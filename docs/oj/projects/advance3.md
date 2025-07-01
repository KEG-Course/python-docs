## Advance 3：分布式评测系统与高级安全

---

### 模块目标

本模块旨在构建一个可扩展的分布式评测系统，实现：

* 多机器并行评测，支持动态扩缩容；
* 使用Redis作为分布式任务队列和锁机制；
* 智能负载均衡和故障转移；
* 高级安全沙箱和反作弊机制；
* 实时监控和性能分析。

---

### 前置知识要求

| 技术点         | 推荐学习内容                         |
| ----------- | ------------------------------ |
| 分布式系统     | Redis、消息队列、分布式锁              |
| 容器技术       | Docker、Kubernetes、容器编排          |
| 网络编程       | `asyncio`、`aiohttp`、WebSocket      |
| 系统编程       | `subprocess`、`resource`、`signal`   |
| 监控与日志     | Prometheus、Grafana、ELK Stack       |
| 安全技术       | seccomp、cgroup、namespace隔离        |

---

### 任务拆解

#### 任务 1：分布式任务队列

**功能描述：**
使用Redis实现分布式任务队列，支持多评测节点并行处理。

**实现方案：**

```python
import redis
import json
import asyncio
from typing import Dict, List

class DistributedJobQueue:
    def __init__(self, redis_url: str):
        self.redis = redis.from_url(redis_url)
        self.queue_name = "oj_job_queue"
        self.processing_key = "oj_processing_jobs"
        self.node_status_key = "oj_node_status"
    
    async def enqueue_job(self, job: Dict) -> str:
        """将任务加入队列"""
        job_id = job["id"]
        job_data = {
            "job": job,
            "enqueue_time": asyncio.get_event_loop().time(),
            "priority": job.get("priority", 0)
        }
        
        # 使用Redis Sorted Set实现优先级队列
        await self.redis.zadd(
            self.queue_name,
            {json.dumps(job_data): -job_data["priority"]}
        )
        return job_id
    
    async def dequeue_job(self, node_id: str) -> Dict:
        """从队列中取出任务"""
        # 原子操作：取出最高优先级任务
        result = await self.redis.zpopmax(self.queue_name)
        if not result:
            return None
        
        job_data = json.loads(result[0][0])
        job = job_data["job"]
        
        # 记录正在处理的任务
        await self.redis.hset(
            self.processing_key,
            job["id"],
            json.dumps({
                "node_id": node_id,
                "start_time": asyncio.get_event_loop().time(),
                "job": job
            })
        )
        
        return job
    
    async def mark_job_completed(self, job_id: str, result: Dict):
        """标记任务完成"""
        await self.redis.hdel(self.processing_key, job_id)
        # 更新任务结果到Redis
        await self.redis.setex(
            f"job_result:{job_id}",
            3600,  # 1小时过期
            json.dumps(result)
        )
    
    async def get_queue_stats(self) -> Dict:
        """获取队列统计信息"""
        queue_size = await self.redis.zcard(self.queue_name)
        processing_count = await self.redis.hlen(self.processing_key)
        
        return {
            "queue_size": queue_size,
            "processing_count": processing_count,
            "node_count": await self.redis.scard(self.node_status_key)
        }
```

#### 任务 2：评测节点管理

**功能描述：**
实现评测节点的注册、发现、健康检查和负载均衡。

**实现方案：**

