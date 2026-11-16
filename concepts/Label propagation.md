---
title: "Label propagation"
tags: [concept, graph]
aliases: ["標籤傳播"]
papers: ["[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]"]
related: ["[[Collaborative filtering]]", "[[Personalized PageRank]]"]
---

# Label propagation

> [!NOTE]
> **一句話**：一種半監督學習方法：圖上少數節點有標籤，讓標籤沿著邊一輪輪擴散到相近的節點，直到穩定。

## 白話解釋
社群網路裡你知道幾個人喜歡爵士樂。每一輪，每個人把朋友們的「喜好分數」平均一下當作自己的新分數（有標籤的人保持不變）。跑幾輪之後，和爵士樂迷關係越近的人，分數越高。

出處：Zhu and Ghahramani, 2002（CMU 技術報告）。

## 在 MemRec 的用法
MemRec 借用這個概念，但傳的不是數字標籤，而是**文字洞察**：使用者和某商品互動後，記憶管理模型把這次互動得到的洞察（例如「喜歡角色發展深刻的故事」）寫進相關鄰居的記憶。只傳一跳，而且整件事用一次 LLM 呼叫完成。

## 和個人化 PageRank 的差別
兩者都在圖上擴散資訊。PageRank 算的是「從起點走到各節點的機率」，標籤傳播算的是「各節點該被貼上什麼標籤」。

## 出現在這些論文
- [[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]

## 相關概念
[[Collaborative filtering]]、[[Personalized PageRank]]
