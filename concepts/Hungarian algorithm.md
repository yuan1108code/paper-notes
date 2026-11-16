---
title: "Hungarian algorithm"
tags: [concept, speech]
aliases: ["匈牙利演算法"]
papers: ["[[【Powerset multi-class cross entropy loss for neural speaker diarization】]]"]
related: ["[[Permutation-invariant training]]"]
---

# Hungarian algorithm

> [!NOTE]
> **一句話**：解決「一對一最佳配對」的經典演算法：給一張成本表，找出總成本最低的配對方式。

## 白話解釋
例如 3 個工人和 3 份工作，每個人做每份工作的成本不同，要怎麼分配讓總成本最低？暴力嘗試要算 3! = 6 種，人數一多就爆炸；匈牙利演算法可以在多項式時間內找到最佳解。

## 在論文裡怎麼用
排列不變訓練時，先算出「模型的每個輸出 × 每個真實說話者」兩兩之間的損失，形成一張成本表，再用匈牙利演算法找最佳對應。

> [!TIP]
> 和最佳化很相關：本質是指派問題（assignment problem），也能寫成整數規劃。排班、派工、資源分配都用得到。

## 出現在這些論文
- [[【Powerset multi-class cross entropy loss for neural speaker diarization】]]

## 相關概念
[[Permutation-invariant training]]
