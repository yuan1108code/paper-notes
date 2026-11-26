---
title: "Chain-of-thought prompting"
tags: [concept, llm]
aliases: ["思維鏈提示", "CoT"]
papers: ["[[【Optimizing generative AI by backpropagating language model feedback】]]", "[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]", "[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]", "[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]", "[[【A Survey on LLM-as-a-Judge】]]", "[[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]", "[[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]", "[[【Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers】]]"]
related: ["[[Prompt optimization]]", "[[Latent reasoning]]", "[[Looped Transformer]]", "[[Knowledge distillation]]"]
---

# Chain-of-thought prompting

> [!NOTE]
> **一句話**：在提示詞裡要求模型「一步一步想」，先寫出推理過程再給答案，通常能提升推理題的正確率。

## 白話解釋
直接要答案時，模型容易跳步出錯。要求它先寫出中間步驟，等於讓它有「打草稿」的空間。最簡單的版本只要在提示詞加一句「Let's think step by step」，叫做零樣本 CoT。

## 在論文裡的角色
常被當作強基準。TextGrad 的提示詞優化就是從零樣本 CoT 提示詞出發，再自動改寫成更具體的步驟指示。

## 出處
Wei et al., *Chain-of-thought prompting elicits reasoning in large language models*, NeurIPS 2022

## 出現在這些論文
- [[【Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers】]]：LOTUS 的平行 latent 工作區、共享迴圈與 gold CoT 監督；與只改提示詞不同。
- [[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]
- [[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]
- [[【Optimizing generative AI by backpropagating language model feedback】]]
- [[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]
- [[【A Survey on LLM-as-a-Judge】]]
- [[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]
- [[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]

## 相關概念
[[Prompt optimization]]

[[Latent reasoning]]、[[Looped Transformer]]、[[Knowledge distillation]]
