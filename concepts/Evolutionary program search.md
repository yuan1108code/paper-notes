---
title: "Evolutionary program search"
tags: [concept, program-search, optimization]
aliases: [進化式程式搜尋, 演化程式搜尋]
papers: ["[[【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】]]", "[[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]", "[[【Self-Taught Optimizer (STOP)：Recursively Self-Improving Code Generation】]]", "[[【Darwin Gödel Machine：Open-Ended Evolution of Self-Improving Agents】]]", "[[【AlphaEvolve：A coding agent for scientific and algorithmic discovery】]]"]
related: ["[[Symbolic regression]]", "[[In-context learning]]"]
---

# Evolutionary program search

> [!NOTE]
> **一句話**：反覆提出可執行程式、以明確評估器打分、保存有用候選，再利用過去候選提出下一批，而不是只相信文字上的自我評價。

## 白話解釋

把「寫一個答案」改成有族群與回饋的探索。候選可能是方程式，也可能是搜尋控制器。LLM 可以是突變／組合候選的提案器；真正決定好壞的是外部評估器。改變的是程式與候選分布，不必更新 LLM 權重。[1]

## 在 LLM-SR 裡如何運作

<center><b>表 1：搜尋閉環</b></center>

| 階段 | LLM-SR 的實際對象 |
|---|---|
| 提案 | 帶佔位係數的方程式骨架 |
| 數值擬合 | BFGS／Adam 對觀測求係數 |
| 評分 | 負 MSE，越接近 0 越好 |
| 記憶 | 多個搜尋島、同分候選群集 |
| 下一輪 | 抽島、依分數抽群、偏好較短同分程式，放兩個例子進提示 |

釋出程式登錄所有有效候選，再另外追蹤各島最佳值；不是只留全域冠軍。多島與抽樣聯合消融不能拆成島數單一因素的因果證明。[1][3]

> [!WARNING]
> 執行候選不等於安全：timeout 不會自動封鎖網路、檔案存取或記憶體濫用。還要防止評估器漏洞、反覆使用同一驗證集的過擬合，以及把不同方法的 iteration 當成等價總成本。[3]


## DGM 的對照

DGM 用成功率 sigmoid 與有效 children 數的倒數抽 parent，保留有編輯能力但不一定高於 parent 的版本；這是譜系探索偏好，不等於量測了行為新奇度。（DGM v3 §3、附錄 C.2、H；https://arxiv.org/html/2505.22954v3 ）


## AlphaEvolve的對照

AlphaEvolve演化完整程式檔、多函式與可選meta-prompts，資料庫借鏡MAP-Elites與islands；精確選樣與收錄規則未公開，不能借用其他系統公式冒稱其實作。（v1 §2–3：https://arxiv.org/html/2506.13131v1 ）

## 出現在這些論文
- [[【AlphaEvolve：A coding agent for scientific and algorithmic discovery】]]：AlphaEvolve演化完整程式檔、多函式與可選meta-prompts，資料庫借鏡MAP-Elites與islands；精確選樣與收錄規則未公開，不能借用其他系統公式冒稱其實作。
- [[【Darwin Gödel Machine：Open-Ended Evolution of Self-Improving Agents】]]：DGM 用成功率 sigmoid 與有效 children 數的倒數抽 parent，保留有編輯能力但不一定高於 parent 的版本；這是譜系探索偏好，不等於量測了行為新奇度。
- [[【Self-Taught Optimizer (STOP)：Recursively Self-Improving Code Generation】]]：用固定語言模型改寫改良器程式，並以改良下游程式的效用作外層評分。

- [[【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】]]：搜尋方程式骨架，數值優化擬合係數。
- [[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]：搜尋探索控制程式，以歷史重播評估；優化對象與 LLM-SR 不同，並非同方法。

## 相關概念

[[Symbolic regression|符號迴歸]]、[[In-context learning|情境學習]]

## Sources

[1] https://arxiv.org/html/2404.18400v3
[3] https://github.com/deep-symbolic-mathematics/LLM-SR