## Advance 4：智能题目管理与自动生成

---

### 模块目标

本模块旨在构建智能的题目管理系统，实现：

* 基于提交数据的自动难度评估；
* 知识点图谱构建与学习路径推荐；
* 智能题目生成与变体创建；
* 题目质量评估与优化建议；
* 个性化题目推荐系统。

---

### 前置知识要求

| 技术点         | 推荐学习内容                         |
| ----------- | ------------------------------ |
| 机器学习       | scikit-learn、numpy、pandas         |
| 自然语言处理   | transformers、spaCy、NLTK          |
| 图算法        | NetworkX、图遍历、最短路径算法          |
| 数据挖掘      | 聚类分析、关联规则、异常检测             |
| 推荐系统      | 协同过滤、内容推荐、混合推荐算法          |
| 文本生成      | GPT、模板生成、参数化题目创建           |

---

### 任务拆解

#### 任务 1：自动难度评估系统

**功能描述：**
基于用户提交数据自动评估题目难度，并动态调整。

**实现方案：**

```python
import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForestRegressor
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

class DifficultyAssessor:
    def __init__(self):
        self.model = RandomForestRegressor(n_estimators=100, random_state=42)
        self.scaler = StandardScaler()
        self.feature_names = [
            'acceptance_rate', 'avg_submission_count', 'avg_time',
            'avg_memory', 'error_rate', 'time_limit_exceeded_rate',
            'wrong_answer_rate', 'compile_error_rate'
        ]
        
    def extract_features(self, problem_id: int, submissions: List[Dict]) -> Dict:
        """提取题目特征"""
        if not submissions:
            return {}
        
        df = pd.DataFrame(submissions)
        
        features = {}
        
        # 通过率
        features['acceptance_rate'] = len(df[df['status'] == 'Accepted']) / len(df)
        
        # 平均提交次数
        user_submissions = df.groupby('user_id').size()
        features['avg_submission_count'] = user_submissions.mean()
        
        # 平均执行时间
        features['avg_time'] = df[df['time'].notna()]['time'].mean()
        
        # 平均内存使用
        features['avg_memory'] = df[df['memory'].notna()]['memory'].mean()
        
        # 错误率统计
        total_submissions = len(df)
        features['error_rate'] = len(df[df['status'] == 'Runtime Error']) / total_submissions
        features['time_limit_exceeded_rate'] = len(df[df['status'] == 'Time Limit Exceeded']) / total_submissions
        features['wrong_answer_rate'] = len(df[df['status'] == 'Wrong Answer']) / total_submissions
        features['compile_error_rate'] = len(df[df['status'] == 'Compile Error']) / total_submissions
        
        return features
    
    def train_model(self, training_data: List[Dict]):
        """训练难度评估模型"""
        X = []
        y = []
        
        for item in training_data:
            features = self.extract_features(item['problem_id'], item['submissions'])
            if features:
                X.append([features.get(f, 0) for f in self.feature_names])
                y.append(item['manual_difficulty'])  # 人工标注的难度
        
        if X:
            X = np.array(X)
            y = np.array(y)
            
            # 标准化特征
            X_scaled = self.scaler.fit_transform(X)
            
            # 训练模型
            self.model.fit(X_scaled, y)
    
    def predict_difficulty(self, problem_id: int, submissions: List[Dict]) -> Dict:
        """预测题目难度"""
        features = self.extract_features(problem_id, submissions)
        
        if not features:
            return {"difficulty": "Unknown", "confidence": 0.0}
        
        # 准备特征向量
        X = np.array([[features.get(f, 0) for f in self.feature_names]])
        X_scaled = self.scaler.transform(X)
        
        # 预测难度
        predicted_difficulty = self.model.predict(X_scaled)[0]
        
        # 计算置信度（基于特征完整性）
        feature_completeness = sum(1 for f in self.feature_names if features.get(f, 0) != 0) / len(self.feature_names)
        
        # 映射到难度等级
        difficulty_level = self._map_to_difficulty_level(predicted_difficulty)
        
        return {
            "difficulty_score": float(predicted_difficulty),
            "difficulty_level": difficulty_level,
            "confidence": feature_completeness,
            "features": features,
            "feature_importance": self._get_feature_importance()
        }
    
    def _map_to_difficulty_level(self, score: float) -> str:
        """将分数映射到难度等级"""
        if score < 2.0:
            return "Easy"
        elif score < 4.0:
            return "Medium"
        elif score < 6.0:
            return "Hard"
        else:
            return "Expert"
    
    def _get_feature_importance(self) -> Dict:
        """获取特征重要性"""
        importance = self.model.feature_importances_
        return dict(zip(self.feature_names, importance))
    
    def analyze_difficulty_trends(self, problem_id: int, time_period: str = "month") -> Dict:
        """分析难度变化趋势"""
        # 获取时间序列数据
        submissions = self._get_submissions_by_period(problem_id, time_period)
        
        trends = []
        for period, period_submissions in submissions.items():
            difficulty = self.predict_difficulty(problem_id, period_submissions)
            trends.append({
                "period": period,
                "difficulty_score": difficulty["difficulty_score"],
                "submission_count": len(period_submissions)
            })
        
        return {
            "problem_id": problem_id,
            "trends": trends,
            "stability_score": self._calculate_stability_score(trends)
        }
```

