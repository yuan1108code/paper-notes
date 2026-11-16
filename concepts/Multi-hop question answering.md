---
title: "Multi-hop question answering"
tags: [concept, llm]
aliases: ["多步驟問答", "多跳問答"]
papers: ["[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]", "[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]", "[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]"]
related: ["[[Retrieval-augmented generation]]", "[[Chain-of-thought prompting]]"]
---

# Multi-hop question answering

> [!NOTE]
> **一句話**：答案不在單一段落裡，必須先找到一個中間事實，再根據它找下一個，串起兩步以上才能回答的問題。

## 白話解釋
例如「Lost Gravity 雲霄飛車是哪個國家製造的？」要先知道製造商是 Mack Rides（第一跳），再查 Mack Rides 是德國公司（第二跳）。每一跳都可能需要不同的文件。

## 常見資料集
| 資料集 | 特色 |
|---|---|
| HotpotQA | 維基百科兩跳問題，常用基準 |
| 2WikiMultihopQA | 結合維基百科與 Wikidata，有固定的推理模式 |
| MuSiQue | 由單跳問題組合而成，2–4 跳，刻意避免捷徑，較難 |
| IIRC | 問題依附在一段主文上，要找主文提到的其他頁面補資訊 |

## 為什麼難
第二跳需要的文件，和原問題的字詞常常沒有重疊，只用問題去檢索找不到。

## 出現在這些論文
- [[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]
- [[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]
- [[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]

## 相關概念
[[Retrieval-augmented generation]]、[[Chain-of-thought prompting]]
