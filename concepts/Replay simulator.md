---
title: "Replay simulator"
tags: [concept, agent, exploration]
aliases: ["歷史重播模擬器"]
papers: ["[[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]"]
related: ["[[World model]]", "[[Pareto hypervolume]]"]
---

# Replay simulator

> [!NOTE]
> 把已經執行並保存結果的歷史，包成能逐步揭露觀測的環境，讓新策略便宜地試走舊路。

## 白話解釋
像拿以前爬山的路線、時間與失敗紀錄練習規劃，不必每個路線規劃都真的重爬。Dream-RSI 中的地圖是一棵發現樹，策略能選分支、深度、並行批次及停止，卻不能憑空知道未記錄路線的後果。[原論文第 3 節](https://arxiv.org/html/2609.14858v2#S3)

## 決策與評分的邊界
| 部分 | 能看什麼 |
|---|---|
| 策略決策 | 當前已揭露前綴、合法動作及允許的結構資訊 |
| 評估器 | 完整凍結世界，用於計算品質正規化與成本基準 |
| 不可混淆 | 評估器知道世界最佳值，不代表策略可偷看最佳節點 |

> [!WARNING]
> 只能評估歷史支援內的選擇。更換模型、工具、提示詞或狀態後，舊的結果未必是新動作的真實反事實；重播改善也不保證新線上探索改善。這不是任意動作的 learned dynamics model。

## 出現在這些論文
- [[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]：線上發現 → 歷史重播 → 改探索控制程式。

## 相關概念
[[World model|世界模型]]、[[Pareto hypervolume|品質成本面積]]。
