---
title: "Permutation-invariant training"
tags: [concept, speech]
aliases: ["排列不變訓練", "PIT"]
papers: ["[[【Powerset multi-class cross entropy loss for neural speaker diarization】]]", "[[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]"]
related: ["[[Hungarian algorithm]]", "[[Speaker diarization]]"]
---

# Permutation-invariant training

> [!NOTE]
> **一句話**：當答案的順序可以任意交換時，訓練時找「讓損失最小的那種對應方式」再計算損失。

## 白話解釋
語者分段模型輸出「第 1、2、3 個說話者」，但這只是編號，沒有固定對應到哪個真實的人。把 A 和 B 的位置對調，答案一樣正確。如果硬要模型猜對編號，就會被不合理地懲罰。

做法：試所有可能的對應方式（或用匈牙利演算法快速找最佳配對），用損失最小的那種來訓練。

## 副作用
模型只保證「同一段內分得開」，不保證「不同段用同一個編號」。所以長錄音切成短片段後，還需要分群把各片段的說話者對起來。

## 出現在這些論文
- [[【Powerset multi-class cross entropy loss for neural speaker diarization】]]
- [[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]

## 相關概念
[[Hungarian algorithm]]、[[Speaker diarization]]
