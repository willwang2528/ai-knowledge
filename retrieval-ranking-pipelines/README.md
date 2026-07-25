# 检索与排序多阶段链路

## 一句话定义

推荐系统和 RAG 都在用多阶段漏斗解决同一个问题：先从巨大数据池中低成本找到候选，再把有限计算集中到少量候选上做精确判断和最终组织。

## 为什么出现

全量对象可能有百万到十亿级，但用户最终只看到少量内容，LLM 也只能接收少量证据。

如果直接对全量对象运行最昂贵的模型，延迟和成本不可接受；如果全程只用便宜模型，最终质量又不够。因此需要逐层缩小候选集。

```text
越靠前：
候选多、计算便宜、重点是不漏

越靠后：
候选少、计算更贵、重点是判准和整体可用
```

## 核心问题

这组名词不应该靠翻译记忆，而应该判断：

1. 该阶段的输入和输出是什么？
2. 候选规模缩小了多少？
3. 它优先保证覆盖、精度，还是列表整体约束？
4. 每个候选允许花多少计算成本？

## 推荐系统核心链路

```text
全量内容池
    ↓
多路召回 / Matching / Candidate Generation / Retrieval
    ↓
合并、去重、硬过滤
    ↓
粗排 / Pre-ranking
    ↓
精排 / Ranking
    ↓
重排 / Re-ranking / Post-ranking
    ↓
最终展示列表
```

| 阶段 | 最重点解决什么 |
|---|---|
| 召回 | 确定候选集合：宁可多捞一些，也不要过早漏掉优质内容，同时能在大规模数据上运行 |
| 合并、过滤 | 去掉重复、无效、无库存、无权限或不能展示的候选 |
| 粗排 | 用有限成本进一步缩小候选，为昂贵精排减负 |
| 精排 | 用丰富特征和复杂模型，准确判断每个候选对当前用户的价值 |
| 重排 | 从整个列表出发处理多样性、新鲜度、公平性和业务约束 |

一句话记忆：

```text
召回保“不漏”
粗排保“算得动”
精排保“判得准”
重排保“整页好”
```

## RAG 核心链路

RAG 需要区分离线建库和在线问答。

### 离线建库

```text
原始文档
    ↓
解析、清洗、切分 Chunk
    ↓
补充元数据和权限
    ↓
生成 Embedding
    ↓
建立关键词索引 / 向量索引
```

### 在线问答

```text
用户问题
    ↓
查询理解 / 改写 / 拆分
    ↓
初次检索：BM25、向量、混合或多数据源
    ↓
权限过滤、去重、结果融合
    ↓
Rerank：精确判断 Query—Chunk 相关性
    ↓
Context Packing：选择并组织最终证据
    ↓
LLM 生成与引用
```

| 阶段 | 最重点解决什么 |
|---|---|
| 查询改写 | 把用户语言变成更适合搜索的查询 |
| Retrieval | 从大规模 Chunk 中取得高覆盖候选，让答案证据进入后续阶段 |
| 融合、过滤 | 合并多路结果，并移除无权限、重复或无效内容 |
| Rerank | 用更强模型纠正初检误排，让真正能回答问题的 Chunk 靠前 |
| Context Packing | 在 Token 预算内选择互补、低重复、可引用的证据 |
| 生成 | 基于证据组织最终答案 |

一句话记忆：

```text
改写让问题“更好搜”
检索让证据“进入候选”
Rerank 让证据“排得更准”
Packing 让上下文“够用且不浪费”
生成让证据“成为答案”
```

## 关键结论

### 1. Recall 不严格等于 Retrieval

```text
中文推荐系统里的“召回阶段”
≈ Retrieval
≈ Candidate Generation

英文 Recall
= 召回率指标
```

`Retrieval` 做检索动作，`Recall@K` 衡量相关对象有没有被漏掉。中文“召回”同时被用作阶段名和指标翻译，才造成歧义。

### 2. RAG Reranker 更接近推荐系统精排

