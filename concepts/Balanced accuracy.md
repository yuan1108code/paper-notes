---
title: "Balanced accuracy"
tags: [concept, evaluation]
aliases: ["平衡正確率", "balanced acc"]
papers: ["[[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]", "[[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]"]
related: ["[[AUROC]]", "[[Industrial anomaly detection]]"]
---

# Balanced accuracy

> [!NOTE]
> **一句話**：正例和負例各自算正確率再平均；不管資料多不平衡，全猜同一個答案都只有 50%。

## 白話解釋
一般正確率在資料不平衡時會騙人：測試集有 73% 是瑕疵品，模型每張都說「有瑕疵」就有 73%。平衡正確率把兩類分開算：

$$\text{Balanced Acc} = \frac{\text{召回率（正例判對的比例）} + \text{特異度（負例判對的比例）}}{2}$$

全猜正例：召回率 100%、特異度 0% → 50%；全猜負例也是 50%。所以數字可以直接和 50% 比。

## 反推特異度
只要論文同時給平衡正確率和召回率，就能算出特異度：特異度 = 2 × 平衡正確率 − 召回率。這招可以快速看出模型是不是幾乎只答一種答案。

<center><b>表 1：同樣的模型，兩種算法（例）</b></center>

| 情境 | 一般正確率 | 平衡正確率 |
|---|---|---|
| MVTec-AD 測試集全答「有瑕疵」（1,258 異常、467 正常） | 72.9% | 50% |
| MMAD 上 LLaVA-1.5 13B（召回率 99.79%） | – | 49.96%（特異度 0.13%） |

> [!TIP]
> 評估警戒、告警這類是／否判斷時，報平衡正確率，再拆出漏報率（1 − 召回率）和誤報率（1 − 特異度），比單一正確率有用得多。

## 出現在這些論文
- [[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]
- [[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]

## 相關概念
[[AUROC]]、[[Industrial anomaly detection]]
