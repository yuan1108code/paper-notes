---
title: "Latent reasoning"
tags: [concept, latent-reasoning]
aliases: ["潛在推理"]
papers: ["[[【Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers】]]"]
related: ["[[Looped Transformer]]", "[[Chain-of-thought prompting]]", "[[Knowledge distillation]]"]
---

# Latent reasoning

> [!NOTE]
> 在連續隱藏向量中承載中間計算，而不一定先把每一步解碼成文字；是否更快與更準，仍需實測。


## 白話解釋
顯式思維鏈先寫出草稿，再回答。潛在推理則把「草稿」留在模型內部向量，最後才輸出答案。向量不等於一段藏起來的完整句子，也不保證更好的推理：逐一產生 latent 仍有序列依賴。

## LOTUS 中的具體做法
LOTUS 在問題與答案間設 K 個 block、每塊 c 個 latent 位置，透過同一 Transformer 重複更新，再由連續表示生成答案。訓練用 gold CoT tokens 提供直接的逐位置監督，加上最終答案損失；推論不先解碼中間文字。

- 平行的是同輪位置計算，不是 K 條獨立、完整推理路徑。
- 讀出某些正確 token，不等於整條鏈正確或因果忠實。
- 更少序列迭代，不等於等比例降低 FLOPs、記憶體與服務延遲。
- Table 10 的替代中間值是在訓練題的未見解法上探測，不能改寫成未見題泛化成果。

## 可遷移的檢查方式
分別量測最終答案、可讀性、thought／prefill／answer 耗時與訓練成本。要宣稱某 latent 對答案重要，需另外做遮蔽或干預；相關讀出不是因果證明。

## 出現在這些論文
- [[【Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers】]]：依 v2 方法、結果與公開程式做區分。

## 來源
[Fan、Svete、Lee，LOTUS，arXiv:2606.31779v2](https://arxiv.org/html/2606.31779v2)，§3–5、Appendices C–F。此頁僅整理此來源的設計與界線，不宣稱已獨立閱讀所有蒸餾或潛在推理研究。

## 相關概念
[[Looped Transformer]]、[[Chain-of-thought prompting]]、[[Knowledge distillation]]
