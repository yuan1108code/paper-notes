---
title: "Self-attention"
tags: [concept, deep-learning]
aliases: ["自注意力", "注意力機制", "attention", "scaled dot-product attention"]
papers: ["[[【Attention is all you need】]]"]
related: ["[[Transformer]]", "[[Positional encoding]]"]
---

# Self-attention

> [!NOTE]
> **一句話**：讓序列中的每個元素去「查詢」同一序列裡所有其他元素，依相關程度加權平均它們的內容，得到考慮了上下文的新表示。

## 白話解釋
每個字產生三個向量：Query（我要找什麼）、Key（我能提供什麼）、Value（被取用時交出的內容）。拿自己的 Query 和每個字的 Key 做內積當相關分數，softmax 成權重，再加權平均所有 Value。

$$\text{Attention}(Q,K,V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V$$

## 重點細節
- **除以 $\sqrt{d_k}$**：內積的變異數會隨維度 $d_k$ 成長，不縮放的話 softmax 會變得極端、梯度消失
- **多頭**：拆成 $h$ 組較小的 Q、K、V 各自做注意力再串接，同樣計算量下能學多種關係
- **遮罩**：生成文字時把未來位置的分數設成 $-\infty$，避免偷看答案
- **和一般注意力的差別**：一般（交叉）注意力是 A 序列看 B 序列（譯文看原文）；自注意力是序列看自己

| 比較 | RNN | 自注意力 |
|---|---|---|
| 照順序的運算 | $O(n)$ | $O(1)$ |
| 任兩個位置的距離 | $O(n)$ | $O(1)$ |
| 每層計算量 | $O(n d^2)$ | $O(n^2 d)$ |

## 出現在這些論文
- [[【Attention is all you need】]]

## 相關概念
[[Transformer]]、[[Positional encoding]]
