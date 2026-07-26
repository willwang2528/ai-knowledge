# RAG 模块链路：方法、职责与实验选型

## 一句话定义

模块化 RAG 是一条把外部文档加工成可检索知识，并在请求时依次完成“判断是否检索、检索候选、精确排序、组织上下文、压缩上下文、生成答案”的系统链路。

## 为什么出现

```text
过去：
只讨论“要不要给 LLM 接检索”，或只优化某一个检索器。

问题：
真实 RAG 由多个模块组成；任一模块的错误、延迟或噪声都会传给后续模块。

导致：
单点方法得分高，不代表整条 RAG 链路效果好；逐模块最优也不等于全局最优。

所以提出：
按模块拆解 RAG，并在统一的端到端链路中比较模块选型。
```

## 核心问题

RAG 的核心不是“把很多文档塞给 LLM”，而是：

> 在可接受的延迟和成本下，把真正有用的少量证据送到生成器最容易利用的位置。

## 核心链路

这张图必须拆成三条逻辑链，不能把所有方框都理解为一次在线请求的连续步骤。

### 1. 离线检索源

```text
原始文档
↓
Chunking（切块）：决定知识的最小检索单元
↓
Embedding（向量化）：把文本映射到可比较的语义空间
↓
Vector Database / Index（向量库 / 索引）：保存并快速搜索知识块
```

### 2. 在线主链

```text
用户问题
↓
Query Classification（查询分类）：判断是否需要外部知识
↓
Retrieval（检索）：从大语料中取得高召回候选
↓
Reranking（重排序）：更精确地判断候选与问题的相关性
↓
Repacking（文档重编排）：调整文档在提示词中的位置
↓
Summarization / Context Compression（摘要 / 上下文压缩）：去掉冗余和噪声
↓
LLM Generation（生成）：基于问题和证据作答
```

### 3. 训练与评估支撑

```text
Fine-tuning（微调）
→ 让生成器学会在“有相关证据，也有噪声”的上下文中作答

Evaluation（评估）
→ 同时检查任务效果、领域效果、RAG 专项能力和延迟
```

## 最容易混淆的四个词

| 术语 | 中文 | 最重要解决什么 |
| --- | --- | --- |
| `Retrieval` | 检索，也可理解为 RAG 的候选召回 | 从百万级语料快速缩小到几十个可能相关的候选，重点是不要过早漏掉证据。 |
| `Reranking` | 重排序 | 用更精确但更慢的模型重新判断候选相关性，把最有用证据排到前面。 |
| `Repacking` | 文档重编排 | 不再判断相关性，而是改变已选文档在 LLM 上下文中的摆放位置。 |
| `Summarization` | 摘要 / 上下文压缩 | 在证据已选定后删除冗余，降低上下文长度、噪声和生成成本。 |

因此：

```text
Retrieval ≠ Reranking ≠ Repacking

检索：选哪些候选
重排序：候选谁更相关
重编排：候选按什么位置交给 LLM
```

## 论文实验选出了什么

| 位置 | 论文中的关键选择 | 一句话理解 |
| --- | --- | --- |
| 切块 | 句子级切块作为默认；局部实验中滑动窗口最好 | 保留语义边界，同时通过重叠减少边界信息丢失，但会增加索引冗余。 |
| 嵌入 | `LLM-Embedder` | 与更大的 `BGE-large-en` 表现接近，模型规模约小三倍，质量与体积更均衡。 |
| 向量库 | `Milvus` | 满足论文列出的四项能力，但这只是功能矩阵，不是吞吐、成本或运维基准。 |
| 查询分类 | 训练一个是否需要检索的分类器 | 不需要外部知识的问题直接生成，减少无效检索及其噪声和延迟。 |
| 检索 | 性能优先用 `HyDE + Hybrid`；效率优先用 `Hybrid` | HyDE 缓解问题与文档表达不一致，Hybrid 结合关键词与语义匹配。 |
| 重排序 | 默认折中用 `monoT5`；固定语料低延迟用 `TILDEv2` | `monoT5` 是质量—延迟折中，不是单项最高分；`TILDEv2` 极快但依赖预索引。 |
| 重编排 | `Reverse` | 在论文端到端设置中平均分略高，但只应视为弱优胜者。 |
| 上下文压缩 | `Recomp`；时延敏感可不压缩 | 压缩有助于长度受限场景，但端到端增益很小且会增加延迟。 |
| 生成器微调 | 相关文档与随机文档混合训练 | 让生成器学会使用正确证据，同时抵抗检索噪声。 |

