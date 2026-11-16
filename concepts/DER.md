---
title: "DER"
tags: [concept, speech]
aliases: ["Diarization Error Rate", "語者分段錯誤率"]
papers: ["[[【Powerset multi-class cross entropy loss for neural speaker diarization】]]", "[[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]", "[[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]"]
related: ["[[Speaker diarization]]"]
---

# DER

> [!NOTE]
> **一句話**：語者分段的主要評估指標，三種錯誤所佔時間的總和比例，越低越好。

## 三種錯誤
| 錯誤 | 英文 | 意思 |
|---|---|---|
| 誤報 | False alarm（FA） | 沒人說話，卻判成有人 |
| 漏抓 | Missed detection（MISS） | 有人說話，卻沒抓到 |
| 認錯人 | Speaker confusion（CONF） | 有抓到，但歸給錯的說話者 |

$$DER = \frac{FA + MISS + CONF}{\text{總說話時間}}$$

> [!WARNING]
> 不同論文的 DER 常常**不能直接比較**：有沒有容忍邊界（forgiveness collar，例如前後 250 毫秒不算錯）、重疊說話算不算分、是否假設語音偵測完全正確，都會讓數字差很多。看數字前先確認設定。

## 出現在這些論文
- [[【Powerset multi-class cross entropy loss for neural speaker diarization】]]
- [[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]
- [[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]

## 相關概念
[[Speaker diarization]]
