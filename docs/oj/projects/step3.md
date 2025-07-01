## Step 3：用户系统

---

### 模块目标

本模块的目标是构建一个完整的多用户管理与权限控制机制，为 OJ 提供身份识别、权限隔离、提交限流等基础能力。完成后，系统应支持：

* 用户注册、更新、查询；
* 用户角色识别与权限控制；
* 提交频率限制机制；
* 管理员可使用人工评测接口。

---

### 前置知识要求

| 技术点         | 推荐内容与说明                         |
| ----------- | ------------------------------- |
| 字典与列表结构设计   | 管理用户信息：`Dict[int, User]`        |
| 时间操作        | 使用 `datetime.datetime` 判断日期是否相同 |
| API 接口约定与设计 | Flask/FastAPI 的路由绑定，状态码返回       |
| 状态持久化（可选）   | JSON 文件或 SQLite 存储用户数据与历史记录     |

---

### 实验任务

---

#### 任务 1：用户注册与查询接口

**功能点：**

* 用户可通过接口注册或更新自身信息；
* 系统支持通过用户 ID 查询所有用户。

**接口设计：**

```http
GET /users
POST /users
```

**数据结构：**

```json
{
  "id": 1,
  "username": "alice",
  "role": "user", // 可为 user, admin, banned
  "submit_quota": {
    "daily_limit": 10,
    "priority": 0
  },
  "stats": {
    "total_submissions": 0,
    "accepted_count": 0,
    "total_score": 0
  }
}
```

**要求：**

* 用户 ID 必须唯一；
* 若已有 ID，允许更新 username 与 role；
* 系统启动时自动创建用户：

```json
{
  "id": 0,
  "username": "root",
  "role": "admin",
  "submit_quota": {
    "daily_limit": -1,
    "priority": -1
  }
}
```

---

#### 任务 2：提交频率限制机制

**背景：**

* 防止用户频繁提交，浪费计算资源；
* 系统应限制普通用户每日最多提交 N 次（如 N = 10）。

**用户结构更新：**

```json
{
  "id": 1,
  "username": "alice",
  "role": "user",
  "submit_history": [
    {
      "timestamp": "2025-06-21T08:30:00",
      "job_id": 42,
      "problem_id": 1001,
      "status": "Accepted",
      "score": 100
    }
  ]
}
```

**实现逻辑：**

* 每次提交任务（`POST /jobs`）前，先检查用户提交记录；
* 统计当天日期的提交次数；
* 若超出限额（如 >10 次），返回：

```http
429 Too Many Requests
```

```json
{
  "error": "submission limit exceeded",
  "daily_limit": 10,
  "current_count": 11
}
```

---

#### 任务 3：角色权限系统

**角色说明：**

| 角色       | 权限说明                      | 提交优先级 | 每日限额 |
| -------- | ------------------------- | ----- | ---- |
| `user`   | 普通用户，可提交评测                | 0     | 10   |
| `admin`  | 管理员，拥有高级权限（可人工判题、查看所有任务等） | -1    | -1   |
| `banned` | 禁用用户，禁止提交评测               | -     | 0    |

**行为控制：**

* 每次提交任务必须验证该用户角色；

  * banned → 拒绝提交（403）；
  * admin/user → 正常提交；
* 后续模块中也需根据角色控制访问权限。

---

#### 任务 4：人工评测接口（仅限 admin）

**接口定义：**

```http
POST /judge/manual
```

**请求体：**

```json
{
  "job_id": 42,
  "status": "Accepted",
  "comment": "管理员手动确认正确",
  "score": 100
}
```

**功能说明：**

* 仅允许 admin 用户调用；
* 用于人工修改某个任务的评测状态；
* 应记录变更日志，包括：

  * 修改时间
  * 操作人 ID
  * 修改前后的状态
  * 备注 comment

日志可保存在：

```
logs/manual_judge.log.jsonl
```

日志格式：

```json
{
  "timestamp": "2025-06-21T10:00:00",
  "admin_id": 0,
  "job_id": 42,
  "old_status": "Wrong Answer",
  "new_status": "Accepted",
  "old_score": 0,
  "new_score": 100,
  "comment": "管理员手动确认正确"
}
```

---

### 测试建议（供学生/助教使用）

| 测试项            | 操作方法                        | 预期结果                 |
| -------------- | --------------------------- | -------------------- |
| 创建新用户          | `POST /users` 添加 ID=1 用户    | 用户列表中新增 alice        |
| 更新用户信息         | 相同 ID 修改 username 或 role    | 变更成功                 |
| 自动创建 root 用户   | 服务启动后查看用户列表                 | 存在 ID=0 且 role=admin |
| 普通用户提交超限       | 连续提交 11 次任务                 | 第 11 次返回 429 错误      |
| banned 用户提交任务  | 将某用户改为 banned，再提交           | 返回 403 Forbidden     |
| admin 调用人工评测接口 | 使用 root 修改任务状态              | 任务状态变更，日志更新          |
| 普通用户调用人工评测接口   | 用 user 身份访问 `/judge/manual` | 返回 403 拒绝            |
| 优先级调度测试        | admin 和普通用户同时提交             | admin 任务优先执行         |

---

### 模块结构建议

可按如下方式组织模块：

* `models/user.py`：定义用户结构体与权限枚举；
* `routes/users.py`：处理用户相关接口；
* `middleware/permission.py`：校验权限装饰器；
* `services/limit.py`：处理频控逻辑；
* `services/priority.py`：处理任务优先级；
* `logs/manual_judge.log.jsonl`：记录人工判题操作。

---

### 能力小结

| 功能      | 支持说明                          |
| ------- | ----------------------------- |
| 用户注册/更新 | 支持 POST /users 创建与更新          |
| 权限系统    | 拥有 user / admin / banned 三种角色 |
| 提交频控    | 每日限额控制，防止滥用资源                 |
| 人工判题支持  | 管理员可手动修改任务状态，并写入审计日志          |
| 优先级调度   | 基于用户角色的任务优先级控制                |