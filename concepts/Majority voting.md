---
title: "Majority voting"
tags: [concept, llm]
aliases: ["多數決", "self-consistency", "自洽性"]
papers: ["[[【A Survey on LLM-as-a-Judge】]]"]
related: ["[[LLM-as-a-judge]]", "[[Chain-of-thought prompting]]", "[[Position bias]]"]
---

# Majority voting

> [!NOTE]
> **一句話**：同一個問題讓 LLM 回答多次（或讓多個模型各答一次），取出現最多次的答案，用來抵消單次輸出的隨機性。

## 白話解釋
LLM 每次輸出可能不同。問 5 次取多數，偶發的錯誤會被其他 4 次蓋過。用在推理題時叫 self-consistency（Wang et al., 2023）：抽樣多條思維鏈，取最常出現的最終答案。

## 用在評審
Gu et al. 的實驗（以 GPT-3.5 為評審、問 5 次）：
| 彙總方式 | 結果 |
|---|---|
| 多數決 | 小幅改善（8 個面向中 4 個變好、3 個變差） |
| 平均 | 沒改善，有偏誤的結果也被算進平均 |
| 取最高分 | 8 個面向全部變差，容易挑到被偏誤灌高的分數 |

多個不同模型投票的效果取決於成員：放進一個偏誤很重的模型，整體就被拖累。

> [!WARNING]
> 多數決需要輸出有變化。temperature = 0 時同一模型每次輸出幾乎一樣，問 5 次等於問 1 次。

## 出現在這些論文
- [[【A Survey on LLM-as-a-Judge】]]

## 相關概念
[[LLM-as-a-judge]]、[[Chain-of-thought prompting]]、[[Position bias]]