#### 任务 2：知识点图谱构建

**功能描述：**
构建题目知识点图谱，支持学习路径推荐。

**实现方案：**

```python
import networkx as nx
from collections import defaultdict
import json

class KnowledgeGraph:
    def __init__(self):
        self.graph = nx.DiGraph()
        self.knowledge_nodes = {}
        self.problem_knowledge_mapping = {}
        
    def add_knowledge_node(self, knowledge_id: str, name: str, 
                          description: str, difficulty: float = 1.0):
        """添加知识点节点"""
        self.graph.add_node(knowledge_id, 
                           name=name,
                           description=description,
                           difficulty=difficulty)
        
        self.knowledge_nodes[knowledge_id] = {
            "id": knowledge_id,
            "name": name,
            "description": description,
            "difficulty": difficulty
        }
    
    def add_knowledge_edge(self, from_knowledge: str, to_knowledge: str, 
                          relationship: str = "prerequisite", weight: float = 1.0):
        """添加知识点关系边"""
        self.graph.add_edge(from_knowledge, to_knowledge,
                           relationship=relationship,
                           weight=weight)
    
    def map_problem_to_knowledge(self, problem_id: int, knowledge_ids: List[str]):
        """将题目映射到知识点"""
        self.problem_knowledge_mapping[problem_id] = knowledge_ids
        
        # 更新知识点的题目统计
        for knowledge_id in knowledge_ids:
            if knowledge_id in self.graph.nodes:
                current_problems = self.graph.nodes[knowledge_id].get('problems', [])
                if problem_id not in current_problems:
                    current_problems.append(problem_id)
                    self.graph.nodes[knowledge_id]['problems'] = current_problems
    
    def analyze_knowledge_dependencies(self) -> Dict:
        """分析知识点依赖关系"""
        # 计算入度和出度
        in_degrees = dict(self.graph.in_degree())
        out_degrees = dict(self.graph.out_degree())
        
        # 识别基础知识点（入度为0）
        basic_knowledge = [node for node, degree in in_degrees.items() if degree == 0]
        
        # 识别高级知识点（出度为0）
        advanced_knowledge = [node for node, degree in out_degrees.items() if degree == 0]
        
        # 计算中心性指标
        betweenness_centrality = nx.betweenness_centrality(self.graph)
        closeness_centrality = nx.closeness_centrality(self.graph)
        
        return {
            "basic_knowledge": basic_knowledge,
            "advanced_knowledge": advanced_knowledge,
            "central_nodes": sorted(betweenness_centrality.items(), 
                                  key=lambda x: x[1], reverse=True)[:10],
            "graph_density": nx.density(self.graph),
            "is_acyclic": nx.is_directed_acyclic_graph(self.graph)
        }
    
    def find_learning_path(self, target_knowledge: str, 
                          user_knowledge: List[str] = None) -> Dict:
        """查找学习路径"""
        if user_knowledge is None:
            user_knowledge = []
        
        # 找到从用户已知知识到目标知识的最短路径
        paths = []
        for start_knowledge in user_knowledge:
            try:
                path = nx.shortest_path(self.graph, start_knowledge, target_knowledge)
                paths.append(path)
            except nx.NetworkXNoPath:
                continue
        
        if not paths:
            # 如果没有路径，从基础知识点开始
            basic_knowledge = [node for node, degree in self.graph.in_degree() if degree == 0]
            for start_knowledge in basic_knowledge:
                try:
                    path = nx.shortest_path(self.graph, start_knowledge, target_knowledge)
                    paths.append(path)
                except nx.NetworkXNoPath:
                    continue
        
        if paths:
            # 选择最短路径
            shortest_path = min(paths, key=len)
            
            # 构建学习路径详情
            learning_path = []
            for i, knowledge_id in enumerate(shortest_path):
                node_data = self.graph.nodes[knowledge_id]
                learning_path.append({
                    "step": i + 1,
                    "knowledge_id": knowledge_id,
                    "name": node_data["name"],
                    "description": node_data["description"],
                    "difficulty": node_data["difficulty"],
                    "recommended_problems": node_data.get("problems", [])[:5]
                })
            
            return {
                "target_knowledge": target_knowledge,
                "path_length": len(shortest_path),
                "learning_path": learning_path,
                "estimated_time": self._estimate_learning_time(learning_path)
            }
        
        return {"error": "No learning path found"}
    
    def _estimate_learning_time(self, learning_path: List[Dict]) -> str:
        """估算学习时间"""
        total_difficulty = sum(step["difficulty"] for step in learning_path)
        estimated_hours = total_difficulty * 2  # 每个难度点约2小时
        
        if estimated_hours < 24:
            return f"{estimated_hours}小时"
        else:
            days = estimated_hours // 24
            hours = estimated_hours % 24
            return f"{days}天{hours}小时"
    
    def get_knowledge_recommendations(self, user_id: int, 
                                    user_history: List[Dict]) -> Dict:
        """基于用户历史推荐知识点"""
        # 分析用户已掌握的知识点
        mastered_knowledge = self._analyze_user_knowledge(user_history)
        
        # 找到相邻的知识点
        adjacent_knowledge = set()
        for knowledge_id in mastered_knowledge:
            successors = list(self.graph.successors(knowledge_id))
            adjacent_knowledge.update(successors)
        
        # 过滤掉已掌握的知识点
        recommended_knowledge = adjacent_knowledge - set(mastered_knowledge)
        
        # 为推荐的知识点计算优先级
        recommendations = []
        for knowledge_id in recommended_knowledge:
            node_data = self.graph.nodes[knowledge_id]
            priority = self._calculate_recommendation_priority(
                knowledge_id, mastered_knowledge, user_history
            )
            
            recommendations.append({
                "knowledge_id": knowledge_id,
                "name": node_data["name"],
                "description": node_data["description"],
                "difficulty": node_data["difficulty"],
                "priority": priority,
                "prerequisites": list(self.graph.predecessors(knowledge_id))
            })
        
        # 按优先级排序
        recommendations.sort(key=lambda x: x["priority"], reverse=True)
        
        return {
            "mastered_knowledge": mastered_knowledge,
            "recommendations": recommendations[:10]
        }
    
    def _analyze_user_knowledge(self, user_history: List[Dict]) -> List[str]:
        """分析用户已掌握的知识点"""
        mastered_knowledge = set()
        
        for submission in user_history:
            if submission["status"] == "Accepted":
                problem_id = submission["problem_id"]
                if problem_id in self.problem_knowledge_mapping:
                    knowledge_ids = self.problem_knowledge_mapping[problem_id]
                    mastered_knowledge.update(knowledge_ids)
        
        return list(mastered_knowledge)
    
    def _calculate_recommendation_priority(self, knowledge_id: str, 
                                         mastered_knowledge: List[str],
                                         user_history: List[Dict]) -> float:
        """计算推荐优先级"""
        priority = 0.0
        
        # 基于难度匹配
        node_data = self.graph.nodes[knowledge_id]
        user_avg_difficulty = self._calculate_user_avg_difficulty(user_history)
        difficulty_match = 1.0 - abs(node_data["difficulty"] - user_avg_difficulty) / 5.0
        priority += difficulty_match * 0.4
        
        # 基于前置知识掌握度
        prerequisites = list(self.graph.predecessors(knowledge_id))
        if prerequisites:
            mastered_prereqs = sum(1 for p in prerequisites if p in mastered_knowledge)
            prereq_ratio = mastered_prereqs / len(prerequisites)
            priority += prereq_ratio * 0.3
        
        # 基于中心性
        betweenness = nx.betweenness_centrality(self.graph)[knowledge_id]
        priority += betweenness * 0.3
        
        return priority
```

