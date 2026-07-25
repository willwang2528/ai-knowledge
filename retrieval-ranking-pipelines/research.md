# 科研视角：多阶段检索与排序如何演进

## 1. 最初的问题

推荐、搜索和 RAG 虽然面向不同产品，研究起点都可以抽象为：

> 如何从巨大对象集合中，在有限延迟和计算预算内，找出少量最相关、最有价值的结果？

如果只用便宜模型扫描全量数据，速度可以接受，但判断不够细；如果对全量数据使用昂贵模型，效果可能更好，却无法上线。

于是形成级联架构：

```text
高覆盖、低成本的候选生成
             ↓
较小集合上的高精度排序
             ↓
面向最终列表或任务的整体优化
```

这不是单一算法的发明，而是效果、延迟、吞吐和成本共同塑造的系统范式。

---

# 2. 推荐系统技术路线

## 2.1 从单一推荐模型到候选生成 + 排序

早期推荐研究常把重点放在协同过滤、矩阵分解或单一预测模型上。大规模在线服务进一步暴露出“全量 Item 无法逐个精算”的问题。

2016 年 YouTube 推荐系统论文清晰描述了两阶段架构：

```text
Candidate Generation
    ↓
Ranking
```

候选生成从巨大视频库中检索数百个候选，Ranking 再用更丰富特征给候选打分。这一工作帮助固定了工业界理解多阶段推荐系统的基本坐标。

## 2.2 从两阶段到多阶段级联

随着内容池、特征数量和排序模型复杂度增长，两阶段之间出现新的计算缺口：

```text
Matching / Retrieval
    ↓
Pre-ranking
    ↓
Ranking
    ↓
Re-ranking
```

Pre-ranking 不再只是“随便做个轻量精排”，而成为独立研究问题：

- 如何保留精排真正看重的候选？
- 如何在严格成本下使用更强交互？
- 如何减少上游候选分布造成的样本选择偏差？
- 如何协调召回、粗排、精排之间的目标？

## 2.3 从逐项相关性到列表级目标

精排通常先估计单个 Item 的点击、转化或时长价值。但最终展示的是一个列表，Item 之间会互相影响：

- 多个相似 Item 可能造成疲劳。
- 热门内容可能挤压长尾供给。
- 新鲜度、公平性和探索无法仅靠单项点击率表达。

因此重排研究逐步引入多样性、公平性、供给生态和长期价值等列表级目标。

## 2.4 当前方向

- 跨阶段联合训练与蒸馏。
- 粗排对召回和精排信号的双向协调。
- 多任务价值建模与长期用户价值。
- 列表级生成式推荐与序列决策。
- 效果、延迟、显存、特征访问成本的联合优化。
- 降低各阶段训练数据由上游筛选带来的样本选择偏差。

---

# 3. 搜索与 RAG 技术路线

## 3.1 从词法检索到学习型稠密检索

经典信息检索依靠倒排索引和词法匹配，例如 BM25。它们对精确关键词、实体和编号很有效，但在同义表达和语义改写上存在限制。

Dense Passage Retrieval（DPR）使用双编码器把问题和段落映射到稠密向量空间，使语义检索可以通过向量近邻搜索扩展到大规模语料。

核心结构：

```text
Query Encoder → Query Vector
Passage Encoder → Passage Vector
             ↓
       相似度与 Top-K
```

## 3.2 RAG 把检索结果变成生成模型的外部记忆

2020 年 RAG 工作将参数化生成模型与非参数化文档索引结合：

```text
问题
  ↓
Retriever 找文档
  ↓
Generator 基于问题与文档生成
```

这使检索不再只服务搜索结果页，而成为生成模型获取可更新知识和证据来源的关键接口。

## 3.3 Retrieve & Re-Rank

双编码器速度快，但 Query 与文档在编码时彼此不可见，细粒度相关性判断有限。

Cross-Encoder 把 Query 和候选文档一起输入模型，允许充分交互，通常更准确，但无法对全量文档低成本运行。

