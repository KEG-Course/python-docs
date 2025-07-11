## Advance 2：智能代码分析与LLM辅助评测

---

### 模块目标

本模块旨在构建基于大语言模型的智能代码分析系统，实现：

* 代码质量自动评估与改进建议；
* 智能错误诊断与修复建议；
* 代码相似度检测与抄袭识别；
* 个性化学习路径推荐；
* 智能题目难度评估与调整。

---

### 前置知识要求

| 技术点         | 推荐学习内容                         |
| ----------- | ------------------------------ |
| LLM API 调用   | OpenAI API、Claude API、本地模型调用    |
| 代码解析       | `ast`、`astroid`、`tree-sitter`      |
| 向量数据库      | `chromadb`、`faiss`、`pinecone`      |
| 自然语言处理    | `transformers`、`sentence-transformers` |
| 机器学习基础    | `scikit-learn`、`numpy`、`pandas`     |
| 异步编程       | `asyncio`、`aiohttp`、`httpx`         |

---

### 任务拆解

#### 任务 1：代码质量智能评估

**功能描述：**
使用LLM对提交的代码进行多维度质量评估，包括代码风格、算法效率、可读性等。

**实现方案：**

```python
class CodeQualityAnalyzer:
    def __init__(self, llm_client):
        self.llm = llm_client
        self.metrics = {
            "readability": 0.0,
            "efficiency": 0.0,
            "maintainability": 0.0,
            "documentation": 0.0
        }
    
    async def analyze_code(self, source_code: str, language: str) -> dict:
        prompt = f"""
        请分析以下{language}代码的质量，从以下维度评分（0-10分）：
        1. 可读性：代码结构清晰，命名规范
        2. 效率：算法复杂度合理，资源使用优化
        3. 可维护性：代码模块化，易于修改扩展
        4. 文档性：注释充分，文档完整
        
        代码：
        {source_code}
        
        请以JSON格式返回评分和建议。
        """
        
        response = await self.llm.chat_completion(prompt)
        return self._parse_quality_response(response)
```

**评估结果结构：**

```json
{
  "quality_score": 8.5,
  "metrics": {
    "readability": 9.0,
    "efficiency": 8.0,
    "maintainability": 8.5,
    "documentation": 8.0
  },
  "suggestions": [
    "建议添加函数文档字符串",
    "变量命名可以更具描述性",
    "考虑使用更高效的算法"
  ],
  "code_improvements": [
    {
      "line": 15,
      "suggestion": "使用列表推导式替代循环",
      "improved_code": "result = [x*2 for x in numbers]"
    }
  ]
}
```

#### 任务 2：智能错误诊断与修复

**功能描述：**
当代码编译或运行时出错，使用LLM分析错误信息并提供修复建议。

**实现方案：**

```python
class ErrorDiagnoser:
    def __init__(self, llm_client):
        self.llm = llm_client
        self.error_patterns = self._load_error_patterns()
    
    async def diagnose_error(self, source_code: str, error_msg: str, 
                           language: str, test_case: str = None) -> dict:
        prompt = f"""
        分析以下{language}代码的错误：
        
        代码：
        {source_code}
        
        错误信息：
        {error_msg}
        
        {f'测试用例：{test_case}' if test_case else ''}
        
        请提供：
        1. 错误原因分析
        2. 修复建议
        3. 修复后的代码
        4. 预防类似错误的建议
        """
        
        response = await self.llm.chat_completion(prompt)
        return self._parse_diagnosis_response(response)
```

**诊断结果结构：**

```json
{
  "error_type": "IndexError",
  "root_cause": "数组越界访问",
  "severity": "high",
  "fix_suggestions": [
    {
      "description": "添加边界检查",
      "code_fix": "if i < len(arr):\n    return arr[i]\nelse:\n    return None",
      "confidence": 0.95
    }
  ],
  "prevention_tips": [
    "使用enumerate()遍历列表",
    "添加边界条件检查",
    "使用try-except处理异常"
  ],
  "learning_resources": [
    "Python列表操作教程",
    "异常处理最佳实践"
  ]
}
```

#### 任务 3：代码相似度检测

