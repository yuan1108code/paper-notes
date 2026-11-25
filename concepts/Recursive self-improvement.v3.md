---
title: "Recursive self-improvement"
tags: [concept, llm, self-improvement]
aliases: ["遞迴自我改良", "RSI", "遞迴自我改善"]
papers: ["[[【Recursive Self-Improvement in AI：From Bounded Self-Refinement to Autonomous Research Loops】]]", "[[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]", "[[【Self-Taught Optimizer (STOP)：Recursively Self-Improving Code Generation】]]", "[[【Darwin Gödel Machine：Open-Ended Evolution of Self-Improving Agents】]]"]
related: ["[[Agent memory]]", "[[LLM-as-a-judge]]", "[[Reward model]]", "[[Test-time training]]"]
---

# Recursive self-improvement

> [!NOTE]
> **一句話**：讓一次改良的成果被保存，並用於下一輪改良的系統；必須分清楚改的是答案、工具與控制程式、模型權重，還是研究方向本身。

## 白話解釋

改寫同一份答案三次，是一次任務內的反覆修正。把成功做法存成可重用的技能或控制程式，才可能讓下次任務受益；更新權重又是不同層次。如果系統連「用什麼標準判好壞」都能改寫，就更需要檢查它是否只是學會討好自己的評審。

RSI 綜述把有邊界的自我修正與更強的自主研究迴圈放在同一張地圖，**不是宣稱所有迴圈已達到無限加速、可自主選定正確科學目標的強 RSI**。

## 三個拆解問題

<center><b>表 1：判斷「自我改良」究竟改了什麼</b></center>

| 問題 | 要記錄的證據 | 常見混淆 |
|---|---|---|
| 哪裡改變？ | 答案、記憶／harness、模型權重、評估器或研究流程的版本 | 存了 Memory 就說訓練模型 |
| 保留多久？ | 單題、單次測試階段、跨任務的持久版本 | 本題變好就說系統永久進步 |
| 誰決定接受？ | 人工每輪批准、人工監督整體、機器依固定標準閉環 | 機器閉環就說目標也是自己選的 |

## 驗證比「再想一次」更重要

- 形式驗證器與執行結果提供可檢查訊號，但只能保證各自形式化／測試覆蓋範圍。
- 模型評審的分數是代理訊號；共享錯誤可能在生成者與評審之間被放大。
- 保留原版本與完整成本，在未用於調整的資料上驗證；同一歷史池不退步，不等於未來任務不退步。
- 人類選定目標與可接受代價，是工具驗證之前的問題；證明解法通過測試，不等於問題值得做。

> [!WARNING]
> 「遞迴」描述回饋結構，不是效能承諾。固定預算、狹窄環境中的改善，不能直接外推成開放世界自主研究能力。


## DGM 的對照

DGM 保存可繼續修改自己的 agent 版本。固定 FM 權重、固定外部搜尋控制下，暫時退步的版本仍可成為跳板；附錄的 logging-marker hacking 也說明評分紀錄須在可改程式之外。（DGM v3 §3、附錄 C.2、H；https://arxiv.org/html/2505.22954v3 ）

## 出現在這些論文
- [[【Darwin Gödel Machine：Open-Ended Evolution of Self-Improving Agents】]]：DGM 保存可繼續修改自己的 agent 版本。固定 FM 權重、固定外部搜尋控制下，暫時退步的版本仍可成為跳板；附錄的 logging-marker hacking 也說明評分紀錄須在可改程式之外。
- [[【Self-Taught Optimizer (STOP)：Recursively Self-Improving Code Generation】]]：用固定語言模型改寫改良器程式，並以改良下游程式的效用作外層評分。

- [[【Recursive Self-Improvement in AI：From Bounded Self-Refinement to Autonomous Research Loops】]]：以改變位置與人類參與程度整理文獻，並討論驗證、崩塌、研究方向與安全限制。
- [[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]：在固定探索歷史上比較控制程式，再回真實探索驗證；改的是控制策略，不是 LLM 權重。

## 來源

- M. Chen, L. Wang, and B. Qu, “Recursive self-improvement in AI: From bounded self-refinement to autonomous research loops,” 2026, arXiv:2607.07663v2. https://arxiv.org/html/2607.07663v2
- Dream-RSI：https://arxiv.org/abs/2609.14858

## 相關概念

[[Agent memory|代理人記憶]]、[[LLM-as-a-judge|模型評審]]、[[Reward model|獎勵模型]]、[[Test-time training|測試時訓練]]