因此形成标准两阶段结构：

```text
BM25 / Bi-Encoder 检索 Top-K
              ↓
Cross-Encoder 重排 Top-K
```

这与推荐系统“召回 + 精排”的工程逻辑同源，只是搜索领域通常把第二阶段称为 Rerank。

## 3.4 Late Interaction

ColBERT 试图缩小两类模型之间的鸿沟：

- 像双编码器一样预计算文档表示。
- 保留 Token 级多向量。
- 在查询时做较轻量的后期交互。

它说明“检索”和“重排”并非只有一条固定边界，而是一条效果—效率连续谱。

## 3.5 从单路检索到混合与 Agentic Retrieval

生产 RAG 通常组合：

- BM25 词法检索
- Dense 向量检索
- 元数据和权限过滤
- 多知识源路由
- 结果融合
- Rerank

进一步的 Agentic Retrieval 会根据问题动态决定：

- 是否需要检索
- 是否拆成多个子问题
- 查询哪个数据源
- 结果不足时是否改写并重试
- 是否需要多跳检索或答案验证

这里的进步不只是“换一个更强 Retriever”，而是把检索从固定调用升级为可规划、可观察、可重试的过程。

---

# 4. 代表论文

## 4.1 Deep Neural Networks for YouTube Recommendations

**论文：** Deep Neural Networks for YouTube Recommendations  
**年份：** 2016  
**作者：** Paul Covington, Jay Adams, Emre Sargin  
**场合：** ACM RecSys 2016

**解决问题：** 如何在超大规模视频库上运行深度推荐模型。

**核心思想：** 把系统拆成 Candidate Generation 和 Ranking 两个神经网络阶段。

**贡献：**

- 清楚展示大规模工业推荐的两阶段架构。
- 候选生成负责大规模缩减，Ranking 使用更丰富特征做精细排序。
- 将模型效果与线上服务约束放在同一个系统中讨论。

**局限：**

- 主要描述两阶段，不覆盖后来工业系统中常见的独立粗排和复杂列表级重排。
- 具体架构来自当时的 YouTube 场景，不能直接视为所有推荐系统标准。

## 4.2 COLD: Towards the Next Generation of Pre-Ranking System

**论文：** COLD: Towards the Next Generation of Pre-Ranking System  
**年份：** 2020  
**作者：** Zhe Wang 等  
**状态：** arXiv 预印本，介绍阿里巴巴广告系统实践

**解决问题：** 粗排既要接近精排效果，又要受到严格线上计算成本约束。

**核心思想：** 从算法—系统协同设计角度，把模型能力和实际计算成本一起优化。

**贡献：**

- 把 Pre-ranking 明确为独立系统问题。
- 强调粗排不是简单缩小版精排。
- 展示在线学习、推理优化和快速实验基础设施的重要性。

**局限：**

- 结果高度依赖特定工业广告系统。
- 线上收益和基础设施细节难以在公开数据上完全复现。

## 4.3 A Hybrid Cross-Stage Coordination Pre-ranking Model

**论文：** A Hybrid Cross-Stage Coordination Pre-ranking Model for Online Recommendation Systems  
**年份：** 2025  
**作者：** Binglei Zhao 等  
**状态：** arXiv 预印本

**解决问题：** 粗排只模仿下游精排会受到样本选择偏差影响，并忽略上游召回信息。

**核心思想：** 同时利用上游检索和下游排序信号，协调粗排在整条链路中的桥梁作用。

**贡献：**

- 将 Pre-ranking 从局部模型问题提升为跨阶段协调问题。
- 显式关注由级联筛选产生的样本选择偏差。

**局限：**

- 公开材料主要是论文和伪代码。
- 需要更多独立数据集和系统复现来判断泛化性。

## 4.4 Dense Passage Retrieval for Open-Domain Question Answering

