---
title: "Procedural generation"
tags: [concept, software]
aliases: ["程序生成", "程序化生成"]
papers: ["[[【Web World Models】]]"]
related: ["[[World model]]", "[[Neuro-symbolic AI]]"]
---

# Procedural generation

> [!NOTE]
> **一句話**：用演算法和隨機種子即時產生內容（地圖、關卡、星球），而不是事先手工做好存起來；同一個種子永遠產生同樣的結果。

## 白話解釋
《Minecraft》的世界看起來無限大，但硬碟裡沒有存整個世界，只存一個種子。走到哪裡，遊戲就用「種子＋座標」算出那裡的地形；離開再回來，算出來的還是一樣。

## 關鍵技巧
- **雜湊座標當種子**：$\text{seed} = h(x, y)$，同一地點永遠得到同一個種子
- **偽隨機數產生器**：同樣的種子產生同樣的一串「隨機」數
- **雜訊函數**（Perlin noise 等）：讓相鄰位置的值連續變化，地形才自然

## 和 LLM 的關係
Web World Model 想用雜湊種子讓 LLM 生成的內容也「每次都一樣」。但程式的偽隨機數是確定的，雲端 LLM 的輸出不是：即使設定 seed，供應商也不保證完全重現，換模型版本就更不可能。要讓 LLM 內容恆存，實際上還是得把第一次的輸出存起來。

> [!TIP]
> 想要可重現的 LLM 內容：用「輸入的雜湊」當鍵，把輸出快取起來。這是最可靠的「程序生成」。

## 出現在這些論文
- [[【Web World Models】]]

## 相關概念
[[World model]]、[[Neuro-symbolic AI]]
