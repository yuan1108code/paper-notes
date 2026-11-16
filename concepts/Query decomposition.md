---
title: "Query decomposition"
tags: [concept, llm]
aliases: ["問題拆解", "子問題拆解"]
papers: ["[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]", "[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]"]
related: ["[[Multi-hop question answering]]", "[[Retrieval-augmented generation]]", "[[ReAct]]", "[[Chain-of-thought prompting]]"]
---

# Query decomposition

> [!NOTE]
> **一句話**：把一個需要多份資料才能回答的大問題，拆成幾個可以各自檢索、各自回答的子問題，最後再把子答案組合起來。

## 白話解釋
「Joseph D. Stewart 擔任校長的學院在哪裡？」可以拆成「他是哪所學院的校長？」和「那所學院在哪裡？」。每個子問題的關鍵字比較集中，比較容易檢索到正確文件。

## 兩種時機
| 做法 | 什麼時候拆 | 代表 | 常見問題 |
|---|---|---|---|
| 逐步式 | 每查完一次再決定下一個子問題 | ReAct、Self-Ask、IRCoT | 前一步錯後面跟著錯；文件越堆越多 |
| 計畫式 | 一開始就把全部子問題規劃好 | ReWOO、PlanRAG、PAR-RAG | 沒看文件就拆，容易多拆或少拆 |

GDP-RAG 的改法：拆之前先用原問題檢索一次，只為文件裡還沒有的資訊出子問題。

## 出現在這些論文
- [[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]
- [[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]

## 相關概念
[[Multi-hop question answering]]、[[Retrieval-augmented generation]]、[[ReAct]]、[[Chain-of-thought prompting]]
