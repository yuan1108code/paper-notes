---
title: "Instruction tuning"
tags: [concept, llm]
aliases: ["指令微調", "visual instruction tuning", "SFT"]
papers: ["[[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]"]
related: ["[[LoRA]]", "[[Prompt tuning]]", "[[Vision-language model]]", "[[Chain-of-thought prompting]]"]
---

# Instruction tuning

> [!NOTE]
> **一句話**：拿大量「指令 → 理想回答」的配對資料做監督式微調，讓預訓練模型學會照人的指令回答。

## 白話解釋
預訓練模型只會接龍，不一定會「回答問題」。指令微調把資料整理成「使用者：請判斷這張圖有沒有瑕疵 A. 有 B. 沒有／助理：答案是 A，因為……」這種格式，用逐字交叉熵訓練，模型就學會這種互動方式。多模態版本（visual instruction tuning，LLaVA 的做法）把影像 token 放進指令裡。

<center><b>表 1：指令微調和相近做法</b></center>

| 做法 | 改什麼 | 資料 |
|---|---|---|
| 指令微調（全參數 SFT） | 所有權重 | 指令 → 回答配對 |
| LoRA 指令微調 | 只加低秩矩陣 | 同上 |
| Prompt tuning | 只學幾個提示向量 | 同上 |
| 思維鏈資料 | 回答裡附推理過程 | 常由 GPT 生成再人工篩 |

## 要注意
- **格式會被學進去**：只有用「製程描述 + 問題」格式訓練過的模型，才會在測試時善用製程描述；其他模型看到同樣的格式反而退步（Triad 的表 1）。比較時要分清「模型懂內容」和「模型習慣格式」
- **會忘記原本的能力**：常見做法是混入一部分原本的通用指令資料（Triad 混了 12K 筆）
- **生成的資料要篩**：GPT 生成的推理過程可能有幻覺，要人工剔除

## 出現在這些論文
- [[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]

## 相關概念
[[LoRA]]、[[Prompt tuning]]、[[Vision-language model]]、[[Chain-of-thought prompting]]
