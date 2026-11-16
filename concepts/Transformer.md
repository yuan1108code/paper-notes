---
title: "Transformer"
tags: [concept, deep-learning]
aliases: ["變換器", "Transformer 架構"]
papers: ["[[【Attention is all you need】]]"]
related: ["[[Self-attention]]", "[[Positional encoding]]", "[[LoRA]]", "[[Vision-language model]]"]
---

# Transformer

> [!NOTE]
> **一句話**：完全用注意力（不用 RNN、不用卷積）組成的神經網路架構，2017 年為了機器翻譯提出，現在所有大型語言模型、嵌入模型、視覺語言模型都建立在它上面。

## 白話解釋
每一層做兩件事：(1) 自注意力：每個字看句子裡所有其他字，依相關程度吸收資訊；(2) 前饋網路：每個字各自再加工一次。兩者外面都包著殘差連接和層正規化，疊很多層。因為沒有遞迴，所有位置可以同時算；順序資訊則靠位置編碼補上。

## 三種變體
| 變體 | 結構 | 代表 | 用途 |
|---|---|---|---|
| 編碼器–解碼器 | 原始架構 | 原論文、T5、BART | 翻譯、摘要 |
| 只用編碼器 | 雙向看整句 | BERT、多數嵌入模型與重排序模型 | 分類、檢索、RAG 的向量化 |
| 只用解碼器 | 只能看前面的字 | GPT、LLaMA、Qwen | 文字生成、現在的 LLM |

## 原論文的設定（base）
6 層、$d_{model} = 512$、8 個頭、FFN 2048 維、6,500 萬參數；8 張 P100 訓練 12 小時。

> [!WARNING]
> 自注意力的計算量是長度的平方。脈絡越長越貴，這是長文件 RAG 要先檢索、篩選的根本原因。

## 出現在這些論文
- [[【Attention is all you need】]]

## 相關概念
[[Self-attention]]、[[Positional encoding]]、[[LoRA]]、[[Vision-language model]]
