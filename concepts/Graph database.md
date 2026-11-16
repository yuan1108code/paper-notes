---
title: "Graph database"
tags: [concept, llm]
aliases: ["圖資料庫", "GDB"]
papers: ["[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]", "[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]", "[[【Interactive and Intelligent Root Cause Analysis in Manufacturing with Causal Bayesian Networks and Knowledge Graphs】]]"]
related: ["[[Retrieval-augmented generation]]", "[[Knowledge graph]]"]
---

# Graph database

> [!NOTE]
> **一句話**：用「節點」和「邊」儲存資料的資料庫，節點和邊都可以帶屬性，適合存關係很多、要沿著關係一層層查的資料。

## 白話解釋
關聯式資料庫（RDB）把資料放在一張張表裡，要串起關係就得 JOIN；圖資料庫直接把關係存成邊。例如「工廠 → 生產 → 商品 → 被誰消耗」，在圖上就是沿著邊走幾步。

最常見的是**標記屬性圖**（labeled property graph，LPG）：節點有標籤（例如 Trade_node），節點和邊都可以有屬性（例如 flow = 0.78）。

## 查詢語言
常用 **Cypher**（Neo4j）：
```
MATCH (s:Trade_node)-[r:source]->(d:Trade_node {name: "Deccan"})
RETURN s.name, r.flow
```

## 什麼時候比 RDB 好
- 關係有好幾層，要多步走訪（供應鏈、社群網路、知識圖譜）
- PlanRAG 發現：Building 情境要沿供應鏈走很多層，用圖資料庫時 PlanRAG 的正確率（49.5%）比用 RDB（40.6%）高

常見產品：Neo4j、Amazon Neptune、ArangoDB。

## 出現在這些論文
- [[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]
- [[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]
- [[【Interactive and Intelligent Root Cause Analysis in Manufacturing with Causal Bayesian Networks and Knowledge Graphs】]]

## 相關概念
[[Retrieval-augmented generation]]
