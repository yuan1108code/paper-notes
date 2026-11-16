---
title: "Personalized PageRank"
tags: [concept, graph]
aliases: ["個人化 PageRank", "PPR"]
papers: ["[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]", "[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]"]
related: ["[[GraphRAG]]", "[[Label propagation]]"]
---

# Personalized PageRank

> [!NOTE]
> **一句話**：PageRank 的變形：隨機漫步時每一步都有一定機率「跳回起點」，所以算出來的分數代表「從這幾個起點出發，各節點有多容易被走到」。

## 白話解釋
在圖上放一個隨機走動的人，每一步沿著邊走到鄰居，但有機率（例如 15%）瞬間傳送回起點。跑久了之後，被造訪越多次的節點分數越高。起點選問題提到的實體，分數就變成「和這題的相關程度」。

## 在 RAG 中的用法
HippoRAG／HippoRAG 2 用它在知識圖上傳遞相關性：問題的實體當起點，算完 PPR 後挑分數高的段落。

## 缺點
因為每走一步都有機率跳回起點（阻尼），**離起點越遠的節點分數衰減越多**。SAG 論文認為這正是 HippoRAG 2 在四跳問題（MuSiQue）上表現較差的原因；SAG 改用 SQL JOIN，擴展時沒有分數衰減。

## 出現在這些論文
- [[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]
- [[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]

## 相關概念
[[GraphRAG]]、[[Label propagation]]
