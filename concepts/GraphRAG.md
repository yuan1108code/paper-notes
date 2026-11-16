---
title: "GraphRAG"
tags: [concept, llm]
aliases: ["圖譜式 RAG", "Graph RAG"]
papers: ["[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]", "[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]"]
related: ["[[Retrieval-augmented generation]]", "[[Graph database]]", "[[Hypergraph]]", "[[Personalized PageRank]]"]
---

# GraphRAG

> [!NOTE]
> **一句話**：先用 LLM 把整個語料庫轉成知識圖譜（實體當節點、關係當邊），查詢時沿著圖找相關資訊，補上一般向量檢索「段落之間沒有關聯」的缺點。

## 白話解釋
一般 RAG 把每段文字各自變成向量，問題來了就找最像的幾段，段落之間互不相干。GraphRAG 一族在建索引時多做一步：讓 LLM 讀每段文字，抽出「誰–做了什麼–對誰」的三元組，合成一張大圖。查詢時從問題提到的實體出發，沿著邊找到相關的實體和段落。

狹義的 GraphRAG 指 Microsoft 的實作（Edge et al., 2024），還會把圖分群、替每群寫摘要，適合回答「整個語料庫的主題是什麼」這類全域問題。

## 常見做法
| 方法 | 特色 |
|---|---|
| Microsoft GraphRAG | 實體圖 + 社群摘要 |
| HippoRAG／HippoRAG 2 | 知識圖 + 個人化 PageRank 傳遞相關性 |
| LightRAG | 支援增量更新，分低階（實體）與高階（關係）檢索 |
| HyperGraphRAG、HyperRAG | 用超圖保留多元關係 |

## 缺點
- 建圖要對整個語料呼叫 LLM，很貴（SAG 論文中 GraphRAG 建 MuSiQue 索引花 10 小時、1.25 億 token）
- 三元組會把一件事拆碎，丟掉「這些關係屬於同一件事」的邊界
- 新文件進來要合併節點，難以增量更新

SAG 的替代做法：不建全域圖，每段只存一個事件和它的實體，查詢時用 SQL JOIN 共同實體臨時串出關聯。

## 出現在這些論文
- [[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]
- [[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]

## 相關概念
[[Retrieval-augmented generation]]、[[Graph database]]、[[Hypergraph]]、[[Personalized PageRank]]