**功能描述：**
使用多种技术检测代码相似度，识别可能的抄袭行为。

**实现方案：**

```python
class SimilarityDetector:
    def __init__(self, vector_db, llm_client):
        self.vector_db = vector_db
        self.llm = llm_client
        self.ast_analyzer = ASTAnalyzer()
    
    async def detect_similarity(self, source_code: str, problem_id: int) -> dict:
        # 1. AST结构分析
        ast_similarity = self.ast_analyzer.compare_structure(source_code)
        
        # 2. 语义向量分析
        semantic_similarity = await self._semantic_analysis(source_code)
        
        # 3. LLM深度分析
        llm_analysis = await self._llm_similarity_analysis(source_code)
        
        return {
            "overall_similarity": self._combine_scores([
                ast_similarity, semantic_similarity, llm_analysis
            ]),
            "details": {
                "ast_similarity": ast_similarity,
                "semantic_similarity": semantic_similarity,
                "llm_analysis": llm_analysis
            },
            "suspicious_submissions": self._find_similar_submissions(
                source_code, problem_id
            )
        }
    
    async def _semantic_analysis(self, code: str) -> float:
        # 使用sentence-transformers生成代码向量
        code_vector = self._encode_code(code)
        similar_codes = self.vector_db.search(code_vector, top_k=10)
        return self._calculate_similarity_score(similar_codes)
    
    async def _llm_similarity_analysis(self, code: str) -> dict:
        prompt = f"""
        分析以下代码的独特性，考虑：
        1. 算法思路的原创性
        2. 代码结构的独特性
        3. 变量命名和注释的原创性
        
        代码：
        {code}
        
        请给出原创性评分（0-10）和详细分析。
        """
        return await self.llm.chat_completion(prompt)
```

#### 任务 4：个性化学习推荐

**功能描述：**
基于用户的学习历史和代码分析结果，推荐个性化的学习路径和练习题目。

**实现方案：**

```python
class LearningRecommender:
    def __init__(self, llm_client, user_db, problem_db):
        self.llm = llm_client
        self.user_db = user_db
        self.problem_db = problem_db
    
    async def generate_recommendations(self, user_id: int) -> dict:
        # 获取用户学习历史
        user_history = self._get_user_history(user_id)
        
        # 分析用户薄弱环节
        weak_areas = await self._analyze_weak_areas(user_history)
        
        # 生成学习建议
        learning_path = await self._generate_learning_path(weak_areas)
        
        # 推荐练习题目
        recommended_problems = self._recommend_problems(weak_areas)
        
        return {
            "weak_areas": weak_areas,
            "learning_path": learning_path,
            "recommended_problems": recommended_problems,
            "estimated_completion_time": self._estimate_time(learning_path)
        }
    
    async def _analyze_weak_areas(self, history: dict) -> list:
        prompt = f"""
        基于以下学习历史，分析用户的薄弱环节：
        
        提交历史：{history['submissions']}
        错误模式：{history['error_patterns']}
        得分分布：{history['score_distribution']}
        
        请识别：
        1. 算法理解薄弱点
        2. 编程语言掌握不足
        3. 代码质量改进空间
        4. 学习进度建议
        """
        
        analysis = await self.llm.chat_completion(prompt)
        return self._parse_weak_areas(analysis)
    
    async def _generate_learning_path(self, weak_areas: list) -> dict:
        prompt = f"""
        为以下薄弱环节设计个性化学习路径：
        {weak_areas}
        
        请提供：
        1. 学习目标优先级
        2. 每个目标的学习资源
        3. 练习计划时间安排
        4. 阶段性评估标准
        """
        
        return await self.llm.chat_completion(prompt)
```

#### 任务 5：智能题目难度评估

**功能描述：**
使用LLM和数据分析自动评估题目难度，并动态调整。

**实现方案：**

