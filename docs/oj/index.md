# PA2: Online Judge 系统设计与实现

## 实验概述

本项目旨在构建一个功能完整的 Online Judge (OJ) 系统，为学生提供在线编程评测平台。通过分阶段实现，你将掌握系统设计、API开发、并发处理、安全控制等核心技能。

## 实验目标

> 构建一个小型但功能完整的 Online Judge 评测服务 (OJ Server)

```
      +-----------+          +------------+         +--------------+
用户   | 提交代码  |  POST →  | OJ 服务器   |  → Run →| 评测结果     |
      +-----------+          |            |         +--------------+
                                  ↓
                             任务队列
                                  ↓
                           任务状态管理
                                  ↓
                        用户查询任务状态 / 排行榜
```

通过本项目的学习，你将能够：

- **掌握架构设计**：理解分布式系统的基本概念和设计原则。
- **精进API开发**：使用FastAPI/Flask构建RESTful API。
- **理解并发编程**：掌握多线程、异步编程和任务调度。
- **学会安全控制**：实现用户权限管理和代码安全沙盒。
- **熟悉数据处理**：使用数据库进行数据持久化和查询优化。
- **理解监控调试**：实现系统监控、日志记录和性能分析。
- **评估代码质量**：实现代码查重、质量评估。

## 项目模块

### 基础模块 (必做)

| 模块 | 名称 | 主要功能 |
| ---- | ---- | -------- |
| Step 1 | 配置解析 + 语言支持 | 构建通用语言运行环境 |
| Step 2 | 评测执行 + 正确判断 | 实现评测核心模块 |
| Step 3 | 用户系统 + 提交频控 | 多用户管理与权限控制 |
| Step 4 | 任务管理与状态流转 | 完整的任务生命周期管理 |
| Step 5 | 排行榜机制 | 用户得分统计与排序 |
| Step 6 | 评测日志与调试 | 记录评测日志，支持Debug |
| Step 7 | 持久化与高可用 | 数据持久化与高可用 |

### 进阶模块 (选做)

| 模块 | 名称 | 主要功能 |
| ---- | ---- | -------- |
| Advance 1 | 智能代码分析与LLM辅助评测 | LLM驱动的代码质量评估 |
| Advance 2 | 分布式评测系统与高级安全 | 可扩展的分布式架构 |
| Advance 3 | 高级调试工具与性能分析 | 强大的调试和分析能力 |
| Advance 4 | 智能题目管理与自动生成 | 智能化的题目管理 |
| Advance 5 | 智能代码查重系统 | 全面的代码查重检测 |

---

## 其他部分（技术栈、评估标准、提交要求、资源等）

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

### 评估标准
- **基础模块 (60分)**:
  - **Step 1-5**: 每个模块12分
  - 功能完整性、代码质量、测试覆盖
  - 文档完整性和API设计合理性
- **进阶模块 (40分)**:
  - **Advance 1-5**: 每个模块8分
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

### 常见问题
- **如何选择进阶模块**?
  A: 建议根据自己的兴趣和技术基础选择1-2个进阶模块。如果对AI感兴趣，可以选择Advance 1；如果对系统架构感兴趣，可以选择Advance 2。
- **项目难度如何**?
  A: 基础模块适合所有学生，进阶模块有一定挑战性。建议先完成基础模块，再尝试进阶功能。
- **需要什么开发环境**?
  A: 需要Python 3.8+、Docker、Redis等。详细环境配置请参考各模块文档。
- **如何获得帮助**?
  A: 可以查看模块文档、参考示例代码、参与讨论。遇到技术问题可以查阅官方文档或寻求助教帮助。
  
---

**祝大家学习愉快，项目顺利！** 🚀