#### 任务 3：智能题目生成

**功能描述：**
基于模板和参数化方法自动生成题目变体。

**实现方案：**

```python
import random
import re
from typing import Dict, List, Tuple
import json

class ProblemGenerator:
    def __init__(self):
        self.templates = self._load_templates()
        self.parameter_ranges = self._load_parameter_ranges()
        
    def _load_templates(self) -> Dict:
        """加载题目模板"""
        return {
            "array_sum": {
                "description": "给定一个长度为{n}的数组，数组中的元素为{min_val}到{max_val}之间的整数。请计算数组中所有元素的和。",
                "input_format": "第一行包含一个整数{n}，表示数组长度。\n第二行包含{n}个整数，表示数组元素。",
                "output_format": "输出一个整数，表示数组元素的和。",
                "constraints": "1 ≤ {n} ≤ {max_n}\n{min_val} ≤ 数组元素 ≤ {max_val}",
                "solution_template": "def solve(n, arr):\n    return sum(arr)",
                "test_case_generator": self._generate_array_sum_test_cases
            },
            "binary_search": {
                "description": "给定一个长度为{n}的有序数组和一个目标值{target}，请使用二分查找算法找到目标值在数组中的位置。如果目标值不存在，返回-1。",
                "input_format": "第一行包含两个整数{n}和{target}。\n第二行包含{n}个有序整数。",
                "output_format": "输出一个整数，表示目标值的位置（从0开始），如果不存在输出-1。",
                "constraints": "1 ≤ {n} ≤ {max_n}\n数组元素是有序的\n-{max_val} ≤ 数组元素 ≤ {max_val}",
                "solution_template": "def solve(n, target, arr):\n    left, right = 0, n-1\n    while left <= right:\n        mid = (left + right) // 2\n        if arr[mid] == target:\n            return mid\n        elif arr[mid] < target:\n            left = mid + 1\n        else:\n            right = mid - 1\n    return -1",
                "test_case_generator": self._generate_binary_search_test_cases
            }
        }
    
    def _load_parameter_ranges(self) -> Dict:
        """加载参数范围"""
        return {
            "n": {"min": 5, "max": 1000},
            "max_n": {"min": 10, "max": 10000},
            "min_val": {"min": -1000, "max": 0},
            "max_val": {"min": 100, "max": 10000},
            "target": {"min": -1000, "max": 10000}
        }
    
    def generate_problem_variant(self, template_name: str, 
                               difficulty_level: str = "medium") -> Dict:
        """生成题目变体"""
        if template_name not in self.templates:
            raise ValueError(f"Template {template_name} not found")
        
        template = self.templates[template_name]
        
        # 根据难度调整参数范围
        adjusted_ranges = self._adjust_parameters_for_difficulty(
            self.parameter_ranges, difficulty_level
        )
        
        # 生成随机参数
        parameters = self._generate_random_parameters(adjusted_ranges)
        
        # 生成题目内容
        problem = {
            "description": template["description"].format(**parameters),
            "input_format": template["input_format"].format(**parameters),
            "output_format": template["output_format"].format(**parameters),
            "constraints": template["constraints"].format(**parameters),
            "solution": template["solution_template"],
            "test_cases": template["test_case_generator"](parameters),
            "difficulty_level": difficulty_level,
            "template_name": template_name,
            "parameters": parameters
        }
        
        return problem
    
    def _adjust_parameters_for_difficulty(self, ranges: Dict, 
                                        difficulty: str) -> Dict:
        """根据难度调整参数范围"""
        adjusted = {}
        
        for param, range_dict in ranges.items():
            min_val = range_dict["min"]
            max_val = range_dict["max"]
            range_size = max_val - min_val
            
            if difficulty == "easy":
                # 缩小范围，使用较小的值
                adjusted[param] = {
                    "min": min_val,
                    "max": min_val + int(range_size * 0.3)
                }
            elif difficulty == "medium":
                # 使用中等范围
                adjusted[param] = {
                    "min": min_val + int(range_size * 0.2),
                    "max": min_val + int(range_size * 0.8)
                }
            else:  # hard
                # 使用较大范围
                adjusted[param] = {
                    "min": min_val + int(range_size * 0.6),
                    "max": max_val
                }
        
        return adjusted
    
    def _generate_random_parameters(self, ranges: Dict) -> Dict:
        """生成随机参数"""
        parameters = {}
        
        for param, range_dict in ranges.items():
            if param == "max_n":
                # max_n应该大于等于n
                n = parameters.get("n", random.randint(range_dict["min"], range_dict["max"]))
                parameters[param] = max(n, range_dict["max"])
            elif param == "target":
                # target应该在数组范围内
                min_val = parameters.get("min_val", range_dict["min"])
                max_val = parameters.get("max_val", range_dict["max"])
                parameters[param] = random.randint(min_val, max_val)
            else:
                parameters[param] = random.randint(range_dict["min"], range_dict["max"])
        
        return parameters
    
    def _generate_array_sum_test_cases(self, parameters: Dict) -> List[Dict]:
        """生成数组求和测试用例"""
        test_cases = []
        
        # 基础测试用例
        n = parameters["n"]
        min_val = parameters["min_val"]
        max_val = parameters["max_val"]
        
        # 小规模测试
        small_n = min(n, 10)
        arr = [random.randint(min_val, max_val) for _ in range(small_n)]
        test_cases.append({
            "input": f"{small_n}\n{' '.join(map(str, arr))}",
            "output": str(sum(arr)),
            "description": "小规模测试"
        })
        
        # 大规模测试
        arr = [random.randint(min_val, max_val) for _ in range(n)]
        test_cases.append({
            "input": f"{n}\n{' '.join(map(str, arr))}",
            "output": str(sum(arr)),
            "description": "大规模测试"
        })
        
        # 边界测试
        arr = [min_val] * n
        test_cases.append({
            "input": f"{n}\n{' '.join(map(str, arr))}",
            "output": str(sum(arr)),
            "description": "最小值边界测试"
        })
        
        return test_cases
    
    def _generate_binary_search_test_cases(self, parameters: Dict) -> List[Dict]:
        """生成二分查找测试用例"""
        test_cases = []
        
        n = parameters["n"]
        target = parameters["target"]
        min_val = parameters["min_val"]
        max_val = parameters["max_val"]
        
        # 生成有序数组
        arr = sorted([random.randint(min_val, max_val) for _ in range(n)])
        
        # 目标值存在的测试
        if target in arr:
            expected = arr.index(target)
        else:
            expected = -1
        
        test_cases.append({
            "input": f"{n} {target}\n{' '.join(map(str, arr))}",
            "output": str(expected),
            "description": "目标值存在测试" if expected != -1 else "目标值不存在测试"
        })
        
        # 边界测试
        arr = [i for i in range(min_val, min_val + n)]
        target = min_val + n // 2
        test_cases.append({
            "input": f"{n} {target}\n{' '.join(map(str, arr))}",
            "output": str(n // 2),
            "description": "边界测试"
        })
        
        return test_cases
    
    def generate_problem_series(self, template_name: str, 
                              count: int = 5) -> List[Dict]:
        """生成题目系列"""
        problems = []
        difficulties = ["easy", "medium", "hard"]
        
        for i in range(count):
            difficulty = difficulties[i % len(difficulties)]
            problem = self.generate_problem_variant(template_name, difficulty)
            problem["series_id"] = f"{template_name}_series_{i+1}"
            problems.append(problem)
        
        return problems
```

