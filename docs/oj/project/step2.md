## Step 2：评测控制

---

### 模块目标

实现评测系统的正确性判断、资源限制和评测日志记录。

---

### 前置知识要求

| 技术点         | 推荐学习内容           |
| -------------- | ---------------------- |
| 子进程执行      | `subprocess.run`, `Popen` |
| 标准输出捕捉    | stdout/stderr/returncode |
| 文件比对        | strip() + splitlines   |
| 时间限制        | `time.monotonic`, `timeout` |
| 内存限制        | `resource`（仅UNIX）   |
| 日志结构        | JSON 格式，写入独立文件 |

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

#### 任务 1：程序执行与资源限制
- 目标：运行用户程序并收集运行时间、内存、stderr等信息。
- 要点：设置最大执行时间和内存，捕捉所有输出。
- 建议：可选用 ps 获取 peak memory。

#### 任务 2：输出比对与状态判定
- 目标：对比用户输出与标准输出，判定评测状态。
- 要点：支持 strict/standard 两种比对模式，状态包括 Accepted、Wrong Answer、TLE、MLE、RE、CE、System Error。
- 建议：返回结构中记录 status、score、time、memory、stderr、input、expected、actual。

#### 任务 3：日志记录
- 目标：每个测试点记录为一条日志项。
- 要点：日志文件为 logs/job_{job_id}.log.jsonl，结构化记录所有细节。
- 建议：每次评测都写完整日志文件，供后续查看。

---

### 能力小结

| 能力项         | 说明                       |
| -------------- | -------------------------- |
| 程序执行       | 支持限时限内存安全运行     |
| 输出比对       | 支持严格/宽松两种比对模式  |
| 状态分类       | 评测状态结构化、标准化     |
| 多测试点       | 支持多组测试点与权重计分   |
| 日志记录       | 结构化日志便于追踪与调试   |

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