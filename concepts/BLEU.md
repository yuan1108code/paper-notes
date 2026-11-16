---
title: "BLEU"
tags: [concept, evaluation]
aliases: ["Bilingual Evaluation Understudy", "機器翻譯指標"]
papers: ["[[【Attention is all you need】]]"]
related: ["[[Transformer]]"]
---

# BLEU

> [!NOTE]
> **一句話**：機器翻譯最常用的自動指標：比較機器譯文和人工參考譯文有多少相同的詞組（n-gram），0–100 分，越高越好。

## 白話解釋
把譯文切成 1 到 4 個字的片段，看有多少片段也出現在參考譯文裡（精確率），取幾何平均；譯文太短會被扣分（brevity penalty），避免只譯幾個有把握的字就拿高分。

## 數字感
WMT 2014 英翻德：2017 年 Transformer big 是 28.4，之前最好的集成系統 26.36。差 1–2 BLEU 在翻譯界就算明顯進步。

> [!WARNING]
> BLEU 只看字面重疊，不理解意思。換句話說但意思正確的譯文會被扣分，所以故事生成、對話、RAG 答案這類沒有標準答案的任務很少用它，改用 LLM 評審或人工評估。

## 出現在這些論文
- [[【Attention is all you need】]]

## 相關概念
[[Transformer]]
