## Advance 5：智能代码查重系统

---

### 模块目标

本模块旨在构建一个全面的代码查重系统，实现：

* 多维度代码相似度检测（AST、语义、风格）；
* 跨语言查重能力；
* 实时查重与批量检测；
* 查重报告生成与可视化；
* 智能阈值调整与误报过滤。

---

### 前置知识要求

| 技术点         | 推荐学习内容                         |
| ----------- | ------------------------------ |
| 代码解析       | `ast`、`tree-sitter`、`clang`        |
| 自然语言处理   | `transformers`、`sentence-transformers` |
| 图算法        | 图相似度、子图匹配、图神经网络           |
| 机器学习       | 聚类、异常检测、特征工程               |
| 向量数据库     | `faiss`、`chromadb`、`pinecone`      |
| 可视化技术     | `matplotlib`、`plotly`、`networkx`   |

---

### 任务拆解

#### 任务 1：多维度代码特征提取

**功能描述：**
从代码中提取多种特征用于相似度计算。

**实现方案：**

```python
import ast
import re
from typing import Dict, List, Set
import hashlib
from collections import Counter

class CodeFeatureExtractor:
    def __init__(self):
        self.feature_types = ['ast', 'semantic', 'style', 'structure']
    
    def extract_features(self, source_code: str, language: str) -> Dict:
        """提取代码特征"""
        features = {}
        
        # AST特征
        features['ast'] = self._extract_ast_features(source_code, language)
        
        # 语义特征
        features['semantic'] = self._extract_semantic_features(source_code)
        
        # 风格特征
        features['style'] = self._extract_style_features(source_code)
        
        # 结构特征
        features['structure'] = self._extract_structure_features(source_code)
        
        return features
    
    def _extract_ast_features(self, source_code: str, language: str) -> Dict:
        """提取AST特征"""
        if language == 'python3':
            return self._extract_python_ast(source_code)
        elif language == 'cpp':
            return self._extract_cpp_ast(source_code)
        else:
            return self._extract_generic_ast(source_code)
    
    def _extract_python_ast(self, source_code: str) -> Dict:
        """提取Python AST特征"""
        try:
            tree = ast.parse(source_code)
            
            features = {
                'node_types': Counter(),
                'function_calls': [],
                'variables': set(),
                'control_structures': [],
                'complexity_metrics': {}
            }
            
            for node in ast.walk(tree):
                # 统计节点类型
                features['node_types'][type(node).__name__] += 1
                
                # 提取函数调用
                if isinstance(node, ast.Call):
                    if hasattr(node.func, 'id'):
                        features['function_calls'].append(node.func.id)
                
                # 提取变量名
                if isinstance(node, ast.Name):
                    features['variables'].add(node.id)
                
                # 提取控制结构
                if isinstance(node, (ast.If, ast.For, ast.While)):
                    features['control_structures'].append(type(node).__name__)
            
            # 计算复杂度指标
            features['complexity_metrics'] = {
                'cyclomatic_complexity': self._calculate_cyclomatic_complexity(tree),
                'nesting_depth': self._calculate_max_nesting_depth(tree),
                'function_count': len([n for n in ast.walk(tree) if isinstance(n, ast.FunctionDef)])
            }
            
            return features
            
        except SyntaxError:
            return {'error': 'Syntax error in code'}
    
    def _extract_semantic_features(self, source_code: str) -> Dict:
        """提取语义特征"""
        # 提取算法模式
        algorithm_patterns = self._detect_algorithm_patterns(source_code)
        
        # 提取数据结构使用
        data_structures = self._detect_data_structures(source_code)
        
        # 提取编程范式
        programming_paradigms = self._detect_programming_paradigms(source_code)
        
        return {
            'algorithm_patterns': algorithm_patterns,
            'data_structures': data_structures,
            'programming_paradigms': programming_paradigms,
            'semantic_fingerprint': self._generate_semantic_fingerprint(source_code)
        }
    
    def _extract_style_features(self, source_code: str) -> Dict:
        """提取代码风格特征"""
        lines = source_code.split('\n')
        
        features = {
            'indentation_pattern': self._analyze_indentation(lines),
            'naming_conventions': self._analyze_naming_conventions(source_code),
            'comment_style': self._analyze_comment_style(source_code),
            'line_length_distribution': self._analyze_line_lengths(lines),
            'whitespace_usage': self._analyze_whitespace_usage(source_code)
        }
        
        return features
    
    def _extract_structure_features(self, source_code: str) -> Dict:
        """提取结构特征"""
        return {
            'code_length': len(source_code),
            'line_count': len(source_code.split('\n')),
            'function_count': len(re.findall(r'def\s+\w+', source_code)),
            'class_count': len(re.findall(r'class\s+\w+', source_code)),
            'import_count': len(re.findall(r'import\s+|from\s+', source_code)),
            'structure_hash': hashlib.md5(source_code.encode()).hexdigest()
        }
```

