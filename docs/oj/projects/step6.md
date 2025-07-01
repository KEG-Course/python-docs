## Step 6：评测日志

---

### 模块目标

本模块旨在为 OJ 系统提供评测日志与调试机制，实现以下能力：

* 记录每次评测中所有测试点的执行详情（含输入输出、错误等）；
* 提供用户/管理员查看日志的 API；
* 支持 Debug 模式，在任务执行时返回 stderr、运行时间等详细数据；
* 支持日志结构化回放接口，为前端可视化展示或审计追踪提供支撑。

---

## 前置知识要求

| 技术点      | 建议掌握内容                                            |
| -------- | ------------------------------------------------- |
| 文件写入     | 使用 `with open(..., "w")` + `.write()` 等方法         |
| JSONL 格式 | 每行一个 JSON 结构，适合多条日志顺序记录                           |
| 错误处理与捕获  | `try/except` 捕捉异常，记录 stderr 和 returncode          |
| 文件下载 API | 使用 FastAPI 的 `FileResponse` 或 Flask 的 `send_file` |

---

## 实验任务拆解

---

### 任务 1：记录评测日志文件

每次评测任务执行后，将每个测试点的评测结果写入日志文件。

#### 推荐文件命名：

```
logs/job_{job_id}.log.jsonl
```

#### 日志格式（每行一条 JSON）：

```json
{
  "job_id": 123,
  "test_id": 2,
  "status": "Wrong Answer",
  "input": "1 2",
  "expected_output": "3",
  "actual_output": "4",
  "stderr": "",
  "time": 0.421,
  "returncode": 0
}
```

每个测试点应独立记录，按顺序逐行写入。

#### 日志字段建议：

| 字段名              | 说明                  |
| ---------------- | ------------------- |
| job\_id          | 当前任务 ID             |
| test\_id         | 测试点编号               |
| status           | AC / WA / TLE 等     |
| input            | 本测试点的输入数据           |
| expected\_output | 正确输出                |
| actual\_output   | 用户程序的实际输出           |
| stderr           | 用户程序运行中的错误输出        |
| time             | 执行用时（单位：秒）          |
| returncode       | 用户程序退出码（如 0 表示正常退出） |

---

### 任务 2：Debug 模式支持

提交任务时允许附带字段：

```json
{
  "source_code": "...",
  "language": "cpp",
  "problem_id": 1001,
  "user_id": 1,
  "debug": true
}
```

若 debug 模式为 true：

* 系统应返回更详细的评测结果字段，如：

  * `stderr`
  * `运行命令`
  * `错误类型说明（如 TLE / RE）`
* 可在 API 返回中直接附带，也可附加专用字段 `debug_info`

若 debug 为 false，则返回结构应尽量精简，仅保留必要状态字段（Accepted / Wrong Answer 等）。

---

### 任务 3：日志查看接口

提供 API：

```http
GET /jobs/{job_id}/log
```

功能说明：

* 返回指定任务的日志文件（JSONL 格式）；
* 支持下载（Content-Type: application/jsonl 或 text/plain）；
* 若日志文件不存在，应返回 404；
* 推荐限制仅本人/管理员可查看（权限检查）。

---

### 任务 4：评测回放接口（结构化展示）

提供接口：

```http
GET /jobs/{job_id}/replay
```

返回结构：

```json
{
  "job_id": 123,
  "tests": [
    {
      "test_id": 1,
      "input": "1 2",
      "expected": "3",
      "actual": "4",
      "status": "Wrong Answer"
    },
    {
      "test_id": 2,
      "input": "5 5",
      "expected": "10",
      "actual": "10",
      "status": "Accepted"
    }
  ]
}
```

回放接口适用于：

* 前端展示测试点对比；
* 提供调试信息时用户更容易定位问题；
* 管理员审计或评测追踪。

---