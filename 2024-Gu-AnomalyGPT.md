---
title: "【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】"
tags: [paper, computer-vision, anomaly-detection, vlm, status-inbox]
concepts: ["[[Industrial anomaly detection]]", "[[Vision-language model]]", "[[Prompt tuning]]", "[[LoRA]]", "[[AUROC]]", "[[In-context learning]]", "[[Contrastive learning]]"]
papers: ["[[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]", "[[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]"]
---

# 【AnomalyGPT：Detecting Industrial Anomalies Using Large Vision-Language Models】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 作者 | Zhaopeng Gu、Bingke Zhu（共同一作）、Guibo Zhu、Yingying Chen（通訊作者）、Ming Tang、Jinqiao Wang；中國科學院自動化研究所、中國科學院大學、Objecteye Inc.（arXiv 版另列武漢人工智慧研究院） |
| 發表 | AAAI 2024（第 38 屆），Vol. 38 No. 3，頁 1932–1940，DOI [10.1609/aaai.v38i3.27963](https://doi.org/10.1609/aaai.v38i3.27963)。arXiv 2308.15366：v1 2023/8/29，v4 2023/12/28 |
| 版本 | 你給的是 AAAI 正式版（9 頁，**沒有附錄**）。附錄 A–D（門檻實驗、正常／異常文字、類別描述、更多案例）只在 arXiv 版。我比對過 arXiv v1–v4 與正式版，結果數字完全相同 |
| 連結 | [AAAI](https://ojs.aaai.org/index.php/AAAI/article/view/27963)・[arXiv](https://arxiv.org/abs/2308.15366)・[HTML v4](https://arxiv.org/html/2308.15366v4)・[GitHub](https://github.com/CASIA-IVA-Lab/AnomalyGPT)（約 1,150 顆星，最後更新 2023/12） |
| 動畫解說 | [papers.poyuanchu.com/2024-gu-anomalygpt/](https://papers.poyuanchu.com/2024-gu-anomalygpt/) |
| 論文類型 | 方法論文：把大型視覺語言模型（LVLM）改造成工業瑕疵檢測工具 |
| 狀態 | 待讀 |
| 與我的相關度 | 2 / 5（工業影像瑕疵檢測不是我的主題；但「把專用模型的輸出變成 LLM 的輸入」「把數值地圖轉成方位文字」「先算全猜同一答案的基準線」三件事可以借到災防系統） |
| 加入日期 | 2026-11-13 |

> [!NOTE]
> **一句話**：工廠瑕疵檢測的傳統方法只給一個「異常分數」，每種產品都要人工設門檻；通用的視覺語言模型（如 LLaVA、MiniGPT-4）會聊天，卻看不出細小瑕疵。AnomalyGPT 在凍結的 PandaGPT（ImageBind + Vicuna-7B）旁邊加一個**輕量解碼器**產生瑕疵熱圖，再用**提示學習器**把熱圖轉成幾個向量塞進 LLM，讓 LLM 直接回答「有沒有異常、在哪裡」，不用設門檻，也能多輪對話和只看 1 張正常圖就套用到新產品。MVTec-AD 正確率 93.3%、Image-AUC 97.4%。

---

## 1. 背景：工業瑕疵檢測（IAD）

### 任務是什麼

**工業瑕疵檢測**（Industrial Anomaly Detection，IAD）：給一張產品照片（瓶子、螺絲、皮革、電路板），判斷**有沒有瑕疵**，以及**瑕疵在哪裡**（逐像素的定位圖）。難處在於瑕疵稀少又千奇百怪，收集不到夠多的瑕疵樣本，所以通常**只用正常品訓練**，把「跟正常品長得不像」的東西當成異常。

兩個常用資料集：

<center><b>表 2：資料集</b></center>

| 資料集 | 類別 | 內容 | 本文用法 |
|---|---|---|---|
| MVTec-AD | 15（瓶子、電纜、膠囊、地毯、榛果、螺絲、拉鍊等） | 訓練 3,629 張（全正常），測試 1,725 張（正常 467、異常 1,258），解析度 700–1,024 | 無監督訓練與測試；或在 VisA 上訓練後 few-shot 測試 |
| VisA | 12（蠟燭、腰果、PCB 1–4、通心粉等） | 9,621 張正常、1,200 張異常，約 1,500 × 1,000 | 1-shot 測試；或用來訓練、再轉到 MVTec |

（MVTec 測試集的正常／異常張數是我依原資料集統計的，論文只寫總數。）

### 先前方法的兩個問題

**問題一：只給分數，門檻要人工定。** 傳統方法（PatchCore、PaDiM 等）輸出 0–1 的異常分數。分數 0.6 到底算不算瑕疵？每個類別的最佳門檻都不同，要找最佳門檻就得先有瑕疵樣本，這違反了「只用正常品」的前提。作者舉例：PatchCore 在 MVTec 的 Image-AUC 有 99.3%，但用**一個共用門檻**時正確率只剩 79.76%（論文第 2 節）。

**問題二：通用 LVLM 不懂瑕疵。** LLaVA、MiniGPT-4、PandaGPT 用網路圖片訓練，看得懂「這是一顆膠囊」，但對局部細節不敏感，也沒有「這個刮痕代表不良品」的領域知識。

<center>
<img src="https://arxiv.org/html/2308.15366v4/fig1.png" alt="圖 1" width="75%">
<br>
<b>圖 1：既有 IAD 方法、既有 LVLM 與 AnomalyGPT 的比較（原論文 Figure 1）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**
> 上半是正常膠囊，下半是印字「500」被刮花的異常膠囊，各自交給三種方法：
> - <b>既有 IAD 方法</b>：只給分數（正常 0.561、異常 0.854）和一張熱圖，右邊的人頭冒問號：「所以到底有沒有瑕疵？」
> - <b>既有 LVLM</b>：會描述「紅色藥瓶、白色標籤」（其實是膠囊），而且兩張都說「沒有異常」，下半的使用者不開心
> - <b>AnomalyGPT</b>：正常圖回答沒有異常、熱圖全黑；異常圖回答「有異常，在圖片右側」，熱圖在印字處有一個亮點
>
> 小細節：AnomalyGPT 把黑橘兩色的膠囊描述成「small, red, round pill」，物體描述其實也不太準，它強的是「有／沒有、在哪」而不是描述。

<center><b>表 3：各類方法具備的功能（原論文 Table 1）</b></center>

| 方法 | Few-shot | 異常分數 | 異常定位 | 直接判斷有無異常 | 多輪對話 |
|---|:-:|:-:|:-:|:-:|:-:|
| 傳統 IAD（PatchCore、InTra、PyramidFlow） | | ✓ | ✓ | | |
| Few-shot IAD（RegAD、GraphCore、WinCLIP） | ✓ | ✓ | ✓ | | |
| LVLM（MiniGPT-4、LLaVA、PandaGPT） | ✓ | | | | ✓ |
| AnomalyGPT | ✓ | ✓ | ✓ | ✓ | ✓ |

---

## 2. 核心想法

1. **不要讓 LLM 自己看細節**：另外接一個專門算瑕疵熱圖的輕量解碼器，LLM 負責「看熱圖 + 看原圖 → 說話」。
2. **熱圖變成 LLM 聽得懂的東西**：用一個小 CNN（提示學習器）把熱圖壓成幾個嵌入向量，接在影像向量後面送進 LLM。
3. **LLM 本身不動**：只訓練解碼器和提示學習器（prompt tuning），LLM 和影像編碼器全部凍結，並且和 PandaGPT 原本的預訓練資料交替訓練，避免忘掉原本的對話能力。
4. **沒有瑕疵樣本就自己做**：在正常圖上剪貼別處的區塊並用泊松編輯融合，產生假瑕疵和對應的問答。
5. **新產品只要 1 張正常圖**：few-shot 時把正常圖的特徵存成記憶庫，拿查詢圖的每一小塊去比對最像的正常小塊，越不像越異常。

---

## 3. 技術細節

### 3.1 整體架構

<center>
<img src="https://arxiv.org/html/2308.15366v4/structure_change.png" alt="圖 2" width="90%">
<br>
<b>圖 2：AnomalyGPT 架構（原論文 Figure 2）</b>
</center>

> [!NOTE]
> **圖 2 怎麼看**
> 圖例在最下方：火焰＝會訓練的模組，雪花＝凍結的模組，⊗ 矩陣乘法，Ⓒ 餘弦相似度，⊕ 逐元素相加。
> - <b>中間主幹</b>：查詢圖（有小凸起的皮革）→ 凍結的 Image Encoder → 凍結的 Linear → 進入 LLM 當影像向量
> - <b>上方紅框 Image Decoder（訓練）</b>：從編碼器 4 個中間層各拿一份小塊特徵，各經一層 Linear，再和「正常文字／異常文字」的文字特徵做矩陣乘法，4 份加起來得到無監督模式的定位圖（右上，黑底小白點）
> - <b>下方橘色虛線框 Few-shot Inference</b>：正常圖經同一個編碼器，4 層特徵存進 Memory，查詢圖的小塊和記憶庫做餘弦相似度，得到 few-shot 模式的定位圖（右下，較模糊）
> - <b>右側 Prompt learner（訓練）</b>：定位圖經「CONV → ReLU → MaxPool」重複 5 次再一層 CONV，和「可學習的基礎提示向量」串接（CONCAT）後送進凍結的 LLM
> - LLM 同時收到影像向量、提示向量和使用者的問題「This is a photo of leather. Is there any anomaly in the image?」，回答「Yes, there is an anomaly in the image, at the bottom left of the image.」
>
> 圖上兩條路徑在 ⊕ 匯合，看起來像是兩種定位圖相加；但公開程式碼在 few-shot 時**只用記憶庫的結果**，相加那行被註解掉了（見 5.4 節）。

<center><b>表 4：元件與設定</b></center>

| 元件 | 用什麼 | 是否訓練 |
|---|---|---|
| 影像編碼器 | ImageBind-Huge，輸入 224 × 224；取第 8、16、24、32 層給解碼器 | 凍結 |
| 文字編碼器 | ImageBind 的文字端，編碼「正常／異常」的模板文字 | 凍結 |
| LLM | Vicuna-7B，從 PandaGPT 的權重初始化 | 凍結 |
| 影像 → LLM 的線性層 | PandaGPT 原有 | 凍結 |
| 影像解碼器 | 每層一個 Linear 投影 | 訓練 |
| 提示學習器 | n₁ 個可學習基礎提示向量 + 一個小 CNN 產生 n₂ 個向量 | 訓練 |
| 訓練 | 2 張 RTX 3090、50 epoch、學習率 1e-3、batch 16、線性暖身 + one-cycle 餘弦衰減；和 PandaGPT 預訓練資料交替訓練 | |

### 3.2 解碼器：小塊特徵和「正常／異常」文字比相似度

想法來自 WinCLIP：CLIP 類模型的影像和文字在同一個空間，拿每一小塊影像特徵去比「完好的皮革照片」和「有瑕疵的皮革照片」兩段文字，比較像後者的小塊就是瑕疵。

問題是中間層的特徵沒有和文字對齊過，不能直接比，所以每層先加一個 Linear 投影到文字空間：

$$M = \text{Upsample}\left(\sum_{i=1}^{4} \text{softmax}\left(\tilde{F}^{i}_{patch} F_{text}^{T}\right)\right)$$

- $\tilde{F}^{i}_{patch}$：第 i 層投影後的小塊特徵（H_i × W_i × C_text）
- $F_{text}$：2 × C_text，一列代表正常、一列代表異常
- softmax 在「正常／異常」兩類之間做，取異常那一類的機率當作每一小塊的異常程度，4 層加總後放大回原圖大小

文字端用 WinCLIP 的**組合式提示集成**：狀態詞（正常 7 種如「flawless [o]」「[o] without defect」，異常 5 種如「damaged [o]」「[o] with flaw」）× 模板 22 種（如「a cropped photo of the [c].」「a photo of a [c] for anomaly detection.」），全部編碼後平均（arXiv 版附錄 B）。

### 3.3 Few-shot：記憶庫比對

新產品沒有訓練資料時，把 k 張正常圖的 4 層小塊特徵存進記憶庫 $B^i$，查詢圖每一小塊找記憶庫中最像的那一塊：

$$M = \text{Upsample}\left(\sum_{i=1}^{4}\left(1 - \max\left(F^{i}_{patch} \cdot {B^{i}}^{T}\right)\right)\right)$$

最像的正常小塊都不太像 → 1 − 相似度大 → 異常。這就是 PatchCore 的做法，只是記憶庫只有 1–4 張圖。

### 3.4 提示學習器：把熱圖翻譯成 LLM 的輸入

- **基礎提示向量** $E_{base}$（n₁ 個）：和輸入無關的可學習向量，相當於給 LLM 一段「你現在在做瑕疵檢測」的隱形提示
- **解碼器提示向量** $E_{dec}$（n₂ 個）：熱圖經 CNN 轉出來
- 兩者合起來 $E_{prompt}$（n₁ + n₂ 個），接在影像向量後面

送進 LLM 的格式：

```
### Human: <Img> E_img </Img> E_prompt [Image Description] Is there any anomaly in the image? ### Assistant:
```

這是 **prompt tuning**：不改 LLM 的權重，只學「塞在前面的幾個向量」。好處是參數少、不容易在只有幾千張圖的資料上過擬合，也比較不會忘記原本的聊天能力。論文沒有寫 n₁、n₂ 的值。

### 3.5 訓練資料：假瑕疵 + 問答

<center>
<img src="https://arxiv.org/html/2308.15366v4/possion.png" alt="圖 3" width="85%">
<br>
<b>圖 3：Cut-paste 與泊松影像編輯的比較（原論文 Figure 3）</b>
</center>

> [!NOTE]
> **圖 3 怎麼看**
> 上排是 Cut-paste、下排是 Poisson，四欄分別是膠囊、榛果、金屬螺帽、藥丸。Cut-paste 直接把一塊方形貼上去，邊緣很明顯（膠囊上方的小橘塊、榛果和藥丸上的黑色方塊）；Poisson 解偏微分方程讓貼上區塊的顏色和邊界跟周圍自然銜接，瑕疵看起來像真的刮痕或缺角。作者採用 NSA 方法（Cut-paste + 泊松編輯）。圖是挑選過的示意，論文沒有比較兩種做法對結果的影響。

每張假瑕疵圖配一組問答：

- 問題 = 類別描述 + 「Is there any anomaly in the image?」。類別描述如「This is a photo of leather for anomaly detection, which should be brown with patterns and without any damage, flaw, defect, scratch, hole or broken part.」（每類一句，arXiv 版附錄 C 表 6、表 7）；實際使用時可以不給描述
- 回答 = 「No, there are no anomalies in the image.」或「Yes, there is an anomaly in the image, at the bottom left of the image.」

<center>
<img src="https://arxiv.org/html/2308.15366v4/3x3.png" alt="圖 4" width="45%">
<br>
<b>圖 4：用 3 × 3 九宮格描述瑕疵位置（原論文 Figure 4）</b>
</center>

> [!NOTE]
> **圖 4 怎麼看**
> 一張皮革圖切成九格：Top Left、Top、Top Right、Left、Center、Right、Bottom Left、Bottom、Bottom Right。左下格有一個小凸起，所以這張圖的標準答案是「at the bottom left」。九宮格讓 LLM 能用文字講出位置，代價是精度只有九分之一張圖；精確位置要看解碼器的熱圖。

### 3.6 損失函數

$$L = \alpha L_{ce} + \beta L_{focal} + \delta L_{dice}，\quad \alpha = \beta = \delta = 1$$

- **交叉熵** $L_{ce}$：LLM 產生的回答和標準答案逐字比較
- **Focal loss**（γ = 2）：瑕疵圖裡大部分像素還是正常的，focal loss 降低「很容易判對的正常像素」的權重，讓模型專心學難的像素
- **Dice loss**：直接最大化預測區域和真實瑕疵區域的重疊

後兩者只作用在解碼器的熱圖上。因為 LLM 的交叉熵會經過提示學習器把梯度傳回解碼器，所以解碼器同時被「畫得準」和「讓 LLM 答得對」兩個目標訓練。

---

## 4. 實驗設定

- **兩種設定**
  - 無監督：在 MVTec-AD 全部 15 類的正常圖（加假瑕疵）上訓練**一個**模型，在 MVTec 測試
  - Few-shot：在一個資料集訓練，換到另一個資料集，每類只給 1、2、4 張正常圖（VisA 訓練 → MVTec 測試，或反過來）；跑 5 次取平均
- **指標**
  - **Image-AUC**：用熱圖的最大值當整張圖的異常分數，算 ROC 曲線下面積，不需要門檻
  - **Pixel-AUC**：每個像素當一個樣本算 AUC，衡量定位
  - **Accuracy**：LLM 回答裡有「Yes」且真的是瑕疵，或有「No」且真的正常，就算對；公開程式碼是**各類別正確率的平均**
- **比較對象**：few-shot 比 SPADE、PaDiM、PatchCore、WinCLIP（數字取自 WinCLIP 論文）；無監督比同樣「一個模型管所有類別」的 UniAD、PaDiM、JNLD。基準方法都沒有 Accuracy，因為它們不會自己做判斷。

> [!WARNING]
> **AUC 不是 LLM 的成績。** Image-AUC 和 Pixel-AUC 都是從解碼器（或記憶庫）的熱圖算出來的，跟 LLM 回答什麼無關。真正衡量「LLM 會不會判斷」的只有 Accuracy 一欄，而這一欄沒有任何基準方法可比。

---

## 5. 結果

### 5.1 Few-shot

<center><b>表 5：Few-shot 結果（原論文 Table 2，5 次平均 ± 標準差）</b></center>

| 設定 | 方法 | MVTec Image-AUC | MVTec Pixel-AUC | MVTec Acc | VisA Image-AUC | VisA Pixel-AUC | VisA Acc |
|---|---|---|---|---|---|---|---|
| 1-shot | SPADE | 81.0 ± 2.0 | 91.2 ± 0.4 | – | 79.5 ± 4.0 | 95.6 ± 0.4 | – |
| | PaDiM | 76.6 ± 3.1 | 89.3 ± 0.9 | – | 62.8 ± 5.4 | 89.9 ± 0.8 | – |
| | PatchCore | 83.4 ± 3.0 | 92.0 ± 1.0 | – | 79.9 ± 2.9 | 95.4 ± 0.6 | – |
| | WinCLIP | 93.1 ± 2.0 | 95.2 ± 0.5 | – | 83.8 ± 4.0 | **96.4 ± 0.4** | – |
| | AnomalyGPT | **94.1 ± 1.1** | **95.3 ± 0.1** | 86.1 ± 1.1 | **87.4 ± 0.8** | 96.2 ± 0.1 | 77.4 ± 1.0 |
| 2-shot | SPADE | 82.9 ± 2.6 | 92.0 ± 0.3 | – | 80.7 ± 5.0 | 96.2 ± 0.4 | – |
| | PaDiM | 78.9 ± 3.1 | 91.3 ± 0.7 | – | 67.4 ± 5.1 | 92.0 ± 0.7 | – |
| | PatchCore | 86.3 ± 3.3 | 93.3 ± 0.6 | – | 81.6 ± 4.0 | 96.1 ± 0.5 | – |
| | WinCLIP | 94.4 ± 1.3 | **96.0 ± 0.3** | – | 84.6 ± 2.4 | **96.8 ± 0.3** | – |
| | AnomalyGPT | **95.5 ± 0.8** | 95.6 ± 0.2 | 84.8 ± 0.8 | **88.6 ± 0.7** | 96.4 ± 0.1 | 77.5 ± 0.3 |
| 4-shot | SPADE | 84.8 ± 2.5 | 92.7 ± 0.3 | – | 81.7 ± 3.4 | 96.6 ± 0.3 | – |
| | PaDiM | 80.4 ± 2.5 | 92.6 ± 0.7 | – | 72.8 ± 2.9 | 93.2 ± 0.5 | – |
| | PatchCore | 88.8 ± 2.6 | 94.3 ± 0.5 | – | 85.3 ± 2.1 | 96.8 ± 0.3 | – |
| | WinCLIP | 95.2 ± 1.3 | **96.2 ± 0.3** | – | 87.3 ± 1.8 | **97.2 ± 0.2** | – |
| | AnomalyGPT | **96.3 ± 0.3** | **96.2 ± 0.1** | 85.0 ± 0.3 | **90.6 ± 0.7** | 96.7 ± 0.1 | 77.7 ± 0.4 |

（粗體是我依數字重標的每欄最高。）

我算的差距（AnomalyGPT − WinCLIP，百分點）：

<center><b>表 6：AnomalyGPT 相對 WinCLIP 的差距（百分點）</b></center>

| 設定 | MVTec Image | MVTec Pixel | VisA Image | VisA Pixel |
|---|---|---|---|---|
| 1-shot | +1.0 | +0.1 | +3.6 | −0.2 |
| 2-shot | +1.1 | −0.4 | +4.0 | −0.4 |
| 4-shot | +1.1 | 0.0 | +3.3 | −0.5 |

- **Image-AUC 六格全贏**：VisA 贏 3.3–4.0 個百分點；MVTec 只贏 1.0–1.1，**比 WinCLIP 自己的標準差（1.3–2.0）還小**
- **Pixel-AUC 只有 1 格贏 0.1，1 格平手，4 格輸**。論文說「competitive」是公允的說法
- **Accuracy 不隨張數上升**：MVTec 1-shot 86.1 → 2-shot 84.8 → 4-shot 85.0，給更多正常圖反而比較差；VisA 77.4 → 77.5 → 77.7 幾乎不動

### 5.2 無監督

<center><b>表 7：MVTec-AD 無監督結果（原論文 Table 3）</b></center>

| 方法（一個模型管所有類別） | Image-AUC | Pixel-AUC | Accuracy |
|---|---|---|---|
| PaDiM（Unified） | 84.2 | 89.5 | – |
| JNLD（Unified） | 91.3 | 88.6 | – |
| UniAD | 96.5 | **96.8** | – |
| AnomalyGPT | **97.4** | 93.1 | 93.3 |

Image-AUC 比 UniAD 高 0.9 個百分點，Pixel-AUC 低 3.7 個百分點。定位能力其實輸給 UniAD 不少，論文只說了贏的部分。

### 5.3 消融實驗

<center><b>表 8：消融實驗（原論文 Table 4，MVTec 無監督／VisA 1-shot）</b></center>

| 解碼器 | 提示學習器 | LLM | LoRA | MVTec Image | MVTec Pixel | MVTec Acc | VisA Image | VisA Pixel | VisA Acc |
|:-:|:-:|:-:|:-:|---|---|---|---|---|---|
| | | ✓ | | – | – | 72.2 | – | – | 56.5 |
| | ✓ | ✓ | | – | – | 73.4 | – | – | 56.6 |
| | | ✓ | ✓ | – | – | 79.8 | – | – | 63.4 |
| ✓ | | ✓ | | 97.1 | 90.9 | 72.2 | 85.8 | 96.2 | 56.5 |
| ✓ | | ✓ | ✓ | 97.1 | 90.9 | 84.2 | 85.8 | 96.2 | 64.7 |
| ✓ | ✓ | ✓ | ✓ | 96.0 | 88.1 | 83.9 | 85.8 | 96.5 | 72.7 |
| ✓ | | | | 97.1 | 90.9 | 90.3 | 85.8 | 96.2 | 75.4 |
| ✓ | ✓ | ✓ | | **97.4** | **93.1** | **93.3** | **87.4** | 96.2 | **77.4** |

沒有 LLM 那一列的判斷方式：拿正常樣本中最高的異常分數當門檻。

怎麼讀這張表：

- **只有 LLM（第 1 列）**：72.2 / 56.5。這幾乎就是「每張都回答有異常」的分數，見下方 WARNING
- **有解碼器但沒有提示學習器（第 4 列）**：和第 1 列完全相同。這是合理的，因為沒有提示學習器時，熱圖根本沒有送進 LLM
- **只用解碼器 + 簡單門檻（第 7 列）**：90.3 / 75.4。這是最重要的比較對象：完整模型（最後一列）93.3 / 77.4，**LLM 加進來只多 3.0 和 2.0 個百分點**
- **LoRA 反而變差**：同樣有解碼器和提示學習器，加 LoRA 微調 LLM 後 MVTec 從 93.3 掉到 83.9（−9.4），VisA 從 77.4 掉到 72.7（−4.7）。作者的解釋是 prompt tuning 比較不會過擬合、轉移能力比較好

> [!WARNING]
> **「只有 LLM」那一列等於全部猜「有異常」。** MVTec 測試集 1,725 張中有 1,258 張是瑕疵品，每張都回答「Yes」的話，總正確率 72.93%，**各類平均（公開程式碼的算法）72.30%**；VisA 依官方 1cls 切分，測試集 962 張正常、1,200 張異常，全猜「Yes」的各類平均是 **56.56%**。論文表上的 72.2 和 56.5 跟這兩個數幾乎一樣，代表凍結的 PandaGPT 基本上每張都說有異常，表上的 72.2 不是「LLM 有一點判斷力」，而是「完全沒有判斷力」。論文沒有列出這條基準線。（張數由我依 MVTec 原資料集與 VisA 官方 `split_csv/1cls.csv` 統計。）

> [!WARNING]
> **LLM 真正的貢獻比看起來小。** 和「解碼器 + 最大正常分數當門檻」這個很粗糙的規則相比，完整模型只多 3.0（MVTec）和 2.0（VisA）個百分點，而且完整模型的解碼器也被重新訓練過（Image-AUC 97.1 → 97.4、Pixel-AUC 90.9 → 93.1），多出來的正確率有一部分可能來自解碼器變好，而不是 LLM 的判斷。論文也沒有比較「解碼器 + 一個用驗證集調好的共用門檻」，這才是公平的對照。

> [!WARNING]
> **VisA 1-shot 的 AUC 在各列不同，和程式碼對不上。** 在 1-shot 設定下，依公開程式碼（`openllama.py`），熱圖**只來自記憶庫比對**，而記憶庫用的是凍結的 ImageBind 特徵，和解碼器、提示學習器、LoRA 都無關，所以各列的 VisA Image-AUC／Pixel-AUC 理論上應該一樣。表上卻有 85.8／96.2、85.8／96.5、87.4／96.2 三種，論文沒有解釋。

### 5.4 我核對公開程式碼的結果

<center><b>表 9：論文說法與程式碼對照</b></center>

| 論文說法 | 程式碼（`code/model/openllama.py`、`code/test_mvtec.py`） | 是否一致 |
|---|---|---|
| 式 (1)：4 層 softmax 結果**相加** | 4 層取**平均**（`torch.mean`） | 只差一個常數倍，不影響 AUC |
| 圖 2：few-shot 時兩種定位圖逐元素相加（⊕） | few-shot 時 `anomaly_map_ret = 1 - sim`，相加的寫法 `(anomaly_map_ret + 1 - sim) / 2` 被註解掉 | 不一致：few-shot 只用記憶庫 |
| 式 (2)：內積取最大 | `F.cosine_similarity` 取最大 | 一致（圖 2 也標 Ⓒ 餘弦相似度） |
| MVTec few-shot 用資料增強的正常圖 | `if 'mvtec' in 'normal_img_paths':` 比對的是字串常數本身，永遠是 False，增強那條路徑從來不會執行 | 程式錯誤 |
| Accuracy | 回答含「Yes」且是瑕疵、或含「No」且是正常 → 對；最後取 15 類平均 | 一致，但「Yes／No」只看有沒有出現這個字 |
| 解碼設定 | MVTec `top_p = 0.1`、VisA `top_p = 0.01`，溫度 1.0 | 論文沒寫；兩個資料集用不同設定 |
| Image-AUC | 用熱圖最大值（`anomaly_map.max()`） | 一致，和 LLM 無關 |

### 5.5 質性範例

<center>
<img src="https://arxiv.org/html/2308.15366v4/case_study_1.png" alt="圖 5" width="70%">
<br>
<b>圖 5：無監督設定的對話範例（原論文 Figure 5）</b>
</center>

> [!NOTE]
> **圖 5 怎麼看**
> 左上是使用者給的藥丸（印「FF」、有紅點，下緣有一塊缺損），左下是模型輸出的熱圖，只在缺損處有一小塊亮點。右邊兩輪對話：問「有異常嗎」答「有，在圖片下方」；再問「這是什麼」答「黑色表面上的白色藥丸」。示範的是「判斷 + 九宮格位置 + 熱圖 + 一般問答」四件事一次做到。

<center>
<img src="https://arxiv.org/html/2308.15366v4/case_study_2.png" alt="圖 6" width="70%">
<br>
<b>圖 6：1-shot 設定的對話範例（原論文 Figure 6）</b>
</center>

> [!NOTE]
> **圖 6 怎麼看**
> 由上到下：給的 1 張正常皮革（Normal input）、有刮痕的查詢圖（Query input）、模型熱圖（Model output）。熱圖在左上方有一塊亮區，但背景雜訊明顯比圖 5 多，這就是作者說的「沒有訓練，定位稍差」。三輪對話：答「有異常，在圖片上方」；材質是皮革；第三輪說皮革「棕色、粗糙，代表是高品質耐用的皮革」，剛說完有瑕疵又誇品質好，可以看出後續聊天並沒有真的整合瑕疵資訊。

<center>
<img src="https://arxiv.org/html/2308.15366v4/case_study_C1.png" alt="圖 7" width="60%">
<br>
<b>圖 7：正常瓶子上四個模型的回答（arXiv 版附錄 D）</b>
</center>

> [!NOTE]
> **圖 7 怎麼看**
> 一個正常瓶子的俯視圖，問「有異常嗎」：PandaGPT 和 LLaVA 都說「有，瓶子倒過來了」（紅字、紅叉），MiniGPT-4 說沒異常但編出一個不存在的「Bordeaux」標籤（黃色半對），AnomalyGPT 答沒有異常、熱圖全黑（綠勾）。第二題問形狀材質，四個模型都答「圓形、玻璃」。通用 LVLM 會把「俯視角度」誤認成異常，這正是缺乏領域知識的表現。這是作者挑的案例，不是統計結果。

### 5.6 為什麼門檻難定（arXiv 版附錄 A）

<center>
<img src="https://arxiv.org/html/2308.15366v4/threshold_patchcore.svg" alt="圖 8" width="90%" style="background:#fff">
<br>
<b>圖 8：PatchCore 在 MVTec 各類別、不同門檻下的正確率（arXiv 版 Figure 7）</b>
</center>

> [!NOTE]
> **圖 8 怎麼看**
> 15 張小圖各是一個類別，橫軸是門檻（0.1–0.95），縱軸是正確率。每條曲線都是「門檻太低全判異常、太高全判正常，中間有一個高峰」。高峰的位置各類差很多（從圖估讀）：膠囊約 0.2、拉鍊約 0.25、地毯和螺絲約 0.3、皮革約 0.35、藥丸約 0.4、磁磚和牙刷約 0.45、電晶體約 0.55、金屬螺帽約 0.6、榛果約 0.7、電纜約 0.75。所以任何單一門檻都會讓某些類別掉很多，例如門檻 0.7 時膠囊正確率只剩 30% 左右。

<center>
<img src="https://arxiv.org/html/2308.15366v4/threshold_winclip.svg" alt="圖 9" width="90%" style="background:#fff">
<br>
<b>圖 9：WinCLIP 在 MVTec 各類別、不同門檻下的正確率（arXiv 版 Figure 8）</b>
</center>

> [!NOTE]
> **圖 9 怎麼看**
> 同樣的 15 格，但橫軸只有 0.485–0.510：WinCLIP 的分數擠在 0.5 附近一個很窄的區間，門檻差 0.005 正確率就可能從 90% 掉到 30%。高峰位置（從圖估讀）從皮革、磁磚、木頭約 0.488 到榛果約 0.501。曲線又陡又窄，比 PatchCore 更難設共用門檻。

這兩張圖是全文最有說服力的動機：分數好（AUC 高）不等於能直接用（正確率高）。但要注意，AnomalyGPT 自己的判斷也是學出來的一個「隱形門檻」，論文沒有給 AnomalyGPT 同樣的「各類正確率」圖來證明它在每一類都穩定。

---

## 6. 總結

AnomalyGPT 是第一批把 LVLM 用在工業瑕疵檢測的工作，架構很務實：凍結大模型，旁邊接一個專門的熱圖解碼器，再用 prompt tuning 把熱圖餵進 LLM。它解決了「使用者不想自己設門檻、想直接問有沒有瑕疵」的需求，也保留了對話能力。

但從數字來看：

- 定位（Pixel-AUC）大多輸給 WinCLIP 和 UniAD，檢測（Image-AUC）在 MVTec 只小贏 1 個百分點左右
- 真正的新東西是 Accuracy，而它只比「解碼器 + 粗糙門檻」高 2–3 個百分點，沒有和公平的門檻方法比
- 純 LLM 的基準等於全猜「有異常」，說明通用 LVLM 在這個任務上幾乎沒有判斷力，判斷力主要來自解碼器的熱圖

> [!TIP]
> **可以學的設計**：當 LLM 看不懂某種專業資料（影像細節、感測器數值、雷達回波），不要硬微調 LLM，而是讓**專用模型先算出結構化結果**（熱圖、分數、等級），再把結果轉成 LLM 的輸入（這篇是用 prompt tuning 轉成向量；不想訓練的話，也可以像九宮格那樣轉成文字）。另外，評估任何「LLM 回答是／否」的系統時，**一定要先算全猜同一個答案的分數**，否則 72% 看起來還不錯，其實什麼都沒學到。

---

## 7. 我的筆記

### 重點筆記

#### Q1：few-shot 記憶庫裡的「4 層小塊特徵」是什麼？

- **小塊（patch）**：ImageBind-Huge 是 ViT，把 224 × 224 的圖切成 14 × 14 像素的小方塊，共 16 × 16 = 256 塊，每塊轉成一個 1280 維向量，代表那一區長什麼樣子。
- **層（layer）**：ViT-Huge 有 32 層，取第 8、16、24、32 層的輸出。每層都是 256 個小塊向量，淺層偏紋理、邊緣（刮痕、髒污），深層偏語意、結構（缺件、變形）。
- **比對流程**：記憶庫 = k 張正常圖 × 4 層 × 256 向量，每層一個 $B^i$。查詢圖的每個小塊在同一層記憶庫找餘弦相似度最高的正常小塊，異常分數 = 1 − 最高相似度；4 層各得一張 16 × 16 分數圖，平均後放大回原圖。
- 一句話：最像的正常小塊都不太像，那一塊就可能有瑕疵（PatchCore 的做法，只是記憶庫只有 1–4 張圖）。

#### Q2：是不是只訓練一個 CNN，其餘只是把參數和 prompt 丟給 LLM？

不完全是。

- **有訓練的是兩個小模組**：影像解碼器（4 個 Linear 投影，不是 CNN）和提示學習器（小 CNN + 可學習的基礎提示向量）。LLM、影像編碼器都凍結。
- **送進 LLM 的不是文字 prompt**，而是 soft prompt：一串連續向量，接在影像向量後面，人寫不出來，只能學。
- **訓練訊號穿過 LLM**：讓凍結的 LLM 回答「Yes, there is an anomaly… at the bottom left」這類標準答案，用 cross-entropy 當損失，梯度傳回提示學習器，但 LLM 權重不更新，這就是 prompt tuning。
- 結論：確實有訓練模型，只是訓練掛在凍結大模型旁邊的輕量模組。論文試過用 LoRA 微調 LLM，MVTec 正確率反而從 93.3% 掉到 83.9%。

#### Q3：Focal loss 和 Dice loss 白話是什麼？

兩者都用來訓練影像解碼器，比對預測熱圖和真實瑕疵遮罩。共同問題：一張瑕疵圖裡瑕疵可能只佔 1% 像素。

- **Focal loss，讓模型多練難題**：逐像素打分，已經很有把握答對的像素權重調低。γ = 2 時，90% 把握的像素權重乘 (1 − 0.9)² = 0.01，60% 把握的乘 0.16，差 16 倍。會的題目少算分，不會的多算分。
- **Dice loss，看整塊有沒有圈對**：Dice = 2 × 重疊面積 ÷（預測面積 + 真實面積），損失 = 1 − Dice。整張都猜正常時逐像素正確率有 99%，但 Dice = 0，損失拉滿。
- 合起來：Focal 管每個像素判得好不好，Dice 管瑕疵的形狀和位置準不準。

#### Q4：圖 3 的 Cut-paste 和 Poisson 是什麼？

真實瑕疵圖少、標註貴，所以作者在正常圖上自己做假瑕疵；瑕疵是自己貼的，位置自動知道，可以直接當訓練標籤。

- **Cut-paste**：剪一小塊直接貼上，像貼貼紙。邊緣明顯，模型容易學到「找方形突兀邊界」的捷徑。
- **Poisson（泊松影像編輯）**：貼上後用數學讓顏色和周圍自然融合，只保留紋理變化（如刮痕線條），像 Photoshop 的修復筆刷。
- 作者用 NSA 方法：Cut-paste + Poisson 融合。

#### Q5：圖 2 裡哪些元件要訓練、哪些是現成的？

- **現成、凍結**：Image Encoder（ImageBind-Huge）、Text Encoder（ImageBind 文字端，編碼「正常／異常」描述）、影像接 LLM 的 Linear（PandaGPT 原有）、LLM（Vicuna-7B）
- **新加、要訓練**：Image Decoder（4 個 Linear，產生熱圖）、Prompt Learner（小 CNN + 基礎提示向量，把熱圖翻成 LLM 讀得懂的向量）
- **不需訓練**：Memory Bank（few-shot 用，只存特徵、做比對，沒有參數）
- 架構裡沒有 Text Decoder，產生文字回答的是 LLM 本身。


### 可以借用的地方
> 災防中心的情境很像：雷達回波圖、淹水潛勢圖、雨量站網格這些「地圖型」資料，LLM 直接看圖並不可靠。可以照 AnomalyGPT 的分工：先由專用模型或規則算出異常區域（例如 1 小時雨量超過 40 毫米的格點），再把結果轉成 LLM 讀得懂的形式，最簡單的是九宮格式的方位文字（「異常集中在臺中市東北方山區」），交給 RAG 助理回答民眾問題。評估警戒判斷時，先算「全部判成不警戒」的正確率當基準線，避免因為資料不平衡高估模型。

### 疑問
> 提示學習器的 n₁、n₂ 是多少？如果把熱圖直接轉成九宮格文字放進提示（完全不訓練），正確率會差多少？AnomalyGPT 在每個類別的正確率是不是也像 PatchCore 一樣起伏很大？VisA 1-shot 各列 AUC 為什麼不同？

### 相關
- 論文：[[【Triad：Empowering LMM-Based Anomaly Detection with Expert-Guided Region-of-Interest Tokenizer and Manufacturing Process】]]（後續工作：用 MuSc 等專家找可疑區、裁原圖放大給 LLaVA，並加入製程描述；把 AnomalyGPT 列為基準，1-shot 86.1% 和本論文自己報的數字完全相同）
- 論文：[[【MMAD：A Comprehensive Benchmark for Multimodal Large Language Models in Industrial Anomaly Detection】]]（後續的評測基準：AnomalyGPT 在 MMAD 平均只有 36.52%，只會答「有沒有瑕疵」，平衡正確率 65.57%，正常品有一半被判成瑕疵）
- 概念：[[Industrial anomaly detection|工業瑕疵檢測]]、[[Vision-language model|視覺語言模型]]、[[Prompt tuning|提示微調]]、[[LoRA]]、[[AUROC]]、[[In-context learning|情境學習]]、[[Contrastive learning|對比學習]]
- 論文引用的相關工作：PatchCore（Roth et al., CVPR 2022，記憶庫最近鄰）；WinCLIP（Jeong et al., CVPR 2023，CLIP 零／少樣本瑕疵檢測）；UniAD（You et al., NeurIPS 2022，一個模型管所有類別）；NSA（Schlüter et al., ECCV 2022，自然合成瑕疵）；PandaGPT（Su et al., 2023）；ImageBind（Girdhar et al., CVPR 2023）
