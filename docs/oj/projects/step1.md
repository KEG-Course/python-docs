## Step 1：配置解析

---

### 模块目标

本模块目标是构建一个**通用语言运行环境系统**，实现如下功能：

* 使用配置文件 `--config config.json` 启动服务；
* 配置中定义支持的语言及运行方式（compile / execute）；
* OJ 系统根据语言名（或后缀）确定如何编译和运行代码；
* 至少支持 Python3 / C++ / Java；
* 提供语言注册 API，支持运行时添加新语言；
* 每种语言支持运行资源限制 profile（时限、内存、网络权限等）；
* 用户提交时允许省略语言字段，系统自动推断语言。

---

### 前置知识要求

| 技术点         | 推荐学习内容                         |
| ----------- | ------------------------------ |
| JSON 配置文件   | `json.load()`                  |
| 字符串模板与替换    | `str.replace()`、`str.format()` |
| 子进程调用       | `subprocess.run()`、`Popen()`   |
| 临时文件 / 目录管理 | `tempfile`, `os`, `shutil`     |
| 文件扩展名提取     | `os.path.splitext(filename)`   |
| REST API 基础 | FastAPI/Flask POST + 参数校验      |

---

### 任务拆解

#### 任务 1：配置文件加载与校验

* 服务启动时必须传入 `--config path` 参数；
* 加载配置文件内容（JSON 格式）；
* 若缺失必要字段（如 `languages`），或路径非法，则启动失败并退出；
* 加载内容存入全局变量（或专用配置模块），供后续使用。

示例配置文件结构：

```json
{
  "default_language": "python3",
  "languages": {
    "python3": {
      "execute": "python3 ${source}",
      "ext": ".py",
      "profile": {
        "time_limit": 2.0,
        "memory_limit": 256,
        "network_allowed": false,
        "compile_time_limit": 5.0,
        "compile_memory_limit": 512
      }
    },
    "cpp": {
      "compile": "g++ ${source} -o ${exe}",
      "execute": "./${exe}",
      "ext": ".cpp",
      "profile": {
        "time_limit": 1.5,
        "memory_limit": 256,
        "network_allowed": false,
        "compile_time_limit": 10.0,
        "compile_memory_limit": 512
      }
    },
    "java": {
      "compile": "javac ${source}",
      "execute": "java -cp ${dir} Main",
      "ext": ".java",
      "profile": {
        "time_limit": 3.0,
        "memory_limit": 512,
        "network_allowed": false,
        "compile_time_limit": 10.0,
        "compile_memory_limit": 1024
      }
    }
  }
}
```

#### 任务 2：语言命令生成器

实现函数：

```python
def get_commands(language_name: str, source_path: str, exe_path: str) -> dict
```

* 根据语言名，从配置中读取对应命令；
* 替换 `${source}`, `${exe}`, `${dir}` 占位符；
* 返回 `{ "compile": "...", "execute": "...", "profile": { ... } }` 字典；
* 若该语言无 `compile` 字段，表示为解释型语言；
* 若语言不存在，抛出自定义异常。

#### 任务 3：自动识别语言

用户提交可不写 language，而是提供：

```json
{
  "source_code": "print('Hello')",
  "filename": "main.py"
}
```

* 系统根据后缀 `.py` 判断为 `python3`；
* 默认支持 `.py`, `.cpp`, `.java`；
* 映射关系来自配置字段 `"ext"`；
* 若无法识别，返回错误；
* 支持 shebang 自动识别（如 `#!/usr/bin/python3`）。

#### 任务 4：POST /languages 接口（语言注册）

实现接口：

```http
POST /languages
```

请求体：

```json
{
  "name": "go",
  "compile": "go build -o ${exe} ${source}",
  "execute": "./${exe}",
  "ext": ".go",
  "profile": {
    "time_limit": 1.0,
    "memory_limit": 128,
    "network_allowed": false,
    "compile_time_limit": 5.0,
    "compile_memory_limit": 512
  }
}
```

* 将新增语言追加进内存中的语言配置结构；
* 不允许覆盖已有语言名（或添加参数控制）；
* 支持写入持久化语言配置文件（如 `languages.json`）；
* 验证字段结构完整性；
* 更新扩展名映射表，用于自动识别。