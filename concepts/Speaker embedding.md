---
title: "Speaker embedding"
tags: [concept, speech]
aliases: ["聲紋向量", "語者嵌入"]
papers: ["[[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]", "[[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]"]
related: ["[[ECAPA-TDNN]]", "[[AAM-softmax]]", "[[Speaker verification]]", "[[Speaker diarization]]"]
---

# Speaker embedding

> [!NOTE]
> **一句話**：把一段人聲壓縮成固定長度的向量，同一個人的向量彼此靠近、不同人的離得遠，可以想成「聲音的指紋」。

## 白話解釋
不管錄音長短，都轉成同樣長度（例如 192 或 256 維）的一串數字。比較兩段聲音是不是同一人，只要算兩個向量有多接近（常用餘弦相似度）。

## 怎麼訓練
一般架構：逐幀處理聲音特徵 → 池化層把整段話彙總成一個向量 → 分類層預測是哪位說話者。訓練完把分類層拿掉，中間那個向量就是聲紋向量。常搭配加邊界的損失函數（例如 AAM-softmax）讓不同人分得更開。

## 用在哪裡
- 語者驗證：是不是本人
- 語者分段：把每個片段的聲紋分群
- 語音合成、變聲：指定「像誰的聲音」

## 出現在這些論文
- [[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]
- [[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]

## 相關概念
[[ECAPA-TDNN]]、[[AAM-softmax]]、[[Speaker verification]]、[[Speaker diarization]]