#### 任务 2：相似度计算引擎

**功能描述：**
实现多种相似度计算算法。

**实现方案：**

```python
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity
from sklearn.feature_extraction.text import TfidfVectorizer
import networkx as nx
from difflib import SequenceMatcher

class SimilarityCalculator:
    def __init__(self):
        self.weights = {
            'ast': 0.3,
            'semantic': 0.3,
            'style': 0.2,
            'structure': 0.2
        }
    
    def calculate_similarity(self, features1: Dict, features2: Dict) -> Dict:
        """计算综合相似度"""
        similarities = {}
        
        # AST相似度
        similarities['ast'] = self._calculate_ast_similarity(
            features1.get('ast', {}), features2.get('ast', {})
        )
        
        # 语义相似度
        similarities['semantic'] = self._calculate_semantic_similarity(
            features1.get('semantic', {}), features2.get('semantic', {})
        )
        
        # 风格相似度
        similarities['style'] = self._calculate_style_similarity(
            features1.get('style', {}), features2.get('style', {})
        )
        
        # 结构相似度
        similarities['structure'] = self._calculate_structure_similarity(
            features1.get('structure', {}), features2.get('structure', {})
        )
        
        # 计算加权综合相似度
        overall_similarity = sum(
            similarities[dim] * self.weights[dim] 
            for dim in similarities.keys()
        )
        
        return {
            'overall_similarity': overall_similarity,
            'detailed_similarities': similarities,
            'confidence': self._calculate_confidence(similarities)
        }
    
    def _calculate_ast_similarity(self, ast1: Dict, ast2: Dict) -> float:
        """计算AST相似度"""
        if 'error' in ast1 or 'error' in ast2:
            return 0.0
        
        # 节点类型分布相似度
        node_types_sim = self._calculate_distribution_similarity(
            ast1.get('node_types', {}), ast2.get('node_types', {})
        )
        
        # 函数调用相似度
        func_calls_sim = self._calculate_set_similarity(
            set(ast1.get('function_calls', [])), 
            set(ast2.get('function_calls', []))
        )
        
        # 变量名相似度
        vars_sim = self._calculate_set_similarity(
            ast1.get('variables', set()), 
            ast2.get('variables', set())
        )
        
        # 控制结构相似度
        control_sim = self._calculate_sequence_similarity(
            ast1.get('control_structures', []), 
            ast2.get('control_structures', [])
        )
        
        # 复杂度指标相似度
        complexity_sim = self._calculate_complexity_similarity(
            ast1.get('complexity_metrics', {}), 
            ast2.get('complexity_metrics', {})
        )
        
        return (node_types_sim + func_calls_sim + vars_sim + 
                control_sim + complexity_sim) / 5.0
    
    def _calculate_semantic_similarity(self, semantic1: Dict, semantic2: Dict) -> float:
        """计算语义相似度"""
        # 算法模式相似度
        algo_sim = self._calculate_set_similarity(
            set(semantic1.get('algorithm_patterns', [])), 
            set(semantic2.get('algorithm_patterns', []))
        )
        
        # 数据结构相似度
        ds_sim = self._calculate_set_similarity(
            set(semantic1.get('data_structures', [])), 
            set(semantic2.get('data_structures', []))
        )
        
        # 编程范式相似度
        paradigm_sim = self._calculate_set_similarity(
            set(semantic1.get('programming_paradigms', [])), 
            set(semantic2.get('programming_paradigms', []))
        )
        
        # 语义指纹相似度
        fingerprint_sim = self._calculate_fingerprint_similarity(
            semantic1.get('semantic_fingerprint', ''), 
            semantic2.get('semantic_fingerprint', '')
        )
        
        return (algo_sim + ds_sim + paradigm_sim + fingerprint_sim) / 4.0
    
    def _calculate_style_similarity(self, style1: Dict, style2: Dict) -> float:
        """计算风格相似度"""
        similarities = []
        
        # 缩进模式相似度
        if 'indentation_pattern' in style1 and 'indentation_pattern' in style2:
            similarities.append(self._calculate_indentation_similarity(
                style1['indentation_pattern'], style2['indentation_pattern']
            ))
        
        # 命名约定相似度
        if 'naming_conventions' in style1 and 'naming_conventions' in style2:
            similarities.append(self._calculate_naming_similarity(
                style1['naming_conventions'], style2['naming_conventions']
            ))
        
        # 注释风格相似度
        if 'comment_style' in style1 and 'comment_style' in style2:
            similarities.append(self._calculate_comment_similarity(
                style1['comment_style'], style2['comment_style']
            ))
        
        return sum(similarities) / len(similarities) if similarities else 0.0
    
    def _calculate_structure_similarity(self, struct1: Dict, struct2: Dict) -> float:
        """计算结构相似度"""
        # 代码长度相似度
        length_sim = 1.0 - abs(struct1.get('code_length', 0) - 
                              struct2.get('code_length', 0)) / max(
                                  struct1.get('code_length', 1), 
                                  struct2.get('code_length', 1)
                              )
        
        # 函数数量相似度
        func_sim = 1.0 - abs(struct1.get('function_count', 0) - 
                            struct2.get('function_count', 0)) / max(
                                struct1.get('function_count', 1), 
                                struct2.get('function_count', 1)
                            )
        
        # 结构哈希相似度
        hash_sim = 1.0 if struct1.get('structure_hash') == struct2.get('structure_hash') else 0.0
        
        return (length_sim + func_sim + hash_sim) / 3.0
    
    def _calculate_distribution_similarity(self, dist1: Dict, dist2: Dict) -> float:
        """计算分布相似度"""
        all_keys = set(dist1.keys()) | set(dist2.keys())
        if not all_keys:
            return 1.0
        
        total_diff = 0
        for key in all_keys:
            val1 = dist1.get(key, 0)
            val2 = dist2.get(key, 0)
            total_diff += abs(val1 - val2)
        
        max_possible_diff = sum(max(dist1.values()) + max(dist2.values()))
        return 1.0 - (total_diff / max_possible_diff) if max_possible_diff > 0 else 1.0
    
    def _calculate_set_similarity(self, set1: Set, set2: Set) -> float:
        """计算集合相似度（Jaccard相似度）"""
        if not set1 and not set2:
            return 1.0
        
        intersection = len(set1 & set2)
        union = len(set1 | set2)
        return intersection / union if union > 0 else 0.0
```

