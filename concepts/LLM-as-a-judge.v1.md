---
title: "LLM-as-a-judge"
tags: [concept, llm]
aliases: ["LLM 當評審", "LLM 評審"]
papers: ["[[【Optimizing generative AI by backpropagating language model feedback】]]", "[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]", "[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]", "[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]", "[[【A Survey on LLM-as-a-Judge】]]"]
related: ["[[Prompt optimization]]", "[[Compound AI system]]", "[[Meta-evaluation]]", "[[Position bias]]", "[[Majority voting]]", "[[Reward model]]", "[[Constrained decoding]]"]
---

# LLM-as-a-judge

> [!NOTE]
> **一句話**：用一個 LLM 來評分或批評另一個 LLM 的輸出，取代或輔助人工評估。

## 白話解釋
人工評估又慢又貴。給 LLM 一份評分標準（評估指令），讓它讀答案、打分數或寫出哪裡不好。在 TextGrad 裡，這份評語就是「損失」，也是反向傳播的起點。

## 用在哪裡
- 自動評估聊天機器人、RAG 問答的品質
- 當作優化的訊號：提示詞優化、答案自我修正

> [!WARNING]
> 評審本身會有偏誤：偏好比較長的答案、偏好自己產生的答案、對位置順序敏感。評分標準要寫清楚，最好和人工評估對照校正。

## 綜述整理的重點（Gu et al., A Survey on LLM-as-a-Judge）
- **四種問法**：打分數、是非題、兩兩比較、多選題。兩兩比較和人的判斷最一致
- **改進的三個位置**：提示詞（拆解步驟／標準、對調位置、結構化輸出）、模型（微調評審）、後處理（多輪多數決、多模型投票、分數平滑）
- **常見偏誤**：位置、長度、具體性（偏好有引用和數字的答案）、自我偏好、名稱（看到「gpt-4」就給高分）
- **實驗結論**：所有模型和人的一致率只有約 51–62%；建議用強模型、正反各問一次、多輪取多數決；取最高分會讓結果變差，同時要求寫理由也可能變差

## 出處
Zheng et al., *Judging LLM-as-a-judge with MT-Bench and Chatbot Arena*, NeurIPS 2023；綜述見 Gu et al., *A Survey on LLM-as-a-Judge*（arXiv 2411.15594）

## 出現在這些論文
- [[【Optimizing generative AI by backpropagating language model feedback】]]
- [[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]
- [[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]
- [[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]
- [[【A Survey on LLM-as-a-Judge】]]

## 相關概念
[[Prompt optimization]]、[[Compound AI system]]、[[Meta-evaluation]]、[[Position bias]]、[[Majority voting]]、[[Reward model]]、[[Constrained decoding]]
