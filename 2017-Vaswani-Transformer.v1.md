---
hackmd_id: "EI1diG4NQmiSjsUNX_tXtQ"
hackmd_url: "https://hackmd.io/@Yuan1108/rJFiMvY5Jx"
last_synced: "2026-11-11 14:29:02"
---

# 【Attention is all you need】

## 介紹
 Transformer 是一種深度學習模型，特別適用於自然語言處理（NLP）和其他序列建模任務，它在NLP領域的出現引發了巨大的關注，並在各種NLP任務中取得了優異的表現，如機器翻譯、語言建模和文本生成。

### Transformer哪裡來？
Transformer是由Google於2017年首次提出的深度學習模型，用於處理序列數據特別是自然語言文本，傳統的序列建模方法，如循環神經網絡（RNN）和卷積神經網絡（CNN），存在一些限制，如難以處理長距離依賴關係和並行處理，Transformer通過引入自注意力機制（Self-Attention）來解決這些問題，使得模型能夠在不同位置之間建立依賴關係，並實現高效的並行計算。

![截圖 2025-02-24 上午11.09.12](https://hackmd.io/_uploads/HkXJmDt5Jg.png)

簡化的版本

![image](https://hackmd.io/_uploads/Hya_EDY9kg.png)

<br>

## Attention 機制
Attention 機制 最早被提出來解決機器翻譯中的編碼器-解碼器架構的瓶頸問題。在傳統的遞迴神經網路（RNN）中，編碼器需要將整個輸入序列壓縮為一個固定維度的上下文向量（Context Vector），解碼器在生成輸出時依賴於這個向量。然而，當輸入序列過長時，這種做法會導致關鍵訊息丟失，尤其是長距離依賴的部分無法被有效捕捉。

- Attention 機制提供了解決方案：在解碼每個輸出時，模型不再只依賴單一的上下文向量，而是根據當前的解碼狀態，對輸入序列中所有位置進行加權計算。具體來說，Attention 機制會計算當前輸出步驟的 **查詢（Query）與輸入序列中每個位置的鍵（Key）之間的相似度，然後根據相似度來對每個位置的值（Value）** 進行加權，最終生成當前步驟的輸出。

- Attention 機制的關鍵步驟：
    1. 計算查詢（Query）與所有鍵（Key）之間的相似度。
    2. 根據相似度加權輸入序列的值（Value）。
    3. 使用加權總和來生成輸出。

## Self-Attention 機制
Self-Attention 機制（自注意力機制）是 Attention 機制的一個特殊變形，廣泛應用於 Transformer 模型中。與 Attention 機制不同，Self-Attention 的作用對象是同一個序列中的不同位置，而不是處理兩個不同序列之間的關係。它的目的是讓序列中的每個位置能夠「關注」到序列中的其他位置，進而捕捉序列內部的長距離依賴性。

在 Self-Attention 機制中，序列中的每個元素同樣被轉換為查詢（Query）、鍵（Key）和值（Value），但這次它們全部來自同一個序列。通過計算每個元素與序列中所有其他元素的相似度，模型可以對整個序列進行加權和總結，從而生成每個位置的最終表示。

Self-Attention 機制的計算步驟：
1. 為序列中的每個元素生成查詢（Query）、鍵（Key）和值（Value）。
2. 計算每個元素的查詢與序列中其他元素的鍵之間的相似度。
3. 根據相似度加權每個元素的值。
4. 將加權總和作為當前元素的最終表示。


## 實際應用中的差異
### Attention 機制與 Self-Attention 機制的比較

<center>
    
| 比較項目 | Attention | Self-Attention |
| :--: | :--: | :--: |
| 處理對象 | 兩個不同的序列（如輸入序列與輸出序列） | 同一個序列中的不同位置 |
| 應用場景 | 編碼器-解碼器結構，如機器翻譯和對話生成 | 單一序列的內部關聯，如語言模型、文本分類等 |
| 關注範圍 | 根據查詢，動態調整對輸入序列不同位置的關注 | 每個位置對整個序列進行自我關注，捕捉序列內部的長距依賴性 |
| 平行計算 | 取決於使用的模型架構（如 RNN 無法平行，但可結合 Attention）| 可完全平行計算，特別適合於 Transformer 這類架構 |
| 長距離依賴性 | 依賴 Attention 機制的設計來解決長距離依賴問題 | 自然處理長距依賴，因為每個位置都可以關注到其他所有位置 |
| 使用的模型	| 常見於有編碼器和解碼器的模型，如 Seq2Seq | 廣泛應用於 Transformer 等模型 |
    
</center>

### LSTM vs Transformer 的比較

<center>

| 比較項目 | LSTM | Transformer |
| :--: | :--: | :--: |
| 核心技術 | 門控機制（遺忘門、輸入門、輸出門） | 自注意力機制（Self-Attention）|
| 長距依賴性	| 可以通過記憶單元保留長期依賴訊息，但隨著序列加長，效果仍會衰減 | 自注意力機制能自然捕捉長距離依賴，不受序列長度的影響 | 
| 計算效率 | 由於遞迴結構，無法進行平行運算，效率相對較低	| 平行處理，適合大規模長序列數據的處理 |
| 模型結構 | 基於 RNN 的遞迴結構 | 基於注意力的非遞迴結構 |
| 訓練難度 | 更容易面臨梯度消失或梯度爆炸問題 | 訓練速度更快，無序列順序依賴，適合大規模數據集 |
    
</center>
    
### Attention 機制：
主要應用於編碼器-解碼器結構中，如機器翻譯、摘要生成等任務。在這些任務中，Attention 使解碼器能夠動態關注輸入序列的不同部分，有效解決了傳統 RNN 在處理長序列時的困難。

### Self-Attention 機制：
廣泛應用於 Transformer 模型中，用於捕捉單一序列中的長距離依賴性。Self-Attention 的平行計算能力使得 Transformer 在處理大型數據集和長序列時比傳統模型更高效，這也是為什麼 Transformer 逐漸取代 RNN 成為 NLP 領域的主流架構。

<br>

## Transformer 核心組件
### 自我注意力機制（Self-Attention）
自注意力機制是Transformer的核心。它允許模型根據輸入序列中的不同位置來賦予不同的注意權重，這對於理解上下文和處理長序列非常重要，自注意力機制的計算過程可以簡單描述
- 對於輸入序列中的每個詞，計算與其他所有詞之間的注意權重。
- 使用這些注意權重對所有詞的表示進行加權總和，以獲得每個詞的新表示。

### 多頭注意力（Multi-Head Attention）
為了更好地捕捉不同類型的關係，Transformer引入了多頭注意力機制。它允許模型在不同子空間中計算注意力，然後將它們組合在一起。多頭注意力可以增加模型的表達能力。

### 前向神經網絡（Feed-Forward Neural Network）
Transformer中的每個注意力子層之後都接了一個前向神經網絡，用於進一步處理特徵。這個簡單的全連接網絡有助於學習特徵的非線性映射。

### 位置編碼（Positional Encoding）
由於Transformer沒有像RNN那樣的隱藏狀態，因此需要引入位置編碼，以區分不同位置的詞。位置編碼是一個固定的向量，添加到詞的嵌入表示中，以考慮詞的相對位置。

不具備序列的時間步概念，因此需要引入位置編碼（Positional Encoding）來讓模型了解輸入序列的順序。

### 堆疊層和殘差連接
Transformer模型由多個堆疊的層組成，每個層都包括自注意力子層和前向神經網絡子層。層之間使用殘差連接，以確保梯度能夠順暢地反向傳播。

這是一種殘差連接，將多頭注意力的輸出和輸入相加，並進行正規化。

<br>

## Transformer 訓練過程
Transformer通常通過監督學習(Supervised learning)進行訓練，例如機器翻譯任務中的原語言到目標語言的映射。訓練過程包括以下步驟：

- 輸入序列的詞嵌入：將輸入序列中的詞轉換為固定維度的詞嵌入向量。
- 位置編碼的添加：將位置編碼添加到詞嵌入中，以考慮詞的位置信息。
- 通過多層Transformer網絡：通過多個堆疊的Transformer層來處理輸入序列。
- 最終的線性映射和Softmax：將模型的輸出映射為輸出詞的概率分佈，並使用交叉熵損失進行訓練。

<br>

## Encoder
先來看左邊的 Encoder 部分，input 先經過 embedding 層轉換為一個向量，然後在進入 layer 前會先與 Positional Encoding 相加 (Input Embedding 跟 Positional Encoding 的維度相等)，這個 Positional Encoding 就是詞語的位置編碼，目的是為了讓模型考慮詞語之間的順序
![image](https://hackmd.io/_uploads/H1cKpPKcJg.png)

\begin{equation}

\end{equation}



### Multi-Header Attention 做自注意
計算輸入它自己每個單詞之間的關聯度。無關順序可以同時處理加快速度。

### Feed Forward
兩層的前饋神經網路，激勵函數用 ReLU 做學習。

<br>

## Decoder
### Masked Multi-Header Attention 做自注意
計算輸出它自己每個單詞之間的關聯度。無關順序可以同時處理加快速度。

由於解碼器是照順序預測，所以預測時不需計算該單詞和它之後的單詞（未來資料）之間的關聯度。因此多了遮罩（Mask）來蓋掉該位置之後的情報。

### Multi-Header Attention 做編碼器-解碼器注意
計算輸出和輸入之間單詞的關聯度。

類似 Seq2Seq 的 Attention，注意輸出和輸入的哪個部分有關聯。

### Feed Forward
兩層的前饋神經網路，激勵函數用 ReLU 做學習。

:::info
**重點筆記**：上述編碼器和解碼器的步驟會 × N 次。
:::

<br>

## 其他的處理
### Positional Encoding
編碼器和解碼器的自注意，由於只計算自己單詞之間的關係，沒有照順序處理，會失去單詞所在位置的情報，所以一開始會透過 位置編碼（Positional Encoding） 把位置的順序情報也編碼進去。

### Add & Norm
每一步之後會做 Add & Norm， Add 是之前介紹過的 Residual（跳接兩個子層）。Norm 是正規化（Normalization）

<br>

## Multi-Header Attention 的計算方式
![image](https://hackmd.io/_uploads/r14jrDtc1g.png)

主要是透過 Scaled Dot-Product（內積）Attention 計算，和一般的 attention 差不多。只是細分成 Query，Key，Value（Q，K，V）。

:::success
**重點筆記**：
- Q = 某個時間步驟（單詞）
- K = Q 作 attention 的對象，用來和 Q 做計算。
- V = Q 作 attention 的對象，K 的原始值。
:::


### Encoder 的步驟
以編碼器的步驟 1. 而言，因為是自注意，Q，K，V都來自輸入自己。
1. 計算注意分數：計算輸入某個單詞 （Q） 對輸入其他所有單詞（K）的關聯性
2. 計算注意分布：將分數轉換成機率分布。
3. 計算 context vector：剛剛的機率分布 和 輸入其他所有的單詞（V）做加權平均。

### Decoder 的步驟
解碼器的步驟 2. 是計算輸出和輸入的關聯，Q 來自輸出，K 和 V 是輸入所有的單詞。
1. 計算注意分數：計算輸出某個單詞 （Q） 對 輸入所有單詞（K）的關聯性
2. 計算注意分布：將分數轉換成機率分布。
3. 計算 context vector：剛剛的機率分布和輸入所有單詞（V）做加權平均。

<br>

## 運作過程
### RNN 或 LSTM 機器翻譯：
在使用 RNN 或 LSTM 進行機器翻譯時，模型會逐字逐句地生成翻譯結果。比如在翻譯句子 "I am a student" 到中文 "我是一個學生" 時，LSTM 會首先接收 "I" 並生成其翻譯，然後接收 "am" ...，依次生成整個句子的翻譯。

### Transformer 機器翻譯：
Transformer 的設計讓它能夠一次性看到整個輸入句子。例如在翻譯 "I am a student" 時，所有的詞會同時被輸入編碼器，通過多頭自注意力機制，模型能夠理解句子內的所有詞之間的關係。解碼器在生成翻譯時，同樣能夠同時考慮已生成的翻譯和原句的全部內容。這使得翻譯更加精確和流暢，特別是在長句子中表現優越。
![image](https://hackmd.io/_uploads/B1xvtPFc1l.png)


<br>

## 自注意的用處

例如輸入：The animal didn't cross the street because it was too tired.
- 可以透過自注意去確認牠（it）是指這句的哪個部分，比如說動物（animal）。
- 多頭（Multi-Header）則透過隨機初始的 Linear 層的訓練，多去注意其他不同的東西。可以看出 牠（it） 不只注意到動物，還連結到累（tire）這個字。

