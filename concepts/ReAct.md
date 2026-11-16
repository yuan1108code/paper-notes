---
title: "ReAct"
tags: [concept, llm]
aliases: ["Reasoning and Acting"]
papers: ["[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]", "[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]", "[[【A Survey on LLM-as-a-Judge】]]", "[[【Web World Models】]]"]
related: ["[[Chain-of-thought prompting]]", "[[Retrieval-augmented generation]]", "[[Compound AI system]]"]
---

# ReAct

> [!NOTE]
> **一句話**：讓 LLM 交替寫出「想法」和「動作」：先想下一步要做什麼，再呼叫工具（搜尋、查資料庫），看到結果後再想下一步。

## 白話解釋
提示詞固定成這種格式，反覆進行：
- **Thought**：我現在要做什麼、為什麼
- **Action**：選一個工具，例如搜尋、資料庫
- **Action input**：給工具的輸入，例如查詢語句
- **Observation**：工具回傳的結果

直到模型寫出「我知道答案了」，再輸出最終答案。這是大多數代理人（agent）框架的基本骨架，LangChain 的代理人也是這樣實作。

## 優缺點
| 優點 | 缺點 |
|---|---|
| 能用外部工具，減少幻覺 | 走一步算一步，沒有整體規劃，容易繞路或漏查 |
| 推理過程看得到，好除錯 | 每一步都要呼叫 LLM，成本高 |

PlanRAG 就是在 ReAct 前面加上「Plan」，每次觀察後再加「Re-plan」，補上整體規劃。

## 出處
Yao et al., *ReAct: Synergizing reasoning and acting in language models*, ICLR 2023

## 出現在這些論文
- [[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]
- [[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]
- [[【A Survey on LLM-as-a-Judge】]]
- [[【Web World Models】]]

## 相關概念
[[Chain-of-thought prompting]]、[[Retrieval-augmented generation]]、[[Compound AI system]]
