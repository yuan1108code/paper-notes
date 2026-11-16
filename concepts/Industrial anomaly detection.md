---
title: "Industrial anomaly detection"
tags: [concept, computer-vision]
aliases: ["工業瑕疵檢測", "工業異常檢測", "IAD"]
papers: ["[[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]", "[[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]", "[[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]"]
related: ["[[AUROC]]", "[[Vision-language model]]", "[[In-context learning]]"]
---

# Industrial anomaly detection

> [!NOTE]
> **一句話**：給一張產品照片，判斷有沒有瑕疵、瑕疵在哪裡；因為瑕疵樣本稀少，通常只用正常品訓練。

## 白話解釋
工廠產線上的瓶子、螺絲、電路板，瑕疵很少見、種類又千奇百怪（刮痕、缺角、汙漬、零件裝反），很難收集到夠多的瑕疵照片來訓練分類器。所以主流做法是**只學正常品長什麼樣子**，測試時把「跟正常品不像」的地方當成瑕疵。

輸出通常有兩種：整張圖的**異常分數**（檢測），和逐像素的**熱圖**（定位）。

## 常見做法

<center><b>表 1：工業瑕疵檢測的主要做法</b></center>

| 類型 | 想法 | 代表方法 |
|---|---|---|
| 重建式 | 訓練模型把圖還原成正常品，還原誤差大的地方是瑕疵 | 自編碼器、GAN、擴散模型（AnoDDPM） |
| 特徵記憶庫 | 存下正常品每一小塊的特徵，測試時找最像的那塊，距離大就是瑕疵 | PatchCore、PaDiM、SPADE |
| 正規化流 | 把正常品的特徵壓成高斯分布，機率低就是瑕疵 | CFlow-AD、PyramidFlow |
| 圖文比對 | 用 CLIP 比較每一小塊和「完好的 X」「有瑕疵的 X」兩段文字 | WinCLIP |
| 視覺語言模型 | 讓 LLM 直接回答有沒有瑕疵、在哪裡 | AnomalyGPT |

## 常用資料集與指標
- **MVTec-AD**：15 類、測試 1,725 張；**VisA**：12 類、約 1 萬張
- **Image-AUC**（整張圖分得開正常和瑕疵嗎）、**Pixel-AUC**（每個像素分得開嗎），都不需要門檻

> [!WARNING]
> AUC 高不代表能直接上線：實際使用要選一個門檻，而各類產品的最佳門檻差很多。PatchCore 在 MVTec 的 Image-AUC 有 99.3%，用一個共用門檻時正確率只剩 79.76%（AnomalyGPT 論文的數字）。

## 出現在這些論文
- [[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]
- [[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]
- [[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]

## 相關概念
[[AUROC]]、[[Vision-language model]]、[[In-context learning]]
