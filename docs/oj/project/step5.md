## Step 5：排行榜机制

---

### 模块目标

本模块旨在为 OJ 系统建立用户得分排行榜，实现答题成绩的统计、排序与展示能力。

系统应支持以下功能：

* 每个用户在每题的得分按**最高值计入总分**；
* 用户总分 = 所有题目得分之和；
* 排行榜支持多种排序方式和 tie-breaker（同分排名控制）；
* 提供规范的 REST API，可供前端直接调用；
* 支持分页和限制返回人数。

---

## 前置知识要求

| 技术点            | 说明                                      |
| -------------- | --------------------------------------- |
| Python 排序      | 使用 `sorted()` 与 key 函数实现自定义排序           |
| 嵌套字典与默认值处理     | `defaultdict(dict)` 管理用户-题目得分表结构        |
| 时间戳与时间比较       | 比较提交时间，支持 earliest / latest tie-breaker |
| RESTful API 设计 | GET 请求参数解析、分页处理、字段组织等基本接口规范             |

---

## 实验任务拆解

---

### 任务 1：题目分值配置

你需要为每道题配置分值。可以将该配置包含在 `config.json` 中或使用单独文件（推荐）：

```json
{
  "problem_scores": {
    "1001": 100,
    "1002": 50,
    "1003": 70
  }
}
```

启动时加载进内存（如 `PROBLEM_SCORES` 字典）。题号推荐使用字符串或整数一致，需统一格式。

---

### 任务 2：得分统计结构与更新机制

建议维护以下两个数据结构：

```python
user_problem_scores = {
    1: {1001: 100, 1002: 30},
    2: {1001: 80}
}

user_score_times = {
    1: datetime(...),
    2: datetime(...)
}
```

在每个评测任务完成后（状态为 `Finished`）：

* 提取 user\_id、problem\_id、score；
* 若用户本题第一次提交，或得分更高 → 更新；
* 同时更新 user 当前总分达到的时间（用于 tie-breaker）。

如果采用持久化结构，建议数据库中维护单独的 `UserScore` 表或缓存结构，每次任务完成后写入更新。

---

### 任务 3：实现排行榜接口

#### 接口定义：

```http
GET /contests/0/ranklist
```

#### 支持参数：

| 参数名           | 类型                               | 默认值        | 描述                    |
| ------------- | -------------------------------- | ---------- | --------------------- |
| `order`       | `asc` / `desc`                   | `desc`     | 按总分升序或降序              |
| `tie_breaker` | `earliest` / `latest` / `random` | `earliest` | 同分用户排序策略              |
| `limit`       | int                              | 全部         | 返回前 N 名               |
| `page`        | int                              | 1          | 页码，配合 page\_size 用于分页 |
| `page_size`   | int                              | 50         | 每页显示用户数量              |

#### 返回结构：

```json
[
  {
    "user_id": 1,
    "username": "alice",
    "total_score": 150,
    "problem_scores": [100, 50, 0],
    "submit_count": 5,
    "last_updated": "2025-06-21T10:15:30"
  },
  ...
]
```

说明：

* `problem_scores` 按照 `problem_id` 升序；
* `submit_count` 表示用户提交的所有评测次数；
* `last_updated` 表示该用户达到当前总分的最后提交时间。

---

### 任务 4：实现 tie-breaker 排序逻辑

你需要为同分用户设置不同排序策略：

```python
def rank_sort_key(user_id):
    score = total_scores[user_id]
    t = user_score_times.get(user_id)
    if tie_breaker == "earliest":
        return (-score, t)
    elif tie_breaker == "latest":
        return (-score, -timestamp(t))
    else:
        return (-score, random.random())
```

记得提前确保 `t` 不为 `None`，否则提供默认时间或 raise 错误。

---

### 任务 5：支持分页与限制结果数

添加参数 `page` 和 `page_size`，对最终排序结果切片：

```python
start = (page - 1) * page_size
end = start + page_size
return sorted_users[start:end]
```

也可以使用 `limit` 参数直接取前 N 名，建议与分页参数二选一。

---

## 自测建议

编写测试脚本或使用 curl 调用，测试如下用例：

| 测试项                     | 方式                     | 预期行为                      |
| ----------------------- | ---------------------- | ------------------------- |
| 多用户多题提交                 | 分别创建多个用户，提交多个题目的评测任务   | 得分正确聚合                    |
| 同用户多次提交同题，仅保留最高分        | 提交多次不同得分任务             | 保留最高分更新总分                 |
| 正确返回默认排序                | 不加参数调用 `/ranklist`     | 总分降序，tie-breaker=earliest |
| `order=asc` 参数排序测试      | 调用时添加参数 `order=asc`    | 总分升序返回                    |
| tie-breaker = latest 测试 | 控制多个同分用户更新时间           | 最新得分用户排前                  |
| 分页参数测试                  | `page=2&page_size=1`   | 正确返回第 2 条记录               |
| problem\_scores 顺序一致测试  | 查看 `problem_scores` 顺序 | 按 `problem_id` 升序排列       |

---

## 助教评测建议

| 测试点         | 评判方式                        |
| ----------- | --------------------------- |
| 每题最高分统计是否正确 | 同题多任务中只取最高得分                |
| 总分计算是否正确    | 题目分数加和是否符合配置值               |
| 排序行为与参数一致   | desc/asc 和 tie-breaker 是否生效 |
| 非法参数处理      | 提供非法参数时是否有默认值或返回 400        |
| 分页与限制是否有效   | page 与 page\_size 参数生效      |
