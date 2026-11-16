---
title: "Prompt optimization"
tags: [concept, llm]
aliases: ["提示詞優化", "自動提示詞優化"]
papers: ["[[【Optimizing generative AI by backpropagating language model feedback】]]"]
related: ["[[LLM-as-a-judge]]", "[[Chain-of-thought prompting]]", "[[Backpropagation]]"]
---

# Prompt optimization

> [!NOTE]
> **一句話**：不靠人工反覆試，而是用演算法自動找出讓 LLM 表現最好的提示詞。

## 白話解釋
同一個模型，提示詞寫法不同，正確率可以差很多。提示詞優化把「提示詞」當成要調的參數，用一小批訓練例子打分數，自動改寫、篩選，找出對新題目也有效的版本。

## 常見做法
| 方法 | 怎麼做 |
|---|---|
| OPRO | 把過去的提示詞和分數給 LLM 看，請它提出新的候選提示詞 |
| DSPy | 把 LLM 程式模組化，自動挑選放進提示詞的示範例子並搜尋組合 |
| TextGrad | LLM 先寫評語（文字梯度）說明哪裡不好，再依評語改寫提示詞，可同時優化指令和示範 |

> [!WARNING]
> 要留驗證集：只有在驗證集變好時才接受新提示詞，否則容易只對訓練例子有效。

## 出現在這些論文
- [[【Optimizing generative AI by backpropagating language model feedback】]]

## 相關概念
[[LLM-as-a-judge]]、[[Chain-of-thought prompting]]、[[Backpropagation]]
