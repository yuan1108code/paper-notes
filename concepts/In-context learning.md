---
title: "In-context learning"
tags: [concept, llm]
aliases: ["情境學習", "上下文學習", "few-shot prompting"]
papers: ["[[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]", "[[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]", "[[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]"]
related: ["[[Prompt tuning]]", "[[Vision-language model]]"]
---

# In-context learning

> [!NOTE]
> **一句話**：不更新模型參數，只在輸入裡放幾個範例，模型就能照著做新任務。

## 白話解釋
GPT-3 之後大家發現，大型語言模型只要在提示裡看幾個「輸入 → 輸出」範例，就能做沒訓練過的任務，完全不需要梯度更新。0 個範例叫 zero-shot，1 個叫 one-shot，幾個叫 few-shot。

## 在視覺任務的變形
工業瑕疵檢測的 few-shot 通常只給**正常品**範例（因為瑕疵難收集）。AnomalyGPT 的做法是把 1–4 張正常圖的小塊特徵存成記憶庫，查詢圖每一小塊去找最像的正常小塊，越不像越可能是瑕疵，再把這張熱圖交給 LLM 判斷。嚴格說這比較像 PatchCore 式的最近鄰比對，而不是 LLM 自己從範例中學習。

> [!WARNING]
> 範例的數量、順序、挑法都會影響結果，而且多給範例不一定變好：AnomalyGPT 在 MVTec 的正確率 1-shot 86.1%、4-shot 反而只有 85.0%。

## 出現在這些論文
- [[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]
- [[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]
- [[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]

## 相關概念
[[Prompt tuning]]、[[Vision-language model]]