#### 任务 4：题目质量评估

**功能描述：**
评估生成题目的质量，提供优化建议。

**实现方案：**

```python
class ProblemQualityAssessor:
    def __init__(self):
        self.quality_metrics = {
            "clarity": 0.0,
            "uniqueness": 0.0,
            "difficulty_consistency": 0.0,
            "test_case_coverage": 0.0,
            "solution_correctness": 0.0
        }
    
    def assess_problem_quality(self, problem: Dict) -> Dict:
        """评估题目质量"""
        assessment = {}
        
        # 清晰度评估
        assessment["clarity"] = self._assess_clarity(problem)
        
        # 独特性评估
        assessment["uniqueness"] = self._assess_uniqueness(problem)
        
        # 难度一致性评估
        assessment["difficulty_consistency"] = self._assess_difficulty_consistency(problem)
        
        # 测试用例覆盖率评估
        assessment["test_case_coverage"] = self._assess_test_case_coverage(problem)
        
        # 解答正确性评估
        assessment["solution_correctness"] = self._assess_solution_correctness(problem)
        
        # 计算总体质量分数
        total_score = sum(assessment.values()) / len(assessment)
        
        # 生成改进建议
        suggestions = self._generate_improvement_suggestions(assessment)
        
        return {
            "overall_score": total_score,
            "detailed_scores": assessment,
            "suggestions": suggestions,
            "quality_level": self._map_quality_level(total_score)
        }
    
    def _assess_clarity(self, problem: Dict) -> float:
        """评估题目清晰度"""
        score = 0.0
        
        description = problem["description"]
        input_format = problem["input_format"]
        output_format = problem["output_format"]
        constraints = problem["constraints"]
        
        # 描述完整性
        if len(description) > 50:
            score += 0.3
        
        # 输入输出格式清晰度
        if "第一行" in input_format and "第二行" in input_format:
            score += 0.2
        
        if "输出" in output_format:
            score += 0.2
        
        # 约束条件明确性
        if "≤" in constraints and "≥" in constraints:
            score += 0.3
        
        return min(score, 1.0)
    
    def _assess_uniqueness(self, problem: Dict) -> float:
        """评估题目独特性"""
        # 这里简化实现，实际需要与现有题目库比较
        score = 0.5  # 基础分数
        
        # 基于参数变化增加独特性
        parameters = problem.get("parameters", {})
        if len(parameters) > 3:
            score += 0.3
        
        # 基于模板类型
        template_name = problem.get("template_name", "")
        if "binary_search" in template_name:
            score += 0.2
        
        return min(score, 1.0)
    
    def _assess_difficulty_consistency(self, problem: Dict) -> float:
        """评估难度一致性"""
        difficulty_level = problem.get("difficulty_level", "medium")
        parameters = problem.get("parameters", {})
        
        score = 0.5  # 基础分数
        
        # 根据参数大小调整难度一致性
        if difficulty_level == "easy":
            if parameters.get("n", 0) <= 100:
                score += 0.5
        elif difficulty_level == "medium":
            if 50 <= parameters.get("n", 0) <= 1000:
                score += 0.5
        elif difficulty_level == "hard":
            if parameters.get("n", 0) >= 500:
                score += 0.5
        
        return min(score, 1.0)
    
    def _assess_test_case_coverage(self, problem: Dict) -> float:
        """评估测试用例覆盖率"""
        test_cases = problem.get("test_cases", [])
        
        if not test_cases:
            return 0.0
        
        score = 0.0
        
        # 测试用例数量
        if len(test_cases) >= 3:
            score += 0.3
        
        # 测试用例多样性
        descriptions = [tc.get("description", "") for tc in test_cases]
        unique_descriptions = set(descriptions)
        if len(unique_descriptions) >= 2:
            score += 0.4
        
        # 边界测试
        if any("边界" in desc for desc in descriptions):
            score += 0.3
        
        return min(score, 1.0)
    
    def _assess_solution_correctness(self, problem: Dict) -> float:
        """评估解答正确性"""
        solution = problem.get("solution", "")
        test_cases = problem.get("test_cases", [])
        
        if not solution or not test_cases:
            return 0.0
        
        # 这里简化实现，实际需要执行代码验证
        score = 0.5  # 基础分数
        
        # 检查解答是否包含关键元素
        if "def" in solution:
            score += 0.2
        
        if "return" in solution:
            score += 0.2
        
        if len(solution.split('\n')) > 3:
            score += 0.1
        
        return min(score, 1.0)
    
    def _generate_improvement_suggestions(self, assessment: Dict) -> List[str]:
        """生成改进建议"""
        suggestions = []
        
        if assessment["clarity"] < 0.7:
            suggestions.append("建议增加题目描述的详细程度")
        
        if assessment["uniqueness"] < 0.6:
            suggestions.append("建议增加题目的独特性，避免与现有题目过于相似")
        
        if assessment["difficulty_consistency"] < 0.7:
            suggestions.append("建议调整参数范围以保持难度一致性")
        
        if assessment["test_case_coverage"] < 0.6:
            suggestions.append("建议增加更多测试用例，特别是边界情况")
        
        if assessment["solution_correctness"] < 0.7:
            suggestions.append("建议验证解答的正确性")
        
        return suggestions
    
    def _map_quality_level(self, score: float) -> str:
        """映射质量等级"""
        if score >= 0.8:
            return "Excellent"
        elif score >= 0.6:
            return "Good"
        elif score >= 0.4:
            return "Fair"
        else:
            return "Poor"
```

