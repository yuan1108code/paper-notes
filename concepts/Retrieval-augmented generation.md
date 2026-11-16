---
title: "Retrieval-augmented generation"
tags: [concept, llm]
aliases: ["檢索增強生成", "RAG"]
papers: ["[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]", "[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]", "[[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]", "[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]", "[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]", "[[【A Survey on LLM-as-a-Judge】]]", "[[【Web World Models】]]", "[[【Attention is all you need】]]", "[[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]"]
related: ["[[BM25]]", "[[Multi-hop question answering]]", "[[Compound AI system]]", "[[Chain-of-thought prompting]]"]
---

# Retrieval-augmented generation

> [!NOTE]
> **一句話**：先從外部知識庫找出相關資料，再交給 LLM 根據這些資料回答，減少幻覺、也能用到模型沒學過或最新的知識。

## 白話解釋
LLM 的知識停在訓練那天，而且會一本正經地編造。RAG 的做法是「開書考」：先用檢索器（關鍵字檢索如 BM25，或向量檢索）從文件庫找出最相關的幾段，貼進提示詞，再請 LLM 根據這些段落回答。

## 基本流程
1. 文件切段、建索引
2. 使用者提問 → 檢索相關段落
3. 段落 + 問題 → LLM 產生答案（最好附上出處）

## 常見問題
- **一次檢索不夠**：答案要串好幾份文件時，第二份往往和原問題沒有共同字詞，搜不到。IRCoT 用「推理一句、檢索一次」解決
- 檢索到不相關的段落，反而誤導模型
- 段落太多塞不進提示詞

## 出處
Lewis et al., *Retrieval-augmented generation for knowledge-intensive NLP tasks*, NeurIPS 2020

## 出現在這些論文
- [[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]
- [[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]
- [[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]
- [[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]
- [[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]
- [[【A Survey on LLM-as-a-Judge】]]
- [[【Web World Models】]]
- [[【Attention is all you need】]]
- [[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]

## 相關概念
[[BM25]]、[[Multi-hop question answering]]、[[Compound AI system]]、[[Chain-of-thought prompting]]
