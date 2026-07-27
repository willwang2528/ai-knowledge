【标题】

RAG 不只是“向量检索”：一篇文章看懂离线索引与在线请求

【公众号摘要】

RAG 的真正难点不是接一个向量库，而是让文档离线变成可搜索知识，再让一次在线请求依次完成判断、召回、排序、组织和生成。本文用两条链路讲清每个模块的位置、输入输出与工程取舍。

【正文图片】

共 5 张，按照正文中的“正文图 1—5”依次插入。图片必须通过公众号编辑器上传；下列本地路径仅供上传定位，不进入最终正文。

【正文】

以前，我对 RAG 的理解停留在：把文档分块并做 Embedding，回答问题时检索向量相似度最高的内容，再交给模型生成答案。

这个理解没有错，但只覆盖了最基础的一小段。

精读《Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks》和《Searching for Best Practices in Retrieval-Augmented Generation》后，我发现完整的 RAG 是 Offline Indexing（离线索引）与 Online Request（在线请求）的双链路。真正落地时，还要处理查询分类、重排序、上下文组织、评测、延迟和成本。

本文只解决一个问题：

一次 RAG 回答，前后究竟经历了什么？

一、为什么需要 RAG？

直接询问模型时，主要依赖 Parametric Memory（参数记忆）：知识被压缩在模型参数中。

参数记忆让模型能够理解问题、组织语言和完成推理，但有三个明显限制：知识难以及时更新，事实来源难以追溯，模型不知道答案时也可能生成一个“听起来合理”的结果。

RAG 为模型增加了 Non-parametric Memory（非参数记忆），也就是保存在外部文档和索引中的知识。它可以更新、替换、检索和检查。

因此，RAG 的基本逻辑是：

先从非参数记忆中查找证据
→ 再由模型利用参数记忆完成理解和表达

这能减少模型对陈旧或模糊内部知识的依赖，让答案更有事实依据。

但 RAG 只能降低错误，不能消灭错误。如果检索结果错误、知识库已经过期，或者模型没有正确使用证据，最终答案仍然可能出错。

二、完整 RAG：两条链路，一组支撑

RAG 不能被看成一条全部在线执行的链路。

第一条是 Offline Indexing（离线索引）：

Documents
→ Chunking
→ Embedding
→ Vector Database / Index

它负责提前把原始文档加工成可搜索的知识源，只在文档接入、更新或重建索引时运行。

第二条是 Online Request（在线请求）：

Query
→ Query Classification
→ Retrieval
→ Reranking
→ Repacking
→ Context Compression
→ LLM Generation

它从用户问题出发，逐步把大规模语料缩小成少量证据，再交给模型生成答案。

Fine-tuning 和 Evaluation 属于全链路支撑能力，不是每次在线请求都必须顺序执行的两个步骤。

〔正文图 1｜RAG 双链路总览〕

上传文件：/Users/will/Library/Containers/com.tencent.xinWeChat/Data/Documents/xwechat_files/wxid_dmeo22emqypj22_fb0b/temp/RWTemp/2026-07/e2e8a7e174488a136df39931fc653e2b/9a0c069c8296b31c38996e44f9e4362c.png

图注：橙色区域是离线索引，蓝色区域是在线请求；微调与评估是支撑能力。

论文进一步列出了每个模块的代表方法。下面这张图应该被理解为“方法地图”，而不是所有场景通用的统一排行榜。局部实验中的最好方法，也不一定能组成全局最优系统。

〔正文图 2｜论文中的 RAG 工作流与方法地图〕

上传文件：/Users/will/Library/Containers/com.tencent.xinWeChat/Data/Documents/xwechat_files/wxid_dmeo22emqypj22_fb0b/temp/RWTemp/2026-07/e2e8a7e174488a136df39931fc653e2b/a8ef9f16e52ebbaa20b0050d33a21851.png

图注：论文比较的模块与代表方法。图中的下划线和颜色只表示论文设定下的选择，不能直接外推到所有工程场景。

理解这些模块时，不要只背名称，而要持续追问三个问题：

输入是什么？
输出变成了什么？
它最重要解决什么问题？

〔正文图 3｜RAG 模块职责总表〕

上传文件：/Users/will/Library/Containers/com.tencent.xinWeChat/Data/Documents/xwechat_files/wxid_dmeo22emqypj22_fb0b/temp/RWTemp/2026-07/e2e8a7e174488a136df39931fc653e2b/97262a3ee113329f48592b4d7b974c5c.png

图注：从输入、输出和核心问题三个角度理解各模块。

三、离线索引：把文档变成可搜索知识

离线链路的数据变化是：

原始文档
→ 可检索文本块
→ 向量与元数据
→ 可搜索知识源

其中最先影响系统效果的是 Chunking（切块）。

块太大，虽然上下文完整，却容易混入无关信息；块太小，虽然更容易精确匹配，却可能切碎完整证据。因此，Chunking 要在语义完整性和检索精度之间寻找平衡。

常见方法包括固定长度、句子级、语义级、Small-to-big、Sliding Window 和 Chunk Metadata。它们没有脱离语料与查询的通用优胜者。

〔正文图 4｜Chunking 方法、优势与代价〕