#### 任务 5：个性化题目推荐

**功能描述：**
基于用户学习历史和偏好推荐个性化题目。

**实现方案：**

```python
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

class PersonalizedRecommender:
    def __init__(self):
        self.user_profiles = {}
        self.problem_features = {}
        self.collaborative_matrix = None
        
    def build_user_profile(self, user_id: int, user_history: List[Dict]) -> Dict:
        """构建用户画像"""
        profile = {
            "user_id": user_id,
            "preferred_difficulties": {},
            "preferred_topics": {},
            "learning_pattern": {},
            "performance_stats": {}
        }
        
        # 分析难度偏好
        difficulty_stats = {}
        for submission in user_history:
            difficulty = submission.get("difficulty_level", "medium")
            status = submission.get("status", "Unknown")
            
            if difficulty not in difficulty_stats:
                difficulty_stats[difficulty] = {"total": 0, "accepted": 0}
            
            difficulty_stats[difficulty]["total"] += 1
            if status == "Accepted":
                difficulty_stats[difficulty]["accepted"] += 1
        
        # 计算难度偏好分数
        for difficulty, stats in difficulty_stats.items():
            if stats["total"] > 0:
                acceptance_rate = stats["accepted"] / stats["total"]
                profile["preferred_difficulties"][difficulty] = acceptance_rate
        
        # 分析主题偏好
        topic_stats = {}
        for submission in user_history:
            problem_id = submission.get("problem_id")
            if problem_id and problem_id in self.problem_features:
                topics = self.problem_features[problem_id].get("topics", [])
                for topic in topics:
                    if topic not in topic_stats:
                        topic_stats[topic] = {"total": 0, "accepted": 0}
                    
                    topic_stats[topic]["total"] += 1
                    if submission.get("status") == "Accepted":
                        topic_stats[topic]["accepted"] += 1
        
        for topic, stats in topic_stats.items():
            if stats["total"] > 0:
                profile["preferred_topics"][topic] = stats["accepted"] / stats["total"]
        
        # 分析学习模式
        profile["learning_pattern"] = self._analyze_learning_pattern(user_history)
        
        # 性能统计
        profile["performance_stats"] = self._calculate_performance_stats(user_history)
        
        self.user_profiles[user_id] = profile
        return profile
    
    def _analyze_learning_pattern(self, user_history: List[Dict]) -> Dict:
        """分析学习模式"""
        pattern = {
            "submission_frequency": 0,
            "preferred_time": "unknown",
            "persistence_level": 0,
            "learning_curve": "unknown"
        }
        
        if not user_history:
            return pattern
        
        # 计算提交频率
        total_days = (user_history[-1]["timestamp"] - user_history[0]["timestamp"]).days + 1
        pattern["submission_frequency"] = len(user_history) / total_days
        
        # 分析坚持程度
        consecutive_days = 0
        max_consecutive = 0
        current_consecutive = 0
        
        for i in range(1, len(user_history)):
            day_diff = (user_history[i]["timestamp"] - user_history[i-1]["timestamp"]).days
            if day_diff <= 1:
                current_consecutive += 1
                max_consecutive = max(max_consecutive, current_consecutive)
            else:
                current_consecutive = 0
        
        pattern["persistence_level"] = max_consecutive
        
        return pattern
    
    def _calculate_performance_stats(self, user_history: List[Dict]) -> Dict:
        """计算性能统计"""
        stats = {
            "total_submissions": len(user_history),
            "accepted_submissions": 0,
            "avg_time": 0,
            "avg_memory": 0,
            "improvement_rate": 0
        }
        
        if not user_history:
            return stats
        
        accepted_submissions = [s for s in user_history if s.get("status") == "Accepted"]
        stats["accepted_submissions"] = len(accepted_submissions)
        
        if accepted_submissions:
            times = [s.get("time", 0) for s in accepted_submissions if s.get("time")]
            memories = [s.get("memory", 0) for s in accepted_submissions if s.get("memory")]
            
            if times:
                stats["avg_time"] = sum(times) / len(times)
            if memories:
                stats["avg_memory"] = sum(memories) / len(memories)
        
        # 计算改进率
        if len(user_history) > 1:
            early_submissions = user_history[:len(user_history)//2]
            late_submissions = user_history[len(user_history)//2:]
            
            early_acceptance = sum(1 for s in early_submissions if s.get("status") == "Accepted")
            late_acceptance = sum(1 for s in late_submissions if s.get("status") == "Accepted")
            
            if len(early_submissions) > 0:
                early_rate = early_acceptance / len(early_submissions)
                late_rate = late_acceptance / len(late_submissions)
                stats["improvement_rate"] = late_rate - early_rate
        
        return stats
    
    def recommend_problems(self, user_id: int, count: int = 10) -> List[Dict]:
        """推荐题目"""
        if user_id not in self.user_profiles:
            return []
        
        user_profile = self.user_profiles[user_id]
        recommendations = []
        
        # 基于内容的推荐
        content_based = self._content_based_recommendation(user_profile, count//2)
        recommendations.extend(content_based)
        
        # 基于协同过滤的推荐
        collaborative_based = self._collaborative_filtering_recommendation(user_id, count//2)
        recommendations.extend(collaborative_based)
        
        # 去重并排序
        unique_recommendations = self._deduplicate_recommendations(recommendations)
        sorted_recommendations = self._sort_recommendations(unique_recommendations, user_profile)
        
        return sorted_recommendations[:count]
    
    def _content_based_recommendation(self, user_profile: Dict, count: int) -> List[Dict]:
        """基于内容的推荐"""
        recommendations = []
        
        # 基于难度偏好
        preferred_difficulties = user_profile["preferred_difficulties"]
        for difficulty, score in sorted(preferred_difficulties.items(), 
                                      key=lambda x: x[1], reverse=True):
            difficulty_problems = [p for p in self.problem_features.values() 
                                 if p.get("difficulty_level") == difficulty]
            
            for problem in difficulty_problems[:count//2]:
                recommendations.append({
                    "problem_id": problem["problem_id"],
                    "score": score,
                    "reason": f"匹配您的{difficulty}难度偏好"
                })
        
        # 基于主题偏好
        preferred_topics = user_profile["preferred_topics"]
        for topic, score in sorted(preferred_topics.items(), 
                                 key=lambda x: x[1], reverse=True):
            topic_problems = [p for p in self.problem_features.values() 
                            if topic in p.get("topics", [])]
            
            for problem in topic_problems[:count//2]:
                recommendations.append({
                    "problem_id": problem["problem_id"],
                    "score": score,
                    "reason": f"匹配您的{topic}主题偏好"
                })
        
        return recommendations
    
    def _collaborative_filtering_recommendation(self, user_id: int, count: int) -> List[Dict]:
        """基于协同过滤的推荐"""
        if self.collaborative_matrix is None:
            return []
        
        # 计算用户相似度
        user_similarities = cosine_similarity([self.collaborative_matrix[user_id]], 
                                            self.collaborative_matrix)[0]
        
        # 找到相似用户
        similar_users = np.argsort(user_similarities)[::-1][1:6]  # 前5个相似用户
        
        # 推荐相似用户喜欢的题目
        recommendations = []
        for similar_user in similar_users:
            user_problems = np.where(self.collaborative_matrix[similar_user] > 0)[0]
            
            for problem_id in user_problems:
                if self.collaborative_matrix[user_id][problem_id] == 0:  # 用户未做过
                    recommendations.append({
                        "problem_id": int(problem_id),
                        "score": user_similarities[similar_user],
                        "reason": f"相似用户推荐"
                    })
        
        return recommendations[:count]
    
    def _deduplicate_recommendations(self, recommendations: List[Dict]) -> List[Dict]:
        """去重推荐结果"""
        seen_problems = set()
        unique_recommendations = []
        
        for rec in recommendations:
            if rec["problem_id"] not in seen_problems:
                seen_problems.add(rec["problem_id"])
                unique_recommendations.append(rec)
        
        return unique_recommendations
    
    def _sort_recommendations(self, recommendations: List[Dict], 
                            user_profile: Dict) -> List[Dict]:
        """排序推荐结果"""
        def sort_key(rec):
            base_score = rec["score"]
            
            # 根据用户学习模式调整分数
            learning_pattern = user_profile["learning_pattern"]
            
            # 如果用户喜欢挑战，提高困难题目的分数
            if learning_pattern.get("persistence_level", 0) > 5:
                problem = self.problem_features.get(rec["problem_id"], {})
                if problem.get("difficulty_level") == "hard":
                    base_score *= 1.2
            
            return base_score
        
        return sorted(recommendations, key=sort_key, reverse=True)
```

