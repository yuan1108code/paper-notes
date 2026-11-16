---
title: "Compound AI system"
tags: [concept, llm]
aliases: ["複合 AI 系統"]
papers: ["[[【Optimizing generative AI by backpropagating language model feedback】]]", "[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]", "[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]", "[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]"]
related: ["[[Prompt optimization]]", "[[LLM-as-a-judge]]", "[[Backpropagation]]"]
---

# Compound AI system

> [!NOTE]
> **一句話**：由多個 LLM 呼叫加上搜尋、程式執行、資料庫、模擬器等工具串成的 AI 系統，而不是單一模型。

## 白話解釋
RAG 就是最常見的例子：查詢改寫 → 檢索 → 重新排序 → 生成答案 → 檢查，每一步都可能是不同的模型或工具。代理人（agent）系統更複雜，會自己規劃要用哪些工具。

## 難點
- 每個元件的提示詞和串接方式多半靠人工調
- 元件之間傳的是文字，很多是黑盒子 API，無法用一般的反向傳播一起優化
- 前面一步錯了，後面會一路錯下去

## 自動優化的方向
DSPy、TextGrad 這類框架，嘗試把整個系統當成一張計算圖，自動調整每個元件的提示詞或輸出。

## 出處
Zaharia et al., *The shift from models to compound AI systems*, BAIR Blog 2024

## 出現在這些論文
- [[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]
- [[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]
- [[【Optimizing generative AI by backpropagating language model feedback】]]
- [[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]

## 相關概念
[[Prompt optimization]]、[[LLM-as-a-judge]]、[[Backpropagation]]
