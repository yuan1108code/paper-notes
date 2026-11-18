---
title: "Pareto hypervolume"
tags: [concept, optimization, metric]
aliases: ["Pareto 超體積", "品質成本面積"]
papers: ["[[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]"]
related: ["[[Replay simulator]]"]
---

# Pareto hypervolume

> [!NOTE]
> 不只比最高品質，也量化「用多小的成本就達到多好的品質」所支配的面積；越大通常代表品質成本取捨越好。

## 白話解釋
兩個方案中，若 A 的成本不高於 B、品質不低於 B，且至少一項嚴格更好，B 就被 A 支配。先留下不被支配的 Pareto frontier，再相對同一參考點量面積，避免只比最昂貴方案的最高分。

## Dream-RSI 的二維例子
依 [原論文式 (1)](https://arxiv.org/html/2609.14858v2#S3)，令歷史世界總呼叫數為 B，品質 a 正規化到 [0,1]，frontier 依成本 n 遞增，a₀=0：

$$HV=\frac1B\sum_j(a_j-a_{j-1})(B-n_j).$$

示意 B=10，點為 (2,0.5)、(4,0.8)、(6,1)、(8,1)，最後一點被 (6,1) 支配；HV=0.4+0.18+0.08=0.66。這是教學數據，不是論文實驗。

| 比較時必須固定 | 原因 |
|---|---|
| 參考成本 B | 換參考點就會改面積 |
| 品質正規化分母 | 不同世界最高分不同，不能直接混算 |
| 分數方向 | 越低越好的誤差必須先轉成合適品質 |
| 被比較點的來源 | 相同歷史內的好取捨不代表未來泛化 |

> [!WARNING]
> Dream-RSI 的最終 reward 還減去序列輪數／呼叫數的並行懲罰；hypervolume 本身不是實際美元成本、牆鐘時間或 online 成功率。釋出程式與論文對零呼叫偏好的懲罰平均分母亦有差異，詳見論文筆記。

## 出現在這些論文
- [[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]：五個預算偏好形成成本品質前緣。

## 相關概念
[[Replay simulator|歷史重播模擬器]]。
