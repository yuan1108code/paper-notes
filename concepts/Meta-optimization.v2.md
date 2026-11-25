---
title: "Meta-optimization"
tags: [concept, optimization, agents]
aliases: ["元優化", "後設最佳化", "優化優化器"]
papers: ["[[【Self-Taught Optimizer (STOP)：Recursively Self-Improving Code Generation】]]", "[[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]", "[[【Darwin Gödel Machine：Open-Ended Evolution of Self-Improving Agents】]]"]
related: ["[[Recursive self-improvement]]", "[[Evolutionary program search]]"]
---

# Meta-optimization

> [!NOTE]
> **一句話**：不只找更好的解法，而是把「產生、搜尋與挑選解法的程序」也當成可優化的對象。

## 白話解釋

一般優化像改良一道菜；元優化則是改良廚師用來試菜、比較與修正食譜的工作流程。工作流程變好，才可能在之後的多道菜上攤提開發成本，但這需要另外驗證，不能由一次分數上升直接推定。

## STOP 的兩層分工

<center><b>表 1：下游優化與元優化（依 STOP §3–4）</b></center>

| 層次 | 可改動對象 | 評分方式 | 保持固定的部分 |
|---|---|---|---|
| 下游 | 任務解法程式 s | 任務效用 u(s) | 評分規則與可呼叫的語言模型 |
| 外層 | 改良器程式 I | 在下游樣本上改出程式後，平均其效用 | 該次外層評估的條件、語言模型權重 |

STOP 的下游呼叫是 s′＝I(u,s,L)；外層把候選改良器放入同樣的評估程序，計算平均 u(I(u,s,L))。接著讓目前改良器改寫自己的原始碼，得到下一版。**這是改呼叫模型的程式，不是訓練模型權重。**

外層資料的平均只是對真正任務分布的估計。若反覆用同一批任務選改良器，就需要另留測試實例；如果還想宣稱跨任務轉移，則要在未用於選擇的新任務類型上檢查。

## 與 Dream-RSI 的連結

Dream-RSI 將「決定走哪條探索分支、並行多少、何時停止」的控制器獨立出來，在固定的真實探索歷史上重播比較。它與 STOP 都把搜尋程序視為可改進的產物，但 STOP 直接以改良器在下游任務上的效用做外層評分，Dream-RSI 的機制則以固定歷史重播及品質—成本目標為核心；兩者不能視為同一個演算法。

> [!WARNING]
> 下游效果較好的優化器，不一定更會改良自己。外層分數又可能有隨機性、選擇偏誤與成本，元優化不保證每輪單調進步，也不等於完整的遞迴智能成長。

## 可借用的工程設計

- 將候選解法、搜尋控制器、評分器與模型版本分開保存。
- 記錄外層開發成本與真正部署效益，不只報部署時呼叫數。
- 固定比較預算，保留舊策略作對照；對外層開發未用過的資料再驗收。
- 讓安全與評測邊界位於可自改的程式之外，避免優化器只是移除限制而拿高分。


## DGM 的對照

DGM 把可改進的 coding agent 作為下一輪修改者，但 archive 與 parent-selection 控制器仍固定；不能把 agent 效能提升說成搜尋控制器也被優化。（DGM v3 §3、附錄 C.2、H；https://arxiv.org/html/2505.22954v3 ）

## 出現在這些論文
- [[【Darwin Gödel Machine：Open-Ended Evolution of Self-Improving Agents】]]：DGM 把可改進的 coding agent 作為下一輪修改者，但 archive 與 parent-selection 控制器仍固定；不能把 agent 效能提升說成搜尋控制器也被優化。

- [[【Self-Taught Optimizer (STOP)：Recursively Self-Improving Code Generation】]]
- [[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]

## 來源

- STOP，COLM 2024，arXiv v3：https://arxiv.org/pdf/2310.02304v3 ，§3–4。
- Dream-RSI，arXiv v2：https://arxiv.org/pdf/2609.14858v2 ，§3–5。

## 相關概念

[[Recursive self-improvement|遞迴自我改良]]、[[Evolutionary program search|演化式程式搜尋]]