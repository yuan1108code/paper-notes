---
title: "Constrained decoding"
tags: [concept, llm]
aliases: ["受限解碼", "結構化輸出", "grammar-constrained decoding"]
papers: ["[[【A Survey on LLM-as-a-Judge】]]", "[[【Web World Models】]]"]
related: ["[[LLM-as-a-judge]]"]
---

# Constrained decoding

> [!NOTE]
> **一句話**：解碼時只允許產生符合指定格式（例如 JSON schema、正規表達式）的字詞，保證輸出一定能被程式解析。

## 白話解釋
請 LLM「只輸出 1 到 5 的數字」，它還是可能寫「我給 4 分，因為……」。受限解碼在每一步把不合格式的字詞機率設成 0，用有限狀態機追蹤目前在格式的哪個位置。

## 代表工具
| 工具 | 特點 |
|---|---|
| XGrammar | 把字詞分成可以事先檢查和需要執行時檢查的兩類，宣稱比既有方法快到 100 倍 |
| SGLang | 程式語言加執行環境一起設計，重複利用 KV cache |
| DOMINO | 保留自然的斷詞方式，搭配預先計算與推測解碼 |
| OpenAI／vLLM 的 structured output | API 直接指定 JSON schema |

## 用在 LLM 評審
評審的分數要自動抽出來，格式一亂整條流程就壞。受限解碼可以保證輸出 `{"score": 4}`。

> [!WARNING]
> 強制格式會扭曲模型原本的機率分布，可能降低判斷品質；先讓模型自由寫理由、最後再受限輸出分數是常見的折衷。

## 出現在這些論文
- [[【A Survey on LLM-as-a-Judge】]]
- [[【Web World Models】]]

## 相關概念
[[LLM-as-a-judge]]
