---
title: "Cost-of-pass"
tags: [concept, llm, evaluation]
aliases: ["每個正確答案的成本"]
papers: ["[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]"]
related: ["[[Retrieval-augmented generation]]", "[[Compound AI system]]"]
---

# Cost-of-pass

> [!NOTE]
> **一句話**：得到一個正確答案平均要花多少錢，算法是「每題成本 ÷ 正確率」。同時懲罰太貴和太常錯的系統。

## 白話解釋
系統 A 每題 0.27 美分、正確率 60%，平均花 0.27 / 0.6 = 0.45 美分才換到一個正確答案。系統 B 每題只要 0.04 美分、但正確率 38%，cost-of-pass 是 0.11，比較低，可是錯一半以上。所以這個指標通常要和正確率一起看。

## 要注意
- 多個資料集時，先各自算再平均，或先平均成本與正確率再相除，結果會不同（GDP-RAG 論文用前者，難的資料集權重較大）
- 成本通常只算 LLM token 費用，不含檢索、延遲

## 出處
Wang et al., 2025（cost-of-pass 框架），GDP-RAG 論文把它當主要指標。

## 出現在這些論文
- [[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]

## 相關概念
[[Retrieval-augmented generation]]、[[Compound AI system]]