| 抽象职责 | 推荐系统 | RAG |
|---|---|---|
| 大规模找候选 | 召回 / Matching | Retrieval |
| 高精度判断 | 精排 / Ranking | Reranker |
| 整理最终集合 | 重排 / Post-ranking | Context Packing |

这里的对应表示工程职责相近，不表示算法和产品目标完全相同。

### 3. 粗排和精排是成本层级，不是固定算法名

候选少、精排便宜时，可以没有独立粗排；候选多、精排昂贵时，可能有一层甚至多层粗排。

### 4. 下游无法找回上游漏掉的对象

推荐系统中，精排无法选择召回阶段没提供的 Item；RAG 中，Reranker 无法重排 Retriever 没有取回的证据。

## 我的理解

可以把整个系统看成招聘：

- 召回/检索：从百万份简历中快速找出可能合适的人。
- 硬过滤：去掉不满足确定条件的人。
- 粗排：用有限信息缩小面试名单。
- 精排/RAG Rerank：投入更多成本做准确判断。
- 推荐重排/RAG Context Packing：组建最终名单时考虑整体互补和约束。

真正要记住的不是五个术语，而是三个动作：

```text
找候选
  ↓
判优先级
  ↓
组织最终结果
```

## 后续可以研究的点

以下内容留作后续独立研究，本次不细究：

- **多路召回与融合策略**：理解不同召回通道怎样分工、合并和分配配额。
- **BM25、Dense Retrieval 与 Hybrid Search**：理解词法检索和语义检索各自的优势与盲区。
- **双塔、Cross-Encoder 与 ColBERT**：理解检索效率与交互精度之间的模型选择。
- **Pointwise、Pairwise、Listwise Ranking**：理解排序模型的三类训练目标。
- **RRF、MMR、DPP**：区分结果融合、相关性—多样性折中和列表多样性建模。
- **分阶段评估**：分别定位召回、Rerank、Context 和生成阶段的质量瓶颈。
- **Agentic Retrieval**：理解查询规划、多跳检索、重试和验证。

## 文件导航

- [engineering.md](engineering.md)：两条链路、核心模块和必要概念
- [research.md](research.md)：技术演进和代表论文
- [article.md](article.md)：公众号文章
- [image_prompt.md](image_prompt.md)：三张视觉知识图 Prompt

## 参考资料

只保留支撑本主题关键结论的论文和官方资料：

1. [Google：Recommendation systems overview](https://developers.google.com/machine-learning/recommendation/overview/types) — Candidate Generation、Scoring、Re-ranking 三阶段职责。
2. [Deep Neural Networks for YouTube Recommendations](https://research.google/pubs/deep-neural-networks-for-youtube-recommendations/) — Candidate Generation + Ranking 两阶段工业架构。
3. [NVIDIA：Best Practices for Building and Deploying Recommender Systems](https://docs.nvidia.com/deeplearning/performance/recsys-best-practices/index.html) — Retrieval、Filtering、Scoring、Ordering。
4. [COLD: Towards the Next Generation of Pre-Ranking System](https://arxiv.org/abs/2007.16122) — 粗排中的效果与计算成本平衡。
5. [Dense Passage Retrieval](https://arxiv.org/abs/2004.04906) — 双编码器稠密检索。
6. [Retrieval-Augmented Generation](https://arxiv.org/abs/2005.11401) — Retriever 与 Generator 结合的 RAG 范式。
7. [Sentence Transformers：Retrieve & Re-Rank](https://www.sbert.net/examples/sentence_transformer/applications/retrieve_rerank/README.html) — Bi-Encoder 初检与 Cross-Encoder 重排。
8. [Microsoft：RAG Information Retrieval Phase](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-information-retrieval) — 查询改写、混合检索、融合和 Rerank。
9. [Stanford IR Book：Precision and Recall](https://nlp.stanford.edu/IR-book/html/htmledition/evaluation-of-unranked-retrieval-sets-1.html) — Recall 和 Precision 的严格定义。
