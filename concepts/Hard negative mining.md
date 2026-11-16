---
title: "Hard negative mining"
tags: [concept, llm]
aliases: ["困難負例挖掘", "假負例過濾"]
papers: ["[[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]"]
related: ["[[Contrastive learning]]"]
---

# Hard negative mining

> [!NOTE]
> **一句話**：訓練檢索模型時，刻意找出「很像正確答案、但其實不對」的例子當負例，讓模型學會分辨細微差別。

## 白話解釋
如果負例都和問題毫不相干，模型很快就學會，進步有限。困難負例的做法是：先用現有模型檢索，把排名很前面、但不是正確答案的結果拿來當負例。

## 最大的陷阱：假負例
同樣的知識常出現在好幾份文件裡。排名前面的「負例」可能其實也有答案，把它當錯的來訓練，等於懲罰模型做對的事。

## 怎麼過濾
PixelRAG 的做法：請 LLM 只看候選文件回答問題，答對的就是假負例，丟掉；答錯或無法回答的才留下。這一步帶來最大的單項進步（+4.7 點）。

## 實務提醒
- 「動態」挖掘：用目前或基礎模型即時檢索，負例會隨資料庫而變
- 每題保留少量（例如 2 個）就夠，太多容易混進雜訊

## 出現在這些論文
- [[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]

## 相關概念
[[Contrastive learning]]
