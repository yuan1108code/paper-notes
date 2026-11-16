---
title: "Root cause analysis"
tags: [concept, manufacturing]
aliases: ["根因分析", "RCA"]
papers: ["[[【Interactive and Intelligent Root Cause Analysis in Manufacturing with Causal Bayesian Networks and Knowledge Graphs】]]"]
related: ["[[Causal discovery]]", "[[Knowledge graph]]"]
---

# Root cause analysis

> [!NOTE]
> **一句話**：出了問題後，沿著「誰影響誰」往回找出真正造成問題的源頭。

## 白話解釋
產線最後發現某個量測值異常、系統告警、或災情發生，表面的現象往往不是源頭。根因分析要回答：哪一個上游因素造成的？修掉哪裡才不會再發生？

## 常見方法

<center><b>表 1：根因分析方法</b></center>

| 方法 | 做法 | 特點 |
|---|---|---|
| 五個為什麼 | 對現象一直問「為什麼」直到源頭 | 簡單，靠個人經驗 |
| FMEA（失效模式與影響分析） | 事先列出每種失效的原因、後果、嚴重度 | 系統化，但要大量人力維護 |
| 故障樹分析 | 把頂層故障拆成「且／或」邏輯樹 | 適合安全分析 |
| 因果圖學習 | 用感測資料學「變數 A → B」的因果圖，沿箭頭往回找 | 能用大量資料，但會學出假關係、變數多時算不動 |
| 知識 + 資料混合 | 先用專家知識排除不可能的關係，再用資料學剩下的 | Wehner et al. 的做法 |

> [!TIP]
> 根因分析的結果最終要交給人去現場確認，所以「能解釋、能修正」比「全自動」重要：工具給候選路徑，專家判斷並回饋。

## 出現在這些論文
- [[【Interactive and Intelligent Root Cause Analysis in Manufacturing with Causal Bayesian Networks and Knowledge Graphs】]]

## 相關概念
[[Causal discovery]]、[[Knowledge graph]]
