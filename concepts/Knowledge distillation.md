---
title: "Knowledge distillation"
tags: [concept, deep-learning]
aliases: ["知識蒸餾"]
papers: ["[[【Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers】]]"]
related: ["[[Latent reasoning]]", "[[Looped Transformer]]", "[[Chain-of-thought prompting]]"]
---

# Knowledge distillation

> [!NOTE]
> 以教師模型的輸出分布、隱藏狀態或其他訊號訓練學生；不同目標保留的資訊與成本不同。


## 白話解釋
不只是把正確答案交給學生，也把教師如何分配機率或形成表示當成訓練目標。學生不一定更小；蒸餾指訊號來源與學習方式，不能光憑模型大小命名。

## 與 LOTUS 直接監督的區別
- 直接 LOTUS：用資料提供的 gold CoT token，透過基礎 LM head 算交叉熵；這不是把教師 hidden state 當蒸餾目標。
- CODI：額外對齊教師 CoT 模型某個指定位置的 hidden state。
- LOTUS+CODI：同時保留直接 token 監督與教師對齊，是獨立變體。
- LOTUS-aux：用另一個共同訓練的 decoder 在 teacher forcing 下重建步驟。名稱中的 teacher forcing 指提供正確前文，不等於此路徑必然是一個凍結教師的蒸餾器。

## 比較時不可忽略的成本
教師 checkpoint 的準備、教師 forward、儲存 target、輔助 decoder 的參數與訓練記憶體都應獨立列出。「推論移除教師／aux」不等於「訓練零額外成本」。

## 讀表警告
LOTUS 3B 的 GSM8K 是 70.0%，LOTUS+CODI 為 70.6%，Explicit CoT 71.5%。「差距在 1 點內」只符合加 CODI 變體，不能把它搬到 plain LOTUS；數字是百分點差，不是相對提升百分比。

## 出現在這些論文
- [[【Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers】]]：依 v2 方法、結果與公開程式做區分。

## 來源
[Fan、Svete、Lee，LOTUS，arXiv:2606.31779v2](https://arxiv.org/html/2606.31779v2)，§3–5、Appendices C–F。此頁僅整理此來源的設計與界線，不宣稱已獨立閱讀所有蒸餾或潛在推理研究。

## 相關概念
[[Latent reasoning]]、[[Looped Transformer]]、[[Chain-of-thought prompting]]
