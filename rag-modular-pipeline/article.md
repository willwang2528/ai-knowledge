RAG 不只是向量检索：从离线索引到在线生成的完整工程链路

以前对 RAG 的理解停留在：把文档分块并做 Embedding，回答问题时检索向量相似度最高的内容，再交给模型生成答案即可。

精读两篇经典论文后才发现，完整的 RAG 是 Offline Indexing（离线索引）＋Online Request（在线请求）的双链路。真正落地到工程中，还需要考虑查询分类、检索方式、重排序、上下文组织、评测、延迟和成本等更多问题。

这篇文章从第一性原理出发，先解释为什么需要 RAG，再沿着数据流拆解完整链路和各模块职责。

━━━━━━━━━━

▌01｜从第一性原理出发：为什么需要 RAG？

直接询问模型时，主要依赖 Parametric Memory（参数记忆），即知识被压缩在模型参数中。

参数记忆能够让模型理解问题、组织语言和完成推理，但存在三个明显问题：

第一，知识难以及时更新。想修改模型内部知识，通常需要重新训练或微调。

第二，知识来源难以追溯。我们很难直接判断模型的某个事实来自哪里。

第三，模型不知道答案时，也可能生成一个“听起来合理”的结果。

RAG 为模型增加了 Non-parametric Memory（非参数记忆），即把知识保存在外部文档和索引中，在回答问题时按需检索。

外部知识可以被更新、替换和检查，因此 RAG 的基本逻辑是：

先从非参数记忆中查找证据
→ 再由模型利用参数记忆完成理解和表达

RAG 能减少模型对陈旧、模糊内部知识的依赖，让答案更有事实依据。

但 RAG 只能降低错误，不能消灭错误。检索结果错误、知识库过期，或者模型没有正确使用证据，仍然会产生错误答案。

━━━━━━━━━━

▌02｜完整 RAG 是“两条链＋一组支撑”

RAG 不能被看成一条全部在线执行的链路。

完整系统包括：

① Offline Indexing（离线索引）

Documents → Chunking → Embedding → Vector Database / Index

② Online Request（在线请求）

Query → Query Classification → Retrieval → Reranking → Repacking → Context Compression → LLM Generation

③ Training & Evaluation（训练评估支撑）

Fine-tuning＋Evaluation

【插图 1｜RAG 双链路总览】

文件：/Users/will/Library/Containers/com.tencent.xinWeChat/Data/Documents/xwechat_files/wxid_dmeo22emqypj22_fb0b/temp/RWTemp/2026-07/e2e8a7e174488a136df39931fc653e2b/9a0c069c8296b31c38996e44f9e4362c.png

图片说明：橙色区域是离线索引，蓝色区域是在线请求。Fine-tuning 和 Evaluation 是支撑能力，不应被理解为每次请求都必须顺序执行的在线模块。

━━━━━━━━━━

▌03｜先看方法地图，再理解模块职责

论文《Searching for Best Practices in Retrieval-Augmented Generation》将 RAG 拆成多个模块，并列举了每个模块的代表方法。

这张图更适合被理解为“方法地图”，不是适用于所有场景的统一排行榜。论文中的局部实验结果也不能直接等同于生产环境中的全局最优组合。

【插图 2｜论文中的 RAG 工作流与方法地图】

文件：/Users/will/Library/Containers/com.tencent.xinWeChat/Data/Documents/xwechat_files/wxid_dmeo22emqypj22_fb0b/temp/RWTemp/2026-07/e2e8a7e174488a136df39931fc653e2b/a8ef9f16e52ebbaa20b0050d33a21851.png

从工程角度看，理解模块时要回答三个问题：

输入是什么？
输出变成了什么？
这个模块最重要解决什么问题？

【插图 3｜RAG 模块职责总表】

文件：/Users/will/Library/Containers/com.tencent.xinWeChat/Data/Documents/xwechat_files/wxid_dmeo22emqypj22_fb0b/temp/RWTemp/2026-07/e2e8a7e174488a136df39931fc653e2b/97262a3ee113329f48592b4d7b974c5c.png

━━━━━━━━━━

▌04｜Offline Indexing：把文档变成可搜索的知识源

1. Chunking（切块）

输入：文档
输出：可检索文本块

Chunking 决定系统能以什么粒度找到知识，同时平衡语义完整性与检索精度。

知识块过大，虽然上下文完整，但可能混入更多无关内容，降低检索精度；知识块过小，虽然更容易精确匹配，却可能切碎完整证据。

常见方法包括：

Token-level：按照固定 token 数切分，简单可控，但可能切断句子。

Sentence-level：以完整句子作为边界，兼顾语义完整性和实现成本。

Semantic-level：通过模型识别语义断点，边界更自然，但成本更高。