```python
class JudgeNode:
    def __init__(self, node_id: str, redis_url: str, capabilities: Dict):
        self.node_id = node_id
        self.redis = redis.from_url(redis_url)
        self.capabilities = capabilities
        self.heartbeat_interval = 30  # 30秒心跳
        self.max_concurrent = capabilities.get("max_concurrent", 4)
        self.current_load = 0
        
    async def start(self):
        """启动评测节点"""
        # 注册节点
        await self._register_node()
        
        # 启动心跳
        asyncio.create_task(self._heartbeat_loop())
        
        # 启动任务处理循环
        asyncio.create_task(self._job_processing_loop())
    
    async def _register_node(self):
        """注册节点到Redis"""
        node_info = {
            "node_id": self.node_id,
            "capabilities": self.capabilities,
            "status": "online",
            "register_time": asyncio.get_event_loop().time(),
            "current_load": 0
        }
        
        await self.redis.hset(
            "oj_nodes",
            self.node_id,
            json.dumps(node_info)
        )
        await self.redis.sadd("oj_online_nodes", self.node_id)
    
    async def _heartbeat_loop(self):
        """心跳循环"""
        while True:
            try:
                # 更新节点状态
                node_info = {
                    "node_id": self.node_id,
                    "current_load": self.current_load,
                    "last_heartbeat": asyncio.get_event_loop().time(),
                    "cpu_usage": await self._get_cpu_usage(),
                    "memory_usage": await self._get_memory_usage()
                }
                
                await self.redis.hset(
                    "oj_nodes",
                    self.node_id,
                    json.dumps(node_info)
                )
                
                await asyncio.sleep(self.heartbeat_interval)
            except Exception as e:
                logger.error(f"Node {self.node_id} heartbeat error: {e}")
    
    async def _job_processing_loop(self):
        """任务处理循环"""
        queue = DistributedJobQueue(self.redis.connection_pool.connection_kwargs)
        
        while True:
            try:
                if self.current_load < self.max_concurrent:
                    job = await queue.dequeue_job(self.node_id)
                    if job:
                        self.current_load += 1
                        asyncio.create_task(self._process_job(job, queue))
                
                await asyncio.sleep(0.1)
            except Exception as e:
                logger.error(f"Job processing error: {e}")
    
    async def _process_job(self, job: Dict, queue: DistributedJobQueue):
        """处理单个任务"""
        try:
            # 创建安全沙箱
            sandbox = SecuritySandbox(job)
            
            # 执行评测
            result = await sandbox.execute()
            
            # 标记任务完成
            await queue.mark_job_completed(job["id"], result)
            
        except Exception as e:
            result = {
                "status": "System Error",
                "error": str(e),
                "job_id": job["id"]
            }
            await queue.mark_job_completed(job["id"], result)
        finally:
            self.current_load -= 1
```

#### 任务 3：高级安全沙箱

**功能描述：**
实现多层安全隔离，防止恶意代码攻击。

**实现方案：**

```python
import os
import signal
import resource
import subprocess
import tempfile
import shutil
from pathlib import Path

class SecuritySandbox:
    def __init__(self, job: Dict):
        self.job = job
        self.timeout = job.get("time_limit", 5.0)
        self.memory_limit = job.get("memory_limit", 256)  # MB
        self.network_allowed = job.get("network_allowed", False)
        self.work_dir = None
        
    async def execute(self) -> Dict:
        """在安全沙箱中执行代码"""
        try:
            # 创建隔离工作目录
            self.work_dir = tempfile.mkdtemp(prefix="oj_sandbox_")
            
            # 设置资源限制
            self._set_resource_limits()
            
            # 创建seccomp过滤器
            seccomp_filter = self._create_seccomp_filter()
            
            # 执行代码
            result = await self._run_code(seccomp_filter)
            
            return result
            
        finally:
            # 清理工作目录
            if self.work_dir and os.path.exists(self.work_dir):
                shutil.rmtree(self.work_dir)
    
    def _set_resource_limits(self):
        """设置资源限制"""
        # CPU时间限制
        resource.setrlimit(resource.RLIMIT_CPU, (int(self.timeout), int(self.timeout)))
        
        # 内存限制
        memory_bytes = self.memory_limit * 1024 * 1024
        resource.setrlimit(resource.RLIMIT_AS, (memory_bytes, memory_bytes))
        
        # 文件大小限制
        resource.setrlimit(resource.RLIMIT_FSIZE, (10 * 1024 * 1024, 10 * 1024 * 1024))
        
        # 进程数限制
        resource.setrlimit(resource.RLIMIT_NPROC, (1, 1))
        
        # 文件描述符限制
        resource.setrlimit(resource.RLIMIT_NOFILE, (10, 10))
    
    def _create_seccomp_filter(self) -> str:
        """创建seccomp过滤器"""
        # 基础允许的系统调用
        allowed_syscalls = [
            "read", "write", "open", "close", "exit", "exit_group",
            "brk", "mmap", "munmap", "mprotect", "fstat", "lstat",
            "stat", "lseek", "getcwd", "chdir", "fchdir", "getuid",
            "getgid", "geteuid", "getegid", "getpid", "getppid",
            "clock_gettime", "time", "nanosleep", "rt_sigreturn"
        ]
        
        if not self.network_allowed:
            # 禁止网络相关系统调用
            blocked_syscalls = [
                "socket", "connect", "bind", "listen", "accept",
                "sendto", "recvfrom", "sendmsg", "recvmsg"
            ]
        
        # 生成seccomp BPF过滤器
        filter_code = self._generate_seccomp_bpf(allowed_syscalls)
        return filter_code
    
    async def _run_code(self, seccomp_filter: str) -> Dict:
        """运行代码"""
        # 写入源代码
        source_file = Path(self.work_dir) / f"main.{self.job['language']}"
        source_file.write_text(self.job['source_code'])
        
        # 构建执行命令
        commands = self._get_execution_commands(source_file)
        
        result = {
            "status": "System Error",
            "output": "",
            "error": "",
            "time": 0,
            "memory": 0
        }
        
        for cmd_type, command in commands.items():
            try:
                # 使用seccomp过滤器执行
                process = await asyncio.create_subprocess_exec(
                    *command,
                    stdout=asyncio.subprocess.PIPE,
                    stderr=asyncio.subprocess.PIPE,
                    cwd=self.work_dir,
                    preexec_fn=lambda: self._apply_seccomp(seccomp_filter)
                )
                
                start_time = asyncio.get_event_loop().time()
                
                try:
                    stdout, stderr = await asyncio.wait_for(
                        process.communicate(),
                        timeout=self.timeout
                    )
                    
                    end_time = asyncio.get_event_loop().time()
                    execution_time = end_time - start_time
                    
                    if process.returncode == 0:
                        result["status"] = "Accepted"
                        result["output"] = stdout.decode()
                        result["time"] = execution_time
                    else:
                        result["status"] = "Runtime Error"
                        result["error"] = stderr.decode()
                        
                except asyncio.TimeoutError:
                    process.kill()
                    result["status"] = "Time Limit Exceeded"
                    result["time"] = self.timeout
                    
            except Exception as e:
                result["status"] = "System Error"
                result["error"] = str(e)
                break
        
        return result
```