#### 任务 3：查重检测引擎

**功能描述：**
实现高效的查重检测算法。

**实现方案：**

```python
import faiss
import numpy as np
from typing import List, Dict, Tuple
import asyncio
from concurrent.futures import ThreadPoolExecutor

class PlagiarismDetector:
    def __init__(self, vector_db_path: str = None):
        self.feature_extractor = CodeFeatureExtractor()
        self.similarity_calculator = SimilarityCalculator()
        self.vector_db = self._initialize_vector_db(vector_db_path)
        self.detection_threshold = 0.7
        
    def detect_plagiarism(self, source_code: str, language: str, 
                         problem_id: int = None) -> Dict:
        """检测代码抄袭"""
        # 提取特征
        features = self.feature_extractor.extract_features(source_code, language)
        
        # 向量化特征
        feature_vector = self._vectorize_features(features)
        
        # 在向量数据库中搜索相似代码
        similar_codes = self._search_similar_codes(feature_vector, problem_id)
        
        # 详细相似度分析
        detailed_analysis = self._analyze_similarities(source_code, similar_codes)
        
        # 生成查重报告
        report = self._generate_plagiarism_report(detailed_analysis)
        
        return report
    
    def _initialize_vector_db(self, db_path: str):
        """初始化向量数据库"""
        if db_path and os.path.exists(db_path):
            return faiss.read_index(db_path)
        else:
            # 创建新的向量数据库
            dimension = 512  # 特征向量维度
            return faiss.IndexFlatIP(dimension)  # 内积索引
    
    def _vectorize_features(self, features: Dict) -> np.ndarray:
        """将特征向量化"""
        vector = []
        
        # AST特征向量化
        ast_vector = self._vectorize_ast_features(features.get('ast', {}))
        vector.extend(ast_vector)
        
        # 语义特征向量化
        semantic_vector = self._vectorize_semantic_features(features.get('semantic', {}))
        vector.extend(semantic_vector)
        
        # 风格特征向量化
        style_vector = self._vectorize_style_features(features.get('style', {}))
        vector.extend(style_vector)
        
        # 结构特征向量化
        structure_vector = self._vectorize_structure_features(features.get('structure', {}))
        vector.extend(structure_vector)
        
        # 填充到固定维度
        while len(vector) < 512:
            vector.append(0.0)
        
        return np.array(vector[:512], dtype=np.float32)
    
    def _search_similar_codes(self, feature_vector: np.ndarray, 
                            problem_id: int = None) -> List[Dict]:
        """搜索相似代码"""
        # 搜索向量数据库
        D, I = self.vector_db.search(
            feature_vector.reshape(1, -1), 
            k=20  # 返回前20个最相似的结果
        )
        
        similar_codes = []
        for i, (distance, index) in enumerate(zip(D[0], I[0])):
            if distance > self.detection_threshold:
                code_info = self._get_code_by_index(index)
                if code_info and (problem_id is None or 
                                code_info.get('problem_id') == problem_id):
                    similar_codes.append({
                        'code_info': code_info,
                        'similarity_score': float(distance),
                        'rank': i + 1
                    })
        
        return similar_codes
    
    def _analyze_similarities(self, source_code: str, 
                            similar_codes: List[Dict]) -> List[Dict]:
        """详细分析相似度"""
        detailed_analysis = []
        
        for similar_code in similar_codes:
            # 提取相似代码的特征
            similar_features = self.feature_extractor.extract_features(
                similar_code['code_info']['source_code'],
                similar_code['code_info']['language']
            )
            
            # 计算详细相似度
            current_features = self.feature_extractor.extract_features(
                source_code, similar_code['code_info']['language']
            )
            
            similarity_details = self.similarity_calculator.calculate_similarity(
                current_features, similar_features
            )
            
            # 代码片段对比
            code_comparison = self._compare_code_snippets(
                source_code, similar_code['code_info']['source_code']
            )
            
            detailed_analysis.append({
                'similar_code': similar_code['code_info'],
                'overall_similarity': similarity_details['overall_similarity'],
                'detailed_similarities': similarity_details['detailed_similarities'],
                'code_comparison': code_comparison,
                'plagiarism_probability': self._calculate_plagiarism_probability(
                    similarity_details
                )
            })
        
        return detailed_analysis
    
    def _compare_code_snippets(self, code1: str, code2: str) -> Dict:
        """比较代码片段"""
        # 使用difflib进行序列匹配
        matcher = SequenceMatcher(None, code1, code2)
        
        # 找到最长的匹配块
        matching_blocks = matcher.get_matching_blocks()
        
        # 计算相似度
        similarity_ratio = matcher.ratio()
        
        # 提取匹配的代码片段
        matching_snippets = []
        for a, b, size in matching_blocks:
            if size > 10:  # 只记录长度大于10的匹配块
                matching_snippets.append({
                    'start1': a,
                    'start2': b,
                    'size': size,
                    'snippet1': code1[a:a+size],
                    'snippet2': code2[b:b+size]
                })
        
        return {
            'similarity_ratio': similarity_ratio,
            'matching_blocks': matching_snippets,
            'total_matching_size': sum(block['size'] for block in matching_snippets)
        }
    
    def _calculate_plagiarism_probability(self, similarity_details: Dict) -> float:
        """计算抄袭概率"""
        overall_sim = similarity_details['overall_similarity']
        detailed_sims = similarity_details['detailed_similarities']
        
        # 基础概率
        base_probability = overall_sim
        
        # 根据各维度相似度调整概率
        if detailed_sims['ast'] > 0.8:
            base_probability *= 1.2  # AST高度相似，增加概率
        
        if detailed_sims['style'] > 0.9:
            base_probability *= 1.1  # 风格高度相似，增加概率
        
        if detailed_sims['semantic'] > 0.7:
            base_probability *= 1.15  # 语义相似，增加概率
        
        return min(base_probability, 1.0)
    
    def _generate_plagiarism_report(self, detailed_analysis: List[Dict]) -> Dict:
        """生成查重报告"""
        if not detailed_analysis:
            return {
                'plagiarism_detected': False,
                'similarity_score': 0.0,
                'similar_codes': [],
                'recommendations': ['未检测到抄袭行为']
            }
        
        # 按相似度排序
        detailed_analysis.sort(key=lambda x: x['overall_similarity'], reverse=True)
        
        # 计算最高相似度
        max_similarity = detailed_analysis[0]['overall_similarity']
        
        # 判断是否抄袭
        plagiarism_detected = max_similarity > self.detection_threshold
        
        # 生成建议
        recommendations = self._generate_recommendations(detailed_analysis)
        
        return {
            'plagiarism_detected': plagiarism_detected,
            'similarity_score': max_similarity,
            'similar_codes': detailed_analysis[:5],  # 返回前5个最相似的
            'recommendations': recommendations,
            'detailed_analysis': detailed_analysis
        }
    
    def _generate_recommendations(self, detailed_analysis: List[Dict]) -> List[str]:
        """生成建议"""
        recommendations = []
        
        max_similarity = detailed_analysis[0]['overall_similarity']
        
        if max_similarity > 0.9:
            recommendations.append('检测到高度相似的代码，建议进行人工审查')
        elif max_similarity > 0.7:
            recommendations.append('检测到相似代码，建议检查是否存在抄袭行为')
        elif max_similarity > 0.5:
            recommendations.append('代码存在一定相似性，建议关注算法思路的原创性')
        
        # 根据具体相似维度给出建议
        detailed_sims = detailed_analysis[0]['detailed_similarities']
        
        if detailed_sims['ast'] > 0.8:
            recommendations.append('代码结构高度相似，建议重新设计算法')
        
        if detailed_sims['style'] > 0.9:
            recommendations.append('代码风格高度相似，建议改进编程风格')
        
        return recommendations
```

