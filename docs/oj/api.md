# OJ 系统 API 文档

本文件统一维护 OJ 系统所有基础与高级接口、参数、响应、异常、状态码、安全说明。各 step/advance 文档如有接口相关内容，均应引用本文件。

---

## 目录
1. 题目管理相关接口（Step 1）
2. 评测相关接口（Step 2 & 4）
3. 用户管理相关接口（Step 3）
4. 日志与权限管理相关接口（Step 6）
5. 高级功能接口
6. 状态码与异常
7. 安全性说明

---

## 1. 题目管理相关接口（Step 1）

### 查看题目列表
- 路径：`GET /api/problems/`
- 权限：公开
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": [
    {"id": "sum_2", "title": "两数之和"},
    {"id": "max_num", "title": "最大数"}
  ]
}
```

### 添加题目
- 路径：`POST /api/problems/`
- 权限：仅管理员（或本地开发可不校验）
- 参数：题目配置（JSON，字段见下）
- 响应：
```json
{"code": 200, "msg": "add success", "data": {"id": "sum_2"}}
```
- 异常：400 字段缺失/格式错误 / 409 id 已存在

### 删除题目
- 路径：`DELETE /api/problems/{problem_id}`
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "delete success", "data": {"id": "sum_2"}}
```
- 异常：404 题目不存在

### 查看题目信息
- 路径：`GET /api/problems/{problem_id}`
- 权限：公开
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": {
    "id": "sum_2",
    "title": "两数之和",
    "description": "输入两个整数，输出它们的和。",
    "input_description": "输入为一行，包含两个整数。",
    "output_description": "输出这两个整数的和。",
    "samples": [
      {"input": "1 2\n", "output": "3\n"}
    ],
    "test_cases": [
      {"input": "1 2\n", "output": "3\n"},
      {"input": "10 20\n", "output": "30\n"}
    ],
    "time_limit": 1.0,
    "memory_limit": 128,
    "score": 100,
    "banned_users": []
  }
}
```
- 异常：404 题目不存在

---

## 2. 评测相关接口（Step 2 & 4）

### 提交评测
- 路径：`POST /api/submissions/`
- 参数：
  - `problem_id` (int, 必填): 题目编号
  - `language` (str, 必填): 语言（如 "python"、"c"、"cpp"）
  - `code` (str, 必填): 用户代码
- 权限：登录用户
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": {"submission_id": 123, "status": "Pending"}
}
```
- 异常：400 参数错误 / 403 用户被禁用 / 404 题目不存在 / 429 提交频率超限

### 查询评测结果
- 路径：`GET /api/submissions/{submission_id}`
- 权限：仅本人或管理员
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": {"status": "Accepted", "score": 100, "time": 0.23, "memory": 128, "stderr": ""}
}
```
- 异常：404 评测不存在 / 403 权限不足

### 查询评测列表
- 路径：`GET /api/submissions/`
- 参数：`user_id`、`problem_id`、`status`、`page`、`page_size`（均可选）
- 权限：本人/管理员
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": {"total": 100, "submissions": [{"submission_id": 1, ...}]}
}
```

### 重新评测
- 路径：`PUT /api/submissions/{submission_id}/rejudge`
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "rejudge started", "data": {"submission_id": 1, "status": "Pending"}}
```
- 异常：404/403

---

## 3. 用户管理相关接口（Step 3）

### 用户注册
- 路径：`POST /api/users/`
- 参数：`username` (str, 必填), `password` (str, 必填)
- 响应：
```json
{"code": 200, "msg": "register success", "data": {"user_id": 1}}
```
- 异常：400 用户名已存在/参数错误

### 查询用户信息
- 路径：`GET /api/users/{user_id}`
- 权限：仅本人或管理员
- 响应：
```json
{"code": 200, "msg": "success", "data": {"user_id": 1, "username": "alice", "role": "user"}}
```
- 异常：404/403

### 用户权限变更
- 路径：`PUT /api/users/{user_id}/role`
- 参数：`role` (str, 必填)
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "role updated", "data": {"user_id": 1, "role": "admin"}}
```
- 异常：404/403

