---
title: "Backpropagation"
tags: [concept, llm]
aliases: ["反向傳播"]
papers: ["[[【Optimizing generative AI by backpropagating language model feedback】]]", "[[【Attention is all you need】]]"]
related: ["[[Prompt optimization]]", "[[Compound AI system]]"]
---

# Backpropagation

> [!NOTE]
> **一句話**：從輸出端的誤差往回推，算出每個參數該往哪個方向調、調多少，是訓練神經網路的核心方法。

## 白話解釋
模型猜錯了，誤差有多大用損失函數量化。反向傳播用微積分的連鎖律，從最後一層開始往前，一層一層算出「這個參數稍微變一下，損失會變多少」（梯度）。接著用梯度下降，把每個參數往讓損失變小的方向挪一點，反覆做就是訓練。

## 前提
每個元件都要可微分（例如矩陣乘法、激活函數），而且要拿得到參數。

## 延伸：文字版的反向傳播
TextGrad 把同樣的流程搬到 LLM 系統：元件之間傳的是文字，梯度換成 LLM 寫的評語，「更新參數」換成 LLM 依評語改寫文字。

> [!TIP]
> 出處：Rumelhart, Hinton & Williams, *Learning representations by back-propagating errors*, Nature 1986

## 出現在這些論文
- [[【Optimizing generative AI by backpropagating language model feedback】]]
- [[【Attention is all you need】]]

## 相關概念
[[Prompt optimization]]、[[Compound AI system]]
