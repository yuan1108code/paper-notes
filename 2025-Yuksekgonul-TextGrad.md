---
title: "【Optimizing generative AI by backpropagating language model feedback】"
tags: [paper, llm, prompt-optimization, compound-ai, status-inbox]
concepts: ["[[Backpropagation]]", "[[Prompt optimization]]", "[[LLM-as-a-judge]]", "[[Compound AI system]]", "[[Chain-of-thought prompting]]"]
papers: ["[[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]", "[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]", "[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]", "[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]", "[[【A Survey on LLM-as-a-Judge】]]", "[[【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】]]"]
---

# 【Optimizing generative AI by backpropagating language model feedback】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 作者 | Mert Yuksekgonul, Federico Bianchi, Joseph Boen, Sheng Liu, Pan Lu, Zhi Huang, Carlos Guestrin, James Zou（史丹佛大學、Chan Zuckerberg Biohub） |
| 發表 | Nature 639, 609–616（2025/3/19）；arXiv 預印本 2024/6 |
| 連結 | [DOI](https://doi.org/10.1038/s41586-025-08661-4)・[arXiv](https://arxiv.org/abs/2406.07496)・[GitHub（textgrad）](https://github.com/zou-group/textgrad) |
| 引用（IEEE） | M. Yuksekgonul et al., "Optimizing generative AI by backpropagating language model feedback," *Nature*, vol. 639, no. 8055, pp. 609–616, Mar. 2025, doi: 10.1038/s41586-025-08661-4. |
| 動畫解說 | [papers.poyuanchu.com/2025-yuksekgonul-textgrad/](https://papers.poyuanchu.com/2025-yuksekgonul-textgrad/) |
| 論文類型 | 方法 + 框架論文：提出 TextGrad，並在 5 類任務上示範 |
| 狀態 | 待讀 |
| 與我的相關度 | 5 / 5（RAG 問答、多個 LLM 串接的系統、用 LLM 評分來改提示詞，都是我正在做的事） |
| 加入日期 | 2026-11-11 |

> [!NOTE]
> **一句話**：把深度學習的「反向傳播」搬到由 LLM 和工具組成的系統上，只是**梯度從數字換成一段文字評語**。由 LLM 說出「這裡哪裡不好、該怎麼改」，評語沿著系統一路往回傳，再由 LLM 照評語改寫提示詞、程式碼、答案，甚至分子結構和放療計畫。

---

## 1. 背景：AI 系統越來越像「拼裝車」

### 從單一模型到複合系統

現在很強的 AI 應用，常常不是單一模型，而是**好幾個 LLM 加上搜尋引擎、程式執行器、模擬器等工具串起來**的複合系統（compound AI system）。例如：

- LLM 搭配符號解題器，能解奧林匹亞等級的數學題
- LLM 搭配搜尋和程式執行，在程式競賽和真實 GitHub 問題上接近人類工程師

但這些系統大多是**專家手工調出來的**：提示詞怎麼寫、每一步怎麼接，靠經驗和反覆嘗試。

### 為什麼不能直接用反向傳播？

過去 15 年深度學習的進展，靠的是**反向傳播**（backpropagation）：每個元件之間用可微分的函數連接（例如矩陣乘法），可以算出「每個參數往哪個方向調，損失會變小」的數字梯度，自動優化。

複合系統做不到這件事，因為：

1. 元件之間傳的是**文字、程式碼、圖片**，不是數字
2. 很多元件是**黑盒子**：例如透過 API 呼叫的 GPT-4o，拿不到模型參數，也算不了梯度
3. 有些元件根本不可微分，例如搜尋引擎、單元測試、分子模擬軟體

> [!TIP]
> 這正是作者的類比：神經網路當年也是「手工調」，直到反向傳播和自動微分讓優化變成「一鍵完成」。TextGrad 想替複合 AI 系統做同一件事。

---

## 2. 核心想法：用文字當梯度

<center>
<img src="https://arxiv.org/html/2406.07496v1/figure_1.png" alt="圖 1" width="90%">
<br>
<b>圖 1：TextGrad 總覽（原論文 Fig. 1，取自 arXiv 版）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**
> 這張圖分成七格，上半部講原理，下半部是四個應用例子。
> - **a（左上）：一般神經網路的反向傳播**。圓圈是神經元，裡面的數字是數值，連線上的數字是權重。藍色彎箭頭是梯度，例如「損失對 h1 的梯度 = 0.267」「損失對 v1 的梯度 = 0.124」，代表從右邊的損失一路往左傳回來。
> - **b（右上）：TextGrad 的反向傳播**。系統換成真實的 LLM 應用：左邊三個輸入（提示詞、查詢、工具提示詞）分別經過 LLM、搜尋引擎、工具呼叫，產生中間輸出，再交給下一個 LLM 產生最終輸出，最後由另一個 LLM 依「評估指令」打分數當作損失。藍色箭頭上的梯度不再是數字，而是一句話：「this response can be improved by…」「this prompt can be improved by…」。
> - **c（中間）：和 PyTorch 的對照表**。數學、PyTorch、TextGrad 三欄一一對應：輸入 `Tensor` 對應 `tg.Variable`、模型 `ResNet50()` 對應 `tg.BlackboxLLM(...)`、損失 `CrossEntropyLoss()` 對應 `tg.TextLoss("Rate the summary.")`、優化器 `SGD` 對應 `tg.TGD`。下方藍色長箭頭說明兩者的訓練步驟一模一樣：`loss = loss_fn(model(input))` → `loss.backward()` → `optimizer.step()`。
> - **d（左中下）：分子優化**。分子經過兩次文字梯度（「加入能增加極性的官能基」「加入極性或芳香環」）逐步改良，結合力分數 Vina 從 -4.3 → -5.5 → -7.5 kcal/mol（越負越好）。
> - **e（右中下）：程式碼優化**。上方紅色標示的是第 t 版有問題的程式碼；右邊文字梯度指出「`nums[i] == k` 這個情況沒有處理好」；下方綠色是第 t+1 版改好的部分。
> - **f（左下）：放療計畫優化**。彩色是劑量分布（藍低紅高），白色和粉紅框是膀胱和直腸。上圖這兩個器官落在高劑量區，文字梯度說「膀胱和直腸的權重太低」，調整後下圖的劑量就避開了它們。
> - **g（右下）：提示詞優化**。原本的提示詞（只說「一步一步想」）正確率 77.8%；優化後的提示詞多了「把每個項目和數量列清楚」「直接加總」「檢查每個項目是否相關」「再核對一次」等指示，正確率升到 91.9%。

### 最簡單的例子：兩次 LLM 呼叫

作者用一個常見結構說明（式 1、2）：

```
預測 = LLM(提示詞 + 問題)
評估 = LLM(評估指令 + 預測)
```

第二行就是常見的「讓 LLM 當評審」（LLM-as-a-judge）。我們想優化的是**提示詞**。

反向傳播的做法，和連鎖律的精神一樣，從後面往前問：

1. **評估 → 預測**：請 LLM 根據評估結果，說明「預測哪裡錯、該怎麼改」，得到「預測的梯度」
2. **預測 → 提示詞**：再請 LLM 根據「預測的梯度」和原本的呼叫，說明「提示詞該怎麼改，預測才會變成那樣」，得到「提示詞的梯度」
3. **更新**：把「提示詞的梯度」和目前的提示詞交給 LLM，請它寫出新版提示詞

### 圖 2 的重點（以文字重現）

原論文 Fig. 2 用三格說明上面的流程。圖中的實際提示詞比較複雜，作者說這裡只是示意：

```
(a) 反向傳播
∂評估/∂預測   = ∇LLM(預測, 評估)
              → 「根據評估，預測是錯的。要修正，你應該改……」
∂評估/∂提示詞 = ∇LLM(提示詞, 預測, ∂評估/∂預測)
              → 「要讓預測產生上面那種改變，你應該換一個策略……」

(b) 文字梯度運算子 ∇LLM(x, y, ∂L/∂y) 的寫法
LLM( 這是一段和 LLM 的對話：{x | y}
     以下是對 {y} 的批評：{∂L/∂y}
     請說明如何改進 {x}。 )

(c) 文字梯度下降（TGD）的一步
x_new = LLM( 以下是對 {x} 的批評：{∂L/∂x}
             請吸收這些批評，產生新的變數。 )
```

<center><b>圖 2：TextGrad 反向傳播與 TGD 更新的示意（依原論文 Fig. 2 以文字重現）</b></center>

<center><b>表 2：數字梯度下降與文字梯度下降的對照</b></center>

| | 一般梯度下降 | 文字梯度下降（TGD） |
|---|---|---|
| 要優化的東西 | 數字參數 θ | 任何文字變數：提示詞、程式碼、答案、分子、權重設定字串 |
| 梯度 | 一組數字，代表調整方向 | 一段評語，說明哪裡不好、該怎麼改 |
| 損失函數 | 要可微分，例如均方誤差 | 可以是黑盒子：LLM 評分、單元測試結果、模擬軟體分數 |
| 更新方式 | θ − 梯度 | 交給 LLM「吸收批評，寫出新版」 |
| 需要模型參數嗎 | 要 | 不用，所以 GPT-4o 這種 API 也能用 |

> [!TIP]
> 關鍵是**梯度不碰模型參數**，只改系統裡的文字。所以不管底下是商用 API、開源模型還是外部工具，都能套用。

---

## 3. 技術細節

### 3.1 一般化：任意計算圖

把系統畫成計算圖：每個變數 $v$ 由它的前一層變數經過某個函數 $f_v$ 產生（式 4）。函數可以是 LLM 呼叫、模擬器或數值解題器。

變數的梯度，是**它被用到的每一個地方傳回來的評語集合**（式 5）：

$$\frac{\partial L}{\partial v} = \bigcup_{w \in \text{SuccessorsOf}(v)} \nabla_f\left(v, w, \frac{\partial L}{\partial w}\right)$$

白話：如果同一個提示詞被用在三個地方，就會收到三份評語，全部合在一起當作它的梯度。

更新用 $v_{new} = \text{TGD.step}(v, \partial L/\partial v)$（式 6）。計算圖有 $n$ 條邊，每一輪優化**最多多呼叫 $n$ 次 LLM** 來算梯度。

### 3.2 兩大類用法

<center><b>表 3：兩種優化情境</b></center>

| 類型 | 優化什麼 | 目標 | 例子 |
|---|---|---|---|
| 測試時優化（test-time optimization） | 這一題的答案本身 | 把眼前這一題做對 | 程式碼、科學題答案、分子、放療計畫 |
| 提示詞優化（prompt optimization） | 給 LLM 的提示詞 | 找一個對**新題目也有效**的提示詞 | 數學推理、計數題的系統提示詞 |

### 3.3 一些實作技巧

- **小批次**：提示詞優化像隨機梯度下降，每輪只拿幾個訓練例子（批次大小 3，共 12 輪，總共看 36 個例子）
- **驗證集把關**：每輪更新後在驗證集上測，**比上一版好才接受**新提示詞
- **提示詞可拆開優化**：把提示詞拆成「指令」和「示範例子」兩個變數一起優化
- **強模型教弱模型**：推理用便宜的 gpt-3.5-turbo，產生梯度用 gpt-4o。花一次優化成本，之後用便宜模型就能跑得更好
- **多數決**：解題時更新 3 次，對所有版本的答案做多數決
- **動量**（放療任務）：把過去幾輪的權重和梯度也放進提示詞，類似梯度下降的動量

---

## 4. 實驗設定：五類任務

除非特別註明，使用 gpt-4o（2024-05-13 版）和 gpt-3.5-turbo（0125 版）。

<center><b>表 4：五類任務與設定</b></center>

| 任務 | 優化的變數 | 損失（評估）來源 | 資料 | 比較對象 |
|---|---|---|---|---|
| 程式碼 | 程式碼 | LLM 依題目、程式碼、本地測試結果給評語 | LeetCode Hard（39 題） | 零樣本、Reflexion |
| 解題 | 答案 | LLM 以「挑剔的科學家」角度檢查答案 | GPQA、MMLU 兩個子集、MathVista、ScienceQA-IMG | CoT、Reflexion |
| 提示詞 | 系統提示詞（指令 + 示範） | 答案與標準答案比對 | BBH 計數、BBH 排序、GSM8k | CoT、OPRO、DSPy |
| 放療計畫 | 各器官的重要性權重（寫成字串） | LLM 比對計畫和臨床目標 | 5 位攝護腺癌病人 | 放射腫瘤科醫師的計畫、貝氏優化 |
| 複合系統 | 每個模組的輸出／提示詞 + 評分提示詞 | LLM 評估最終答案 | ScienceQA-IMG（Chameleon）、HQH 空間推理 | Chameleon、CoT、Reflexion |

另外還有一個分子設計的概念驗證：用 Autodock Vina（結合力）和 QED（類藥性）兩個模擬分數組成「文字損失」，在 58 個標靶蛋白上測試。

> [!WARNING]
> Reflexion 在程式碼任務用了 1 個示範例子，TextGrad 一個都沒用（零樣本）。比較時這點對 TextGrad 不利，所以作者說結果更有說服力。但反過來說，**各方法的 LLM 呼叫次數和成本並不完全一樣**，要對照補充資料的成本分析。

---

## 5. 結果

### 5.1 程式碼、解題、提示詞

<center><b>表 5：主要結果（越高越好，數字取自原論文）</b></center>

| 任務 | 資料集 | 基準 | 最強的對照 | TextGrad |
|---|---|---|---|---|
| 程式碼（通過率） | LeetCode Hard | 零樣本 26% | Reflexion 31 ± 1.2% | **36 ± 1.8%** |
| 解題（正確率） | GPQA | CoT 51.0% | GPT-4o 官方數字 53.6% | **55.0%** |
| | MMLU 機器學習 | CoT 85.7% | Reflexion 85.7% | **88.4%** |
| | MMLU 大學物理 | CoT 91.2% | Reflexion 94.1% | **95.1%** |
| | MathVista | CoT 63.8% | Reflexion 64.0% | **66.1%** |
| | ScienceQA-IMG | CoT 91.6% | Reflexion 90.2% | **92.4%** |
| 提示詞（正確率） | BBH 計數 | CoT 77.8% | DSPy 84.9% | **91.9%** |
| | BBH 排序 | CoT 76.7% | DSPy 79.8% | **80.8%** |
| | GSM8k | CoT 76.7% | DSPy 79.8% | **80.8%** |

- 提示詞優化：**比 OPRO 好**，在排序和 GSM8k 和 DSPy 差不多，計數題比 DSPy 高 7.0 個百分點
- 消融實驗：**指令和示範一起優化最好**；只優化指令也已經贏過其他方法
- 換成開源的 Llama-3.1-70B 一樣有效；為 gpt-3.5 優化的提示詞，搬到 Qwen-2.5、Llama 也能提升
- 補充資料的成本分析顯示，TextGrad 在「成本－正確率」上是更好的取捨（Pareto 改善）

> [!WARNING]
> 原論文 Extended Data Fig. 5a 裡，**排序和 GSM8k 兩列的數字完全一樣**（76.7 / 77.8 / 79.8 / 80.8），同一張圖的 GSM8k 提示詞例子卻寫 72.9% → 81.1%。這兩列可能有一列印錯，引用時要小心。

### 5.2 放療計畫

<center>
<img src="https://arxiv.org/html/2406.07496v1/treatment_planning_trajectories.png" alt="圖 3" width="80%">
<br>
<b>圖 3：放療計畫優化（原論文 Fig. 3，取自 arXiv 版）</b>
</center>

> [!NOTE]
> **圖 3 怎麼看**
> 背景：放療要讓腫瘤區（PTV）拿到足夠劑量，同時保護膀胱、直腸等健康器官。計畫軟體會依每個部位的「重要性權重」算出劑量分布，難的是權重怎麼設，現在靠人反覆試。TextGrad 負責調這組權重。
> - **a（上排）：劑量分布的變化**。彩色是劑量（藍低紅高），中間紅色大塊是腫瘤，上方白框是膀胱，下方粉紅框是直腸。初始版劑量往四周擴散；第 1 輪的文字梯度說「腫瘤區沒達標，提高 PTV 權重」，劑量就集中到腫瘤上；後來的梯度說「膀胱和直腸的權重太低」，到第 5 輪兩個器官受到的劑量明顯降低。
> - **b（左下）：腫瘤區平均劑量**。藍點是 TextGrad 每一輪的結果，淡紫色帶是變異範圍，紅虛線是臨床目標 70.2 Gy。從初始約 79 Gy，第 2 輪起就貼在目標線上，而且變異範圍越縮越窄。
> - **c（右下）：膀胱（灰）和直腸（粉紅）的平均劑量**。紅虛線是臨床允許的上限 30 Gy。第 1 輪因為只顧腫瘤，兩者都升高（膀胱一度略超過上限）；之後梯度要求保護器官，第 5 輪降到約 19 和 22 Gy，都在上限以下。
> - **d、e（最下方的表）：和醫師計畫比較**。數字見下表。

<center><b>表 6：放療計畫與醫師計畫的比較（5 位病人平均，括號為標準差）</b></center>

| 部位 | 指標 | 醫師計畫 | 貝氏優化 | TextGrad |
|---|---|---|---|---|
| 腫瘤 PTV | 平均劑量與目標的差 | +1.97 (0.36) | +4.84 (1.65) | **+0.51** (0.09) |
| 腫瘤 PTV | D95 與目標的差 | −0.10 (0.15) | +0.00 (0.07) | **+0.00** (0.00) |
| 膀胱 | 平均劑量（Gy，越低越好） | 22.39 (5.55) | 26.15 (5.64) | **20.92** (0.79) |
| 直腸 | 平均劑量（Gy，越低越好） | 23.88 (6.45) | 24.47 (9.30) | **17.18** (4.20) |

D95 是「95% 的腫瘤體積至少拿到的劑量」，和目標 70.2 Gy 剛好相等最理想。貝氏優化平均要 21.3 輪才收斂，TextGrad 最多 10 輪。

> [!WARNING]
> 只有 5 位病人，而且作者明說這是**展示潛力，不代表臨床效果**，真正使用需要臨床驗證。

### 5.3 分子設計（概念驗證）

<center>
<img src="https://arxiv.org/html/2406.07496v1/molecule_design.png" alt="圖 4" width="75%">
<br>
<b>圖 4：用文字優化分子（原論文 Extended Data Fig. 1，取自 arXiv 版）</b>
</center>

> [!NOTE]
> **圖 4 怎麼看**
> - **a（最上排）**：從一個苯環開始，兩次文字梯度（「加入能形成氫鍵或疏水作用的官能基」「加入疏水基團或芳香環，同時維持親水和疏水的平衡」）把分子逐步改大。結合力 Vina 從 −4.2 → −5.5 → −7.5 kcal/mol（越負越好），類藥性 QED 從 0.44 → 0.59 → 0.79（越高越好）。
> - **b（左中）**：橫軸是類藥性 QED，縱軸是結合力。灰色等高線是起始分子，集中在左上（結合力差）；紅色是 DrugBank 裡已上市的藥；藍色是 TextGrad 設計的分子，和上市藥落在差不多的結合力區域，但**類藥性更高、更偏右**。
> - **c（右中）**：同一個標靶 PPARA 上，TextGrad 十輪優化的軌跡（黑點虛線）從左上往右下走，最後落在三個上市藥（紅星）附近或更好的位置。
> - **d、e（下排）**：最後設計出的分子和最相近的上市藥 Fenofibric Acid 長得不太像（Tanimoto 相似度 0.38），右邊是它卡進蛋白質口袋的模擬姿勢。

在 58 個標靶上，TextGrad 用更少的模擬次數，找到比其他分子優化演算法更好的分子。作者也提醒：分數全靠電腦模擬，**實際價值有限，只算概念驗證**。

### 5.4 複合系統

<center><b>表 7：複合系統的結果（正確率）</b></center>

| 資料集 | 方法 | 正確率 |
|---|---|---|
| ScienceQA-IMG | Chameleon（原始系統） | 77.5% |
| | TextGrad 優化 2 輪 | 83.2% |
| | TextGrad 優化 3 輪 | **85.2%** |
| HQH 空間推理 | Chain of thought | 70.0% |
| | Reflexion | 72.0% |
| | TextGrad（提示詞 + 答案一起優化） | **79.0%** |

- **Chameleon** 是一個代理人系統：規劃器先決定要用哪些工具（看圖、偵測文字、查知識、產生答案），再依序執行。原本沒有回饋機制，前面錯了會一路錯下去。TextGrad 把每個模組的輸出都當成變數，從最後的答案往回傳評語，逐一修正。原論文 Fig. 4b 的例子：知識檢索模組被要求「把三個物體的關鍵特性講清楚」，答案生成模組被指出「烏龜殼其實不亮，是霧面的」，最後從錯的「亮」改成對的「不透明」
- **HQH**：同時優化三個變數：回答用的指令、**評分用的損失提示詞**，以及答案本身。連評審的提示詞都能被優化

---

## 6. 總結

1. TextGrad 把系統畫成計算圖，用 **LLM 寫的評語當梯度**，沿著圖反向傳播，再用文字梯度下降更新任何文字變數
2. 語法刻意模仿 PyTorch（`Variable`、`TextLoss`、`backward()`、`TGD.step()`），學過 PyTorch 就會用；完全開源
3. 同一個框架、不做任務特化，在程式碼、科學解題、提示詞、放療計畫、分子設計、代理人系統上都有提升
4. 不需要模型參數，黑盒子 API 和外部工具都能接
5. 限制：分子和放療只做了電腦模擬驗證；作者也說反向傳播文字回饋的演算法設計空間還很大

> [!TIP]
> **可以帶走的思路**
> - **RAG 系統可以這樣調**：檢索查詢的改寫提示詞、生成答案的提示詞、評分的 LLM 評審，全部當變數，用「四項品質檢核」當損失，讓系統自己改提示詞，而不是人工一版一版試
> - **強模型教弱模型**：用 gpt-4o 等強模型產生評語，優化便宜模型或地端開源模型的提示詞，部署成本低。這對工業場域只能用地端模型的情況很實用
> - **數值優化器的外層調參**：放療的做法是「內層照舊用數值優化器，外層讓 LLM 依評語調權重」。排班、排程的限制式權重，也許可以用同樣方式調
> - **一定要有驗證集把關**：只在比上一版好時才接受更新，避免越改越差

---

## 7. 我的筆記

### 重點筆記
>

### 可以借用的地方
> 國家災防中心的 RAG 問答：能不能把四項品質檢核寫成 TextLoss，讓 TextGrad 自動優化檢索與生成的提示詞？智慧排班的限制式權重，能不能仿照放療的「外層 LLM 調權重」？

### 疑問
> 每一輪要多呼叫幾次 LLM？和 DSPy 比，實際花費差多少？評審 LLM 自己有偏誤時，會不會把系統帶歪？

### 相關
- 概念：[[Backpropagation|反向傳播]]、[[Prompt optimization|提示詞優化]]、[[LLM-as-a-judge|LLM 當評審]]、[[Compound AI system|複合 AI 系統]]、[[Chain-of-thought prompting|思維鏈提示]]
- 對照方法：DSPy（Khattab et al., ICLR 2024）、OPRO（Yang et al., ICLR 2024）、Reflexion（Shinn et al., NeurIPS 2023）、Self-Refine（Madaan et al., NeurIPS 2023）
- 「文字梯度」一詞來源：Pryzant et al., *Automatic prompt optimization with "gradient descent" and beam search*, EMNLP 2023
- 相關論文：[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]（LLM 決策代理人，規劃提示詞可以用 TextGrad 優化）
- 被優化的代理人系統：Chameleon（Lu et al., NeurIPS 2023）
- 相關論文：[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]（多步驟 RAG 系統，可以用 TextGrad 優化它的推理和檢索提示詞）
- 相關論文：[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]（MemRec：記憶管理與推薦拆成兩個 LLM 角色的複合系統，提示詞與規則可用 TextGrad 優化）
- 相關論文：[[【A Survey on LLM-as-a-Judge】]]（TextGrad 的「損失」就是 LLM 評審的評語；這篇綜述整理的位置、長度、自我偏好等偏誤，會直接影響優化方向）

- 相關論文：[[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]；[動畫解說](https://papers.poyuanchu.com/2026-zheng-dreamrsi/)（都不改模型權重；Dream-RSI 改探索控制程式並以固定歷史重播評分，非文字梯度傳播）。


### 延伸閱讀：方程式搜尋

- [[【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】]]：同樣結合固定 LLM 與外部回饋；LLM-SR 改方程式骨架並擬合數值係數，不是改提示文字或探索控制器。
