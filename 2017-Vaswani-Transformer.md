---
title: "【Attention is all you need】"
tags: [paper, deep-learning, nlp, transformer, status-inbox]
concepts: ["[[Transformer]]", "[[Self-attention]]", "[[Positional encoding]]", "[[BLEU]]", "[[Backpropagation]]", "[[LoRA]]", "[[Vision-language model]]", "[[Retrieval-augmented generation]]"]
papers: []
---

# 【Attention is all you need】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 論文全名 | Attention Is All You Need |
| 作者 | Ashish Vaswani、Noam Shazeer、Niki Parmar、Jakob Uszkoreit、Llion Jones、Aidan N. Gomez、Łukasz Kaiser、Illia Polosukhin（八位同等貢獻，排名隨機；Google Brain、Google Research、多倫多大學） |
| 發表 | NIPS 2017（第 31 屆，長灘）；arXiv 1706.03762，v1 2017/6/12，目前最新 v7 2023/8/2。本筆記依 v7 |
| 連結 | [arXiv](https://arxiv.org/abs/1706.03762)・[PDF](https://arxiv.org/pdf/1706.03762v7)・[HTML v7](https://arxiv.org/html/1706.03762v7)・[官方程式碼 tensor2tensor](https://github.com/tensorflow/tensor2tensor) |
| 動畫解說 | [papers.poyuanchu.com/2017-vaswani-transformer/](https://papers.poyuanchu.com/2017-vaswani-transformer/) |
| 論文類型 | 方法論文：提出新的模型架構 Transformer |
| 狀態 | 已讀（2025/2 第一版筆記，2026/11 依新格式改寫） |
| 與我的相關度 | 4 / 5（我用的 LLM、嵌入模型、重排序模型全部是 Transformer。這篇不是能直接搬進 RAG 的方法，但讀懂注意力的計算量和位置編碼，才知道長脈絡為什麼貴、為什麼放在中間的段落容易被忽略） |
| 加入日期 | 2025-02-24（改寫 2026-11-13） |

> [!NOTE]
> **一句話**：2017 年以前，翻譯模型靠 RNN 一個字一個字往下讀，無法平行、長句子也記不住。這篇把 RNN 和 CNN 全部拿掉，**只用注意力**（attention）組出 Transformer：每個字直接看句子裡所有其他字，一次平行算完。在 WMT 2014 英翻德拿到 28.4 BLEU（比之前最好的系統高 2 以上），訓練成本只要對手的幾分之一。之後的 BERT、GPT、所有大型語言模型都是這個架構。

---

## 1. 背景：RNN 卡在「一次只能讀一個字」

### 序列轉換任務

**序列轉換**（sequence transduction）是把一串符號變成另一串符號，例如翻譯「I am a student」→「我是一個學生」。當時最好的做法是**編碼器–解碼器**（encoder–decoder）：編碼器讀完原文，壓成一組表示；解碼器再依序吐出譯文，每吐一個字都會參考已經吐出的字（稱為**自迴歸**，auto-regressive）。

### 既有做法的問題

<center><b>表 2：三種處理序列的方式</b></center>

| 方式 | 怎麼讀句子 | 問題 |
|---|---|---|
| RNN／LSTM／GRU | 一個字一個字讀，第 $t$ 步的狀態 $h_t$ 取決於 $h_{t-1}$ | 必須照順序算，**無法平行**；句子越長越慢，遠處的字容易被遺忘 |
| CNN（ByteNet、ConvS2S） | 用卷積同時處理一小段 | 可以平行，但兩個遠距離的字要經過很多層才連得起來：ConvS2S 隨距離線性增加，ByteNet 隨距離對數增加 |
| RNN＋注意力（Bahdanau 2014） | 解碼時回頭看原文的所有位置 | 解決了「壓成一個向量會丟資訊」，但底下仍是 RNN，還是慢 |

**注意力**最早是為了解決 RNN 翻譯的瓶頸：編碼器要把整句話壓成一個固定長度的向量，句子一長就會丟資訊。注意力讓解碼器在產生每個字時，回頭依相關程度加權所有原文位置。

作者的問題是：**既然注意力這麼有用，能不能乾脆只用注意力？**

---

## 2. 核心想法：只用注意力

Transformer 保留編碼器–解碼器的架構，但裡面完全不用 RNN 或卷積，改用**自注意力**（self-attention）：同一個句子裡的每個字，直接和所有其他字算關聯程度。

好處有三個（第 4 節的論證）：
1. **可以平行**：所有位置同時算，不必等前一個字
2. **任兩個字的距離都是 1**：第 1 個字和第 100 個字在同一層就直接相連，長距離關係比較好學
3. **比較好解釋**：可以畫出每個字在「看」哪些字（見第 5.4 節）

代價是：每一層要算 $n \times n$ 的關聯矩陣，句子長度 $n$ 加倍，計算量變四倍。

<center>
<img src="https://arxiv.org/html/1706.03762v7/Figures/ModalNet-21.png" alt="圖 1" width="55%">
<br>
<b>圖 1：Transformer 架構（原論文 Figure 1）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**
> 左半邊是<b>編碼器</b>，右半邊是<b>解碼器</b>，灰色大框旁的「N×」代表這一塊疊 N 次（論文 N = 6）。
> - 由下往上讀：輸入的字先變成向量（粉紅色 Input Embedding），加上<b>位置編碼</b>（圓圈裡的正弦波，⊕ 是相加），再進入灰框
> - 編碼器每一層：橘色<b>多頭注意力</b> → 黃色 Add & Norm → 藍色<b>前饋網路</b> → Add & Norm。從框外繞進 Add & Norm 的箭頭就是<b>殘差連接</b>（把輸入直接加回輸出）
> - 解碼器每一層多一塊：最下面是<b>遮罩多頭注意力</b>（只能看已經產生的字），中間的多頭注意力有一條線從編碼器頂端拉過來，這是<b>編碼器–解碼器注意力</b>（看原文）
> - 解碼器的輸入是「往右移一格的輸出」（Outputs shifted right），最後經過紫色 Linear 和綠色 Softmax 變成下一個字的機率
>
> 注意 Add & Norm 畫在每個子層<b>之後</b>（稱為 Post-LN）。後來的研究發現把 LayerNorm 移到子層之前（Pre-LN）訓練比較穩定，現在的 LLM 幾乎都用 Pre-LN。

---

## 3. 技術細節

### 3.1 縮放點積注意力（Scaled Dot-Product Attention）

注意力可以想成查資料：每個字同時扮演三個角色。

<center><b>表 3：Query、Key、Value 的意思</b></center>

| 角色 | 比喻 | 意思 |
|---|---|---|
| Query（Q，查詢） | 我要找什麼 | 目前這個字想要哪種資訊 |
| Key（K，鍵） | 每份資料的標籤 | 每個字「我能提供什麼」 |
| Value（V，值） | 資料本身 | 被取用時，實際交出去的內容 |

公式：

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V$$

步驟：
1. $QK^\top$：每個 Query 和每個 Key 做內積，得到「關聯分數」，內積越大代表越相關
2. 除以 $\sqrt{d_k}$（$d_k$ 是 Key 的維度）
3. softmax：把每一列分數變成加總為 1 的權重
4. 乘上 $V$：用權重把所有 Value 加權平均，得到這個字的新表示

<center>
<img src="https://arxiv.org/html/1706.03762v7/Figures/ModalNet-19.png" alt="圖 2a" width="18%">
&nbsp;&nbsp;&nbsp;&nbsp;
<img src="https://arxiv.org/html/1706.03762v7/Figures/ModalNet-20.png" alt="圖 2b" width="28%">
<br>
<b>圖 2：縮放點積注意力（左）與多頭注意力（右）（原論文 Figure 2）</b>
</center>

> [!NOTE]
> **圖 2 怎麼看**
> 左圖由下往上就是上面的公式：Q 和 K 先矩陣相乘（MatMul）→ 縮放（Scale，除以 $\sqrt{d_k}$）→ 遮罩（Mask，可選，解碼器用）→ SoftMax → 再和 V 矩陣相乘。右圖是多頭注意力：V、K、Q 各自經過 h 組不同的線性轉換（Linear），平行做 h 次左圖的注意力（疊在一起的紫框），結果串接（Concat）後再經過一次線性轉換。

#### 為什麼要除以 $\sqrt{d_k}$

論文的註腳 4：假設 $q$ 和 $k$ 每一維都是平均 0、變異數 1 的獨立隨機變數，那麼內積 $q \cdot k = \sum_{i=1}^{d_k} q_i k_i$ 的平均是 0、**變異數是 $d_k$**。$d_k = 64$ 時，內積的標準差是 8，分數之間差距很大，softmax 會幾乎把所有權重給最大的那一個，梯度變得極小、學不動。除以 $\sqrt{d_k}$ 把標準差拉回 1。

作者也比較了兩種注意力：**加法注意力**（用一層小網路算相似度）和**點積注意力**。理論複雜度差不多，但點積可以用高度最佳化的矩陣乘法，快很多、也省記憶體。$d_k$ 小時兩者差不多；$d_k$ 大時，不縮放的點積注意力比加法注意力差，所以要縮放。

### 3.2 多頭注意力（Multi-Head Attention）

只做一次注意力，所有關係會被平均成一種。多頭注意力把 Q、K、V 用 $h$ 組不同的矩陣投影到較小的維度，各自做注意力，再串接起來：

$$\text{MultiHead}(Q,K,V) = \text{Concat}(\text{head}_1,\dots,\text{head}_h)\,W^O, \quad \text{head}_i = \text{Attention}(QW_i^Q,\ KW_i^K,\ VW_i^V)$$

論文用 $h = 8$ 個頭，每個頭 $d_k = d_v = d_{\text{model}}/h = 512/8 = 64$ 維。因為每個頭的維度變小，**總計算量和一個 512 維的單頭差不多**。不同的頭可以學不同的關係，例如一個頭看語法、一個頭看代名詞指涉。

### 3.3 注意力在模型裡的三種用法

<center><b>表 4：三種注意力</b></center>

| 位置 | Q 來自 | K、V 來自 | 作用 |
|---|---|---|---|
| 編碼器自注意力 | 編碼器上一層 | 編碼器上一層 | 原文每個字看原文所有字 |
| 解碼器遮罩自注意力 | 解碼器上一層 | 解碼器上一層 | 譯文每個字只能看自己和前面的字（把未來位置的分數設成 $-\infty$，softmax 後權重為 0） |
| 編碼器–解碼器注意力 | 解碼器上一層 | 編碼器的最終輸出 | 譯文每個字看原文所有字，等同於傳統 seq2seq 的注意力 |

遮罩的原因：解碼器是一個字一個字產生的，訓練時不能偷看答案的後半段。

### 3.4 其他元件

- **逐位置前饋網路**（position-wise FFN）：每個位置各自通過兩層全連接網路，中間用 ReLU。$\text{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2$，輸入輸出 512 維、中間 2048 維。每個位置用同一組參數，但每一層的參數不同
- **殘差連接與層正規化**：每個子層的輸出是 $\text{LayerNorm}(x + \text{Sublayer}(x))$。為了能相加，所有子層和嵌入層的輸出都是 $d_{\text{model}} = 512$ 維
- **嵌入與 softmax**：輸入嵌入、輸出嵌入、最後的線性層三者**共用同一個權重矩陣**；嵌入層的權重乘上 $\sqrt{d_{\text{model}}}$

### 3.5 位置編碼（Positional Encoding）

自注意力只看「誰和誰相關」，不管順序：把句子裡的字打亂，算出來的結果一樣。所以要把位置資訊加進輸入：

$$PE_{(pos,\,2i)} = \sin\!\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right), \qquad PE_{(pos,\,2i+1)} = \cos\!\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)$$

$pos$ 是第幾個字，$i$ 是第幾維。每一維都是一條正弦波，**低維的波長短（變化快），高維的波長長（變化慢）**，波長從 $2\pi$ 到 $10000 \cdot 2\pi$ 呈等比級數。像時鐘的秒針、分針、時針，組合起來就能唯一表示每個位置。

選這個函數的理由：對任何固定的距離 $k$，$PE_{pos+k}$ 都可以寫成 $PE_{pos}$ 的線性函數（三角函數的和角公式），模型比較容易學會「相對位置」。作者也試了**可學習的位置嵌入**，結果幾乎一樣（表 7 的 E 列），選正弦版本是因為它可能可以**外推**到比訓練時更長的句子。

### 3.6 為什麼用自注意力（三個比較指標）

<center><b>表 5：各種層的比較（原論文 Table 1）</b></center>

| 層的類型 | 每層計算量 | 必須照順序的運算數 | 最長路徑長度 |
|---|---|---|---|
| 自注意力 | $O(n^2 \cdot d)$ | $O(1)$ | $O(1)$ |
| 循環（RNN） | $O(n \cdot d^2)$ | $O(n)$ | $O(n)$ |
| 卷積 | $O(k \cdot n \cdot d^2)$ | $O(1)$ | $O(\log_k n)$ |
| 受限自注意力（只看附近 $r$ 個字） | $O(r \cdot n \cdot d)$ | $O(1)$ | $O(n/r)$ |

$n$ 是句子長度，$d$ 是向量維度，$k$ 是卷積核大小。

- **照順序的運算數**決定能不能平行：自注意力是常數，RNN 要 $n$ 步
- **最長路徑長度**是任兩個字之間訊號要走幾步：越短，長距離關係越好學
- **計算量**：當 $n < d$ 時，自注意力比 RNN 便宜。2017 年的翻譯句子通常只有幾十個子詞，$d = 512$，條件成立

> [!WARNING]
> 「$n < d$ 時比較便宜」是 2017 年翻譯任務的前提。現在 LLM 的脈絡動輒幾萬到幾十萬個 token，$n \gg d$，$O(n^2)$ 正是最大的瓶頸。作者在結論也提到要研究「受限的局部注意力」來處理圖片、音訊、影片等長輸入，後來 Longformer、FlashAttention、各種稀疏注意力都是在解這個問題。

---

## 4. 實驗設定

<center><b>表 6：訓練設定</b></center>

| 項目 | 內容 |
|---|---|
| 英翻德資料 | WMT 2014 英德，約 450 萬句對；byte-pair encoding（BPE），共用詞彙約 37,000 個 |
| 英翻法資料 | WMT 2014 英法，3,600 萬句；word-piece 詞彙 32,000 個 |
| 批次 | 依長度分組，每批約 25,000 個原文 token 和 25,000 個譯文 token |
| 硬體 | 一台機器、8 張 NVIDIA P100 |
| 訓練時間 | base：每步 0.4 秒，10 萬步，約 12 小時；big：每步 1.0 秒，30 萬步，3.5 天 |
| 最佳化器 | Adam，$\beta_1 = 0.9$、$\beta_2 = 0.98$、$\epsilon = 10^{-9}$ |
| 學習率 | 先線性升高 4,000 步（warmup），再依步數的平方根倒數下降：$lrate = d_{\text{model}}^{-0.5} \cdot \min(step^{-0.5},\ step \cdot warmup^{-1.5})$ |
| 正則化 | 殘差 dropout 0.1（big 英德 0.3）；label smoothing 0.1（讓模型不要太有把握，困惑度變差但 BLEU 變好） |
| 推論 | beam search，beam 大小 4，長度懲罰 $\alpha = 0.6$；base 平均最後 5 個 checkpoint，big 平均最後 20 個 |
| 指標 | **BLEU**：機器譯文和人工譯文的 n-gram 重疊程度，0–100，越高越好 |

---

## 5. 結果

### 5.1 機器翻譯

<center><b>表 7：翻譯品質與訓練成本（原論文 Table 2，newstest2014）</b></center>

| 模型 | 英德 BLEU | 英法 BLEU | 英德訓練量（FLOPs） | 英法訓練量（FLOPs） |
|---|---|---|---|---|
| ByteNet | 23.75 | | | |
| Deep-Att + PosUnk | | 39.2 | | 1.0 × 10²⁰ |
| GNMT + RL | 24.6 | 39.92 | 2.3 × 10¹⁹ | 1.4 × 10²⁰ |
| ConvS2S | 25.16 | 40.46 | 9.6 × 10¹⁸ | 1.5 × 10²⁰ |
| MoE | 26.03 | 40.56 | 2.0 × 10¹⁹ | 1.2 × 10²⁰ |
| Deep-Att + PosUnk（集成） | | 40.4 | | 8.0 × 10²⁰ |
| GNMT + RL（集成） | 26.30 | 41.16 | 1.8 × 10²⁰ | 1.1 × 10²¹ |
| ConvS2S（集成） | 26.36 | 41.29 | 7.7 × 10¹⁹ | 1.2 × 10²¹ |
| **Transformer（base）** | 27.3 | 38.1 | **3.3 × 10¹⁸** | 同左 |
| **Transformer（big）** | **28.4** | **41.8** | 2.3 × 10¹⁹ | 同左 |

**集成**（ensemble）是把多個模型的預測合起來，通常比單一模型好，但成本也乘上好幾倍。

我核對的數字：
- 英德 big 比之前最好的 ConvS2S 集成高 28.4 − 26.36 = **2.04 BLEU**，符合「超過 2.0」
- 連 base（27.3）都贏過所有先前的模型和集成，訓練量只有 3.3 × 10¹⁸，是 ConvS2S 單模型的約 1/3、GNMT 集成的約 1/55
- 訓練量是作者自己估的：訓練時間 × GPU 數 × 每張 GPU 的持續算力（P100 取 9.5 TFLOPS）。base：12 小時 × 8 張 × 9.5 × 10¹² = 3.28 × 10¹⁸ ✓；big：3.5 天 × 8 張 × 9.5 × 10¹² = 2.30 × 10¹⁹ ✓
- 英法 big 41.8 比之前最好的集成 41.29 高 0.51，訓練量 2.3 × 10¹⁹ 是 MoE 單模型（1.2 × 10²⁰）的約 0.19 倍，符合「不到 1/4」

> [!WARNING]
> **英法 BLEU 前後不一**：摘要和表 2 都寫 big 模型英法是 **41.8**，但第 6.1 節正文寫「our big model achieves a BLEU score of **41.0**」。v7 沒有統一，以表格的 41.8 為準。

### 5.2 模型變化（消融實驗）

所有變化都在英德開發集（newstest2013）上測，沒有平均 checkpoint。

<center><b>表 8：架構變化（原論文 Table 3，節錄；沒寫的設定和 base 相同）</b></center>

| 列 | 改了什麼 | PPL（越低越好） | BLEU | 參數量 |
|---|---|---|---|---|
| base | N=6、d=512、FFN=2048、h=8、$d_k$=64、dropout 0.1、LS 0.1 | 4.92 | 25.8 | 6,500 萬 |
| (A) | h=1、$d_k$=512（單頭） | 5.29 | 24.9 | |
| (A) | h=4、$d_k$=128 | 5.00 | 25.5 | |
| (A) | h=16、$d_k$=32 | 4.91 | 25.8 | |
| (A) | h=32、$d_k$=16 | 5.01 | 25.4 | |
| (B) | $d_k$=16 | 5.16 | 25.1 | 5,800 萬 |
| (B) | $d_k$=32 | 5.01 | 25.4 | 6,000 萬 |
| (C) | N=2 | 6.11 | 23.7 | 3,600 萬 |
| (C) | N=8 | 4.88 | 25.5 | 8,000 萬 |
| (C) | d=1024、$d_k$=128 | 4.66 | 26.0 | 1.68 億 |
| (C) | FFN=4096 | 4.75 | 26.2 | 9,000 萬 |
| (D) | dropout 0 | 5.77 | 24.6 | |
| (D) | label smoothing 0 | 4.67 | 25.3 | |
| (E) | 可學習的位置嵌入取代正弦 | 4.92 | 25.7 | |
| big | N=6、d=1024、FFN=4096、h=16、dropout 0.3、30 萬步 | 4.33 | 26.4 | 2.13 億 |

PPL（perplexity，困惑度）是每個子詞的困惑度，衡量模型對正確答案有多不意外。

作者的結論：
- **(A) 頭數**：單頭比最好的設定差 0.9 BLEU（24.9 對 25.8），但頭太多（32）也會變差
- **(B) Key 維度**：$d_k$ 變小會變差，作者推測判斷相關性並不簡單，可能需要比內積更複雜的函數
- **(C)(D)**：模型越大越好，dropout 對防止過擬合很重要
- **(E)**：正弦和可學習的位置編碼幾乎一樣

> [!NOTE]
> 細看 (C)：N = 8（8,000 萬參數）的 PPL 4.88 比 base 低，BLEU 卻是 25.5，比 base 的 25.8 差。「越大越好」在 BLEU 上不是每一列都成立，只是整體趨勢。(D) 也看得到 label smoothing 的取捨：拿掉它 PPL 變好（4.67），BLEU 變差（25.3）。

### 5.3 英文成分句法分析

為了證明 Transformer 不只會翻譯，作者拿它做**成分句法分析**（constituency parsing：把句子拆成「名詞片語、動詞片語」的樹狀結構）。這個任務的輸出比輸入長很多、而且有嚴格的結構限制。用 4 層 Transformer、$d_{\text{model}} = 1024$，在 Penn Treebank 的 WSJ 部分（約 4 萬句）訓練，幾乎不調參數。

<center><b>表 9：句法分析結果（原論文 Table 4，WSJ 第 23 節 F1）</b></center>

| 方法 | 訓練方式 | F1 |
|---|---|---|
| Vinyals & Kaiser（2014） | 只用 WSJ，判別式 | 88.3 |
| Petrov et al.（2006） | 只用 WSJ，判別式 | 90.4 |
| Zhu et al.（2013） | 只用 WSJ，判別式 | 90.4 |
| Dyer et al.（2016，RNNG） | 只用 WSJ，判別式 | **91.7** |
| **Transformer（4 層）** | 只用 WSJ，判別式 | 91.3 |
| Zhu et al.（2013） | 半監督 | 91.3 |
| Huang & Harper（2009） | 半監督 | 91.3 |
| McClosky et al.（2006） | 半監督 | 92.1 |
| Vinyals & Kaiser（2014） | 半監督 | 92.1 |
| **Transformer（4 層）** | 半監督（約 1,700 萬句） | 92.7 |
| Luong et al.（2015） | 多任務 | 93.0 |
| Dyer et al.（2016，RNNG） | 生成式 | **93.3** |

> [!WARNING]
> **「除了 RNNG 以外都贏」少算了一個**：正文說 Transformer「優於所有先前的模型，只輸給 Recurrent Neural Network Grammar」，但表上 Luong et al.（2015）多任務的 93.0 也高於 Transformer 半監督的 92.7。比較公平的說法是：在相同訓練方式下（只用 WSJ 或半監督），只輸給 RNNG。

另外值得注意：只用 4 萬句訓練時，RNN 的 seq2seq 模型（Vinyals & Kaiser，88.3）連傳統的 Berkeley Parser（90.4）都贏不了，Transformer 則可以（91.3）。

### 5.4 注意力視覺化（附錄）

<center>
<img src="https://arxiv.org/html/1706.03762v7/making_more_difficult5_new.svg" alt="圖 3" width="85%" style="background:#fff">
<br>
<b>圖 3：「making」的注意力，編碼器第 5 層（原論文 Figure 3）</b>
</center>

> [!NOTE]
> **圖 3 怎麼看**
> 句子是「It is in this spirit that a majority of American governments have passed new laws since 2009 making the registration or voting process more difficult.」上排灰色標出「making」，往下的線是它在各個頭的注意力，每種顏色是一個頭（左上色票，共 8 個頭），線越深代表權重越大。最粗的幾條（棕色、灰色）連到很遠的「more」「difficult」，代表模型學會了「making…more difficult」這個跨越 7 個字的片語結構。下方的色塊是每個頭在各位置的權重。原圖是透明背景，深色模式下文字會看不見，所以這裡加了白底。

<center>
<img src="https://arxiv.org/html/1706.03762v7/anaphora_resolution_new.svg" alt="圖 4" width="80%" style="background:#fff">
<br>
<b>圖 4：代名詞「its」的注意力（原論文 Figure 4 上半）</b>
</center>

> [!NOTE]
> **圖 4 怎麼看**
> 句子是「The Law will never be perfect, but its application should be just - this is what we are missing, in my opinion.」這是第 5 層第 5 個頭的完整注意力：上排每個字連到下排它注意的字。原論文 Figure 4 的下半（HTML 版另一張圖）只畫「its」：第 5 頭幾乎全部看向「Law」，第 6 頭看向「Law」和「application」，而且權重非常集中。也就是模型自己學會了代名詞「its」指的是「Law」，作者稱為指代消解（anaphora resolution）。

<center>
<img src="https://arxiv.org/html/1706.03762v7/attending_to_head_new.svg" alt="圖 5" width="80%" style="background:#fff">
<br>
<b>圖 5：不同的頭學到不同的結構（原論文 Figure 5 上半）</b>
</center>

> [!NOTE]
> **圖 5 怎麼看**
> 同一句話、同一層的另一個頭。原論文 Figure 5 有兩個頭：綠色的頭有很多線集中到「Law」「application」「missing」「opinion」等關鍵字；紅色的頭（HTML 版的另一張圖）幾乎都是連到相鄰的字，像在看前後文。作者的結論是「不同的頭清楚地學會了不同的工作」。這只是挑出來的例子，沒有量化分析。

---

## 6. 總結

1. RNN 必須照順序計算，無法平行，長距離關係也難學
2. Transformer 完全拿掉 RNN 和卷積，只用注意力：$\text{softmax}(QK^\top/\sqrt{d_k})V$
3. 多頭注意力讓模型同時學多種關係；位置編碼補回順序資訊；殘差、LayerNorm、FFN 組成每一層
4. 英德 28.4 BLEU、英法 41.8 BLEU，都是當時最佳，訓練成本是對手的幾分之一；也能做句法分析
5. 代價是計算量隨長度平方成長，2017 年的短句子不是問題，長脈絡時就是

> [!WARNING]
> **論文本身的小錯**
> - 英法 BLEU：摘要和表 2 是 41.8，第 6.1 節寫 41.0
> - 句法分析「只輸給 RNNG」漏了 Luong et al. 的 93.0
> - 第 5.1 節「byte-pair encoding [3]」引錯文獻：[3] 是 Britz et al. 的 NMT 架構探索，BPE 應該是 [31] Sennrich et al.
> - 消融只跑一次、沒有誤差範圍；BLEU 差 0.1–0.3 的變化（例如 h = 16 和 h = 8）不一定有意義
> - 架構圖是 Post-LN，後續研究（Xiong et al., 2020）發現 Pre-LN 更容易訓練、可以不用 warmup

> [!TIP]
> **可以帶走的觀念**
> - **注意力的計算量是長度的平方**：RAG 塞進去的段落越多，LLM 的成本和延遲不是線性增加。先用檢索和重排序把段落篩少，比把整份文件丟進去划算
> - **位置很重要**：Transformer 本身不知道順序，全靠位置編碼。後來的研究也發現 LLM 對放在中間的資訊比較不敏感（lost in the middle），重要的段落放在開頭或結尾
> - **縮放的道理到處適用**：內積的變異數隨維度成長，所以要除以 $\sqrt{d}$。自己算向量相似度、做 softmax 時也要注意尺度
> - **多頭＝多種視角**：同樣的計算量，拆成 8 個 64 維的頭比 1 個 512 維的頭好（表 8 A 列）
> - **用消融實驗說明每個零件的價值**：表 3 一次只改一個設定，是寫論文時值得模仿的做法

---

## 7. 我的筆記

### 重點筆記
> 以下整理自我 2025/2 的第一版筆記（完整原文保存在本機 `2017-Vaswani-Transformer.v1.md`）：
> - Q = 目前這個字；K = Q 拿來比對的對象；V = 被取用時交出的內容（K 的原始值）
> - 編碼器自注意力：Q、K、V 都來自輸入自己。步驟是算注意分數 → softmax 變成分布 → 對 V 加權平均得到 context vector
> - 編碼器–解碼器注意力：Q 來自輸出，K、V 來自輸入
> - 解碼器的遮罩：預測時不能看到後面的字（未來資料）
> - 每個子層後面都有 Add & Norm：Add 是殘差連接，Norm 是層正規化
> - 「The animal didn't cross the street because it was too tired.」：自注意力可以讓「it」連到「animal」，多頭還會連到「tired」。這個例子出自 Google 的部落格文章，論文本身用的是「its → Law」（圖 4）
> - 第一版用了「Multi-Header Attention」，正確名稱是 Multi-Head Attention

<center><b>表 10：LSTM 與 Transformer 的比較（第一版筆記）</b></center>

| 比較項目 | LSTM | Transformer |
|---|---|---|
| 核心技術 | 門控機制（遺忘門、輸入門、輸出門） | 自注意力 |
| 長距離依賴 | 靠記憶單元，序列變長效果仍會衰減 | 任兩個位置直接相連，路徑長度是常數 |
| 計算效率 | 遞迴結構，無法平行 | 可以平行，但計算量隨長度平方成長 |
| 模型結構 | 遞迴 | 非遞迴，靠位置編碼補順序 |
| 訓練 | 容易梯度消失或爆炸 | 訓練較快，需要 warmup 等技巧 |

### 可以借用的地方
> 做國家災防中心的 RAG 時，Transformer 的平方成本直接決定架構：與其把整份災情報告塞給 LLM，不如先用嵌入模型（也是 Transformer 編碼器）檢索、用交叉編碼器重排序，只把最相關的幾段放進提示詞，而且把最重要的段落放在開頭。選嵌入模型時也要看它的最大長度（位置編碼決定的上限），超過就會被截斷。

### 疑問
> 正弦位置編碼號稱可以外推到更長的句子，但實務上效果有限，後來的 RoPE、ALiBi 是怎麼改進的？為什麼現在的 LLM 幾乎都只用解碼器（GPT 式），不用完整的編碼器–解碼器？

### 相關
- 概念：[[Transformer]]、[[Self-attention|自注意力]]、[[Positional encoding|位置編碼]]、[[BLEU]]、[[Backpropagation|反向傳播]]、[[LoRA]]（在 Transformer 的注意力權重上做低秩微調）、[[Vision-language model|視覺語言模型]]（把 Transformer 用到圖片）、[[Retrieval-augmented generation|RAG]]
- 前身：Bahdanau et al., *Neural Machine Translation by Jointly Learning to Align and Translate*（2014，第一個注意力翻譯模型）；ConvS2S（Gehring et al., 2017）；ByteNet（Kalchbrenner et al., 2016）
- 後續：BERT（2018，只用編碼器）、GPT 系列（只用解碼器）、ViT（2020，用在圖片）；Pre-LN（Xiong et al., 2020）、FlashAttention（2022）