完整方法表和实验解释见 [engineering.md](engineering.md)。

## 两套配方

### 性能优先

```text
查询分类
→ HyDE + Hybrid
→ monoT5
→ Reverse
→ Recomp
→ LLM
```

核心代价：`HyDE` 需要额外生成伪文档，检索链路更慢。

### 效率均衡

```text
查询分类
→ Hybrid
→ TILDEv2
→ Reverse
→ Recomp
→ LLM
```

核心代价：`TILDEv2` 适合固定语料；新增文档需要重新预处理。

## 我的理解

这篇论文最值得保留的不是某个“最佳模型”，而是模块化排障方法：

```text
没有找到证据
→ 查 Chunking / Embedding / Retrieval

找到证据但顺序不好
→ 查 Reranking

证据正确但 LLM 没利用
→ 查 Repacking / Compression / Generator

系统太慢
→ 查 Query Classification、HyDE 调用、Reranker 和 Compression
```

## 关键结论

1. `Retrieval` 在 RAG 中承担候选召回职责；它与推荐系统的召回在功能上相似，但语料、目标和指标不同。
2. `HyDE + Hybrid` 在论文的 `mAP` 和深召回上领先，但 `nDCG@10` 不如单独 `HyDE`，延迟也更高；“最好”必须绑定指标。
3. `RankLLaMA` 在独立重排实验中质量最高，`TILDEv2` 最快，`monoT5` 是作者采用的折中点。
4. `Reverse` 相对 `Sides`、`Forward` 的优势很小，且无显著性检验，不能当成跨任务定律。
5. `Recomp` 不是默认必开：端到端平均分仅从不压缩的 `0.441` 提升到 `0.446`，延迟从 `10.97s` 增至 `11.70s`。

## 证据边界

必须同时记住：

1. 论文采用顺序式、逐模块贪心选择，不是穷举组合，因此逐模块选择不等于全局最优。
2. 切块、检索、重排、压缩和端到端实验使用的数据集与指标并不完全一致，不能横向比较不同表中的绝对分数。
3. “效率均衡配方”是把局部低延迟选项组合出的建议，论文没有单独报告这套完整组合的端到端结果。
4. Discussion 声称性能配方平均分为 `0.483`，但 Table 1 最终 `Recomp` 行的 `Avg Score` 为 `0.446`；原文没有给出足以消解差异的口径说明，复现时应重新计算。

## 后续可以研究的点

- 自适应检索：比按任务类型二分类更细地判断“何时检索、检索多少”。
- 多跳检索：查询分解在真正的多跳任务中如何迭代检索与合并证据。
- 全局模块搜索：验证模块间交互，而不是按固定顺序贪心锁定。
- 生产评测：在真实语料上同时测答案质量、召回、延迟、成本和知识更新。

## 参考资料

- [Wang et al., 2024. Searching for Best Practices in Retrieval-Augmented Generation（ACL Anthology）](https://aclanthology.org/2024.emnlp-main.981/)
- [论文 PDF](https://aclanthology.org/2024.emnlp-main.981.pdf)
- [作者官方代码仓库：FudanDNN-NLP/RAG](https://github.com/FudanDNN-NLP/RAG)
- [本地 Obsidian 深读笔记](</Users/will/research/note/GUI-AGENT/20 Papers/RAG/Searching_for_Best_Practices_in_Retrieval_Augmented_Generation/Searching_for_Best_Practices_in_Retrieval_Augmented_Generation.md>)
