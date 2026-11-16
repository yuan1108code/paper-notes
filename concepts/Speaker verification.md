---
title: "Speaker verification"
tags: [concept, speech]
aliases: ["語者驗證", "聲紋驗證"]
papers: ["[[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]"]
related: ["[[Speaker embedding]]", "[[AAM-softmax]]"]
---

# Speaker verification

> [!NOTE]
> **一句話**：判斷「這段聲音是不是某個人說的」，是一題是非題。

## 白話解釋
先替每個人註冊一段或多段聲音，得到他的聲紋向量。之後來一段新聲音，算它和註冊聲紋的相似度，超過門檻就判定是本人。

## 評估指標
- EER（等錯誤率）：調整門檻，讓「把別人當成本人」和「把本人拒絕」兩種錯誤一樣多時的錯誤率，越低越好
- minDCF（最小檢測成本）：依實際應用給兩種錯誤不同權重後的最低成本，越低越好

## 打分方式
- 餘弦相似度：最簡單，搭配加邊界損失訓練的聲紋很好用
- PLDA：較傳統的統計式打分
- 分數正規化（例如 AS-Norm）：校正不同錄音條件造成的分數偏移

## 出現在這些論文
- [[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]

## 相關概念
[[Speaker embedding]]、[[AAM-softmax]]
