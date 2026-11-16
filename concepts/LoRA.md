---
title: "LoRA"
tags: [concept, llm]
aliases: ["Low-Rank Adaptation", "低秩微調"]
papers: ["[[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]", "[[【Attention is all you need】]]", "[[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]"]
related: ["[[Vision-language model]]", "[[Contrastive learning]]"]
---

# LoRA

> [!NOTE]
> **一句話**：微調大模型時不動原本的權重，只在旁邊加一對很小的低秩矩陣來學習調整，省記憶體又省時間。

## 白話解釋
大模型的權重矩陣 $W$ 很大。LoRA 假設微調需要的改變 $\Delta W$ 可以用兩個小矩陣相乘表示：$\Delta W = BA$，其中 $A$、$B$ 的秩（rank）很小，例如 8 或 16。訓練時只更新 $A$、$B$，原本的 $W$ 凍結。

## 好處
- 要訓練的參數通常不到原本的 1%
- 一張 GPU 就能微調大模型
- 不同任務可以各存一組小小的 LoRA 權重，切換方便

## 要調哪些部分
VLM 有視覺編碼器和語言模型兩塊。過去的文件檢索研究（ColPali）發現只調語言模型比較好；PixelRAG 發現網頁截圖彼此長得很像，**連視覺編碼器一起調**反而更好（+2.3 點）。

## 出處
Hu et al., *LoRA: Low-rank adaptation of large language models*, ICLR 2022

## 出現在這些論文
- [[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]
- [[【Attention is all you need】]]
- [[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]

## 相關概念
[[Vision-language model]]、[[Contrastive learning]]
