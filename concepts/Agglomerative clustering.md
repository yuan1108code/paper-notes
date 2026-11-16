---
title: "Agglomerative clustering"
tags: [concept, speech]
aliases: ["階層式聚合分群", "Agglomerative hierarchical clustering"]
papers: ["[[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]"]
related: ["[[Speaker diarization]]", "[[Speaker embedding]]"]
---

# Agglomerative clustering

> [!NOTE]
> **一句話**：由下往上的分群方法：一開始每筆資料自成一群，每次把最接近的兩群合併，直到距離超過門檻為止。

## 白話解釋
不用事先知道要分幾群，只要設定「多遠就不再合併」的門檻。這正適合語者分段：事先不知道錄音裡有幾個人。

## 「兩群有多近」的算法（連結方式）
| 連結方式 | 兩群距離怎麼算 |
|---|---|
| 單一連結 | 兩群中最近的兩點 |
| 完全連結 | 兩群中最遠的兩點 |
| 平均連結 | 所有點對距離的平均 |
| 質心連結（UPGMC） | 兩群中心點的距離 |
| Ward | 合併後群內變異增加多少 |

pyannote.audio 2.1 在每個驗證集上都發現**質心連結最好**，第二名是平均連結。

> [!TIP]
> 選它的另一個原因是只有一個超參數，而且不要求資料按固定時間間隔排列。

## 出現在這些論文
- [[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]

## 相關概念
[[Speaker diarization]]、[[Speaker embedding]]
