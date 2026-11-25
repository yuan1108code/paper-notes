---
title: "Open-ended search"
tags: [concept, evolutionary-search, agents]
aliases: ["開放式搜尋", "開放式探索"]
papers: ["[[【Darwin Gödel Machine：Open-Ended Evolution of Self-Improving Agents】]]", "[[【AlphaEvolve：A coding agent for scientific and algorithmic discovery】]]"]
related: ["[[Evolutionary program search]]", "[[Recursive self-improvement]]", "[[Meta-optimization]]"]
---

# Open-ended search

> [!NOTE]
> **一句話：** 不只追逐目前最好的單一解，而是保留多條可繼續發展的探索路徑；暫時低分的版本也可能成為後續改進的跳板。

## 白話解釋

只留眼前最高分，像登山只准走上坡：遇到必須先下降再爬向更高山峰的路，就會停住。開放式搜尋允許保留不同路線，之後可以回到較早的分支繼續探索。但「保留路線」不自動保證找到更高山峰，也不代表有限次實驗已證明無限創新。

## DGM 的具體做法

DGM 保存仍具程式編輯能力的 coding agent 版本及父子關係。新版本不必勝過 parent 才能留存；抽下一輪 parent 時，用成功率與有效直接 children 數共同決定機率。固定基礎模型的權重、外部搜尋控制器與 benchmark 並未一起演化。[2] §3、附錄 C.2

<center><b>表 1：保留什麼，與下一輪選誰，是兩個問題</b></center>

| 決策 | DGM 論文的做法 | 不應混淆 |
|---|---|---|
| Archive 入庫 | 編譯成功且保有基本編輯功能 | 不要求當前分數超過 parent |
| Parent 選擇 | 成功率 sigmoid × 1／(1＋有效直接 children 數)，再正規化 | 不是每次只選最高分，也不是均勻亂選 |
| 論文抽樣資格 | 排除成功率已達 1 的版本 | 公開 release 未實作同樣排除 |
| 探索偏好 | Children 較少的版本有較高未探索獎勵 | 沒有直接量測行為距離的新奇度指標 |
| 邊界 | 固定目標、固定控制器、有限候選預算 | 不等於開放世界任意自主研究 |

論文 C.2 的權重為：

$$
w_i=\frac{1}{1+\exp[-10(\alpha_i-0.5)]}\cdot\frac{1}{1+n_i},\quad
p_i=\frac{w_i}{\sum_{j\in E}w_j},\quad E=\{i:\alpha_i<1\}.
$$

$\alpha_i$ 是成功率，$n_i$ 僅計有功能的直接 children。所有有資格的版本仍有非零機率；不能推論有限成本內一定探索到它們，也不能把 Polyglot 圖表的 50 題分數當成 release 實際選 parent 的 10 題分數。[2] 附錄 C.2；[3] `DGM_outer.py`、`self_improve_step.py`

## 與其他搜尋方式比較

- <b>Greedy（貪婪選擇）</b>：只從目前最高分繼續，容易忽略要先退一步的路線。
- **只留最新可用版本**：沿單一路徑往前，無法回頭發展早期不同分支。
- **累積 archive**：保存多條可用路線，但需控制評估成本、重複候選與資源消耗。

> [!WARNING]
> Best-so-far 曲線不下降，可能只是持續保留歷史最高分，不代表每個 child 都改善。DGM 附錄 H 還展示改掉 logging markers 後取得高分的情況；搜尋目標與外部不可改的紀錄、測試及安全邊界必須分開。[2]

## 可借用的工程原則

保存候選版本、父版本、修改內容、逐題成功／失敗、評估條件與成本；允許保留有價值的低分分支，但把最後測試集與正式部署權限留在外部。這可作為 RAG 或離線工業 AI 流程的待驗證設計，不是本文已做過的實驗。


## AlphaEvolve的對照

AlphaEvolve以人定義的可計分任務保存多樣程式，借鏡MAP-Elites／island populations；它保留探索多樣性，但未展示無邊界自主選題或整個controller自改。（v1 §2–3：https://arxiv.org/html/2506.13131v1 ）

## 出現在這些論文
- [[【AlphaEvolve：A coding agent for scientific and algorithmic discovery】]]：AlphaEvolve以人定義的可計分任務保存多樣程式，借鏡MAP-Elites／island populations；它保留探索多樣性，但未展示無邊界自主選題或整個controller自改。

- [[【Darwin Gödel Machine：Open-Ended Evolution of Self-Improving Agents】]]：以版本 archive 保留 stepping stones，搜尋可遞迴自改的 coding agents。

## 相關概念

[[Evolutionary program search|演化式程式搜尋]]、[[Recursive self-improvement|遞迴自我改進]]、[[Meta-optimization|後設最佳化]]

## Sources

[2] https://arxiv.org/html/2505.22954v3
[3] https://github.com/jennyzzt/dgm