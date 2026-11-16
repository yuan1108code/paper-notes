---
title: "Speaker diarization"
tags: [concept, speech]
aliases: ["語者分段", "說話者分段"]
papers: ["[[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]", "[[【Powerset multi-class cross entropy loss for neural speaker diarization】]]", "[[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]"]
related: ["[[Speaker embedding]]", "[[Agglomerative clustering]]", "[[DER]]", "[[Permutation-invariant training]]"]
---

# Speaker diarization

> [!NOTE]
> **一句話**：回答「一段錄音裡，誰在什麼時候說話？」把同一個人說的片段歸在一起，但不需要知道他是誰。

## 白話解釋
輸出是一條時間軸，例如「說話者 A：0–3 秒、說話者 B：2–7 秒」。只區分「不同的人」，不辨認真實身分；要知道是誰，得再接語者辨識。

## 常見做法
| 做法 | 怎麼做 | 優點 | 缺點 |
|---|---|---|---|
| 傳統流水線 | 語音偵測 → 抽聲紋 → 分群 | 任意長度、任意人數 | 不擅長兩人同時說話，前面的錯誤會一路傳下去 |
| 端到端（EEND） | 一個神經網路直接輸出誰在說話 | 能處理重疊 | 人數多、錄音長就吃力，很吃訓練資料 |
| 混合式 | 短片段用神經網路，再用聲紋分群接回整段 | 兼顧重疊與長錄音 | 流程較長、超參數較多 |

> [!TIP]
> 應用：會議紀錄、客服錄音分析、影片字幕、多人語音問答的前處理。

## 出現在這些論文
- [[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]
- [[【Powerset multi-class cross entropy loss for neural speaker diarization】]]
- [[【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】]]

## 相關概念
[[Speaker embedding]]、[[Agglomerative clustering]]、[[DER]]、[[Permutation-invariant training]]