#### 任务 4：批量查重与实时检测

**功能描述：**
支持批量查重和实时检测功能。

**实现方案：**

```python
class BatchPlagiarismDetector:
    def __init__(self, detector: PlagiarismDetector):
        self.detector = detector
        self.executor = ThreadPoolExecutor(max_workers=4)
    
    async def batch_detect(self, submissions: List[Dict]) -> Dict:
        """批量查重检测"""
        results = []
        
        # 并发处理
        tasks = []
        for submission in submissions:
            task = asyncio.create_task(
                self._detect_single_submission(submission)
            )
            tasks.append(task)
        
        # 等待所有任务完成
        batch_results = await asyncio.gather(*tasks)
        
        # 分析批量结果
        batch_analysis = self._analyze_batch_results(batch_results)
        
        return {
            'total_submissions': len(submissions),
            'plagiarism_count': batch_analysis['plagiarism_count'],
            'suspicious_groups': batch_analysis['suspicious_groups'],
            'individual_results': batch_results,
            'batch_statistics': batch_analysis['statistics']
        }
    
    async def _detect_single_submission(self, submission: Dict) -> Dict:
        """检测单个提交"""
        loop = asyncio.get_event_loop()
        
        result = await loop.run_in_executor(
            self.executor,
            self.detector.detect_plagiarism,
            submission['source_code'],
            submission['language'],
            submission.get('problem_id')
        )
        
        return {
            'submission_id': submission['id'],
            'user_id': submission['user_id'],
            'problem_id': submission.get('problem_id'),
            'detection_result': result
        }
    
    def _analyze_batch_results(self, results: List[Dict]) -> Dict:
        """分析批量结果"""
        plagiarism_count = 0
        suspicious_groups = []
        similarity_scores = []
        
        for result in results:
            if result['detection_result']['plagiarism_detected']:
                plagiarism_count += 1
            
            similarity_scores.append(
                result['detection_result']['similarity_score']
            )
        
        # 识别可疑组
        suspicious_groups = self._identify_suspicious_groups(results)
        
        # 计算统计信息
        statistics = {
            'plagiarism_rate': plagiarism_count / len(results) if results else 0,
            'avg_similarity': np.mean(similarity_scores) if similarity_scores else 0,
            'max_similarity': max(similarity_scores) if similarity_scores else 0,
            'similarity_distribution': self._calculate_similarity_distribution(similarity_scores)
        }
        
        return {
            'plagiarism_count': plagiarism_count,
            'suspicious_groups': suspicious_groups,
            'statistics': statistics
        }
    
    def _identify_suspicious_groups(self, results: List[Dict]) -> List[List[Dict]]:
        """识别可疑组"""
        suspicious_groups = []
        processed = set()
        
        for i, result1 in enumerate(results):
            if i in processed:
                continue
            
            group = [result1]
            processed.add(i)
            
            for j, result2 in enumerate(results[i+1:], i+1):
                if j in processed:
                    continue
                
                # 检查是否与组内成员相似
                if self._are_results_similar(result1, result2):
                    group.append(result2)
                    processed.add(j)
            
            if len(group) > 1:
                suspicious_groups.append(group)
        
        return suspicious_groups
    
    def _are_results_similar(self, result1: Dict, result2: Dict) -> bool:
        """检查两个结果是否相似"""
        # 检查是否有共同的相似代码
        similar_codes1 = {code['similar_code']['id'] 
                         for code in result1['detection_result']['similar_codes']}
        similar_codes2 = {code['similar_code']['id'] 
                         for code in result2['detection_result']['similar_codes']}
        
        return len(similar_codes1 & similar_codes2) > 0
```

