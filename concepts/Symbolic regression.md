---
title: "Symbolic regression"
tags: [concept, symbolic-regression, scientific-discovery]
aliases: [符號迴歸, 方程式發現]
papers: ["[[【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】]]"]
related: ["[[Evolutionary program search]]", "[[Neuro-symbolic AI]]"]
---

# Symbolic regression

> [!NOTE]
> **一句話**：不只調已知公式的係數，而是連要用哪些運算、如何組合成公式，都從資料裡尋找。

## 白話解釋

一般回歸先定好 y=ax+b，只求 a、b；符號迴歸還要決定是否使用平方、指數、相乘或分段。目標不只是讓預測準確，也希望得到可檢查且能泛化的關係。LLM-SR 將骨架提案交給固定 LLM，係數交給數值優化器，觀測誤差則用來推動結構搜尋。[1]

## 結構與係數要分開

<center><b>表 1：兩種不同的未知量</b></center>

| 層 | 例子 | 可用工具 |
|---|---|---|
| 離散結構 | 阻尼用 v 還是 v³？ | 遺傳程式設計、LLM 提案、受限文法搜尋 |
| 連續係數 | 阻尼強度是 0.2 還是 0.5？ | 最小平方、BFGS、Adam |

> [!WARNING]
> 在有限資料上，兩條不等價方程式可能都擬合得很好。低 NMSE 不等於恢復唯一物理定律；應另外測量綱、穩定性、新初值與更遠工況。LLM-SR 附錄 G 的近似恢復力就是例子。[1]

## 出現在這些論文

- [[【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】]]：領域先驗提案結構，以數值擬合與多島經驗搜尋迭代。

## 相關概念

[[Evolutionary program search|進化式程式搜尋]]、[[Neuro-symbolic AI|神經符號 AI]]

## Sources

[1] https://arxiv.org/html/2404.18400v3
