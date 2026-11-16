---
title: "NDCG"
tags: [concept, evaluation]
aliases: ["Normalized Discounted Cumulative Gain", "Hit@K", "命中率"]
papers: ["[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]"]
related: ["[[Recall@k]]", "[[Collaborative filtering]]"]
---

# NDCG

> [!NOTE]
> **一句話**：衡量排序品質的指標：正確答案排得越前面分數越高，第 1 名得滿分，往後依對數遞減。常和 Hit@K（正確答案有沒有進前 K 名）一起看。

## 白話解釋
只有一個正確答案時：
- **Hit@K**：正確答案排在前 K 名就算 1，否則 0
- **NDCG@K**：排在第 r 名（r ≤ K）得 $1/\log_2(r+1)$ 分，所以第 1 名 1 分、第 2 名 0.63、第 3 名 0.5、第 5 名 0.39

| 名次 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| NDCG 得分 | 1.00 | 0.63 | 0.50 | 0.43 | 0.39 |

## 實用的檢查
只有一個正確答案時，一定有 $\text{Hit@1} \le \text{NDCG@K} \le \text{Hit@K}$，而且 NDCG@5 − NDCG@3 不會超過 (Hit@5 − Hit@3) × 0.43。MemRec 論文的主表用這個方法檢查，只有 Yelp 的 P5 那一格不成立（應是筆誤）。

## 隨機基準
10 個候選隨便排：Hit@1 = 0.1、Hit@3 = 0.3、Hit@5 = 0.5。看推薦論文的數字時要記得這條底線。

## 出現在這些論文
- [[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]

## 相關概念
[[Recall@k]]、[[Collaborative filtering]]
