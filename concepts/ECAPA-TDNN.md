---
title: "ECAPA-TDNN"
tags: [concept, speech]
papers: ["[[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]", "[[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]", "[[【Powerset multi-class cross entropy loss for neural speaker diarization】]]"]
related: ["[[Speaker embedding]]", "[[Speaker verification]]"]
---

# ECAPA-TDNN

> [!NOTE]
> **一句話**：一種常用的聲紋向量模型，是 TDNN 的改良版，VoxSRC 2020 語者驗證比賽的冠軍系統。

## 白話解釋
TDNN（時延神經網路）用不同寬度的時間視窗看聲音，捕捉短時間和較長時間的聲音特徵。ECAPA-TDNN 在這基礎上加了三個改良：
- 通道注意力：讓模型自己判斷哪些特徵比較重要
- 多層特徵整合：把淺層和深層的資訊一起用
- 注意力池化：彙總整段話時，讓重要的片段權重較高

## 為什麼常見
效果好、模型不大，有現成的預訓練版本（例如 SpeechBrain）。pyannote.audio 2.1 比較了四種聲紋模型，它在 VoxConverse 上表現最好（DER 10.7%）。

## 出處
Desplanques et al., *ECAPA-TDNN: Emphasized Channel Attention, Propagation and Aggregation in TDNN Based Speaker Verification*, Interspeech 2020

## 出現在這些論文
- [[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]
- [[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]
- [[【Powerset multi-class cross entropy loss for neural speaker diarization】]]

## 相關概念
[[Speaker embedding]]、[[Speaker verification]]
