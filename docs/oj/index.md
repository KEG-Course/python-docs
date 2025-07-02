# PA2: Online Judge 系统设计与实现

## 实验概述

本项目旨在构建一个功能完整的 Online Judge (OJ) 系统，分阶段实现，逐步掌握系统设计、API开发、安全控制等核心能力。

## 实验目标

> 构建一个小型但功能完整的 Online Judge 评测服务（OJ Server）

---

## 基础模块（必做）

| Step  | 名称         | 主要功能描述                                   | 详细文档 |
|-------|--------------|-----------------------------------------------|----------|
| Step1 | 基础评测     | 题目配置、评测流程、请求与响应、异常处理       | [step1.md](project/step1.md) |
| Step2 | 多语言支持   | 至少支持 C/C++/Python，动态注册语言           | [step2.md](project/step2.md) |
| Step3 | 评测列表     | 查看/管理评测列表、单个评测、重新评测         | [step3.md](project/step3.md) |
| Step4 | 用户管理     | 用户注册、信息管理、权限控制                   | [step4.md](project/step4.md) |
| Step5 | Ranklist     | 查看比赛排行榜、分数统计、排序                | [step5.md](project/step5.md) |
| Step6 | Log          | 查看请求响应历史、评测日志、报错信息           | [step6.md](project/step6.md) |
| Step7 | 持久化       | 评测记录、排行榜等信息持久化，断点恢复         | [step7.md](project/step7.md) |

---

## 进阶模块（选做）

| Advance | 名称         | 主要功能描述                                   |
|---------|--------------|-----------------------------------------------|
| Adv1    | Special Judge| 支持多种评测方式，题目配置可选评测策略         |
| Adv2    | 前端交互     | 极简前端（HTML+JS），可与后端交互             |
| Adv3    | 安全机制     | 沙箱隔离、内存限制、命令过滤等                 |
| Adv4    | 查重系统     | 代码查重、抄袭检测                             |

---

## API 文档

所有接口、参数、异常、状态码等详见 [api.md](api.md)。

---

## 其他部分（技术栈、评分标准、提交要求、资源等）

### 技术栈
- **后端技术**: FastAPI / Flask
- **异步编程**: asyncio, aiohttp
- **数据库**: SQLite / PostgreSQL
- **缓存**: Redis
- **任务队列**: Celery / Redis Queue

### 系统编程
- **进程管理**: subprocess, multiprocessing
- **资源控制**: resource, cgroups
- **安全沙箱**: seccomp, namespaces
- **容器技术**: Docker

### 机器学习与AI
- **LLM集成**: OpenAI API, Claude API
- **代码分析**: ast, tree-sitter
- **向量数据库**: FAISS, ChromaDB
- **机器学习**: scikit-learn, numpy

### 监控与可视化
- **性能监控**: Prometheus, Grafana
- **日志系统**: ELK Stack
- **数据可视化**: matplotlib, plotly
- **调试工具**: DAP, pdb

### 评分标准
- **基础模块 (60分)**:
  - **Step 1-5**: 每个模块12分
  - 功能完整性、代码质量、测试覆盖
  - 文档完整性和API设计合理性
- **进阶模块 (40分)**:
  - **Advance 1-4**: 每个模块10分
  - 技术难度、创新性、实用性
  - 系统设计和架构合理性

### 加分项
- 性能优化 (5分)
- 用户体验 (5分)
- 创新功能 (5分)
- 项目展示 (5分)

### 提交要求
- **代码提交**:
  - 完整的源代码和配置文件
  - 清晰的代码注释和文档
  - 单元测试和集成测试
  - 性能测试报告
- **文档提交**:
  - 系统设计文档
  - API接口文档
  - 部署和运维文档
  - 用户使用手册
- **演示要求**:
  - 功能演示视频
  - 系统架构展示
  - 性能测试演示
  - 问题解答

### 学习资源
- **官方文档**:
  - [FastAPI 官方文档](https://fastapi.tiangolo.com/)
  - [Python asyncio 文档](https://docs.python.org/3/library/asyncio.html)
  - [Redis 官方文档](https://redis.io/documentation)
- **技术教程**:
  - [系统设计基础](https://github.com/donnemartin/system-design-primer)
  - [Python 并发编程](https://docs.python.org/3/library/concurrent.futures.html)
  - [Docker 容器技术](https://docs.docker.com/)
- **参考项目**:
  - [Codeforces](https://codeforces.com/) - 知名OJ平台
  - [LeetCode](https://leetcode.com/) - 编程练习平台
  - [HackerRank](https://www.hackerrank.com/) - 技术评测平台