```python
class DifficultyAssessor:
    def __init__(self, llm_client, submission_db):
        self.llm = llm_client
        self.submission_db = submission_db
    
    async def assess_difficulty(self, problem_id: int) -> dict:
        # 获取题目信息
        problem_info = self._get_problem_info(problem_id)
        
        # 获取提交数据
        submission_stats = self._get_submission_stats(problem_id)
        
        # LLM分析题目复杂度
        complexity_analysis = await self._analyze_complexity(problem_info)
        
        # 统计分析难度指标
        statistical_difficulty = self._calculate_statistical_difficulty(
            submission_stats
        )
        
        # 综合评估
        final_difficulty = self._combine_difficulty_scores([
            complexity_analysis, statistical_difficulty
        ])
        
        return {
            "problem_id": problem_id,
            "difficulty_score": final_difficulty,
            "difficulty_level": self._map_to_level(final_difficulty),
            "analysis": {
                "complexity_analysis": complexity_analysis,
                "statistical_analysis": statistical_difficulty
            },
            "recommendations": await self._generate_difficulty_recommendations(
                problem_info, final_difficulty
            )
        }
    
    async def _analyze_complexity(self, problem_info: dict) -> dict:
        prompt = f"""
        分析以下题目的算法复杂度：
        
        题目描述：{problem_info['description']}
        输入输出：{problem_info['io_spec']}
        约束条件：{problem_info['constraints']}
        
        请评估：
        1. 最优解的时间复杂度
        2. 空间复杂度要求
        3. 算法思路的复杂度
        4. 实现难度
        5. 常见陷阱和难点
        """
        
        return await self.llm.chat_completion(prompt)
```

---

### API接口设计

#### POST /analysis/quality
```json
{
  "source_code": "def solve(n): return n * 2",
  "language": "python3",
  "problem_id": 1001
}
```

#### POST /analysis/diagnose
```json
{
  "source_code": "def solve(n): return n / 0",
  "error_message": "ZeroDivisionError: division by zero",
  "language": "python3",
  "test_case": "n = 0"
}
```

#### POST /analysis/similarity
```json
{
  "source_code": "def solve(n): return n * 2",
  "problem_id": 1001,
  "threshold": 0.8
}
```

#### GET /recommendations/{user_id}
```json
{
  "weak_areas": ["动态规划", "图论算法"],
  "learning_path": [
    {
      "topic": "动态规划基础",
      "resources": ["教程链接", "练习题目"],
      "estimated_time": "2小时"
    }
  ],
  "recommended_problems": [1002, 1005, 1008]
}
```

---

### 测试建议

| 测试项                    | 期望行为                                |
| ---------------------- | ----------------------------------- |
| 代码质量评估准确性           | 高质量代码获得高分，低质量代码获得低分           |
| 错误诊断准确性             | 能准确识别错误原因并提供有效修复建议            |
| 相似度检测有效性           | 相似代码被正确识别，不同代码不被误判             |
| 推荐系统个性化            | 不同用户获得不同的学习建议和题目推荐            |
| 难度评估准确性            | 评估结果与实际通过率相符                 |
| LLM响应时间              | 在合理时间内返回分析结果（<30秒）            |
| 系统稳定性               | 大量并发请求时系统稳定运行                |

---

### 模块结构建议

```
llm_analysis/
├── clients/
│   ├── openai_client.py
│   ├── claude_client.py
│   └── local_model_client.py
├── analyzers/
│   ├── quality_analyzer.py
│   ├── error_diagnoser.py
│   ├── similarity_detector.py
│   └── difficulty_assessor.py
├── recommenders/
│   ├── learning_recommender.py
│   └── problem_recommender.py
├── utils/
│   ├── ast_analyzer.py
│   ├── vector_encoder.py
│   └── prompt_templates.py
└── routes/
    ├── analysis.py
    └── recommendations.py
```

---

### 能力小结

| 功能      | 支持说明                          |
| ------- | ----------------------------- |
| 代码质量评估 | 多维度质量分析，提供改进建议              |
| 智能错误诊断 | 准确识别错误原因，提供修复方案            |
| 相似度检测   | 多技术融合，有效识别代码相似性            |
| 学习推荐    | 个性化学习路径，智能题目推荐             |
| 难度评估    | 动态难度调整，基于数据和LLM分析          |
| 实时分析    | 异步处理，支持高并发分析请求             |