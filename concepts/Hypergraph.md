---
title: "Hypergraph"
tags: [concept, graph]
aliases: ["超圖", "超邊", "hyperedge"]
papers: ["[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]"]
related: ["[[GraphRAG]]", "[[Graph database]]"]
---

# Hypergraph

> [!NOTE]
> **一句話**：一般的圖一條邊只連兩個節點；超圖的一條「超邊」可以同時連任意多個節點，適合表示多方共同參與的一件事。

## 白話解釋
「A 公司在 D 日以 X 元收購 B 公司」牽涉四個東西：收購方、被收購方、金額、日期。
- 一般的圖要拆成好幾條兩兩相連的邊（A–收購–B、A–金額–X…），「這幾條邊是同一件收購」的資訊就不見了，除非另外加一個「事件」節點
- 超圖直接用**一條超邊**把四個節點綁在一起

## 怎麼存
超圖可以存成一張「關聯表」（incidence table）：每列是（超邊 id, 節點），用超邊 id 分組就能還原每條超邊。SAG 就是這樣：事件是超邊，實體是節點，存在 SQL 的多對多表裡。

## 和二部圖的關係
把「超邊」也當成一種節點，超圖就變成二部圖（兩種節點，邊只連不同種）。在二部圖上走「超邊 → 節點 → 超邊」兩步，就是找到共用某個節點的其他超邊。

## 出現在這些論文
- [[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]

## 相關概念
[[GraphRAG]]、[[Graph database]]