---

### API接口设计

#### POST /problems/generate
```json
{
  "template_name": "array_sum",
  "difficulty_level": "medium",
  "count": 5
}
```

#### GET /knowledge/graph
```json
{
  "nodes": [
    {
      "id": "dp_basic",
      "name": "动态规划基础",
      "difficulty": 3.0,
      "problem_count": 15
    }
  ],
  "edges": [
    {
      "from": "dp_basic",
      "to": "dp_advanced",
      "relationship": "prerequisite"
    }
  ]
}
```

#### GET /recommendations/{user_id}
```json
{
  "recommendations": [
    {
      "problem_id": 1001,
      "score": 0.85,
      "reason": "匹配您的动态规划主题偏好",
      "difficulty": "medium",
      "estimated_time": "30分钟"
    }
  ],
  "learning_path": [
    {
      "knowledge_id": "dp_basic",
      "name": "动态规划基础",
      "progress": 0.7
    }
  ]
}
```

#### POST /problems/assess
```json
{
  "problem": {
    "description": "...",
    "test_cases": [...]
  }
}
```

---

### 测试建议

| 测试项                    | 期望行为                                |
| ---------------------- | ----------------------------------- |
| 难度评估准确性           | 评估结果与实际通过率相符               |
| 知识点图谱完整性         | 知识点关系正确，学习路径合理            |
| 题目生成质量            | 生成的题目清晰、正确、有挑战性          |
| 推荐系统个性化          | 不同用户获得不同的推荐结果             |
| 质量评估有效性          | 能准确识别题目质量问题               |
| 学习路径推荐           | 推荐的学习路径符合用户当前水平         |
| 系统性能              | 推荐和生成功能响应时间合理            |

