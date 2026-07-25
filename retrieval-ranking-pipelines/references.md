# 参考资料

检索日期：2026-07-25

本主题优先使用论文、官方文档和官方工程资料。下面按用途列出，不按数量堆砌。

## 一、推荐系统链路

### 1. Google Developers：Recommendation systems overview

- 链接：[Recommendation systems overview](https://developers.google.com/machine-learning/recommendation/overview/types)
- 类型：官方课程
- 主要依据：Google 将常见推荐架构概括为 Candidate Generation、Scoring、Re-ranking，并解释三者的规模和职责。

### 2. TensorFlow Recommenders：Recommending movies: retrieval

- 链接：[Recommending movies: retrieval](https://www.tensorflow.org/recommenders/examples/basic_retrieval)
- 类型：官方工程文档
- 主要依据：真实推荐系统常拆为 Retrieval 与 Ranking；Retrieval 面向大规模候选池，Ranking 将候选缩为最终短列表。

### 3. Google Research：Deep Neural Networks for YouTube Recommendations

- 链接：[论文页面](https://research.google/pubs/deep-neural-networks-for-youtube-recommendations/)
- PDF：[论文 PDF](https://research.google.com/pubs/archive/45530.pdf)
- 类型：RecSys 2016 论文
- 主要依据：大规模推荐的 Candidate Generation + Ranking 两阶段架构。

### 4. NVIDIA：Best Practices for Building and Deploying Recommender Systems

- 链接：[官方文档](https://docs.nvidia.com/deeplearning/performance/recsys-best-practices/index.html)
- 类型：官方工程文档
- 主要依据：Retrieval、Filtering、Scoring、Ordering 四阶段设计；检索偏效率，排序偏精度，最终 Ordering 处理业务优先级。

### 5. COLD: Towards the Next Generation of Pre-Ranking System

- 链接：[arXiv:2007.16122](https://arxiv.org/abs/2007.16122)
- 类型：2020 arXiv 预印本，工业系统论文
- 主要依据：Matching、Pre-ranking、Ranking 的级联；粗排中的效果—计算成本联合设计。

### 6. A Hybrid Cross-Stage Coordination Pre-ranking Model

- 链接：[arXiv:2502.10284](https://arxiv.org/abs/2502.10284)
- 类型：2025 arXiv 预印本
- 主要依据：Retrieval、Pre-ranking、Ranking、Re-ranking 的跨阶段协调与样本选择偏差。

### 7. Personalized Re-ranking for Improving Diversity in Live Recommender Systems

- 链接：[arXiv:2004.06390](https://arxiv.org/abs/2004.06390)
- 类型：2020 论文
- 主要依据：推荐系统的 Re-ranking 可作为主排序后的列表级组件，用于平衡准确性和多样性。

## 二、RAG、检索与 Rerank

### 8. Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks

- 链接：[arXiv:2005.11401](https://arxiv.org/abs/2005.11401)
- 类型：NeurIPS 2020 论文
- 主要依据：参数化生成模型与可检索非参数记忆的结合，是现代 RAG 的代表性起点。

### 9. Dense Passage Retrieval for Open-Domain Question Answering

- 链接：[arXiv:2004.04906](https://arxiv.org/abs/2004.04906)
- 类型：2020 论文
- 主要依据：双编码器 Dense Retrieval，通过问题和段落向量进行大规模候选检索。

### 10. Sentence Transformers：Retrieve & Re-Rank

- 链接：[Retrieve & Re-Rank](https://www.sbert.net/examples/sentence_transformer/applications/retrieve_rerank/README.html)
- 类型：官方工程文档
- 主要依据：先用词法或 Bi-Encoder 检索较大候选集，再用 Cross-Encoder 对 Query—候选对进行精确重排。

### 11. ColBERT

- 链接：[arXiv:2004.12832](https://arxiv.org/abs/2004.12832)
- 类型：SIGIR 2020 论文
- 主要依据：Late Interaction 在独立编码效率与细粒度 Query—文档交互之间折中。

### 12. Microsoft Azure Architecture Center：Design and develop a RAG solution

- 链接：[RAG solution design and evaluation guide](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-solution-design-and-evaluation-guide)
- 类型：官方架构文档
- 主要依据：RAG 离线数据链路包括 Chunking、元数据增强、Embedding 和索引持久化；标准在线链路包括查询、搜索、上下文组装和生成。

### 13. Microsoft Azure Architecture Center：Information retrieval phase

- 链接：[RAG information-retrieval phase](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-information-retrieval)
- 类型：官方架构文档
- 主要依据：查询改写、分解、混合检索、RRF 融合、Semantic Rerank 以及各步骤的顺序。

### 14. NVIDIA RAG Blueprint：Query-to-Answer Pipeline

- 链接：[Query-to-Answer Pipeline](https://docs.nvidia.com/rag/latest/query-to-answer-pipeline.html)
- 类型：官方工程文档
- 主要依据：Query Rewriter → Retriever → Context Reranker → LLM Generation 的线上链路。

### 15. NVIDIA RAG Blueprint：Agentic RAG

- 链接：[Agentic RAG](https://docs.nvidia.com/rag/latest/agentic-rag.html)
- 类型：官方工程文档
- 主要依据：针对歧义、多跳和跨文档问题进行规划、子任务检索、重试、综合与验证。

### 16. Large Language Models are Effective Text Rankers with Pairwise Ranking Prompting

- 链接：[Google Research 论文页面](https://research.google/pubs/large-language-models-are-effective-text-rankers-with-pairwise-ranking-prompting/)
- 类型：NAACL 2024 论文
- 主要依据：LLM Pointwise、Listwise、Pairwise 排序以及 Pairwise Ranking Prompting。

## 三、指标与基础概念

### 17. Stanford Introduction to Information Retrieval：Precision and Recall

- 链接：[Evaluation of unranked retrieval sets](https://nlp.stanford.edu/IR-book/html/htmledition/evaluation-of-unranked-retrieval-sets-1.html)
- 类型：经典教材在线版
- 主要依据：Precision 是返回对象中的相关比例；Recall 是全部相关对象中被找回的比例。

### 18. Stanford Introduction to Information Retrieval：Ranked retrieval evaluation

- 链接：[Evaluation of ranked retrieval results](https://nlp.stanford.edu/IR-book/html/htmledition/evaluation-of-ranked-retrieval-results-1.html)
- 类型：经典教材在线版
- 主要依据：Top-K、Precision–Recall 与排序结果评价的关系。

## 四、来源使用说明

- 文中的候选规模均标注为示意值，不视为行业固定标准。
- COLD 与 HCCP 的工业收益只用于理解研究动机，不外推为其他业务的预期收益。
- “RAG 的 Reranker 在功能上近似推荐系统精排”是基于系统位置和职责做出的跨领域映射，不是术语标准规定。
- 各公司和团队可能使用不同命名，应以组件的输入、输出、候选规模、目标和延迟预算为准。
