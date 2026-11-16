---
title: "Positional encoding"
tags: [concept, deep-learning]
aliases: ["位置編碼", "位置嵌入", "positional embedding"]
papers: ["[[【Attention is all you need】]]"]
related: ["[[Transformer]]", "[[Self-attention]]"]
---

# Positional encoding

> [!NOTE]
> **一句話**：自注意力本身不知道字的順序，位置編碼把「第幾個位置」變成向量加進輸入，讓模型分得出「狗咬人」和「人咬狗」。

## 白話解釋
原始 Transformer 用不同頻率的正弦和餘弦波，每一維一條波：低維變化快、高維變化慢，像時鐘的秒針、分針、時針，組合起來能唯一表示每個位置。

$$PE_{(pos,2i)} = \sin\!\left(\frac{pos}{10000^{2i/d}}\right),\quad PE_{(pos,2i+1)} = \cos\!\left(\frac{pos}{10000^{2i/d}}\right)$$

## 幾種做法
| 做法 | 例子 | 特性 |
|---|---|---|
| 正弦（固定） | 原始 Transformer | 不用學；相對位移可以寫成線性轉換 |
| 可學習的絕對位置 | BERT、GPT-2 | 原論文實驗中和正弦幾乎一樣，但無法超過訓練長度 |
| 旋轉位置編碼（RoPE） | LLaMA、Qwen | 把位置轉成向量旋轉，直接編碼相對距離，現在的主流 |
| ALiBi | BLOOM | 依距離直接扣注意力分數，外推性較好 |

> [!TIP]
> 嵌入模型和 LLM 的最大長度由位置編碼決定，超過就會被截斷。做 RAG 切段落時要先查清楚模型的上限。

## 出現在這些論文
- [[【Attention is all you need】]]

## 相關概念
[[Transformer]]、[[Self-attention]]