上传文件：/Users/will/Library/Containers/com.tencent.xinWeChat/Data/Documents/xwechat_files/wxid_dmeo22emqypj22_fb0b/temp/RWTemp/2026-07/e2e8a7e174488a136df39931fc653e2b/321bfcc3a3c077f9833efe18aec5564b.png

图注：不同切块方法解决的问题不同，也会引入不同成本。

更稳妥的工程起点，是使用句子边界加适度重叠，再根据真实查询和评测结果调整，而不是照搬某个固定块大小。

Embedding（向量化）把文本块与查询映射到同一个语义空间，让措辞不同但含义相近的内容能够匹配。模型榜单只能用于初筛，最终仍要用自己的语料评估召回质量、速度、向量维度与运行成本。

Vector Database / Index（向量库 / 索引）负责保存向量、原文和元数据，并在大规模语料中执行近邻或混合搜索。选型不能只看功能清单，还要考虑吞吐、P99 延迟、过滤性能、扩缩容和运维成本。

四、在线请求：把问题变成有证据的答案

在线链路的第一步不是检索，而是 Query Classification（查询分类）。

它要回答的是：这个问题是否需要外部知识？

翻译、改写或总结用户已经提供的内容，可能不需要检索；涉及最新信息、专业知识或模型参数之外的事实时，才更需要进入检索链。这个判断可以减少无效延迟，也能避免无关文档污染上下文。

确定需要检索后，Retrieval（检索）从全量语料中取得 Top-k 候选。它的优先目标是高召回：先快速缩小候选集合，尽量不要过早漏掉证据。

BM25 擅长专有名词、数字和稀有词；向量检索擅长字面不同但语义接近的表达。Query Rewriting、Query Decomposition、HyDE 和 Hybrid Search 则从不同角度改善查询表达或融合多种检索信号。

〔正文图 5｜Retrieval 方法、优势与代价〕

上传文件：/Users/will/Library/Containers/com.tencent.xinWeChat/Data/Documents/xwechat_files/wxid_dmeo22emqypj22_fb0b/temp/RWTemp/2026-07/e2e8a7e174488a136df39931fc653e2b/cd9a833645ad8577d97e333a0d032747.png

图注：检索方法需要在召回范围、语义能力、调用成本和延迟之间取舍。

Retrieval 得到候选之后，Reranking（重排序）使用 Query＋Top-k Candidates 做更精确的相关性判断，把真正有用的证据排到前面。

这也意味着 Reranking 不能脱离 Query，只根据候选内容重新排序。重排序本身主要改变顺序；如果需要删除候选，还要增加 Top-n 或相关性阈值。

Repacking（文档重编排）与 Reranking 不同。候选已经确定后，它不再判断相关性，而是调整证据在上下文中的位置，缓解模型对中间内容利用不足的问题。

Summarization / Context Compression（摘要 / 上下文压缩）负责删除冗余和噪声，降低上下文长度和生成成本。但它并非默认必开：上下文不长、噪声不高时，错误压缩反而可能删掉关键证据。

最后，LLM Generation 使用 Query＋Evidence 生成答案。这里需要同时检查两个指标：

Answer Correctness（答案正确性）：答案是否符合客观事实或标准答案。

Faithfulness（忠实度）：答案中的陈述是否得到当前检索上下文支持。

两者并不等价。检索文档本身错误时，模型可能忠实地复述错误信息；模型依靠参数记忆答对时，也可能缺少当前上下文依据。

五、真正需要内化的，不是某个“最佳模型”

这次学习后，我真正保留下来的不是一张模型排行榜，而是以下几个工程判断。

第一，完整 RAG 不是“向量检索＋拼接 Prompt”，而是离线索引和在线请求两条链路。

第二，Query Classification 解决“是否检索”；Query Rewriting 和 Query Decomposition 解决“怎样检索”，不能混为同一层。

第三，Retrieval 负责高召回地取得候选；Reranking 使用 Query＋Candidates 提高排序精度；Repacking 则改变证据的上下文位置。

第四，HyDE 生成的是帮助检索的假设文档，不是可以直接引用的事实证据。

第五，Summarization 和 Fine-tuning 都不是工程上的必选项。是否启用，应该由真实瓶颈和端到端评测决定。

Fine-tuning 可以让生成器学习从正确证据与噪声混合的上下文中提取答案。Evaluation 则必须同时覆盖通用能力、特定领域、检索能力、答案正确性、忠实度和延迟，避免只优化某个局部指标。

后续仍值得继续研究：不同数据形态如何切块与索引，自适应检索如何判断是否检索、检索多少和何时停止，多跳问题如何迭代检索并合并证据，以及生产系统如何处理知识更新、权限隔离、错误回退和全链路监控。

RAG 正在从早期的“单轮检索—生成”，走向混合检索、GraphRAG 和 Agentic RAG 等更加多元的形态。

但无论架构如何变化，理解完整链路始终是排查问题、选择模块和评估系统的起点。

参考文献

［1］WANG X, WANG Z, GAO X, et al. Searching for Best Practices in Retrieval-Augmented Generation［C］. Proceedings of the 2024 Conference on Empirical Methods in Natural Language Processing. Association for Computational Linguistics, 2024: 17716–17736.

［2］LEWIS P, PEREZ E, PIKTUS A, et al. Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks［C］. Advances in Neural Information Processing Systems, 2020, 33: 9459–9474.
