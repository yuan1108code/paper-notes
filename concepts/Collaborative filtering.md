---
title: "Collaborative filtering"
tags: [concept, recommender-system]
aliases: ["協同過濾", "CF"]
papers: ["[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]"]
related: ["[[Label propagation]]", "[[NDCG]]", "[[Agent memory]]"]
---

# Collaborative filtering

> [!NOTE]
> **一句話**：推薦系統最經典的想法：「和你品味相似的人喜歡的東西，你可能也會喜歡」，只看誰和什麼互動過，不需要理解商品內容。

## 白話解釋
把所有使用者和商品排成一張大表，格子裡是評分或有沒有互動。A 和 B 買過很多相同的書，B 買了一本 A 沒買過的書，就推薦給 A。

## 演變
| 做法 | 例子 |
|---|---|
| 鄰居式 | User-based CF、Item-based CF（Sarwar et al., 2001） |
| 矩陣分解 | 把大表拆成使用者向量 × 商品向量（Koren et al., 2009） |
| 深度學習 | NCF、序列模型 SASRec |
| 圖模型 | LightGCN：在使用者–商品互動圖上傳遞向量，捕捉多跳關聯 |
| LLM 代理人 | MemRec：把鄰居的文字記憶濃縮成偏好面向，交給 LLM 推理 |

## 常見問題
- **冷啟動**：新使用者、新商品沒有互動紀錄
- **稀疏**：大部分格子是空的（MemRec 論文中 Books 的密度只有 0.0233%）

## 出現在這些論文
- [[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]

## 相關概念
[[Label propagation]]、[[NDCG]]、[[Agent memory]]
