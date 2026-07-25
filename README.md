# AI Knowledge

个人 AI 知识库。

## 目录规则

每个主题使用一个独立目录，并直接放在仓库根目录，与 `AGENTS.md` 同层。

```text
ai-knowledge/
├── AGENTS.md
├── README.md
├── rag/
├── agent-memory/
└── gui-agent/
```

不使用 `topics/` 汇总目录。所有主题统一在 Git `main` 分支维护。

## 已收录主题

- [检索与排序多阶段链路](retrieval-ranking-pipelines/README.md)：用推荐系统与 RAG 两条数据链路讲清召回、检索、粗排、精排、Ranking、Rerank、过滤、融合和 Context Packing。
