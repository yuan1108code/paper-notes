---
title: "Position bias"
tags: [concept, evaluation]
aliases: ["位置偏誤", "順序偏誤"]
papers: ["[[【A Survey on LLM-as-a-Judge】]]", "[[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]"]
related: ["[[LLM-as-a-judge]]", "[[Meta-evaluation]]", "[[Majority voting]]"]
---

# Position bias

> [!NOTE]
> **一句話**：LLM 評審比較兩個答案時，會因為答案放在前面或後面而改變判斷，跟內容好壞無關。

## 白話解釋
同一組答案 A、B，「A 在前」時評審說 A 好，「B 在前」時又說 B 好，代表它其實在看位置。有研究發現，ChatGPT 當評審時，Vicuna-13B 只要放在第二個位置就能贏過 ChatGPT。GPT-4 傾向選第一個，ChatGPT 傾向選第二個，不同模型偏好的位置不一樣。

## 怎麼量
**位置一致性**：每題正反各問一次，兩次選同一個答案的比例。

$$\text{Position Consistency} = \frac{1}{N}\sum_{i=1}^{N} \mathbb{I}(S_i^{AB} = S_i^{BA})$$

另外還有 conflict rate（對調後結論不同的比例）、preference fairness（偏好某個位置的程度）。

## 怎麼減輕
- 正反各問一次，取平均分（Wang et al.）
- 兩次結論不同就判平手（PandaLM）
- 清單排序時把順序打亂多次再彙總（permutation self-consistency）

> [!TIP]
> 位置一致性很便宜（只要多問一次），可以當作評審的健康指標長期監控。在 Gu et al. 的實驗中，GPT-4-turbo 是 80.31%，LLaMA3-8B 只有 38.85%。

## 出現在這些論文
- [[【A Survey on LLM-as-a-Judge】]]
- [[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]

## 相關概念
[[LLM-as-a-judge]]、[[Meta-evaluation]]、[[Majority voting]]
