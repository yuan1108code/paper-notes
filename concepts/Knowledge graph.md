---
title: "Knowledge graph"
tags: [concept, knowledge-representation]
aliases: ["知識圖譜", "KG"]
papers: ["[[【Interactive and Intelligent Root Cause Analysis in Manufacturing with Causal Bayesian Networks and Knowledge Graphs】]]"]
related: ["[[Graph database]]", "[[GraphRAG]]", "[[Causal discovery]]"]
---

# Knowledge graph

> [!NOTE]
> **一句話**：用「實體—關係—實體」的三元組存知識的有向標籤圖，可以沿關係推理。

## 白話解釋
知識圖譜把知識寫成很多條「主詞 — 關係 — 受詞」：例如「焊接站 — hasProcessStep — 焊接」「濕度 — hasNoImpact — 熱輸入」。節點是實體，邊是有名字的關係，整體是一張有向標籤圖，通常存在圖資料庫（Neo4j 等）裡，用 Cypher、SPARQL 查詢。

## 能做什麼
- **推理**：沿著關係推出沒寫明的事實，例如「A 在 B 之前、B 在 C 之前 → A 在 C 之前」
- **約束其他模型**：把硬規則交給機器學習，例如因果發現時排除「後面工序影響前面工序」的邊
- **給 LLM 當檢索來源**：GraphRAG 沿著圖譜的關係找相關證據

## 設計重點
- **結構（schema）先定好**：哪幾類節點、哪幾種關係（如 Line → Station → Process Step → Sensor Variable）
- **順序關係**（hasSuccessor）很有用：一條就能推出大量「不可能」的關係
- **否定知識**也要存（如 hasNoImpact），專家的修正才能累積

> [!TIP]
> 災防領域可以把「雨量站 → 流域 → 鄉鎮 → 避難所」、「降雨 → 土石流潛勢 → 道路中斷」這類上下游關係做成知識圖譜，同時給因果分析和 RAG 使用。

## 出現在這些論文
- [[【Interactive and Intelligent Root Cause Analysis in Manufacturing with Causal Bayesian Networks and Knowledge Graphs】]]

## 相關概念
[[Graph database]]、[[GraphRAG]]、[[Causal discovery]]
