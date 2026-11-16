---
title: "Causal discovery"
tags: [concept, causal-inference]
aliases: ["因果發現", "因果結構學習", "causal structure learning"]
papers: ["[[【Interactive and Intelligent Root Cause Analysis in Manufacturing with Causal Bayesian Networks and Knowledge Graphs】]]"]
related: ["[[Root cause analysis]]", "[[Knowledge graph]]"]
---

# Causal discovery

> [!NOTE]
> **一句話**：從觀察資料推出變數之間「誰是因、誰是果」的有向圖，而不只是相關。

## 白話解釋
相關不代表因果：冰淇淋銷量和溺水人數相關，是因為天氣熱同時影響兩者。**因果發現**想從資料推出一張有向無環圖（DAG），箭頭 A → B 表示「改變 A、其他不變，B 也會變」。這張圖加上機率就是**因果貝氏網路**。

## 三大難題
1. **混淆**：第三個變數同時影響兩者，造成假關係
2. **方向**：只看相關分不出誰影響誰
3. **組合爆炸**：可能的圖數量隨變數數超指數成長

## 主要方法

<center><b>表 1：因果發現方法</b></center>

| 類型 | 代表 | 想法 |
|---|---|---|
| 條件獨立檢定 | PC、FCI | 一對變數在控制其他變數後若獨立，就沒有直接邊 |
| 結構方程模型（SEM） | LiNGAM、CAM、NOTEARS | 每個變數 = 原因的函數 + 雜訊，找最能解釋資料的圖 |
| 加入背景知識 | 時間先後、禁止邊、根／葉變數 | 先排除不可能的邊，大幅縮小搜尋空間 |

**CAM（因果加性模型）**：每個原因各自一個非線性函數再相加，先找變數的因果順序（每個變數只用排在前面的去預測，殘差越小越好），再用迴歸挑出真正的邊。

## 怎麼評估
**結構漢明距離（SHD）**：學到的圖要加、刪、反轉幾條邊才會變成真實的圖，越小越好。真實世界通常沒有完整的真實圖，只能和部分已知的關係比，這是應用論文最常見的弱點。

> [!WARNING]
> 「學到的邊變少」不等於「假的邊變少」：沒有真實因果圖時，少掉的也可能是真的關係。

## 出現在這些論文
- [[【Interactive and Intelligent Root Cause Analysis in Manufacturing with Causal Bayesian Networks and Knowledge Graphs】]]

## 相關概念
[[Root cause analysis]]、[[Knowledge graph]]