#### 任务 4：智能负载均衡

**功能描述：**
实现基于节点负载和能力的智能任务分发。

**实现方案：**

```python
class LoadBalancer:
    def __init__(self, redis_url: str):
        self.redis = redis.from_url(redis_url)
        self.node_cache = {}
        self.cache_ttl = 5  # 5秒缓存
        
    async def select_best_node(self, job: Dict) -> str:
        """选择最佳节点执行任务"""
        # 获取在线节点
        nodes = await self._get_online_nodes()
        
        if not nodes:
            raise Exception("No available nodes")
        
        # 计算节点评分
        node_scores = []
        for node_id, node_info in nodes.items():
            score = await self._calculate_node_score(node_info, job)
            node_scores.append((node_id, score))
        
        # 选择评分最高的节点
        best_node = max(node_scores, key=lambda x: x[1])[0]
        return best_node
    
    async def _calculate_node_score(self, node_info: Dict, job: Dict) -> float:
        """计算节点评分"""
        score = 0.0
        
        # 负载评分（负载越低分数越高）
        current_load = node_info.get("current_load", 0)
        max_concurrent = node_info.get("capabilities", {}).get("max_concurrent", 1)
        load_ratio = current_load / max_concurrent
        score += (1.0 - load_ratio) * 40  # 负载权重40%
        
        # 能力匹配评分
        required_language = job.get("language")
        supported_languages = node_info.get("capabilities", {}).get("languages", [])
        if required_language in supported_languages:
            score += 30  # 语言匹配权重30%
        
        # 性能评分
        cpu_usage = node_info.get("cpu_usage", 0.5)
        memory_usage = node_info.get("memory_usage", 0.5)
        performance_score = (1.0 - cpu_usage) * 0.6 + (1.0 - memory_usage) * 0.4
        score += performance_score * 20  # 性能权重20%
        
        # 历史成功率评分
        success_rate = await self._get_node_success_rate(node_info["node_id"])
        score += success_rate * 10  # 成功率权重10%
        
        return score
    
    async def _get_online_nodes(self) -> Dict:
        """获取在线节点信息"""
        # 检查缓存
        cache_key = "online_nodes_cache"
        cached = await self.redis.get(cache_key)
        if cached:
            return json.loads(cached)
        
        # 从Redis获取在线节点
        online_node_ids = await self.redis.smembers("oj_online_nodes")
        nodes = {}
        
        for node_id in online_node_ids:
            node_info = await self.redis.hget("oj_nodes", node_id)
            if node_info:
                nodes[node_id] = json.loads(node_info)
        
        # 更新缓存
        await self.redis.setex(cache_key, self.cache_ttl, json.dumps(nodes))
        return nodes
```

#### 任务 5：实时监控与告警

**功能描述：**
实现系统监控、性能分析和异常告警。

**实现方案：**

