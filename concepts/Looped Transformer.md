---
title: "Looped Transformer"
tags: [concept, transformer]
aliases: ["迴圈 Transformer"]
papers: ["[[【Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers】]]"]
related: ["[[Latent reasoning]]", "[[Transformer]]", "[[Chain-of-thought prompting]]"]
---

# Looped Transformer

> [!NOTE]
> 重複使用同一組 Transformer 權重，以更多計算輪數增加深度；不是每跑一輪就新增一套參數。


## 白話解釋
可以把它想成同一組計算器反覆修正工作區，而不是排出多組不同計算器。訓練仍可更新共享權重；「共享」不等於「凍結」。

## LOTUS 的工作區與深度
K 控制 block 數，c 控制每塊的位置數，R 控制 refinement 深度。固定 Kc 改 R 與固定 R 改 c 是不同實驗。

```text
h0 = f(E | prefix cache)
ht = f(E + h(t−1) | prefix cache), t=1…R
```

原始 embedding E 每輪注入。式 (2) 及公開程式都先有初始化，再做 R 次 refinement，因此工作區模型通過數是 R+1，另有 prefix／suffix／答案處理。Kc(R+1) 只是位置通過次數，不是 FLOPs 或時間估算。

## 注意力與實作邊界
同時更新多個位置，仍可以使用 causal attention；「平行」不表示每個位置能看未來。LOTUS 的概念文字與公開程式在 BoT cache 邊界及 shifted CE 等細節不同，重現時需固定程式版本，不由示意圖推斷。

## 增加深度不保證永遠改善
LOTUS Table 6 在 R=6 訓練的模型，推論 R=6 為 70.0%、R=7 為 69.3%。這是指定模型與數學題的觀察，不能直接推論任何共享迴圈模型都在第六輪最佳。

## 出現在這些論文
- [[【Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers】]]：依 v2 方法、結果與公開程式做區分。

## 來源
[Fan、Svete、Lee，LOTUS，arXiv:2606.31779v2](https://arxiv.org/html/2606.31779v2)，§3–5、Appendices C–F。此頁僅整理此來源的設計與界線，不宣稱已獨立閱讀所有蒸餾或潛在推理研究。

## 相關概念
[[Latent reasoning]]、[[Transformer]]、[[Chain-of-thought prompting]]