Small-to-big：使用小块匹配查询，命中后返回包含它的父级大块。

Sliding Window：相邻块保留重叠内容，减少边界信息丢失，但会增加索引冗余。

Chunk Metadata：为知识块附加标题、关键词或假设问题，增加过滤和匹配信号。

【插图 4｜Chunking 方法、优势与代价】

文件：/Users/will/Library/Containers/com.tencent.xinWeChat/Data/Documents/xwechat_files/wxid_dmeo22emqypj22_fb0b/temp/RWTemp/2026-07/e2e8a7e174488a136df39931fc653e2b/321bfcc3a3c077f9833efe18aec5564b.png

工程上不存在通用最佳块大小。更合理的做法是先使用句子边界加适度重叠，再根据真实查询、语料结构和评测结果调整。

2. Embedding（向量化）

输入：文本块或查询
输出：向量

Embedding 让不同措辞但语义相近的查询和文档能够通过相似度进行匹配。

常见模型包括 LLM-Embedder、BAAI/bge 系列、intfloat/e5 系列、Jina-embeddings-v2、GTE 系列和 all-mpnet-base-v2。

模型榜单只能作为初筛依据。真正选型时仍要使用自己的语料和查询评测召回质量、速度、向量维度与运行成本。

3. Vector Database / Index（向量库 / 索引）

输入：向量和元数据
输出：可搜索知识源

它负责保存向量、原文和元数据，让系统能够在大规模语料中低延迟地执行近邻或混合搜索。

常见方案包括 Milvus、Faiss、Weaviate、Qdrant 和 Chroma。实际选型不仅要看功能，还要评估吞吐、P99 延迟、过滤性能、内存成本、扩缩容和运维复杂度。

━━━━━━━━━━

▌05｜Online Request：把用户问题变成有证据的答案

1. Query Classification（查询分类）

输入：用户问题
输出：检索或不检索

它解决的不是“怎么检索”，而是“当前问题是否需要检索”。

翻译、改写、总结用户已经提供的内容，可能不需要访问外部知识；涉及最新信息、专业知识或模型参数之外的事实时，才更需要进入检索链。

Query Classification 可以减少无效延迟，也能避免把不相关文档加入上下文。

2. Retrieval（检索）

输入：查询＋全量语料
输出：Top-k 候选

Retrieval 的目标是从大规模语料中快速缩小候选集合，优先保证不要过早漏掉证据。

常见方法包括：

BM25（关键词检索）：擅长专有名词、数字和稀有词，但不擅长同义表达和语义匹配。

Contriever / LLM-Embedder（向量检索）：能够匹配字面不同但语义相近的文本，但需要运行嵌入模型并维护向量索引。

Query Rewriting（查询重写）：让 LLM 把问题改写成更适合检索的表达，但会增加一次模型调用，也可能导致语义偏移。

Query Decomposition（查询分解）：将复杂问题拆成多个子问题分别检索，更适合组合问题和多跳问题，但会增加检索与结果合并成本。

HyDE（假设文档生成）：先生成假设文档，再用其向量查找真实文档，可以缩小短查询和完整文档之间的表达差距，但也可能引入错误信息。

Hybrid Search（混合检索）：融合关键词检索和向量检索，兼顾精确词项匹配与语义匹配，但需要设计分数归一化与融合策略。

【插图 5｜Retrieval 方法、优势与代价】

文件：/Users/will/Library/Containers/com.tencent.xinWeChat/Data/Documents/xwechat_files/wxid_dmeo22emqypj22_fb0b/temp/RWTemp/2026-07/e2e8a7e174488a136df39931fc653e2b/cd9a833645ad8577d97e333a0d032747.png

3. Reranking（重排序）

输入：Query＋Top-k 候选
输出：更准确的相关性顺序

Retrieval 重覆盖，Reranking 重精度。它使用更精确但更慢的模型重新判断候选与查询的相关性，把真正有用的内容排到前面。

常见方法包括 monoT5、monoBERT、RankLLaMA 和 TILDEv2。

需要注意：Reranking 不能脱离 Query，只根据候选内容重新排序。重排序本身也主要负责改变顺序；如果需要筛除候选，还要增加 Top-n 或相关性阈值。

4. Repacking（文档重编排）

输入：已排序候选
输出：重组后的上下文顺序

Repacking 不再判断相关性，也不会增加新证据。它利用 LLM 对上下文首尾位置更敏感的特点，决定重要证据在提示词中如何摆放。

常见方法包括 Forward、Reverse 和 Sides。

5. Summarization / Context Compression（摘要 / 上下文压缩）

输入：多篇候选
输出：更短的证据上下文

它负责删除冗余和无关内容，降低上下文长度、噪声和生成成本。

常见方法包括 BM25、Contriever、抽取式与生成式 Recomp、LongLLMLingua 和 SelectiveContext。

