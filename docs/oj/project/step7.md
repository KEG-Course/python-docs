## Step 7：数据持久化

---

### 模块目标

为 OJ 系统建立数据持久化机制，确保系统重启或部署后：

* 用户信息、任务记录、排行榜等核心数据不会丢失；
* 支持多种存储后端（JSON 或 SQLite）；
* 所有读写行为规范安全，可扩展为异步或高并发场景。

---

## 前置知识要求

| 技术点          | 推荐工具或内容                                  |
| ------------ | ---------------------------------------- |
| JSON 文件操作    | `json.load()` / `json.dump()`            |
| SQLite 数据库   | 使用 Python `sqlite3` 或 ORM 库如 SQLModel    |
| 文件/线程锁机制     | 使用 `threading.Lock()` 或 `filelock` 控制写操作 |
| 应用生命周期控制与初始化 | 启动前加载持久数据，退出前或变更时落盘                      |

---

## 实验任务拆解

---

### 任务 1：数据模型与持久结构设计

你需要持久化以下几类核心数据：

| 类型    | 字段结构示意                                                                    |
| ----- | ------------------------------------------------------------------------- |
| 用户信息  | id, username, role, submit\_history                                       |
| 任务记录  | id, user\_id, problem\_id, status, result, score, create/update\_time 等字段 |
| 配置/常量 | 如 problem\_scores（建议仍从 config.json 读取）                                    |
| 日志文件  | 已在 Step 6 中建立，无需修改路径结构                                                    |

---

### 任务 2：使用 JSON 实现持久化（基础实现）

默认采用 JSON 文件方案：

```
data/
├── users.json
├── jobs.json
├── job_id_counter.txt
```

* 所有文件在启动时自动加载；
* 每次数据更新后同步写入文件；
* 推荐封装为统一模块，统一接口写入/读取。

#### 示例结构

**users.json**：

```json
[
  { "id": 0, "username": "root", "role": "admin" },
  { "id": 1, "username": "alice", "role": "user" }
]
```

**jobs.json**：

```json
[
  {
    "id": 1,
    "user_id": 1,
    "problem_id": 1001,
    "status": "Finished",
    "score": 100,
    "create_time": "...",
    "update_time": "...",
    "result": [...]
  }
]
```

**job\_id\_counter.txt**：

```
42
```

---

### 任务 3：支持 SQLite 模式（推荐实现）

建立结构化表定义：

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  username TEXT,
  role TEXT,
  submit_history TEXT   -- 可使用 JSON 字符串
);

CREATE TABLE jobs (
  id INTEGER PRIMARY KEY,
  user_id INTEGER,
  problem_id INTEGER,
  status TEXT,
  score INTEGER,
  result TEXT,          -- 可使用 JSON 序列化后存入
  create_time TEXT,
  update_time TEXT
);
```

推荐使用 SQLite 内建 `AUTOINCREMENT` 机制分配 `job_id`。

可选 ORM 框架：

* SQLModel（推荐 + FastAPI 兼容）
* SQLAlchemy
* Tortoise ORM

---

### 任务 4：统一存储接口封装

不管使用 JSON 还是 SQLite，推荐使用统一的 `storage.py` 模块暴露接口：

```python
def load_users() -> List[Dict]: ...
def save_users(users: List[Dict]): ...

def load_jobs() -> List[Dict]: ...
def save_jobs(jobs: List[Dict]): ...

def get_next_job_id() -> int: ...
```

业务逻辑无需关注具体使用哪种后端存储。可以通过配置 `storage_backend` 切换使用方式。

---

### 任务 5：数据读写安全控制

为避免并发写冲突：

* 所有写操作必须使用线程锁（如 `threading.Lock()`）保护；
* 或使用 `filelock`（对 JSON 文件更友好）；
* 在 SQLite 模式中可依赖事务机制；

建议写操作函数如下封装：

```python
data_lock = threading.Lock()

def save_users_safe(users):
    with data_lock:
        with open("data/users.json", "w") as f:
            json.dump(users, f, indent=2)
```

---

### 任务 6：程序启动加载 + 初始化机制

系统启动时应执行：

* 加载 JSON/SQLite 数据；
* 若文件不存在，创建空白结构（如空列表）；
* 加载计数器（或数据库主键自增）；
* 初始化必要目录结构（如 `data/`, `logs/`）；
* 检查日志路径是否存在，必要时创建。