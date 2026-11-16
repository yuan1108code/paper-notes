---
title: "Meta-evaluation"
tags: [concept, evaluation]
aliases: ["後設評估", "評估評審"]
papers: ["[[【A Survey on LLM-as-a-Judge】]]"]
related: ["[[LLM-as-a-judge]]", "[[Position bias]]"]
---

# Meta-evaluation

> [!NOTE]
> **一句話**：評估「評審」本身準不準：拿有人工標註的資料，看自動評估方法（指標或 LLM 評審）和人的判斷有多一致、有沒有系統性偏誤。

## 白話解釋
用 LLM 評審給 RAG 答案打分之前，要先知道這個評審可不可信。做法是準備一批人已經判斷過的例子，讓評審也判一次，比較兩者。

## 常用指標
| 指標 | 意思 |
|---|---|
| 一致率（percentage agreement） | 評審和人結論相同的比例 |
| Cohen's kappa | 扣掉「碰巧一致」之後的一致程度，0 代表跟亂猜一樣 |
| Spearman 相關 | 評審的分數排序和人的排序有多像 |
| 位置一致性 | 兩個答案對調順序後，評審是否選同一個 |
| 偏誤準確率 | 在刻意設計的陷阱題（例如長但沒內容的答案）上選對的比例 |

## 常用資料集
MTBench、Chatbot Arena、LLMEval²（2,553 題兩兩比較）、EvalBiasBench（6 種偏誤、80 題）、CALM（12 種偏誤）。

> [!TIP]
> 實務上不需要大資料集：50–100 題自己領域的人工標註，就足以比較幾個候選評審、抓出明顯的偏誤。

> [!WARNING]
> 題數少時，一題就差好幾個百分點（24 題時一題 = 4.17 個百分點）。比較策略時要看差了幾題，不要只看百分比。

## 出現在這些論文
- [[【A Survey on LLM-as-a-Judge】]]

## 相關概念
[[LLM-as-a-judge]]、[[Position bias]]
