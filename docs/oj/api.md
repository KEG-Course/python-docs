# OJ 系统 API 文档

本文件统一维护 OJ 系统所有基础与高级接口、参数、响应、异常、状态码、安全说明。各 step/advance 文档如有接口相关内容，均应引用本文件。

---

## 目录
1. 通用说明
2. 基础接口
   - 2.1 评测相关
   - 2.2 用户相关
   - 2.3 语言相关
   - 2.4 排行榜相关
   - 2.5 日志相关
   - 2.6 数据导出/恢复
   - 2.7 题目管理相关
3. 高级功能接口
   - 3.1 Special Judge
   - 3.2 查重系统
   - 3.3 安全机制
   - 3.4 前端交互
4. 状态码与异常
5. 安全性说明

---

## 1. 通用说明
- 所有接口均采用 RESTful 风格，推荐 JSON 作为请求和响应格式。
- 所有接口响应均包含 `code`（状态码）、`msg`（信息）、`data`（数据，若有）。
- 所有异常情况需返回合理 HTTP 状态码及结构化错误信息。
- 权限相关接口建议使用 Cookie/Session 传递用户身份，body 传 role 仅作演示或加分项。

---

## 2. 基础接口

### 2.1 评测相关

#### 评测提交
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

#### 查询评测结果
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

#### 查询评测列表
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

#### 重新评测
- 路径：`PUT /api/submissions/{submission_id}/rejudge`
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "rejudge started", "data": {"submission_id": 1, "status": "Pending"}}
```
- 异常：404/403

---

### 2.2 用户相关

#### 用户注册/登录
- 路径：`POST /api/users/`
- 参数：`username` (str, 必填), `password` (str, 必填)
- 响应：
```json
{"code": 200, "msg": "register success", "data": {"user_id": 1}}
```
- 异常：400 用户名已存在/参数错误

#### 查询用户信息
- 路径：`GET /api/users/{user_id}`
- 权限：仅本人或管理员
- 响应：
```json
{"code": 200, "msg": "success", "data": {"user_id": 1, "username": "alice", "role": "user"}}
```
- 异常：404/403

#### 用户权限变更
- 路径：`PUT /api/users/{user_id}/role`
- 参数：`role` (str, 必填)
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "role updated", "data": {"user_id": 1, "role": "admin"}}
```
- 异常：404/403

#### 用户列表
- 路径：`GET /api/users/`，参数：`username`、`role`、`page`、`page_size`（可选）
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "success", "data": {"total": 2, "users": [{"user_id": 1, ...}]}}
```

---

### 2.3 语言相关

#### 动态注册新语言
- 路径：`POST /api/languages/`
- 参数：`name` (str, 必填), `compile_cmd` (str, 可选), `run_cmd` (str, 必填)
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "language registered", "data": {"name": "go"}}
```
- 异常：400/403

#### 查询支持语言列表
- 路径：`GET /api/languages/`
- 响应：
```json
{"code": 200, "msg": "success", "data": [{"name": "python", ...}]}
```

---

### 2.4 排行榜相关

#### 查询排行榜
- 路径：`GET /api/ranklist/`
- 参数：`problem_id`、`order`、`tie_breaker`、`page`、`page_size`（可选）
- 响应：
```json
{"code": 200, "msg": "success", "data": {"total": 2, "ranklist": [{"user_id": 1, ...}]}}
```

---

### 2.5 日志相关

#### 查询评测日志
- 路径：`GET /api/submissions/{submission_id}/log`
- 权限：仅本人或管理员
- 响应：
```json
{"code": 200, "msg": "success", "data": [{"test_id": 1, ...}]}
```
- 异常：404/403

#### 日志回放/调试
- 路径：`GET /api/submissions/{submission_id}/replay`，参数：`debug` (bool, 可选)
- 权限：仅本人或管理员，debug 仅管理员
- 响应：
```json
{"code": 200, "msg": "success", "data": {"steps": [...], "debug_info": {...}}}
```
- 异常：404/403

---

### 2.6 数据导出/恢复

#### 数据导出
- 路径：`GET /api/export/`，参数：`format` (str, 可选)
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "success", "data": {"users": [...], "problems": [...], "submissions": [...]}}
```
- 异常：403/500

#### 数据恢复
- 路径：`POST /api/import/`，参数：`file` (上传)
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "import success", "data": null}
```
- 异常：400/403/500

---

### 2.7 题目管理相关

#### 查看题目列表
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

#### 添加题目
- 路径：`POST /api/problems/`
- 参数：题目配置（JSON，字段见下）
- 权限：仅管理员（或本地开发可不校验）
- 响应：
```json
{"code": 200, "msg": "add success", "data": {"id": "sum_2"}}
```
- 异常：400 字段缺失/格式错误 / 409 id 已存在

#### 删除题目
- 路径：`DELETE /api/problems/{problem_id}`
- 权限：仅管理员
- 响应：
```json
{"code": 200, "msg": "delete success", "data": {"id": "sum_2"}}
```
- 异常：404 题目不存在

#### 查看题目信息
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

## 3. 高级功能接口

### 3.1 Special Judge
- 上传 SPJ 脚本：`POST /api/problems/{problem_id}/spj`（仅管理员）
- 删除 SPJ 脚本：`DELETE /api/problems/{problem_id}/spj`（仅管理员）
- 查询题目评测策略：`GET /api/problems/{problem_id}`

### 3.2 查重系统
- 发起查重检测：`POST /api/plagiarism/`（仅管理员）
- 查询查重结果：`GET /api/plagiarism/{task_id}`（仅管理员）
- 下载查重报告：`GET /api/plagiarism/{task_id}/report`（仅管理员）

### 3.3 安全机制
- 评测相关接口均需沙箱隔离、资源限制、命令校验，超限返回 TLE/MLE

### 3.4 前端交互
- 前端所有交互均通过上述 REST API 实现

---

## 4. 状态码与异常

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

## 5. 安全性说明
- 用户身份建议通过 Cookie/Session 传递，避免 body 明文 role。
- 评测执行需沙箱隔离，防止恶意代码危害系统。
- 资源限制（如内存、CPU 时间）需严格执行。
- 日志和敏感信息需妥善保护，避免泄露。

---

**本文件为 OJ 系统接口唯一规范，所有接口变更需同步更新。**