上下文不长、噪声不高时，这一模块并非必须；错误压缩反而可能删除关键证据。

6. LLM Generation（大模型生成）

输入：Query＋Evidence
输出：Answer

生成模型负责综合用户问题和外部证据作答，而不是只依赖参数记忆。

这一步最终需要同时关注答案正确性和忠实度：

Answer Correctness（答案正确性）：答案是否符合客观事实或标准答案。

Faithfulness（忠实度）：答案中的陈述是否得到当前检索上下文的支持。

两者并不等价。检索文档本身错误时，模型可能忠实地复述错误信息；模型依靠参数记忆答对时，也可能缺少当前上下文依据。

━━━━━━━━━━

▌06｜Training & Evaluation：它们是支撑，不是在线必经链路

1. Fine-tuning（微调）

它让生成器学会从有用证据与噪声混合的上下文中提取答案。

论文比较了 Normal、Random 和 Disturb 等上下文构造方式。其中 Disturb 将相关文档与随机文档混合，更接近真实检索结果中“正确证据与干扰项共存”的状态。

但 Fine-tuning 在工程上不是必选项。如果现成模型已经能够正确利用证据，应先验证是否真的存在需要通过训练解决的问题。

2. Evaluation（评估）

Evaluation 用于判断某项优化是否真正改善了系统，而不是只让某个局部指标变高。

评估至少应该覆盖：

General Performance（通用能力）
Specific Domains（特定领域）
Retrieval Capability（检索能力）
Answer Correctness（答案正确性）
Faithfulness（忠实度）
Latency（延迟）

生产系统还需要记录 Recall@K、nDCG、最终上下文 token 数、P50 / P95 / P99 延迟以及单次请求成本。

━━━━━━━━━━

▌07｜从数据流角度重新理解整条链路

离线链路的数据变化：

Documents（原始文档）
→ Chunking
→ Retrievable Chunks（可检索文本块）
→ Embedding
→ Vectors＋Metadata（向量＋元数据）
→ Vector Database / Index
→ Searchable Knowledge Source（可搜索知识源）

这条链路通常只在文档接入、更新或重建索引时执行，不需要每次用户提问都重新运行。

在线链路的数据变化：

User Query
→ Query Classification

如果不需要检索：

→ LLM Generation
→ Answer

如果需要检索：

→ Retrieval
→ Top-k Candidates
→ Reranking：Query＋Candidates
→ Ranked Candidates
→ Repacking
→ Reorganized Context
→ Context Compression
→ Shorter Evidence Context
→ LLM Generation：Query＋Evidence
→ Answer

与其只记模块名称，不如持续追问：数据经过当前模块后，究竟发生了什么变化？

━━━━━━━━━━

▌08｜这次真正内化的内容

第一，完整 RAG 不是“向量检索＋拼接 Prompt”，而是离线索引和在线请求两条链路。

第二，Query Classification 很重要。不是所有问题都需要检索，未来还应该继续研究更细粒度的自适应检索：是否检索、何时检索、检索多少以及何时停止。

第三，Query Classification 解决“是否检索”；Query Rewriting 和 Query Decomposition 解决“怎样检索”，不能混为同一层。

第四，HyDE 生成的是帮助检索的假设文档，不是可以直接引用的事实证据。

第五，Reranking 必须同时使用 Query 和 Top-k Candidates；Repacking 则是在候选已经确定后改变上下文摆放顺序。

第六，Summarization 和 Fine-tuning 在工程上都不是必选项。是否使用，要由真实瓶颈和评测结果决定。

━━━━━━━━━━

▌09｜后续学习路线

这篇文章只是先把 RAG 的完整链路串联起来，建立对各模块位置和职责的整体认识。每个模块内部仍有很多工程细节需要继续研究，例如：

不同数据形态应该如何解析、切块和建立索引；

如何实现更细粒度的自适应检索；

多跳问题如何迭代检索并合并证据；

如何端到端评估答案质量、检索质量、延迟和成本；

如何处理知识更新、权限隔离、错误回退和全链路监控。

RAG 也正在从早期的“单轮检索—生成”两段式结构，向混合检索、GraphRAG、Agentic RAG 等更加多元的形态演进。

理解完整链路，是继续研究这些进阶方向的起点。

━━━━━━━━━━

参考文献

［1］WANG X, WANG Z, GAO X, et al. Searching for Best Practices in Retrieval-Augmented Generation［C］. Proceedings of the 2024 Conference on Empirical Methods in Natural Language Processing. Association for Computational Linguistics, 2024: 17716–17736.

［2］LEWIS P, PEREZ E, PIKTUS A, et al. Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks［C］. Advances in Neural Information Processing Systems, 2020, 33: 9459–9474.
