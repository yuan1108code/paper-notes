---
title: "Neuro-symbolic AI"
tags: [concept, ai]
aliases: ["神經符號 AI", "神經–符號混合"]
papers: ["[[【Web World Models】]]", "[[【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】]]"]
related: ["[[World model]]", "[[Constrained decoding]]", "[[Procedural generation]]"]
---

# Neuro-symbolic AI

> [!NOTE]
> **一句話**：把神經網路（擅長模糊的理解和生成）和符號系統（規則、邏輯、程式，擅長精確和一致）結合起來，各做自己擅長的部分。

## 白話解釋
LLM 會寫故事、懂語意，但算錯數字、前後矛盾；程式和規則不會出錯，但只能處理事先想到的情況。神經符號的做法是讓兩者分工：例如程式維護遊戲狀態、檢查規則，LLM 負責寫描述和發明新內容，再由程式檢查 LLM 的輸出是否合法。

## 常見的分工方式
| 方式 | 例子 |
|---|---|
| 程式管狀態，LLM 寫敘述 | Web World Model 的物理層／想像層 |
| LLM 產生結構化輸出，程式執行 | AI Spire 的卡片效果代碼、LLM 寫 SQL 或程式碼再執行 |
| 用知識圖譜追蹤狀態 | 文字冒險遊戲中用圖譜記錄世界變化 |
| LLM 寫規則，程式套用 | MemRec 讓 LLM 離線寫篩選規則 |

> [!TIP]
> 實務上最有用的原則：會被追究對錯的事（數字、規則、權限）交給程式，需要彈性和語言能力的事交給 LLM，中間用型別或 schema 當契約。

## 出現在這些論文
- [[【Web World Models】]]

- [[【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】]]：LLM 提案符號／程式結構，數值優化擬合係數，再用觀測誤差篩選。

## 相關概念
[[World model]]、[[Constrained decoding]]、[[Procedural generation]]
