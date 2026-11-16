---
title: "Vision-language model"
tags: [concept, llm]
aliases: ["視覺語言模型", "VLM"]
papers: ["[[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]", "[[【Attention is all you need】]]", "[[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]", "[[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]", "[[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]"]
related: ["[[Retrieval-augmented generation]]", "[[Contrastive learning]]", "[[LoRA]]"]
---

# Vision-language model

> [!NOTE]
> **一句話**：同時看得懂圖片和文字的模型：把圖片轉成一串視覺 token，和文字一起交給語言模型處理。

## 白話解釋
典型架構有三塊：
1. **視覺編碼器**（常用 ViT）：把圖片切成小方塊（patch），每塊轉成向量
2. **投影層**：把視覺向量轉成語言模型看得懂的格式，常會合併相鄰的方塊來減少 token
3. **語言模型**：把視覺 token 和文字 token 放在一起，產生回答

## 能做什麼
看圖回答問題、OCR 讀字、讀表格和圖表、看文件截圖。近年進步很快，新一代模型讀截圖的能力已經能和直接讀文字相比。

## 和成本的關係
很多 VLM 支援動態解析度：圖片越大，視覺 token 越多。所以把圖片縮小就能省 token，這是純文字模型沒有的調整方式。

常見模型：Qwen-VL 系列、LLaVA、Llama-Vision、GPT-4o、Gemini。

## 出現在這些論文
- [[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]
- [[【Attention is all you need】]]
- [[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]
- [[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]
- [[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]

## 相關概念
[[Retrieval-augmented generation]]、[[Contrastive learning]]、[[LoRA]]
