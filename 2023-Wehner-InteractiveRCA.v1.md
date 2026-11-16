---
title: "【Interactive and Intelligent Root Cause Analysis in Manufacturing with Causal Bayesian Networks and Knowledge Graphs】"
tags: [paper, causal-inference, knowledge-graph, manufacturing, status-inbox]
concepts: ["[[Root cause analysis]]", "[[Causal discovery]]", "[[Knowledge graph]]", "[[Graph database]]"]
papers: ["[[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]"]
---

# 【Interactive and Intelligent Root Cause Analysis in Manufacturing with Causal Bayesian Networks and Knowledge Graphs】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 作者 | Christoph Wehner（班貝格大學認知系統組）、Maximilian Kertel（BMW 電池芯技術開發）、Judith Wewerka（BMW 數位化與創新） |
| 發表 | 2023 IEEE 97th Vehicular Technology Conference（VTC2023-Spring），義大利佛羅倫斯，2023 年 6 月，頁 1–7。DOI [10.1109/VTC2023-Spring57618.2023.10199563](https://doi.org/10.1109/VTC2023-Spring57618.2023.10199563) |
| 版本 | 你給的 IEEE 連結需要訂閱，我連不進去（IEEE 擋自動存取），所以本筆記依作者放在 arXiv 的版本（2402.00043v1，2024/1/20，7 頁）。書目資料取自 IEEE 頁面的存檔，標題、作者、摘要和 IEEE 版相同；內文是否和 IEEE 版逐字一樣我無法確認 |
| 連結 | [IEEE Xplore](https://ieeexplore.ieee.org/document/10199563)・[arXiv](https://arxiv.org/abs/2402.00043)・[PDF](https://arxiv.org/pdf/2402.00043v1) |
| 動畫解說 | [papers.poyuanchu.com/2023-wehner-rca/](https://papers.poyuanchu.com/2023-wehner-rca/) |
| 論文類型 | 系統／應用論文：BMW 電動車產線的根因分析工具，評估很少（三個小實驗，沒有和其他方法比較） |
| 狀態 | 待讀 |
| 與我的相關度 | 3 / 5（產線不是我的主題，但「用知識圖譜的先後順序和專家規則，先排除不可能的因果關係，再讓資料學剩下的部分」「專家點一下就把錯的邊加進黑名單、下次重學」這兩個做法，可以借到災害成因分析和災防知識圖譜） |
| 加入日期 | 2026-11-15 |

> [!NOTE]
> **一句話**：產線出了瑕疵，要找是哪一道工序造成的（根因分析，RCA）。純靠資料學因果圖（因果貝氏網路）在上千個感測變數時算不動，還會學出假的因果關係。作者把 BMW 一條電動車產線的「站點 → 工序 → 感測變數」先後順序和專家規則存進 Neo4j 知識圖譜，用它先排除不可能的因果邊、縮小搜尋空間，再做一個介面讓專家把錯的邊加進黑名單、標記變數類型，下次重學時就會照著改。論文報告訓練時間少 79%、學到的因果邊少 60.6%。

---

## 1. 背景

### 根因分析是什麼

產線最後的品檢發現某個量測值異常（例如焊接的熱輸入太高），但問題可能是前面某一道工序造成的。**根因分析**（Root Cause Analysis，RCA）就是沿著「誰影響誰」往回找。

傳統做法是 ISO/IEC 31010 裡的方法：

- **五個為什麼**（Five Whys）：一直問「為什麼」直到找到源頭
- **失效模式與影響分析**（FMEA）：事先列出每種失效的原因和後果
- **故障樹分析**（Fault Tree Analysis）：把故障拆成邏輯樹

這些都是把專家經驗整理成手冊，很花人力，也用不到產線感測器每小時產生的大量資料。

### 用資料學因果圖的難處

**因果貝氏網路**（Causal Bayesian Network）是一張有方向的圖：節點是感測變數，箭頭 A → B 代表「改變 A，即使其他變數不變，B 也會變」。學出這張圖就能沿箭頭往回找根因。但只靠資料有三個問題：

1. **混淆**（confounding）：兩個變數相關，可能只是因為第三個變數（例如同一位操作員）同時影響它們
2. **方向**：就算真的有關係，也看不出誰是因、誰是果
3. **組合爆炸**：可能的圖的數量隨變數數超指數成長，上千個變數根本不可能全部試

先前的研究用 FMEA 或本體論（ontology）定結構，但本體論只描述「類別」，無法針對個別量測值排除；也有研究證明加入專家知識能改善因果圖（作者之一 Kertel 的前作）。

---

## 2. 核心想法

1. **把產線的先後順序存成知識圖譜**：前面工序量到的變數可能影響後面的，反過來不可能。光這一條就能刪掉大量候選邊
2. **加上專家規則**：哪些變數是「根」（只能當原因，例如環境濕度）、「葉」（只能當結果，例如最後一道工序的量測）、「無關」（不影響任何東西，排除），以及「確定沒影響」的變數對
3. **只在剩下的空間裡用資料學因果圖**：用因果加性模型（CAM），而且只需要在同一道工序內部找順序
4. **專家回饋寫回知識圖譜**：介面上看到錯的箭頭，勾「加入黑名單」就寫入一條 hasNoImpact；下次重學會自動避開

---

## 3. 技術細節

### 3.1 系統架構

<center><b>圖 1：系統架構（依原論文 Fig. 1 以文字重現）</b></center>

```
 製程專家 ──Interacts with──▶ [Frontend：Interactive User Interface]
                                   │ Requests Retraining / Grabs Meta-Infos / Updates Blacklist
                                   ▼
 [Data Layer：Sensor Network] ◀── Grabs Data / Issues Data Preprocessing / Grabs Metadata ──┐
                                                                                          │
 [Knowledge Graph] ◀── Grabs Blacklist / Updates Knowledge Graph ── [Application Logic：Controller]
                                                                      │            ▲
                    Issues Iteration（Retraining）/ Gives Blacklist   ▼            │ Pulls Edges and Weights
                                                      [Model：Bayesian Network] ───┘
                                                                      │
                                   Frontend ◀── Grabs Bayesian Network ┘
```

> [!NOTE]
> **圖 1 怎麼看**
> 五個微服務，全部透過中間的 <b>Controller</b> 溝通：
> - <b>Data Layer</b>（左上）：從產線感測器網路抓資料、做前處理
> - <b>Knowledge Graph</b>（左下）：存專家知識和黑名單
> - <b>Bayesian Network</b>（上中）：Controller 交給它資料和黑名單、要求重新訓練，再取回學到的邊和權重
> - <b>Interactive User Interface</b>（右）：製程專家在這裡看根因圖、要求重訓、更新黑名單
>
> 關鍵是右下到左下那條回路：專家在介面上的修正（Updates Blacklist）經 Controller 寫進知識圖譜（Updates Knowledge Graph），下一輪訓練時 Bayesian Network 再拿到（Gives Blacklist）。

### 3.2 知識圖譜的設計

<center><b>圖 2：知識圖譜的結構（依原論文 Fig. 2 以文字重現）</b></center>

```
 Line ──hasSuccessorLine──▶ Line
  │ hasStation
  ▼
 Station ──hasSuccessorStation──▶ Station
  │ hasProcessStep
  ▼
 Process Step ──hasSuccessorProcessStep──▶ Process Step
  │ hasVariable
  ▼
 Sensor Variable ──hasNoImpact──▶ Sensor Variable
 （子類別：Root／Leaf／Irrelevant Variable）
```

> [!NOTE]
> **圖 2 怎麼看**
> 四層，每層都有一條「下一個」關係（自己指向自己的迴圈），表示順序：產線（Line）有多個站點（Station），站點有多道工序（Process Step），工序量測多個感測變數（Sensor Variable）。最底層多一條 hasNoImpact，表示專家確定兩個變數之間沒有因果關係。感測變數還可以標成三種子類別之一。

<center><b>圖 3：知識圖譜範例（依原論文 Fig. 3 以文字重現）</b></center>

```
 站點：   Sorting Station ──▶ Welding Station ◀── Gluing Station
             │                    │                 │         │
 工序：    Sorting             Welding        Surface Prep. ──▶ Gluing
           │      │               │                 │          │      │
 變數：  Weight  Sorting Time  Heat Input ◀─╳─ Humidity    Pressure  Amount Adhesive
         (Root)  (Irrelevant)   (Leaf)  hasNoImpact (Root)
```

> [!NOTE]
> **圖 3 怎麼看**
> 一個小產線：分揀站和塗膠站都在焊接站之前。塗膠站有兩道工序，先做表面處理、再塗膠。六個感測變數：
> - <b>Weight</b>（重量，根變數）、<b>Sorting Time</b>（分揀時間，無關變數）：在分揀工序量測
> - <b>Heat Input</b>（熱輸入，葉變數，最後一道工序）：在焊接量測
> - <b>Humidity</b>（濕度，根變數，環境因素不受產線影響）：在表面處理量測
> - <b>Pressure</b>（壓力）、<b>Amount Adhesive</b>（膠量）：在塗膠量測，同一道工序所以彼此順序未知
>
> Humidity 到 Heat Input 有一條 hasNoImpact（專家說濕度不影響熱輸入）。
>
> 原圖站點之間的箭頭標成「hasSuccessorStep」，和圖 2 的「hasSuccessorStation」不一致，應該是筆誤。

### 3.3 推理如何縮小搜尋空間

<center><b>圖 4：候選因果邊（依原論文 Fig. 4–6 以文字重現）</b></center>

```
 Fig. 4 不用知識圖譜：6 個變數兩兩之間都可能 → 6 × 5 = 30 條有向邊
 Fig. 5 用知識圖譜後：7 條
   Humidity → Amount Adhesive      Humidity → Pressure
   Amount Adhesive ↔ Pressure（同工序，方向未知，算 2 條）
   Amount Adhesive → Heat Input    Pressure → Heat Input
   Weight → Heat Input
   （Sorting Time 是無關變數，完全排除；Humidity → Heat Input 被 hasNoImpact 排除）
 Fig. 6 假設的真實因果圖：5 條
   Humidity → Amount Adhesive      Humidity → Pressure
   Pressure → Amount Adhesive      Amount Adhesive → Heat Input
   Weight → Heat Input
```

> [!NOTE]
> **圖 4 怎麼看**
> 從 30 條候選縮到 7 條，真實的 5 條都在這 7 條裡面。要學的只剩：7 條裡哪些是真的、以及同一工序內 Pressure 和 Amount Adhesive 誰影響誰。這就是全文的核心示範。（Fig. 5 裡分揀站的 Weight 和塗膠站的變數之間沒有任何候選邊，雖然兩站之間沒有先後關係；論文沒說明這條規則。）

> [!WARNING]
> **範例的文字、公式和圖互相矛盾。**
> - 3.3 節說「Weight 和 Sorting Time 可能影響 Heat Input，但 Humidity、Amount Adhesive、Pressure 不行」。但圖 3 的塗膠站在焊接站之前，Fig. 5 也列出 Amount Adhesive → Heat Input 和 Pressure → Heat Input，Fig. 6 的真實因果圖更把 Amount Adhesive → Heat Input 當成真的；Sorting Time 又被標成「無關變數」要排除
> - 式 (2) 寫 HeatInput = f₁(Humidity) + f₂(AmountAdhesive) + Noise，用的正好是圖 3 被 hasNoImpact 禁止的 Humidity；式 (1) 和 Fig. 6 都是 Weight 和 Amount Adhesive
> - 文中說「Weight 和 Humidity 是根變數，只要考慮把它們放在前兩個位置的排列，排列數減少 30 倍」。6 個變數共 6! = 720 種排列；兩個根變數放在前兩位、順序不限是 2! × 4! = 48 種，**只減少 15 倍**；要減少 30 倍得連兩者的先後也固定（720 / 24）

### 3.4 因果貝氏網路怎麼學

**結構方程模型**（SEM）的想法：每個變數 = 它的原因的函數 + 雜訊。作者用**因果加性模型**（CAM），每個原因各自一個非線性函數再相加：

$$\text{HeatInput} = f_1(\text{Weight}) + f_2(\text{AmountAdhesive}) + \text{Noise}$$

（式子依 Fig. 6 修正；論文式 (2) 誤寫成 Humidity。）雜訊是常態分布、f 非線性時，理論上可以從觀察資料辨認出真正的因果圖。CAM（Bühlmann et al., 2014）分兩步：

1. **找因果順序**：在所有排列 π 中找使下式最小的那個，f 用最大概似估計

$$S(\pi) = \sum_{k=1}^{p} \log\left[\sum_{\ell=1}^{N}\left(x_{\ell k} - \sum_{\pi(j)<\pi(k)} \hat{f}^{\pi}_{kj}(x_{\ell j})\right)^{2}\right]$$

意思是：每個變數只用排在它前面的變數去預測，殘差越小代表這個順序越合理。

2. **找因果邊**：順序定了以後，用迴歸挑出真正有影響的前面變數

如果知識圖譜給出完整順序，第 1 步可以跳過。實際上同一工序的變數無法排序，所以用 Kertel et al.（2022）的方法：**只在每道工序內部找順序**，跨工序的順序直接從知識圖譜取得。

### 3.5 互動介面

原論文 Fig. 7 是工具「RootFinder」的截圖（沒有公開圖檔，以下用文字描述）：

> [!NOTE]
> **圖 5 怎麼看（原論文 Fig. 7，介面截圖）**
> - <b>左邊大圖</b>：學到的根因圖，藍色節點是感測變數，<b>紅色</b>是出問題的變數（範例搜尋「Heat Input」），<b>黃色</b>是在根因路徑上的變數
> - <b>中間「Node」面板</b>：變數名稱、影響最大的三個因素，以及標記 ② 的屬性勾選框（Root Node、Irrelevant Node、Leaf Node），範例中 Heat Input 勾了 Leaf Node
> - <b>右邊「Edge」面板</b>：選取的邊 FROM Pressure TO Heat Input，權重 0.1340846945，下面是標記 ① 的「Add To Blacklist」勾選框
>
> 兩個數字就是兩種回饋：① 把錯的邊加進黑名單、② 改變數的子類別。專家也可以選產品和時間範圍，只看出問題那段時間的資料。

---

## 4. 實驗設定

- **場景**：BMW 一條真實電動車產線
- **知識圖譜**：Neo4j，用 Cypher 查詢；整個圖譜 100,015 個節點、417,944 條關係；評估用的那條產線有 53 個站點、96 道工序、1,683 個感測變數、2,143 條關係
- **資料**：一天、一種產品型號的資料；前處理：刪掉只有一個值的欄位、相關係數超過 0.95 的欄位對刪一個、缺值超過 50% 的欄和列刪掉，其餘缺值補平均
- **三個實驗**：
  1. 隨機取 25%、50%、75%、100% 的知識圖譜資訊，看訓練時間（跑 5 次取平均）
  2. 同樣設定，看學到的因果邊數量
  3. 隨機抽 100 組、每組 8 個變數，用時空順序學因果圖，和「部分已知的根因圖」比較 aSHD（修改版結構漢明距離，越低越好）；再加一條 hasNoImpact 看變化

> [!WARNING]
> **沒有「不用知識圖譜」的基準，也沒有和其他方法比較。** 實驗 1、2 的 x 軸從 25% 開始，所以所有「減少多少」都是 25%（或 50%）對 100% 的比較，不是「沒有知識圖譜 vs 有」。也沒有和純資料驅動的方法（NOTEARS、DirectLiNGAM、PC 演算法）或 FMEA 式方法比較。

---

## 5. 結果

### 5.1 知識圖譜比例與學到的因果邊

<center><b>表 2：知識圖譜比例與學到的因果邊數（依原論文 Fig. 8 估讀）</b></center>

| 使用的知識圖譜比例 | 25% | 50% | 75% | 100% |
|---|---|---|---|---|
| 學到的因果邊數（約略值，從圖估讀） | 約 972 | 約 1,010 | 約 955 | 約 398 |

> [!NOTE]
> **表 2 怎麼看（原論文 Fig. 8）**
> 橫軸是使用多少比例的知識圖譜資訊，縱軸「Number (n)」，只有一條紅線、四個點。25% 到 75% 幾乎持平（50% 還比 25% 高），只有到 100% 才從約 955 掉到約 398。圖說寫「學習時間和 CER 數量都下降」，但圖上只有一條線，也沒有圖例。

> [!WARNING]
> **兩個主要數字都對不太上圖。**
> - <b>「訓練時間減少 79.0%」</b>：Fig. 8 只有一條線、縱軸是「數量」，圖上找不到訓練時間，論文也沒有任何表列出秒數。這個數字無法驗證
> - <b>「刪掉 60.6% 的假因果邊」</b>：用估讀值算，(1,010 − 398) / 1,010 = 60.6%，正好是從**最高點 50%** 算到 100%；從 25% 起算是 59.1%。更大的問題是它量的是「學到的邊少了多少」，不是「假的邊少了多少」：這條產線沒有真實因果圖，少掉的邊也可能是真的。摘要和結論都寫成「減少假因果關係」，證據不支持
> - 曲線在 25%–75% 幾乎不動、只有 100% 突然下降，比較像某一類資訊（例如根／葉／無關變數標記）只在 100% 時才完整，而不是「知識越多越好」的漸進效果；論文沒有分析

### 5.2 專家回饋

<center><b>表 3：加一條 hasNoImpact 前後（實驗 3，100 組 × 8 個變數）</b></center>

| | aSHD 平均 | 標準差 |
|---|---|---|
| 只用時空順序 | 0.47 | 0.74 |
| 再加 1 條 hasNoImpact | 0.44 | 0.70 |

- 平均改善 (0.47 − 0.44) / 0.47 = **6.4%**（論文寫「almost 6%」，其實超過 6%）
- 作者說標準差變小代表「更穩定」

> [!WARNING]
> **6% 的改善在雜訊範圍內。** 差距 0.03，但標準差約 0.7；就算當成兩組獨立樣本，差距的標準誤約 0.10，差距只有 0.3 個標準誤。論文沒有做檢定、沒說加的是哪一條邊、也沒說 100 組裡有幾組受到影響。aSHD 平均才 0.47 代表多數組本來就是 0（隨機抽 8 個變數，在部分已知的根因圖裡很可能幾乎沒有已知的邊），能改善的空間本來就很小。

---

## 6. 總結

這篇的價值在**系統設計**，不在實驗：它示範了一個實際工廠能用的流程：產線的拓撲和專家知識存在圖資料庫 → 推理排除不可能的因果邊 → 在剩下的空間用資料學因果圖 → 視覺化給專家 → 專家修正寫回圖譜 → 重學。這個「人機回圈」對任何需要專家把關的因果分析都適用。

但評估太薄：沒有基準方法、沒有真實因果圖、訓練時間的數字在圖上找不到、「假因果邊減少」其實只是「邊變少」，專家回饋的效果在雜訊範圍內。範例的文字、公式、圖之間還有好幾處矛盾。可以參考做法，不要引用它的數字當證據。

> [!TIP]
> **可以學的做法**：(1) <b>先用結構知識排除，再用資料學習</b>：時間先後、空間上下游、「確定無關」這類硬規則存在圖譜裡，能把搜尋空間從 30 條縮到 7 條；(2) <b>回饋要能累積</b>：專家的每次修正都寫成圖譜裡的一條關係（hasNoImpact），而不是只改這一次的結果；(3) <b>變數分類</b>（根／葉／無關）是很便宜但效果很大的先驗。

---

## 7. 我的筆記

### 重點筆記
>

### 可以借用的地方
> 災害成因分析可以照這個架構：把「降雨 → 土壤含水 → 坡地崩塌 → 道路中斷」這類鏈條的上下游關係存成知識圖譜，加上「根變數」（雨量、地震規模這種外部驅動，不會被其他變數影響）、「葉變數」（災損、傷亡這種結果），以及專家確定無關的變數對；分析某次災情時，只在圖譜允許的候選邊上用觀測資料學因果強度，結果用介面給防災專家看，錯的邊勾黑名單寫回圖譜。同樣的圖譜也能給 GraphRAG 用：回答「這次淹水的可能原因」時，只沿著圖譜允許的因果方向檢索證據。

### 疑問
> 79% 的訓練時間是從哪裡來的？不用知識圖譜（0%）的時候學到幾條邊、要訓練多久？1,683 個變數裡標了多少根／葉／無關變數？「部分已知的根因圖」有幾條已知的邊、從哪來？

### 相關
- 論文：[[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]（同樣主張用製程知識找瑕疵成因，但把製程文字當多模態模型的情境，而不是約束因果圖）
- 概念：[[Root cause analysis|根因分析]]、[[Causal discovery|因果發現]]、[[Knowledge graph|知識圖譜]]、[[Graph database|圖資料庫]]
- 論文引用的相關工作：CAM（Bühlmann, Peters & Ernest, Annals of Statistics 2014，因果加性模型）；Kertel, Harmeling & Pauly（2022，arXiv 2210.14573，製造業用 SEM 學因果圖並加入專家知識，本文直接採用的演算法）；NOTEARS 非參數版（Zheng et al., AISTATS 2020）；DirectLiNGAM（Shimizu et al., JMLR 2011）；Knowledge Graphs 綜述（Hogan et al., ACM Computing Surveys 2021）