**论文：** Dense Passage Retrieval for Open-Domain Question Answering  
**年份：** 2020  
**作者：** Vladimir Karpukhin 等

**解决问题：** 传统词法检索难以覆盖问题与答案段落之间的语义表达差异。

**核心思想：** 用两个编码器分别表示问题和段落，通过稠密向量相似度检索。

**贡献：**

- 推动 Dense Retrieval 成为开放域问答的重要范式。
- 证明双编码器可以兼顾语义匹配和大规模索引。

**局限：**

- 单向量独立编码限制了 Query—段落的细粒度交互。
- 对训练负样本、领域迁移和索引配置敏感。

## 4.5 Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks

**论文：** Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks  
**年份：** 2020  
**作者：** Patrick Lewis 等  
**场合：** NeurIPS 2020

**解决问题：** 纯参数模型难以可靠更新知识、给出来源，并在知识密集任务中精确使用事实。

**核心思想：** 将生成模型的参数记忆与可检索的非参数文档索引结合。

**贡献：**

- 奠定现代 RAG 的代表性范式。
- 将 Retriever 与 Generator 置于统一任务框架中。
- 强调外部知识的可更新性和来源价值。

**局限：**

- 原始系统相对今天的生产 RAG 简化，没有完整覆盖权限、混合检索、Rerank、Context Packing 和安全治理。
- 最终质量受到检索候选上限约束。

## 4.6 ColBERT

**论文：** ColBERT: Efficient and Effective Passage Search via Contextualized Late Interaction over BERT  
**年份：** 2020  
**作者：** Omar Khattab, Matei Zaharia  
**场合：** SIGIR 2020

**解决问题：** Cross-Encoder 相关性强但太慢，双编码器可扩展但交互较弱。

**核心思想：** Query 和文档分别编码为 Token 级表示，在后期执行可扩展的细粒度交互。

**贡献：**

- 提出 Late Interaction 这一重要折中路线。
- 说明更丰富交互可以进入端到端检索，而不只存在于第二阶段 Rerank。

**局限：**

- 多向量索引的存储和服务复杂度高于单向量检索。
- 工程效率仍依赖压缩、剪枝和专用检索实现。

## 4.7 Large Language Models are Effective Text Rankers with Pairwise Ranking Prompting

**论文：** Large Language Models are Effective Text Rankers with Pairwise Ranking Prompting  
**年份：** 2024  
**作者：** Zhen Qin 等  
**场合：** NAACL 2024

**解决问题：** 如何让通用大模型更有效地执行文本排序。

**核心思想：** 用 Pairwise Ranking Prompting 降低模型直接理解复杂排序任务的难度。

**贡献：**

- 展示 LLM 作为文本 Ranker 的能力。
- 比较 Pointwise、Listwise 和 Pairwise 提示策略。

**局限：**

- 大模型 Rerank 的成本、延迟和稳定性仍限制大候选集应用。
- 排序质量依赖模型、提示、候选构成和领域。

---

# 5. 最重要的科研结论

## 5.1 多阶段不是历史包袱，而是计算约束的自然结果

只要同时存在“全量对象巨大”和“高精度模型昂贵”，漏斗结构就会出现。

## 5.2 每个阶段都改变下游的数据分布

召回决定粗排能看见什么，粗排决定精排的训练与推理分布。上游误杀会成为不可恢复的上限，也会引入样本选择偏差。

## 5.3 名词边界会移动

更高效的交互模型可能把过去只能用于 Rerank 的能力前移到 Retrieval；更强的生成或 Agent 系统也可能把多次检索组合成动态流程。

因此术语不是固定算法类别，而是系统在当前成本条件下划出的职责边界。

## 5.4 最终目标正从“单个候选相关”扩展为“系统整体有效”

- 推荐系统关注列表体验、生态、公平性和长期价值。
- RAG 关注证据组合、可引用性、冲突处理、验证和完整回答。

这使后排序、Context Packing 和 Agentic Retrieval 成为越来越重要的研究对象。