#### 任务 5：查重报告可视化

**功能描述：**
生成可视化的查重报告。

**实现方案：**

```python
import matplotlib.pyplot as plt
import plotly.graph_objects as go
import plotly.express as px
from plotly.subplots import make_subplots

class PlagiarismReportVisualizer:
    def __init__(self):
        self.colors = {
            'low': '#00ff00',
            'medium': '#ffff00', 
            'high': '#ff0000'
        }
    
    def generate_visual_report(self, detection_result: Dict) -> Dict:
        """生成可视化报告"""
        visualizations = {}
        
        # 相似度分布图
        visualizations['similarity_distribution'] = self._create_similarity_distribution(
            detection_result
        )
        
        # 相似代码网络图
        visualizations['similarity_network'] = self._create_similarity_network(
            detection_result
        )
        
        # 代码对比图
        visualizations['code_comparison'] = self._create_code_comparison(
            detection_result
        )
        
        # 特征相似度雷达图
        visualizations['feature_radar'] = self._create_feature_radar(
            detection_result
        )
        
        return visualizations
    
    def _create_similarity_distribution(self, result: Dict) -> go.Figure:
        """创建相似度分布图"""
        if not result.get('similar_codes'):
            return go.Figure()
        
        similarities = [code['overall_similarity'] 
                       for code in result['similar_codes']]
        
        fig = go.Figure()
        
        fig.add_trace(go.Histogram(
            x=similarities,
            nbinsx=20,
            name='相似度分布',
            marker_color='lightblue'
        ))
        
        fig.add_vline(
            x=0.7,  # 阈值线
            line_dash="dash",
            line_color="red",
            annotation_text="检测阈值"
        )
        
        fig.update_layout(
            title="代码相似度分布",
            xaxis_title="相似度分数",
            yaxis_title="频次",
            showlegend=True
        )
        
        return fig
    
    def _create_similarity_network(self, result: Dict) -> go.Figure:
        """创建相似代码网络图"""
        if not result.get('similar_codes'):
            return go.Figure()
        
        # 构建网络节点和边
        nodes = []
        edges = []
        
        # 添加中心节点（当前代码）
        nodes.append({
            'id': 'current',
            'label': '当前代码',
            'group': 0
        })
        
        # 添加相似代码节点
        for i, similar_code in enumerate(result['similar_codes']):
            node_id = f"similar_{i}"
            nodes.append({
                'id': node_id,
                'label': f"相似代码 {i+1}",
                'group': 1,
                'similarity': similar_code['overall_similarity']
            })
            
            # 添加边
            edges.append({
                'from': 'current',
                'to': node_id,
                'weight': similar_code['overall_similarity']
            })
        
        # 创建网络图
        fig = go.Figure()
        
        # 添加节点
        node_x = []
        node_y = []
        node_text = []
        node_color = []
        
        for node in nodes:
            if node['id'] == 'current':
                node_x.append(0)
                node_y.append(0)
                node_color.append('red')
            else:
                angle = (len(nodes) - 1) * 2 * np.pi / (len(nodes) - 1)
                node_x.append(np.cos(angle) * 2)
                node_y.append(np.sin(angle) * 2)
                node_color.append('lightblue')
            
            node_text.append(node['label'])
        
        fig.add_trace(go.Scatter(
            x=node_x,
            y=node_y,
            mode='markers+text',
            marker=dict(
                size=20,
                color=node_color
            ),
            text=node_text,
            textposition="top center",
            name="代码节点"
        ))
        
        # 添加边
        for edge in edges:
            from_idx = next(i for i, node in enumerate(nodes) if node['id'] == edge['from'])
            to_idx = next(i for i, node in enumerate(nodes) if node['id'] == edge['to'])
            
            fig.add_trace(go.Scatter(
                x=[node_x[from_idx], node_x[to_idx]],
                y=[node_y[from_idx], node_y[to_idx]],
                mode='lines',
                line=dict(width=edge['weight'] * 5),
                showlegend=False
            ))
        
        fig.update_layout(
            title="代码相似度网络图",
            xaxis=dict(showgrid=False, zeroline=False, showticklabels=False),
            yaxis=dict(showgrid=False, zeroline=False, showticklabels=False),
            showlegend=True
        )
        
        return fig
    
    def _create_feature_radar(self, result: Dict) -> go.Figure:
        """创建特征相似度雷达图"""
        if not result.get('similar_codes'):
            return go.Figure()
        
        # 获取第一个相似代码的详细相似度
        detailed_sims = result['similar_codes'][0]['detailed_similarities']
        
        categories = list(detailed_sims.keys())
        values = list(detailed_sims.values())
        
        fig = go.Figure()
        
        fig.add_trace(go.Scatterpolar(
            r=values,
            theta=categories,
            fill='toself',
            name='相似度分数'
        ))
        
        fig.update_layout(
            polar=dict(
                radialaxis=dict(
                    visible=True,
                    range=[0, 1]
                )),
            showlegend=True,
            title="各维度相似度分析"
        )
        
        return fig
```

