---
title: "AAM-softmax"
tags: [concept, speech]
aliases: ["Additive angular margin softmax", "ArcFace loss"]
papers: ["[[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]"]
related: ["[[Speaker embedding]]", "[[Speaker verification]]"]
---

# AAM-softmax

> [!NOTE]
> **一句話**：加了「角度邊界」的分類損失函數：不只要分對，還要分得夠開，讓抽出的向量更好區分。

## 白話解釋
一般 softmax 只要求正確類別的分數最高就好。AAM-softmax 在計算正確類別的分數時，故意把向量和類別中心的夾角加上一個邊界 $m$，等於提高難度，逼模型把同一人的向量聚得更緊、不同人推得更遠。

## 家族
- A-softmax：乘法式的角度邊界
- AM-softmax：在餘弦值上減一個邊界
- AAM-softmax：在角度上加一個邊界（人臉辨識的 ArcFace 也是這個）

## 訓練技巧
- 邊界排程：一開始邊界設 0，再逐漸加到目標值，避免模型一開始學不起來
- 大邊界微調：訓練好後用更大的邊界（例如 0.5）和更長的片段再微調一次

## 出現在這些論文
- [[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]

## 相關概念
[[Speaker embedding]]、[[Speaker verification]]