```python
import prometheus_client
from prometheus_client import Counter, Gauge, Histogram, Summary

class MonitoringSystem:
    def __init__(self):
        # 定义监控指标
        self.job_counter = Counter('oj_jobs_total', 'Total jobs processed', ['status', 'language'])
        self.job_duration = Histogram('oj_job_duration_seconds', 'Job execution time', ['language'])
        self.queue_size = Gauge('oj_queue_size', 'Current queue size')
        self.node_count = Gauge('oj_online_nodes', 'Number of online nodes')
        self.node_load = Gauge('oj_node_load', 'Node load', ['node_id'])
        
        # 启动Prometheus指标服务器
        prometheus_client.start_http_server(8000)
    
    def record_job_completion(self, job: Dict, result: Dict, duration: float):
        """记录任务完成指标"""
        status = result.get("status", "Unknown")
        language = job.get("language", "unknown")
        
        self.job_counter.labels(status=status, language=language).inc()
        self.job_duration.labels(language=language).observe(duration)
    
    async def update_system_metrics(self):
        """更新系统指标"""
        # 更新队列大小
        queue_size = await self.redis.zcard("oj_job_queue")
        self.queue_size.set(queue_size)
        
        # 更新节点数量
        node_count = await self.redis.scard("oj_online_nodes")
        self.node_count.set(node_count)
        
        # 更新节点负载
        nodes = await self._get_online_nodes()
        for node_id, node_info in nodes.items():
            load = node_info.get("current_load", 0)
            self.node_load.labels(node_id=node_id).set(load)
    
    async def check_alerts(self):
        """检查告警条件"""
        alerts = []
        
        # 检查队列积压
        queue_size = await self.redis.zcard("oj_job_queue")
        if queue_size > 100:
            alerts.append({
                "level": "warning",
                "message": f"Queue backlog detected: {queue_size} jobs",
                "timestamp": asyncio.get_event_loop().time()
            })
        
        # 检查节点健康状态
        nodes = await self._get_online_nodes()
        for node_id, node_info in nodes.items():
            last_heartbeat = node_info.get("last_heartbeat", 0)
            current_time = asyncio.get_event_loop().time()
            
            if current_time - last_heartbeat > 60:  # 60秒无心跳
                alerts.append({
                    "level": "critical",
                    "message": f"Node {node_id} appears to be down",
                    "timestamp": current_time
                })
        
        # 发送告警
        if alerts:
            await self._send_alerts(alerts)
```

---

### API接口设计

#### GET /cluster/status
```json
{
  "nodes": [
    {
      "node_id": "judge-1",
      "status": "online",
      "current_load": 2,
      "max_concurrent": 4,
      "cpu_usage": 0.45,
      "memory_usage": 0.6
    }
  ],
  "queue_stats": {
    "queue_size": 15,
    "processing_count": 8,
    "node_count": 3
  }
}
```

#### POST /cluster/scale
```json
{
  "action": "scale_up",
  "node_count": 2,
  "node_type": "standard"
}
```

#### GET /cluster/metrics
```json
{
  "prometheus_endpoint": "http://localhost:8000/metrics",
  "grafana_dashboard": "http://localhost:3000/d/oj-cluster"
}
```

---

### 测试建议

| 测试项                    | 期望行为                                |
| ---------------------- | ----------------------------------- |
| 节点注册与发现           | 新节点能正确注册并被调度器发现            |
| 任务分发负载均衡         | 任务能根据节点负载智能分发               |
| 故障转移机制            | 节点宕机时任务能自动迁移到其他节点         |
| 安全沙箱隔离           | 恶意代码无法突破安全限制                |
| 性能监控准确性          | 监控指标能准确反映系统状态              |
| 高并发处理能力          | 支持100+并发任务同时处理              |
| 资源限制有效性          | 内存、CPU限制能正确生效               |

---

### 模块结构建议

```
distributed/
├── queue/
│   ├── redis_queue.py
│   └── priority_queue.py
├── nodes/
│   ├── judge_node.py
│   ├── node_manager.py
│   └── load_balancer.py
├── security/
│   ├── sandbox.py
│   ├── seccomp_filter.py
│   └── resource_limits.py
├── monitoring/
│   ├── metrics.py
│   ├── alerts.py
│   └── dashboard.py
└── api/
    ├── cluster.py
    └── health.py
```

---

### 能力小结

| 功能      | 支持说明                          |
| ------- | ----------------------------- |
| 分布式队列 | Redis实现，支持优先级和持久化         |
| 节点管理   | 自动注册发现，健康检查，负载均衡        |
| 安全沙箱   | 多层隔离，资源限制，系统调用过滤        |
| 智能调度   | 基于负载和能力的任务分发              |
| 实时监控   | Prometheus指标，Grafana可视化        |
| 故障恢复   | 自动故障检测，任务重调度，节点恢复       |
| 弹性伸缩   | 支持动态添加移除评测节点             |