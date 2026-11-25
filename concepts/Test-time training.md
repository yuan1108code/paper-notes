---
title: "Test-time training"
tags: [concept, llm, adaptation]
aliases: ["測試時訓練", "TTT", "Test-time weight adaptation"]
papers: ["[[【Recursive Self-Improvement in AI：From Bounded Self-Refinement to Autonomous Research Loops】]]"]
related: ["[[Recursive self-improvement]]", "[[Agent memory]]", "[[LoRA]]"]
---

# Test-time training

> [!NOTE]
> **一句話**：模型已部署後，利用當前任務可取得的訓練訊號，更新部分或全部可訓練參數；不是只多想幾步或改提示詞。

## 白話解釋

三種做法看起來都像「用的時候變聰明」，改變位置卻不同：一是重新寫答案，二是把經驗存進外部記憶，三是真的調整模型參數。測試時訓練屬第三種。所用訊號可能是自監督目標、可驗證結果或其他部署時回饋，不代表有權讀取評測答案。

<center><b>表 1：三種部署階段改變的差別（依 RSI 綜述 Figure 3 整理）</b></center>

| 機制 | 改變位置 | 權重是否改變 | 保留範圍要查什麼 |
|---|---|---|---|
| 推論時答案修正／搜尋 | 本次輸出與推理軌跡 | 否 | 常限單題／單次 episode |
| 測試時訓練／權重適應 | 可訓練參數，例如部分 adapter | 是 | 本次 session 後保留還是重設 |
| 系統／harness 演化 | 提示詞、記憶、工具、技能或控制程式 | 不必改基礎權重 | 是否跨任務持久保存及回滾 |

「測試時」是部署階段，不表示每一種方法都會永久保存更新；重設規則、可讀取的資訊與計算預算必須一起說明。RSI 綜述的部署分類也包含權重適應，因此不能把「部署時演化」全部等同於「模型權重固定」。

## 評估要分開報告

- 更新前與更新後的獨立任務表現，以及新領域得益是否犧牲舊領域能力。
- 適應所用資料與最後評測資料是否隔離；不能反覆用最終測試答案選參數。
- 額外訓練步數、推論成本、記憶體、延遲與更新失敗率。
- 更新範圍、重設頻率、壞更新的偵測與回滾方式。

> [!WARNING]
> 外部記憶、較長推理與權重更新不應混報成同一機制。沒有清楚的更新紀錄與獨立驗證，不能由答案改善推定模型已學會可泛化的新能力。

## 出現在這些論文

- [[【Recursive Self-Improvement in AI：From Bounded Self-Refinement to Autonomous Research Loops】]]：在部署時演化的圖與章節中，區分答案修正、測試時權重適應及系統／harness 演化。

## 來源

M. Chen, L. Wang, and B. Qu, “Recursive self-improvement in AI: From bounded self-refinement to autonomous research loops,” 2026, arXiv:2607.07663v2, Figure 3 與部署階段相關章節。https://arxiv.org/html/2607.07663v2

## 相關概念

[[Recursive self-improvement|遞迴自我改良]]、[[Agent memory|代理人記憶]]、[[LoRA|低秩適應]]
