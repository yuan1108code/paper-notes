---
title: "AUROC"
tags: [concept, evaluation]
aliases: ["ROC 曲線下面積", "AUC", "Image-AUC", "Pixel-AUC"]
papers: ["[[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]"]
related: ["[[Industrial anomaly detection]]"]
---

# AUROC

> [!NOTE]
> **一句話**：把所有可能的門檻都試一遍，看模型把正例排在負例前面的能力；0.5 是亂猜，1 是完美。

## 白話解釋
模型輸出分數後，要選門檻才能判斷「是／否」。ROC 曲線把每個門檻下的**真陽性率**（瑕疵被抓到的比例）對**偽陽性率**（正常品被誤判的比例）畫出來，曲線下面積就是 AUROC（常簡稱 AUC）。

等價的直覺：隨機抽一個正例、一個負例，模型給正例的分數比較高的機率。

## 優點和盲點
- **不用選門檻**、不受正負比例影響，適合比較不同模型的「排序能力」
- 但它**不告訴你實際用時要設多少門檻**，也不保證任何單一門檻下的正確率好

<center><b>表 1：瑕疵檢測常見的兩種 AUC</b></center>

| 名稱 | 一個樣本是什麼 | 衡量什麼 |
|---|---|---|
| Image-AUC | 一張圖（常用熱圖最大值當分數） | 檢測：整張圖有沒有瑕疵 |
| Pixel-AUC | 一個像素 | 定位：瑕疵畫得準不準 |

> [!WARNING]
> Pixel-AUC 被大量正常像素主導，瑕疵很小時數字容易偏高；很多論文會另外報 PRO 或 AP。

## 出現在這些論文
- [[【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】]]

## 相關概念
[[Industrial anomaly detection]]
