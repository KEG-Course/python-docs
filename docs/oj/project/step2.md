## Step 2：评测控制

---

### 模块目标

本模块目标是实现评测系统的**正确性判断、性能限制控制与评测细节记录**，包括：

* 运行程序时记录运行时间、内存、stderr；
* 支持评测多个测试点（输入-输出组）；
* 对输出进行 strict / standard 模式比对；
* 对运行过程施加资源限制（时间 / 内存）；
* 所有评测过程输出需记录为日志条目；
* 支持多测试点按权重得分计算。

---

### 前置知识要求

| 技术点    | 学习建议或模块                        |
| ------ | ------------------------------ |
| 子进程执行  | `subprocess.run`, `Popen`      |
| 标准输出捕捉 | stdout/stderr/returncode       |
| 文件比对   | 输出规范化、strip() + splitlines     |
| 时间限制   | `time.monotonic`, `timeout` 参数 |
| 内存限制   | `resource`（仅限类 UNIX 系统）        |
| 日志结构   | JSON 格式，写入到独立文件                |

---

### 配置支持

在题目配置（例如 `problem_1001.json`）中定义测试点与得分：

```json
{
  "time_limit": 1.0,
  "memory_limit": 256,
  "compare_mode": "standard",
  "tests": [
    {
      "input": "1 2",
      "output": "3",
      "score": 20
    },
    {
      "input": "1000 2000",
      "output": "3000",
      "score": 80
    }
  ],
  "stop_on_failure": false
}
```

---

### 任务拆解

#### 任务 1：执行程序并收集信息

* 运行程序的方式（compile 已完成）：

  * 使用 `subprocess.run()` 执行命令；
  * 设置最大执行时间（time\_limit）；
  * 设置最大内存（memory\_limit，使用 `resource.setrlimit()`）；
* 捕捉：

  * stdout, stderr；
  * returncode；
  * wall time（`start = time.monotonic()`）；
  * 运行后内存（可选，用 ps 取 peak memory）；

---

#### 任务 2：比对输出并分类状态

* 提供两种对比模式：

  1. `strict`：完全逐字符一致；
  2. `standard`：忽略末尾空行、行尾空格。

* 判断结果状态：

  * `Accepted`：完全正确
  * `Wrong Answer`：答案错误
  * `Time Limit Exceeded`：超时
  * `Memory Limit Exceeded`：超内存
  * `Runtime Error`：运行时错误（非 0 退出码）
  * `Compile Error`：编译错误
  * `System Error`：系统错误（评测系统异常）

* 返回结构中记录以下字段：

```json
{
  "status": "Wrong Answer",
  "score": 0,
  "time": 0.381,
  "memory": 32,
  "stderr": "...",
  "input": "...",
  "expected": "...",
  "actual": "..."
}
```

---

#### 任务 3：支持多测试点，支持按分计分

* 每道题包含多组 input/output；
* 每组带一个 `score` 字段；
* 总得分 = 所有测试点通过得分的加权和；
* 失败测试点得分为 0；
* 返回最终结果结构中要包含 `score` 总和。

---

#### 任务 4：测试点日志写入

* 每个测试点记录为一条日志项，格式如下：

```json
{
  "job_id": 42,
  "test_id": 1,
  "status": "Accepted",
  "input": "...",
  "expected": "...",
  "actual": "...",
  "score": 20,
  "stderr": "...",
  "time": 0.123,
  "memory": 22,
  "create_time": "2025-06-21T10:00:00",
  "update_time": "2025-06-21T10:00:30"
}
```

* 日志文件写入路径应为：

  ```
  logs/job_{job_id}.log.jsonl
  ```

* 每次评测都写完整日志文件，供后续查看。

---

### 测试建议

| 测试项                    | 期望行为                                |
| ---------------------- | ----------------------------------- |
| 程序运行超过时间限制             | 返回 TLE，时间字段 > time\_limit           |
| 程序运行超过内存限制             | 返回 MLE，stderr 显示 `MemoryError` 或被杀死 |
| 程序输出错误                 | 返回 WA，附 expected 与 actual 输出        |
| strict / standard 比对差异 | 行尾空格敏感性体现                           |
| 停止在第一个错误点              | stop\_on\_failure = true 时只评一个测试点   |
| 日志记录完整                 | 每个测试点写入日志一行，包括 stderr、时间、得分等        |
| 多测试点得分按权重正确            | 正确得出最终得分，例如 20 + 0 = 20             |

---

### 模块结构建议

你可以封装为：

* `runner.py`：程序运行器，负责限时执行、收集输出
* `comparator.py`：结果比对逻辑（含标准与严格模式）
* `scorer.py`：计分模块，根据测试点与权重计算总分
* `logger.py`：统一写入 JSONL 日志结构

主循环可以是：

```python
for i, test in enumerate(problem.tests):
    result = run_one_test(test, ...)
    write_log(job_id, result)
    if config.stop_on_failure and result["status"] != "Accepted":
        break
```