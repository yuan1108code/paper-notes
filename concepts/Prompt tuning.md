---
title: "Prompt tuning"
tags: [concept, llm]
aliases: ["提示微調", "軟提示", "soft prompt"]
papers: ["[[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]"]
related: ["[[LoRA]]", "[[Vision-language model]]", "[[Prompt optimization]]"]
---

# Prompt tuning

> [!NOTE]
> **一句話**：不改模型權重，只學幾個接在輸入前面的「虛擬 token」向量，讓凍結的模型適應新任務。

## 白話解釋
一般的提示是人寫的文字；prompt tuning 的「提示」是一串**可訓練的向量**，長度跟 token 嵌入一樣，接在真正的輸入前面。訓練時模型本身完全凍結，只有這幾個向量會更新。可以想成：學一段人類看不懂、但模型很受用的「隱形提示」。

## 和其他微調方式比較

<center><b>表 1：微調方式比較</b></center>

| 方式 | 改什麼 | 可訓練參數 | 特點 |
|---|---|---|---|
| 全參數微調 | 所有權重 | 全部 | 效果好但貴，小資料容易過擬合、忘記原本能力 |
| LoRA | 在權重旁加低秩矩陣 | 約 0.1–1% | 現在最常用的省參數微調 |
| Prompt tuning | 只學輸入前面的幾個向量 | 極少 | 最省；模型原本的能力完全保留 |
| 人工提示／提示優化 | 文字 | 0 | 不用訓練，但只能用文字表達 |

## 延伸：讓向量帶資訊
AnomalyGPT 的提示向量分兩種：一種是固定可學的（基礎提示），另一種由小 CNN 從瑕疵熱圖**算出來**，所以每張圖的提示都不一樣，等於把另一個模型的輸出「翻譯」成 LLM 的輸入。

> [!TIP]
> 在 AnomalyGPT 的消融實驗中，同樣的架構改用 LoRA 微調 LLM，MVTec 正確率從 93.3% 掉到 83.9%：資料只有幾千張時，動越少參數越不容易壞。

## 出現在這些論文
- [[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]

## 相關概念
[[LoRA]]、[[Vision-language model]]、[[Prompt optimization]]
