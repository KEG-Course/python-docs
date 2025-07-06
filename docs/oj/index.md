# OJ 系统实验说明

## 实验目标

构建一个小型但功能完整的 Online Judge (OJ) 系统，分阶段实现，逐步掌握系统设计、API开发、安全控制等核心能力。

---

## 基础模块（必做，共30分）

| Step  | 名称         | 主要功能描述                                   | 详细文档 |
|-------|--------------|-----------------------------------------------|----------|
| Step1 | 配置解析     | 题目配置加载、字段校验、异常处理               | [step1.md](project/step1.md) |
| Step2 | 评测控制     | 程序执行、资源限制、输出比对、动态注册语言     | [step2.md](project/step2.md) |
| Step3 | 用户系统     | 用户注册/更新、权限管理、人工判题接口          | [step3.md](project/step3.md) |
| Step4 | 任务状态管理 | 评测任务流转、调度、API                       | [step4.md](project/step4.md) |
| Step5 | 评测日志     | 日志结构化记录、日志接口、权限                 | [step5.md](project/step5.md) |
| Step6 | 高级安全     | 沙箱隔离、命令过滤、资源限制                   | [step6.md](project/step6.md) |

---

## 进阶模块（选做，最多加10分）

| Advance | 名称         | 主要功能描述                                   |
|---------|--------------|-----------------------------------------------|
| Adv1    | Special Judge| 特殊题目评测，支持多种评测方式                 |
| Adv2    | 前端交互     | 极简前端界面（如 Streamlit），与后端交互        |
| Adv3    | 安全机制     | Docker 容器控制、命令过滤、资源限制            |
| Adv4    | 代码查重     | 查重算法实现、抄袭检测                         |

---

## API 文档

所有接口、参数、异常、状态码等详见 [api.md](api.md)。

---

## 评分标准

参见 [requirements.md](requirements.md)

---

### 学习资源

- **技术教程**:
  - [系统设计基础](https://github.com/donnemartin/system-design-primer)
  - [Python 并发编程](https://docs.python.org/3/library/concurrent.futures.html)
  - [Docker 容器技术](https://docs.docker.com/)

- **参考项目**:
  - [Codeforces](https://codeforces.com/) - 知名OJ平台
  - [LeetCode](https://leetcode.com/) - 编程练习平台
  - [HackerRank](https://www.hackerrank.com/) - 技术评测平台