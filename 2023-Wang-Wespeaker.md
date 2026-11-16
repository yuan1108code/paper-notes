---
title: "【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】"
tags: [paper, speech, speaker-embedding, speaker-verification, toolkit, status-inbox]
concepts: ["[[Speaker embedding]]", "[[Speaker verification]]", "[[Speaker diarization]]", "[[ECAPA-TDNN]]", "[[DER]]", "[[AAM-softmax]]"]
papers: ["[[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]", "[[【Powerset multi-class cross entropy loss for neural speaker diarization】]]"]
---

# 【Wespeaker：A Research and Production Oriented Speaker Embedding Learning Toolkit】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 作者 | Hongji Wang, Chengdong Liang, Shuai Wang, Zhengyang Chen, Binbin Zhang, Xu Xiang, Yanlei Deng, Yanmin Qian（上海交通大學 X-LANCE、騰訊、NVIDIA、WeNet 開源社群等） |
| 發表 | ICASSP 2023；arXiv 2022/10/31 |
| 連結 | [arXiv](https://arxiv.org/abs/2210.17016)・[DOI](https://doi.org/10.1109/ICASSP49357.2023.10096626)・[GitHub](https://github.com/wenet-e2e/wespeaker)・[線上 Demo](https://huggingface.co/spaces/wenet/wespeaker_demo) |
| 引用（IEEE） | H. Wang et al., "Wespeaker: A research and production oriented speaker embedding learning toolkit," in *Proc. IEEE Int. Conf. Acoust., Speech Signal Process. (ICASSP)*, Rhodes Island, Greece, 2023, pp. 1–5, doi: 10.1109/ICASSP49357.2023.10096626. |
| 動畫解說 | [papers.poyuanchu.com/2023-wang-wespeaker/](https://papers.poyuanchu.com/2023-wang-wespeaker/) |
| 論文類型 | 工具包（toolkit）論文：介紹一套開源軟體的設計與基準結果，不是提出單一新演算法 |
| 狀態 | 待讀 |
| 與我的相關度 | 3 / 5（語音領域；但「研究到部署」的工程設計、大量資料的管理方式，和工業 AI 落地很像） |
| 加入日期 | 2026-11-11 |

> [!NOTE]
> **一句話**：Wespeaker 是一套專門用來訓練「聲紋向量」（speaker embedding）的開源工具，把比賽冠軍系統用的技巧整理成乾淨的程式碼，而且訓練好的模型可以**直接匯出、部署到正式環境**，縮短研究和產品之間的距離。

---

## 1. 背景：什麼是聲紋向量？為什麼需要新工具？

### 聲紋向量（speaker embedding）

把一段人說話的聲音，壓縮成一個固定長度的數字向量（例如 256 維），讓**同一個人的聲音向量彼此靠近、不同人的離得遠**。可以把它想成「聲音的指紋」。

有了這個向量，很多工作就變簡單了：

- **語者驗證**（speaker verification）：這段聲音是不是本人？算兩個向量的相似度，超過門檻就判定是同一人。常見的比對方法有餘弦相似度（cosine similarity）和 PLDA（一種統計式的打分模型）
- **語者分段**（speaker diarization）：一段錄音裡誰在什麼時候說話？把每一小段的聲紋向量分群
- 其他：語音辨識的說話者適應、語音合成和變聲時指定「像誰的聲音」

> [!TIP]
> [Powerset 語者分段論文](https://hackmd.io/@Yuan1108/H1RfDsbAMe)的流程第二步就是「用 ECAPA-TDNN 抽聲紋向量再分群」。Wespeaker 就是專門把這一步做好、做快的工具。

### 現有工具的問題

作者指出三個缺口：

1. **論文和比賽結果對不上**：比賽冠軍系統用了很多工程技巧，但這些技巧很少寫進論文，一般研究者重現不出同樣的成績
2. **現有開源工具偏研究**：例如 SpeechBrain、ESPnet 是「什麼語音任務都做」的大型工具，程式碼複雜，而且**沒有考慮怎麼搬到正式產品上**
3. **舊工具依賴重**：早期的 Kaldi 很強，但安裝和使用門檻高

所以作者參考自家做語音辨識的 WeNet（強調「研究到部署」），做了一套專注於聲紋向量的工具。名字裡的 "We" 取自 WeChat，代表連結與分享。

---

## 2. Wespeaker 的六個特色

1. **成績有競爭力**：在 VoxCeleb、CNCeleb、VoxConverse 上都達到很好的結果，並重新實作了多項比賽冠軍系統的技巧
2. **輕量**：只依賴 PyTorch 生態系，**不需要 Kaldi**
3. **統一資料讀取（UIO）**：從幾小時到幾百萬小時的資料，都用同一套介面讀取
4. **即時特徵處理**：切段、資料增強、特徵擷取都在訓練時即時做，不先存到硬碟
5. **分散式訓練**：用 PyTorch 的 DistributedDataParallel，支援多台機器、多張 GPU
6. **可直接上線**：模型能匯出成 TorchScript（JIT）或 ONNX 格式，附部署範例程式

---

## 3. 整體架構

<center>
<img src="https://arxiv.org/html/2210.17016v2/figs/wespeaker.png" alt="圖 1" width="85%">
<br>
<b>圖 1：Wespeaker 的整體架構與流程（原論文 Figure 1）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**
> 照箭頭方向讀，顏色代表三個階段：
> - **藍色：資料端**。左上「Data preparation」準備資料，有兩種存法：raw（一個個小檔案）或 shard（打包成大檔）。接著「On-the-fly Feature Preparation」在訓練時即時切段、做資料增強、擷取特徵
> - **綠色：模型端**。「Model training」訓練完後，往左到「Model Export and deployment」匯出並部署
> - **橘色與灰色：應用端**。部署好的模型負責「Speaker Embedding Extraction」抽出聲紋向量，再分給下游任務：**語者驗證**、**語者分段**，以及最下面那條沒有標名字的箭頭，代表其他還沒列出的應用
>
> 重點是這是**一條從資料到上線的完整流程**，不是只管訓練。

一個標準的聲紋向量模型由四部分組成：

1. **幀級別層**（frame-level layers）：一小段一小段（例如每 10 毫秒）處理聲音特徵
2. **池化層**（pooling）：把整段話所有小段的資訊彙總成一個向量。Wespeaker 支援統計式（算平均和標準差）和注意力式（讓模型自己決定哪幾段比較重要）
3. **段級別轉換層**：再經過一兩層，把向量映射到說話者類別
4. **損失函數**：訓練時讓模型學會分辨不同說話者

---

## 4. 資料管理：這篇最工程、也最值得學的部分

### 4.1 統一資料讀取（Unified IO，UIO）

<center>
<img src="https://arxiv.org/html/2210.17016v2/figs/uio.png" alt="圖 2" width="70%">
<br>
<b>圖 2：Wespeaker 的統一資料讀取系統（原論文 Figure 2）</b>
</center>

> [!NOTE]
> **圖 2 怎麼看**
> 最上面是統一的入口「Unified IO System」，下面依資料量分成左右兩條路：
> - **左邊：小資料（raw）**。直接讀本機硬碟上的原始檔案，只需要兩個清單檔：wav.scp（每個音檔的路徑）和 utt2spk（每個音檔是誰說的）
> - **右邊：大資料（shard）**。把大量小音檔打包成 1.tar、2.tar……N.tar 這樣的大檔，每個 tar 裡面是「音檔 + 說話者編號」（下方小框：1.wav id1、2.wav id2……）。這些大檔可以放在本機，也可以放在雲端儲存（S3、OSS、HDFS 等）
>
> 對訓練程式來說，**不管資料多大、放在哪裡，讀法都一樣**。

為什麼要打包？正式產品等級的語料常有數萬小時，由幾百萬個小檔案組成。一次開那麼多小檔案，容易**記憶體不足**或**讀取太慢**拖累訓練。把它們用 tar 打包成大檔、訓練時按順序邊讀邊解壓，就能避免這些問題。這個做法參考了 TensorFlow 的 TFRecord 和 AIStore。

> [!TIP]
> 這和工業場域的大量感測器資料、影像資料很像：檔案很多很小時，**先打包成分片（shard）再串流讀取**，是很通用的解法，不限於語音。

### 4.2 即時特徵處理（On-the-fly feature preparation）

傳統做法是**離線**的：先把資料重新取樣、做資料增強、切段、擷取特徵，全部存成新檔案，訓練時再讀。Wespeaker 改成**訓練時即時處理**，好處有兩個：

1. **省硬碟空間**：不用另外存增強過的音檔和特徵
2. **模型更穩健**：每一輪（epoch）看到的增強結果都不一樣，等於資料變多了，模型比較不容易只記住訓練資料

<center>
<img src="https://arxiv.org/html/2210.17016v2/figs/feat_pre.png" alt="圖 3" width="95%">
<br>
<b>圖 3：即時特徵處理的流水線（原論文 Figure 3）</b>
</center>

> [!NOTE]
> **圖 3 怎麼看**
> 每個方框是資料流過該步驟後的樣子，框裡列出每筆資料目前有哪些欄位；寫著「same」代表欄位沒有增加，只是內容被改了。**紅字**標出新加入的欄位。顏色代表階段：
> - **綠色到藍色：分配資料**。資料（shard 檔或原始檔清單）先依機器數量分好（Distributed Partition），每台機器只拿自己那一份
> - **黃色：讀進來**。解開 tar（Untar）或從硬碟讀取後，每筆資料有 key（編號）、spk（說話者）、wav（聲音）、sample_rate（取樣率）
> - **橘色：處理聲音**。依序是：局部打亂（Local Shuffle）→ 把說話者名字轉成數字編號（Spk2id，新增紅字 label）→ 重新取樣（Resample）→ 調整語速（Speed Perturb）→ 隨機切成固定長度（Random Chunk）→ 加雜訊或回音（Noise/Reverb）
> - **紫色：轉成特徵**。擷取濾波器組特徵（Compute Fbank，這時 wav 被換成紅字 feat）→ 正規化（CMVN）→ 頻譜遮罩增強（SpecAug）
> - **粉紅色：組成批次**。最後打包成訓練用的一批資料，只剩 keys、feats、labels

每個步驟的白話說明：

| 步驟 | 做什麼 |
|---|---|
| Local Shuffle | 在一個小緩衝區裡打亂順序，讓每一輪資料順序不同 |
| Spk2id | 把說話者名字對應成 0 到 N−1 的編號 |
| Resample | 統一取樣率 |
| Speed Perturb | 以一定機率把語速調快或調慢。語速改變會讓音高跟著變，所以**調速後的聲音被當成新的說話者** |
| Random Chunk | 隨機切成一樣長，太短就補零 |
| Noise / Reverb | 加背景雜訊或房間回音，模擬真實環境 |
| Compute Fbank | 從原始聲波算出濾波器組特徵（模仿人耳對不同頻率的感受） |
| CMVN | 每段話各自做平均與變異數正規化，減少錄音設備差異 |
| SpecAug | 在頻譜上隨機遮掉一些時間或頻率區塊；預設關閉，因為和其他增強方式常不相容 |
| Batch | 組成固定大小的批次 |

---

## 5. 模型、損失函數與訓練技巧

### 支援的模型

| 模型 | 說明 |
|---|---|
| TDNN（x-vector） | 開啟深度聲紋向量時代的里程碑模型 |
| ResNet（r-vector）與更深的版本 | VoxSRC 2019 與 CNSRC 2022 比賽的最佳系統 |
| ECAPA-TDNN | TDNN 的改良版，VoxSRC 2020 冠軍系統 |

### 損失函數

除了一般的 softmax 交叉熵，還支援幾種**加邊界**（margin）的版本：A-softmax、AM-softmax、AAM-softmax。

白話說：一般的分類只要求「分對」；加邊界的版本要求「**分對，而且要分得夠開**」。訓練時故意把正確答案的分數扣掉一點（邊界 margin），逼模型把不同說話者推得更遠，抽出來的聲紋向量更好區分。

### 訓練技巧

**學習率排程**：先暖身再遞減。前 $T_{warm}$ 步學習率從 0 線性升高（暖身），之後從初始值 $\eta_0$ 以指數方式下降到最終值 $\eta_T$。

$$lr(t) = g(t)\,h(t),\quad g(t)=\begin{cases}\dfrac{t}{T_{warm}}, & t<T_{warm}\\ 1, & T_{warm}\le t<T\end{cases},\quad h(t)=\eta_0\cdot\exp\!\left(\frac{t}{T}\ln\frac{\eta_T}{\eta_0}\right)$$

**邊界排程**：邊界 margin 不是一開始就加滿，而是分三段。一開始是 0，讓模型先學會基本分類；中間逐漸（線性或對數）增加；最後固定在目標值 $M$。

$$m(t)=\begin{cases}0, & t<T_1\\ f(t), & T_1\le t<T_2\\ M, & T_2\le t<T\end{cases}$$

**大邊界微調**（large margin fine-tuning）：模型訓練好之後，再多一個微調階段，用**更大的邊界（0.5）和更長的訓練片段（6 秒）**。這是多個比賽冠軍系統的常用技巧。

### 打分與部署

- **打分方式**：用加邊界損失訓練的聲紋向量，直接用**餘弦相似度**就很好用；也支援較傳統的 PLDA，以及分數正規化（AS-Norm）來校正分數
- **部署**：模型可匯出成 ONNX 或 TensorRT 格式，放到 NVIDIA 的 Triton 推論伺服器上；也提供 Python 套件，用 `pip` 安裝就能直接用預訓練模型

> [!TIP]
> 「訓練完能直接匯出 ONNX、丟到 Triton、或包成 pip 套件」這種**從第一天就考慮部署**的設計，對工業 AI 專案很有參考價值：很多研究原型卡在「做得出來，但搬不到現場」。

---

## 6. 實驗設定

所有實驗共用的設定：

| 項目 | 設定 |
|---|---|
| 資料讀取 | shard 模式的 UIO |
| 雜訊 / 回音 | MUSAN 資料集當背景雜訊、模擬的房間脈衝響應當回音；每段話有 60% 機率加其中一種（不同時加） |
| 調速 | 0.9 倍、1.0 倍、1.1 倍各三分之一；調速後視為新說話者 |
| 特徵 | 80 維對數梅爾濾波器組（Fbank），每 10 毫秒一幀、視窗 25 毫秒 |
| 切段 | 每段 200 幀（約 2 秒），只做平均正規化 |
| SpecAug | 所有實驗都沒用 |
| 語者驗證的打分 | 大邊界微調後，用餘弦相似度 + AS-Norm |

評估指標：

- **EER**（等錯誤率）：調整門檻，讓「把別人誤認成本人」和「把本人拒絕」兩種錯誤一樣多時的錯誤率。越低越好
- **minDCF**（最小檢測成本）：依實際應用給兩種錯誤不同權重後的最低成本。越低越好
- **DER**（語者分段錯誤率）：漏抓（MISS）、誤報（FA）、認錯人（SC）三種錯誤的總和。越低越好

---

## 7. 結果

### 7.1 語者驗證：VoxCeleb

訓練資料是 VoxCeleb2 dev（5,994 位說話者、超過 100 萬段音檔）。測試用 VoxCeleb1 的三種測試清單：O（原始）、E（擴充，更多比對組合）、H（困難，同國籍同性別互比）。

<center><b>表 2：VoxCeleb 上不同模型的結果（原論文 Table 1；數字越低越好）</b></center>

| 模型 | Vox1-O EER(%) | Vox1-O minDCF | Vox1-E EER(%) | Vox1-E minDCF | Vox1-H EER(%) | Vox1-H minDCF |
|---|---|---|---|---|---|---|
| TDNN | 1.590 | 0.166 | 1.641 | 0.170 | 2.726 | 0.248 |
| ECAPA-TDNN（原論文） | 0.870 | 0.107 | 1.120 | 0.132 | 2.120 | 0.210 |
| ECAPA-TDNN（Wespeaker） | 0.728 | 0.099 | 0.929 | 0.100 | 1.721 | 0.169 |
| ResNet34（原論文） | 1.310 | 0.154 | 1.380 | 0.163 | 2.500 | 0.233 |
| ResNet34（Wespeaker） | 0.723 | 0.069 | 0.867 | 0.097 | 1.532 | 0.146 |
| ResNet221 | 0.505 | 0.045 | 0.676 | 0.067 | 1.213 | 0.111 |
| ResNet293 | **0.447** | **0.043** | **0.657** | **0.066** | **1.183** | **0.111** |

重點：

- 同樣的模型，Wespeaker 重新實作後**都比原論文好**。例如 ResNet34 在 Vox1-O 的 EER 從 1.310% 降到 0.723%，幾乎少一半；這就是「把比賽技巧補齊」帶來的差距
- **ResNet 越深越好**：從 34 層到 293 層，Vox1-O 的 EER 從 0.723% 降到 0.447%

### 7.2 語者驗證：CNCeleb

CNCeleb 是中文名人資料集，涵蓋唱歌、訪談、直播等多種場景，比 VoxCeleb 難。訓練用 CNCeleb2 的 1,996 人加上 CNCeleb1 dev 的 797 人。

資料特殊處理：很多錄音**短於 2 秒**，所以先把同一人、同類型場景的短音檔接起來，湊成 5 秒以上。測試時每位說話者有多段註冊錄音，把它們的聲紋向量**取平均**當作這個人的代表，作者說這明顯提升了表現。

<center><b>表 3：CNCeleb 測試集結果（原論文 Table 2）</b></center>

| 模型 | EER(%) | minDCF |
|---|---|---|
| TDNN | 8.960 | 0.446 |
| ECAPA-TDNN | 7.395 | 0.372 |
| ResNet34（ASV-Subtools 工具包） | 9.141 | 0.463 |
| ResNet34（Wespeaker） | 6.492 | 0.354 |
| ResNet221 | **5.655** | **0.330** |

同樣是 ResNet34，Wespeaker 的 EER（6.492%）比另一個開源工具 ASV-Subtools（9.141%）低很多。

### 7.3 語者分段：VoxConverse

這個實驗示範怎麼把訓練好的聲紋模型拿去做語者分段：用預訓練的 ResNet34 抽聲紋向量，再用譜分群（spectral clustering）分出說話者。VoxConverse 是從 YouTube 影片收集的「真實環境」資料。

<center><b>表 4：VoxConverse 開發集的語者分段結果（原論文 Table 3）</b></center>

| 系統 | 漏抓 MISS(%) | 誤報 FA(%) | 認錯人 SC(%) | DER(%) |
|---|---|---|---|---|
| 資料集原論文基準（系統偵測語音段） | 2.4 | 2.3 | 3.0 | 7.7 |
| Wespeaker（系統偵測語音段） | 4.4 | 0.6 | 2.1 | 7.1 |
| Wespeaker（使用正確的語音段標註） | 2.3 | 0.0 | 1.9 | **4.2** |

- 「系統偵測語音段」是用 Silero-VAD 自動判斷哪裡有人說話；「正確標註」則是直接給標準答案，代表語音偵測完全正確時的上限
- 同樣用自動偵測，Wespeaker 的 DER 7.1% 比基準 7.7% 低；**認錯人的比例（SC）從 3.0% 降到 2.1%**，這直接反映聲紋向量品質比較好
- 但漏抓變多（2.4% → 4.4%），主要是語音偵測那一步造成的，不是聲紋模型的問題

> [!WARNING]
> - 這是**工具包論文**，重點是「把已知方法實作好、整理好」，不是提出新演算法；和原論文比較的進步，主要來自訓練技巧和工程細節
> - 受限於篇幅，論文只列主要結果；消融實驗（拿掉某個技巧會差多少）、PLDA 的結果都放在 GitHub 的 recipe 裡，**論文本身看不出每個技巧各貢獻多少**
> - 語者分段只測了 VoxConverse 開發集，而且漏抓率偏高
> - PLDA 目前只支援 two-covariance 版本

---

## 8. 總結與未來方向

1. Wespeaker 是一套**專注於聲紋向量**、程式碼輕量的開源工具
2. 重新實作比賽冠軍技巧後，成績**超越各模型的原始論文**，ResNet 加深後效果更好
3. **資料管理**是亮點：UIO 讓幾小時到幾百萬小時的資料用同一套方式讀取，即時特徵處理省空間又讓模型更穩健
4. **可以直接上線**：匯出 ONNX / TensorRT、部署到 Triton，或用 pip 安裝使用

作者規劃的下一步：

- 自監督學習（不用標註資料也能學聲紋）
- 給資源有限設備用的小模型
- 持續加入新模型、新的打分方法並優化訓練策略

> [!TIP]
> **可以帶走的思路**
> - **研究和部署一起設計**：從一開始就規劃匯出格式、推論伺服器和套件化，才不會做完研究還要重寫一次
> - **大量小檔案先打包成分片再串流讀取**，適用於任何大規模資料訓練
> - **資料增強放在訓練時即時做**，省空間，也讓每一輪的資料都不一樣
> - **比賽和論文之間的差距常常在工程細節**：學習率暖身、邊界排程、大邊界微調、分數正規化，單看都不起眼，加起來效果很大

---

## 9. 我的筆記

### 重點筆記
>

### 可以借用的地方
> UIO 分片讀取能不能用在工研院的大量感測器或影像資料？ONNX + Triton 的部署流程能不能套用到工業 AI 模型？

### 疑問
>

### 相關
- 概念：[[Speaker embedding|聲紋向量]]、[[Speaker verification|語者驗證]]、[[Speaker diarization|語者分段]]、[[ECAPA-TDNN]]、[[DER]]、[[AAM-softmax]]
- 相關論文：[[【pyannote.audio 2.1 speaker diarization pipeline：principle, benchmark, and recipe】]]（另一套語者分段工具，第二步同樣用 ECAPA-TDNN 抽聲紋）、[[【Powerset multi-class cross entropy loss for neural speaker diarization】]]
- 姊妹工具：WeNet（同團隊的端到端語音辨識工具）
- 經典模型：x-vector（Snyder et al., ICASSP 2018）、ECAPA-TDNN（Desplanques et al., Interspeech 2020）
