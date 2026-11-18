---
title: "World model"
tags: [concept, agent]
aliases: ["世界模型"]
papers: ["[[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]", "[[【Web World Models】]]"]
related: ["[[Neuro-symbolic AI]]", "[[Procedural generation]]", "[[Agent memory]]"]
---

# World model

> [!NOTE]
> **一句話**：能預測「做了某個動作之後，世界會變成什麼樣子」的模型，讓代理人可以在腦中先模擬、再行動。

## 白話解釋
人在過馬路前會先想像車子會開到哪裡，這就是一個內在的世界模型。AI 的世界模型做同樣的事：輸入目前狀態和動作，輸出下一個狀態。

## 幾種做法
| 做法 | 例子 | 特性 |
|---|---|---|
| 學出來的神經網路 | Ha & Schmidhuber（2018）的 World Models、Dreamer | 代理人在「夢」裡學策略 |
| 預測抽象表示 | LeCun 的 JEPA（I-JEPA、V-JEPA） | 不預測像素，只預測重點 |
| 影片／3D 生成 | Genie、擴散模型做的互動遊戲 | 畫面豐富，但難控制、會前後矛盾 |
| 用 LLM 模擬 | WebDreamer（模擬網頁操作的結果）、RAP（LLM 同時當世界模型和推理者） | 利用 LLM 的常識 |
| 程式＋LLM 混合 | Web World Model | 規則寫在程式裡，LLM 只補內容；可控但規則要人寫 |

> [!WARNING]
> 「世界模型」這個詞被用得很寬：有人指可以拿來規劃的預測模型，有人只是指一個可以互動的生成環境。讀論文時要先確認它指的是哪一種。

## 歷史重播的特例
Dream-RSI 把已執行的探索樹當 replay world，只揭露已記錄結果，不預測未探索動作。它借用世界模型的「先在環境裡評估策略」想法，但不是學出的生成式 dynamics model。[論文](https://arxiv.org/html/2609.14858v2)

## 出現在這些論文
- [[【Web World Models】]]

- [[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]

## 相關概念
[[Neuro-symbolic AI]]、[[Procedural generation]]、[[Agent memory]]
