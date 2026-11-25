---
title: "Reward model"
tags: [concept, llm]
aliases: ["獎勵模型", "過程獎勵模型", "PRM"]
papers: ["[[【A Survey on LLM-as-a-Judge】]]", "[[【Recursive Self-Improvement in AI：From Bounded Self-Refinement to Autonomous Research Loops】]]"]
related: ["[[LLM-as-a-judge]]", "[[Majority voting]]"]
---

# Reward model

> [!NOTE]
> **一句話**：給模型輸出打分數的模型，分數拿來當強化學習的訓練訊號，或在推論時從多個候選中挑最好的。

## 白話解釋
RLHF 的做法：先收集人對兩個回答的偏好，訓練一個獎勵模型學會「人會比較喜歡哪個」，再用它的分數去強化學習（PPO）訓練語言模型。

| 種類 | 打分對象 | 例子 |
|---|---|---|
| 結果獎勵模型（ORM） | 只看最終答案 | RLHF 的偏好模型 |
| 過程獎勵模型（PRM） | 每一個推理步驟 | Lightman et al.（Let's Verify Step by Step）、Math-Shepherd |
| 規則式獎勵 | 用程式檢查答案對錯、格式 | DeepSeek-R1 的推理任務 |

## 和 LLM 評審的關係
LLM 評審可以直接當獎勵模型（不用另外訓練），也可以用它大量產生偏好標籤來訓練獎勵模型。推論時的 Best-of-N 也是讓評審從 N 個答案挑一個。

> [!WARNING]
> 獎勵模型會被「鑽漏洞」（reward hacking）：被訓練的模型學會討好評審的表面特徵（例如寫長一點），而不是真的變好。這也是 DeepSeek-R1 在推理任務改用規則式獎勵的原因之一。

## 出現在這些論文
- [[【Recursive Self-Improvement in AI：From Bounded Self-Refinement to Autonomous Research Loops】]]：討論自我改進迴圈中，訊號可靠性、跨任務持續性與共享錯誤的風險。
- [[【A Survey on LLM-as-a-Judge】]]

## 相關概念
[[LLM-as-a-judge]]、[[Majority voting]]
