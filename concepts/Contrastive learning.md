---
title: "Contrastive learning"
tags: [concept, llm]
aliases: ["對比學習", "InfoNCE"]
papers: ["[[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]", "[[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]"]
related: ["[[Hard negative mining]]", "[[Vision-language model]]"]
---

# Contrastive learning

> [!NOTE]
> **一句話**：訓練嵌入模型的方法：讓配對的東西（問題和答案所在的文件）向量靠近，不配對的推遠。

## 白話解釋
每筆訓練資料是一組「查詢、正例、負例」。損失函數（最常用 InfoNCE）把它當成多選題：在正例和一堆負例中，模型要讓正例的相似度最高。

$$\mathcal{L} = -\log\frac{e^{\mathrm{sim}(q,p)/\tau}}{e^{\mathrm{sim}(q,p)/\tau}+\sum_n e^{\mathrm{sim}(q,n)/\tau}}$$

$\tau$ 是溫度，控制分數差距被放大的程度。

## 負例從哪來
- **同批次負例**（in-batch）：同一批裡其他問題的正例，免費但通常太簡單
- **困難負例**：長得像但不對的文件，學習效果最好，但要小心混進其實也正確的「假負例」

## 用在哪裡
檢索用的嵌入模型（文字、圖片、截圖）、CLIP 這類圖文對齊模型、自監督表徵學習。

## 出現在這些論文
- [[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]
- [[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]

## 相關概念
[[Hard negative mining]]、[[Vision-language model]]