---

### API接口设计

#### POST /plagiarism/detect
```json
{
  "source_code": "def solve(n): return n * 2",
  "language": "python3",
  "problem_id": 1001
}
```

#### POST /plagiarism/batch
```json
{
  "submissions": [
    {
      "id": 1,
      "source_code": "...",
      "language": "python3",
      "problem_id": 1001
    }
  ]
}
```

#### GET /plagiarism/report/{submission_id}
```json
{
  "plagiarism_detected": true,
  "similarity_score": 0.85,
  "similar_codes": [
    {
      "submission_id": 2,
      "similarity_score": 0.85,
      "detailed_analysis": {...}
    }
  ],
  "visualizations": {
    "similarity_distribution": "base64_image",
    "similarity_network": "base64_image"
  }
}
```

---

### 测试建议

| 测试项                    | 期望行为                                |
| ---------------------- | ----------------------------------- |
| 相似代码检测准确性        | 能准确识别相似代码，减少误报            |
| 跨语言查重能力           | 支持不同编程语言的查重检测              |
| 批量处理性能            | 能高效处理大量提交的批量查重            |
| 实时检测响应时间         | 单次检测响应时间小于5秒                |
| 可视化报告质量          | 生成的图表清晰易懂，信息完整            |
| 阈值调整灵活性          | 支持动态调整检测阈值                   |
| 误报过滤效果           | 能有效过滤正常相似，减少误报            |

---

### 模块结构建议

```
plagiarism_detection/
├── feature_extraction/
│   ├── ast_extractor.py
│   ├── semantic_extractor.py
│   └── style_extractor.py
├── similarity/
│   ├── calculator.py
│   ├── algorithms.py
│   └── metrics.py
├── detection/
│   ├── detector.py
│   ├── batch_detector.py
│   └── realtime_detector.py
├── visualization/
│   ├── report_generator.py
│   ├── charts.py
│   └── network_visualizer.py
├── storage/
│   ├── vector_db.py
│   ├── code_index.py
│   └── cache_manager.py
└── api/
    ├── detection_api.py
    ├── batch_api.py
    └── report_api.py
```

---

### 能力小结

| 功能      | 支持说明                          |
| ------- | ----------------------------- |
| 多维度检测 | AST、语义、风格、结构多维度相似度分析      |
| 跨语言支持 | 支持Python、C++、Java等多种语言        |
| 批量处理   | 高效的批量查重检测，支持并发处理          |
| 实时检测   | 快速响应的实时查重检测                |
| 可视化报告 | 丰富的图表和网络图可视化              |
| 智能阈值   | 动态阈值调整，减少误报和漏报           |
| 误报过滤   | 智能过滤正常相似，提高检测准确性         |