---
title: "Agent memory"
tags: [concept, llm]
aliases: ["代理人記憶", "LLM 記憶", "長期記憶"]
papers: ["[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]", "[[【Web World Models】]]"]
related: ["[[Retrieval-augmented generation]]", "[[Compound AI system]]", "[[Collaborative filtering]]"]
---

# Agent memory

> [!NOTE]
> **一句話**：讓 LLM 代理人在多次互動之間記住事情的機制，通常存成一段會被持續改寫的文字，或是可以檢索的紀錄。

## 白話解釋
LLM 本身每次對話都從零開始。要讓代理人「越用越懂你」，就得在外面另存記憶：每次互動後整理重點，下次再讀回來。

## 常見設計
| 類型 | 做法 | 例子 |
|---|---|---|
| 檢索式 | 把過去紀錄存進向量資料庫，需要時取回 | RAG |
| 分層管理 | 像作業系統管理記憶體，主動搬進搬出上下文 | MemGPT |
| 反思式 | 定期把零碎紀錄總結成高層次的洞察 | Generative Agents |
| 圖結構 | 記憶存成知識圖譜或時間圖 | Zep |
| 協同式 | 記憶連成圖，互動後同時更新相關的鄰居 | MemRec |

## 設計重點（MemRec 的經驗）
- **管記憶和用記憶分開**：便宜的模型整理，強模型只讀精簡內容
- **更新放背景**：不讓記憶整理拖慢回應
- **風險**：記憶寫錯會一直留著，協同式更新還會把錯誤擴散給別人

## 出現在這些論文
- [[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]
- [[【Web World Models】]]

## 相關概念
[[Retrieval-augmented generation]]、[[Compound AI system]]、[[Collaborative filtering]]
