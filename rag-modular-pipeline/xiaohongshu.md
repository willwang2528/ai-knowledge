RAG 不只是向量检索 🧠🔍

以前我对 RAG 的理解是：文档分块、做 Embedding，回答时检索向量相似度最高的内容，再交给模型即可。

精读两篇经典论文后才发现，完整 RAG 是“两条链＋一组支撑”：

📦 Offline Indexing
Documents → Chunking → Embedding → Vector Database / Index

🔍 Online Request
Query → Query Classification → Retrieval → Reranking → Repacking → Context Compression → LLM Generation

🛠 Training & Evaluation
Fine-tuning＋Evaluation

为什么 RAG 能减少事实性错误？

直接问模型主要依赖 Parametric Memory，知识被压缩在模型参数中，难更新、难追溯。RAG 增加了可检索、可更新的 Non-parametric Memory：先查找外部证据，再由模型完成理解和表达。

这次内化了几点：

① 不是所有问题都要检索，Query Classification 可以减少无效延迟和错误上下文。

② Retrieval 负责高召回地找出 Top-k 候选；Reranking 必须结合 Query＋Candidates 做更精确的排序。

③ Repacking 不再判断相关性，而是调整证据在上下文中的位置。

④ Summarization 和 Fine-tuning 都不是必选项，要根据上下文长度、噪声和实际评测决定。

⑤ RAG 只能降低错误，不能消灭错误。检索错误、知识过期或模型没有正确使用证据，仍会影响答案。

这篇先建立完整链路，后续再研究切块选型、自适应检索、多跳检索、评测与生产工程。

#RAG #大模型 #AI工程 #AgentMemory #向量检索 #知识库
