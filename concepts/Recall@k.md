---
title: "Recall@k"
tags: [concept, evaluation]
aliases: ["召回率@k", "R@k"]
papers: ["[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]", "[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]"]
related: ["[[Retrieval-augmented generation]]", "[[BM25]]", "[[NDCG]]"]
---

# Recall@k

> [!NOTE]
> **一句話**：檢索結果的前 k 名裡，找回了多少比例的「正確答案需要的文件」。

## 白話解釋
一題需要 3 份支持文件，系統回傳的前 5 段裡有 2 份 → Recall@5 = 2/3 ≈ 66.7%。再把所有題目平均。
- k 越大，越容易找齊，所以數字一定越來越高（Recall@1 ≤ Recall@2 ≤ Recall@5 ≤ Recall@10）
- 多步驟問答特別在意它：只要漏一份，證據鏈就斷了

## 要注意
- 只看「有沒有找到」，不看排第幾（排序品質要看 MRR、nDCG）
- 只能比較會回傳段落清單的方法；直接回傳答案或摘要的 GraphRAG 類方法沒有 Recall@k
- k 要和實際交給 LLM 的段落數對齊才有意義（SAG 論文問答用前 5 段，所以主表報 Recall@5）

## 出現在這些論文
- [[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]
- [[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]

## 相關概念
[[Retrieval-augmented generation]]、[[BM25]]、[[NDCG]]
