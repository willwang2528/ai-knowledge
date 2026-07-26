# RAG 模块链路视觉图 Prompt

## 图片 1：三条链路知识地图

主题：RAG 模块链路总览  
图片数量：1  
图片目标：让读者第一眼区分离线索引、在线请求、训练评估支撑。  
视觉结构：横向三泳道；上方为离线检索源，中间为在线主链，下方为训练与评估反馈。  
必须元素：

- Offline Indexing（离线索引）：Documents → Chunking → Embedding → Vector Database / Index
- Online Request（在线请求）：Query → Query Classification → Retrieval → Reranking → Repacking → Context Compression → LLM Generation
- Training & Evaluation（训练与评估）：Fine-tuning、General Performance、Specific Domains、Retrieval Capability、Latency
- 用虚线反馈箭头从 Evaluation 指回各模块

文字内容：每个模块只写英文与中文名称，不写算法细节。  
风格：白底、蓝色在线链、橙色离线链、绿色评估链，扁平技术信息图，16:9，中文清晰，无装饰性人物。

## 图片 2：检索到生成的漏斗架构

主题：RAG 在线主链如何逐步缩小和整理信息  
图片数量：1  
图片目标：解释 Retrieval、Reranking、Repacking、Compression 的区别。  
视觉结构：从左到右的漏斗加上下文容器。
必须元素：

- Retrieval（检索）：全量语料 → Top-k 候选；标注“高召回，尽量不漏”
- Reranking（重排序）：Top-k → 更准确顺序；标注“高精度”
- Repacking（文档重编排）：相同候选，展示 Forward / Reverse / Sides 三种摆放
- Compression（上下文压缩）：长文档 → 短证据；标注“去冗余，但可能丢信息”
- LLM：Query + Evidence → Answer

文字内容：突出“选哪些 → 谁更相关 → 放哪里 → 留什么”。  
风格：工程架构图，深蓝和青色，候选数量逐步收缩，信息密度中等，16:9。

## 图片 3：方法选择与证据强度

主题：论文中的性能配方、效率配方与证据边界  
图片数量：1  
图片目标：避免把蓝色方法或逐模块胜者误解为全局最优。  
视觉结构：左右两条配方，中间为对比标尺，底部为三条警示。
必须元素：

- 性能优先：Classifier → HyDE + Hybrid → monoT5 → Reverse → Recomp
- 效率均衡：Classifier → Hybrid → TILDEv2 → Reverse → Recomp
- 标尺：Quality、Latency、Fixed vs Updating Corpus
- 三条警示：
  1. Greedy module search ≠ global optimum
  2. Different modules use different datasets and metrics
  3. Balanced recipe lacks a separately reported end-to-end result
- 数字冲突：Discussion `0.483` vs Table 1 `0.446`，标注“needs reproduction”

文字内容：英文术语后跟中文，保持短句。  
风格：研究证据图，性能路线用紫色、效率路线用绿色、警示用红色边框，白底，16:9。

## 后续可以研究的点

- 为真实系统指标生成动态路由决策图。
- 为多跳 RAG 生成迭代检索时序图。
- 为模块级错误诊断生成可观测性面板图。