### 用户列表查询
- 路径：`GET /api/users/`，参数：`username`、`role`、`page`、`page_size`（可选）
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "success", "data": {"total": 2, "users": [{"user_id": 1, ...}]}}
```

---

## 4. 日志与权限管理相关接口（Step 6）

### 查询评测日志
- 路径：`GET /api/submissions/{submission_id}/log`
- 权限：仅本人或管理员
- 响应：
```json
{"code": 200, "msg": "success", "data": [{"test_id": 1, ...}]}
```
- 异常：404/403

### 配置日志/测例可见性
- 路径：`PUT /api/problems/{problem_id}/log_visibility`
- 权限：仅管理员
- 参数：
  - `public_cases` (bool)：是否允许所有用户查看测例详情
- 响应：
```json
{
  "code": 200,
  "msg": "log visibility updated",
  "data": {"problem_id": 1001, "public_cases": true}
}
```
- 异常：404 题目不存在 / 403 权限不足

### 日志访问审计
- 路径：`GET /api/logs/access/`
- 权限：仅管理员
- 参数：
  - `user_id` (int, 可选)：按用户筛选
  - `problem_id` (int, 可选)：按题目筛选
  - `page` (int, 可选)：页码
  - `page_size` (int, 可选)：每页数量
- 响应：
```json
{
  "code": 200,
  "msg": "success",
  "data": [
    {"user_id": 1, "problem_id": 1001, "action": "view_log", "time": "2024-06-01T12:00:00"}
  ]
}
```
- 异常：403 权限不足

---

## 5. 高级功能接口

### 动态注册新语言
- 路径：`POST /api/languages/`
- 参数：
  - `name` (str, 必填)：语言名称，如 "python"、"cpp"、"go"
  - `file_ext` (str, 必填)：代码文件扩展名，如 ".py"、".cpp"
  - `compile_cmd` (str, 可选)：编译命令（如 C++/Go 需编译，Python 可省略）
  - `run_cmd` (str, 必填)：运行命令，支持变量占位（如 `{src}` 表示源文件，`{exe}` 表示可执行文件）
  - `source_template` (str, 可选)：代码执行模板（如需包裹用户代码，可用此字段）
  - `time_limit` (float, 可选)：该语言默认时间限制（秒）
  - `memory_limit` (int, 可选)：该语言默认内存限制（MB）
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "language registered", "data": {"name": "go"}}
```
- 异常：400/403

- 示例：
```json
{
  "name": "cpp",
  "file_ext": ".cpp",
  "compile_cmd": "g++ {src} -o {exe} -O2 -std=c++17",
  "run_cmd": "{exe}",
  "source_template": "{code}",
  "time_limit": 1.0,
  "memory_limit": 128
}
```
```json
{
  "name": "python",
  "file_ext": ".py",
  "run_cmd": "python3 {src}",
  "source_template": "{code}",
  "time_limit": 1.0,
  "memory_limit": 128
}
```

### 查询支持语言列表
- 路径：`GET /api/languages/`
- 响应：
```json
{"code": 200, "msg": "success", "data": [{"name": "python", ...}]}
```

### Special Judge、查重、前端交互等
- 上传 SPJ 脚本：`POST /api/problems/{problem_id}/spj`（仅管理员）
- 删除 SPJ 脚本：`DELETE /api/problems/{problem_id}/spj`（仅管理员）
- 查询题目评测策略：`GET /api/problems/{problem_id}`
- 发起查重检测：`POST /api/plagiarism/`（仅管理员）
- 查询查重结果：`GET /api/plagiarism/{task_id}`（仅管理员）
- 下载查重报告：`GET /api/plagiarism/{task_id}/report`（仅管理员）

---

## 6. 状态码与异常

| HTTP 状态码 | 说明           | 示例场景           |
|-------------|----------------|--------------------|
| 200         | 正常           | 一切正常           |
| 400         | 参数错误       | 缺少/错误参数      |
| 403         | 权限不足/禁用  | banned 用户/无权限 |
| 404         | 资源不存在     | 题目/评测不存在    |
| 429         | 频率超限       | 提交过于频繁       |
| 500         | 服务器异常     | 未知错误           |

- 所有错误响应建议格式：
```json
{"code": 404, "msg": "problem not found", "data": null}
```

---

## 7. 安全性说明
- 用户身份建议通过 Cookie/Session 传递，避免 body 明文 role。
- 评测执行需沙箱隔离，防止恶意代码危害系统。
- 资源限制（如内存、CPU 时间）需严格执行。
- 日志和敏感信息需妥善保护，避免泄露。

---

**本文件为 OJ 系统接口唯一规范，所有接口变更需同步更新。**