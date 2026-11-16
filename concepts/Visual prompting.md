---
title: "Visual prompting"
tags: [concept, computer-vision]
aliases: ["視覺提示", "visual prompt"]
papers: ["[[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]", "[[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]"]
related: ["[[Vision-language model]]", "[[Industrial anomaly detection]]"]
---

# Visual prompting

> [!NOTE]
> **一句話**：不改模型，直接在輸入圖片上畫方框、輪廓、色塊或加參考圖，引導多模態模型注意特定區域。

## 白話解釋
文字提示是在問題裡加說明；視覺提示是在**圖片本身**加提示：把要看的地方框起來、塗上半透明顏色、畫輪廓，或多放一張正常樣本當對照。

## 常見形式

<center><b>表 1：視覺提示的形式</b></center>

| 形式 | 做法 | 例子 |
|---|---|---|
| 方框（bbox） | 在目標周圍畫矩形 | MMAD 把瑕疵區框起來給 MLLM |
| 輪廓（contour） | 沿區域邊界描線 | 同上 |
| 疊色（highlight） | 把區域塗成半透明紅色 | MMAD 出題時把真實遮罩塗紅給 GPT-4V |
| 參考圖 | 多給一張正常樣本 | MMAD 的 1-shot 設定 |

## 要注意
- 提示錯了會誤導模型：MMAD 把傳統瑕疵模型的熱圖畫給 MLLM，正常圖也被畫上熱點，異常判別最多掉 12.6 個百分點
- 同樣的資訊轉成文字（「瑕疵在左上角、約占 5%」）有時比畫在圖上更有效

## 出現在這些論文
- [[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]
- [[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]

## 相關概念
[[Vision-language model]]、[[Industrial anomaly detection]]