---

### 模块结构建议

```
intelligent_management/
├── difficulty/
│   ├── assessor.py
│   ├── model_trainer.py
│   └── trend_analyzer.py
├── knowledge/
│   ├── graph_builder.py
│   ├── path_finder.py
│   └── recommendation_engine.py
├── generation/
│   ├── problem_generator.py
│   ├── template_manager.py
│   └── test_case_generator.py
├── quality/
│   ├── assessor.py
│   ├── validator.py
│   └── optimizer.py
├── recommendation/
│   ├── personalized_recommender.py
│   ├── collaborative_filter.py
│   └── content_based_filter.py
└── api/
    ├── problem_management.py
    ├── knowledge_api.py
    └── recommendation_api.py
```

---

### 能力小结

| 功能      | 支持说明                          |
| ------- | ----------------------------- |
| 难度评估   | 基于机器学习的自动难度评估，动态调整        |
| 知识图谱   | 构建知识点关系图，支持学习路径推荐         |
| 题目生成   | 基于模板的智能题目生成，支持多种变体        |
| 质量评估   | 多维度题目质量评估，提供改进建议          |
| 个性化推荐 | 基于用户画像和协同过滤的智能推荐          |
| 学习分析   | 用户学习模式分析，进度跟踪             |
| 自动优化   | 基于反馈的题目和推荐系统自动优化         |