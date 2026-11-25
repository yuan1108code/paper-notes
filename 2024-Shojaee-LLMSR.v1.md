---
title: "【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】"
tags: [paper, symbolic-regression, scientific-discovery, llm, status-inbox]
concepts: ["[[Symbolic regression]]", "[[Evolutionary program search]]", "[[In-context learning]]", "[[Neuro-symbolic AI]]"]
papers: ["[[【Optimizing generative AI by backpropagating language model feedback】]]", "[[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]"]
---

# 【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 作者 | Parshin Shojaee、Kazem Meidani、Shashank Gupta、Amir Barati Farimani、Chandan K. Reddy；前兩位共同第一作者 |
| 機構 | Virginia Tech、Carnegie Mellon University、Allen Institute for AI（依 v3 PDF 首頁） |
| 發表 | ICLR 2025 Oral；不是 2024 年 ICLR 論文。[2][3] |
| 版本 | 本筆記依 arXiv:2404.18400v3（2025-03-20）；v1 為 2024-04-29、v2 為 2024-06-02。PDF 共 32 頁，主文第 1–10 頁、參考文獻第 11–13 頁、附錄 A–G 第 14–32 頁。所有表號、結果和圖均取 v3，不混用舊版。[1][2] |
| 連結 | [arXiv](https://arxiv.org/abs/2404.18400)／[PDF v3](https://arxiv.org/pdf/2404.18400v3)／[HTML v3](https://arxiv.org/html/2404.18400v3)／[官方程式與資料](https://github.com/deep-symbolic-mathematics/LLM-SR) |
| 引用（IEEE） | P. Shojaee, K. Meidani, S. Gupta, A. B. Farimani, and C. K. Reddy, "LLM-SR: Scientific equation discovery via programming with large language models," in *Proc. Int. Conf. Learn. Represent. (ICLR)*, 2025, arXiv:2404.18400. |
| 動畫解說 | [papers.poyuanchu.com/2024-shojaee-llmsr/](https://papers.poyuanchu.com/2024-shojaee-llmsr/) |
| 論文與素材授權 | arXiv 論文 CC BY 4.0；本筆記原圖均標註原論文圖號。官方倉庫根目錄 MIT，但部分沿用 FunSearch 檔案保留 Apache-2.0 header，使用時應保留個別檔案原授權聲明。[2][3] |
| 檢查範圍 | 讀完主文與附錄 A–G、檢視全部 26 圖、轉錄原始 5 表，靜態檢查程式與 12 份 CSV；未執行下載的候選程式，未重跑完整 LLM 搜尋實驗 |
| 與我的相關度 | 4 / 5：適合工業 AI 的可解釋代理模型、材料曲線、設備動態與結構／數值分離優化；不是現成 RAG、異常偵測或通用預測模型 |
| 整理日期 | 2026-11-20 |

> [!NOTE]
> **一句話**：讓 LLM 當「想公式結構的人」，讓數值優化器當「調係數的人」，用真實資料評分，再把不同搜尋島上的好例子放回提示詞繼續改公式。**有學到數值係數、有累積搜尋經驗，但沒有更新基礎 LLM 權重。**[1][3]

## 1. 背景：為什麼擬合出一條線，還不算發現規律？

### 1.1 符號迴歸的任務

<b>符號迴歸（symbolic regression）</b>不是在已知 $y=ax+b$ 後只求 $a,b$；它連「要不要平方、指數、相乘、分段」都要選。輸入一批觀測 $\mathcal D=\{(\mathbf x_i,y_i)\}$，輸出可閱讀、可執行的數學關係 $\tilde f$，希望既貼近觀測，也能預測未看過的狀態。[1]

工業例子是：已知溫度、應變與應力的量測，希望得到可檢查的材料模型，而不只有一個黑箱神經網路。但只要允許很多運算子，公式結構的組合便急遽增加；資料有限時，不同公式還可能幾乎同樣準確。

### 1.2 既有方法與本文切入點

傳統遺傳程式設計把公式畫成樹，再做突變與交叉；DSR 以強化學習產生表達式；NeSymReS、E2E 先用大量合成公式訓練 Transformer，再由數據猜公式。這些方法各有強項，本文強調的缺口是：搜尋時很難自然使用「這是阻尼振盪器」「細菌有最佳溫度」等文字領域背景，以及多行程式的表達彈性。[1]

本文把 LLM 的科學先驗與程式生成能力接到數值評估器。它不是宣稱資料已經不重要；**先驗決定往哪些結構找，資料決定哪些結構與係數值得留下**。如果不讓資料反駁先驗，整個流程就只是寫出看起來像物理的故事。

### 1.3 為什麼不用著名物理公式就結案？

Feynman benchmark 包含廣為流傳的公式。LLM 即使沒從新觀測推理，也可能直接背出正確表達式。作者因而改造兩個振盪器、一個合成細菌生長關係，並加入真實鋁材應力應變資料，合計四題、三個領域。[1]

> [!WARNING]
> **新題不等於已證明無污染**：附錄 C 的較高困惑度與較慢搜尋，是「不像直接背誦」的間接證據，不是訓練集交叉比對或污染證明。領域背景本來就刻意提供；須區分有用的科學先驗、精確答案洩漏和真正由觀測辨識的部分。[1]

## 2. 核心想法：結構、係數、搜尋記憶分開處理

<center>
<img src="https://arxiv.org/html/2404.18400v3/fig1_main.png" alt="圖 1：結構提案、資料擬合與經驗管理的閉環" width="95%" style="background:#fff">
<br>
<b>圖 1：結構提案、資料擬合與經驗管理的閉環（原論文 Figure 1）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**：左上把指令／任務描述／歷史例子組成 prompt，中上 LLM 產生骨架；右下先以觀測調 params，再評分。左下多島 buffer 經 experience manager 抽範例回到 prompt；箭頭不是更新 LLM 權重。[1]


### 2.1 三個角色各自做什麼？

<center><b>表 2：LLM-SR 各層改變的對象</b></center>

| 層 | 輸入 | 輸出／被改變的對象 | 不應誤認成 |
|---|---|---|---|
| LLM 提案 | 任務描述、變數意義、評估程式、歷史範例 | 含 `params` 佔位的方程式程式骨架 | LLM 一次直接估準全部係數 |
| 數值優化 | 一個固定骨架與訓練觀測 | 該骨架的連續係數，例如阻尼強度 | 更新 LLM 神經網路權重 |
| 搜尋記憶 | 候選、訓練分數、來源島 | 候選族群、提示範例與搜尋分布 | 跨任務永久學習的權重 |
| 測試 | 固定候選與獨立 ID／OOD 資料 | 泛化誤差 | 把測試分數回灌選模仍稱獨立測試 |

LLM 像熟悉領域的工程師，提出「驅動力＋阻尼＋恢復力」；優化器像校準儀器的人，調出每項的強度；經驗庫像保存不同研究方向的實驗簿。這裡的「進化」主要發生在程式與提示範例，而非模型權重。[1][3]

### 2.2 方程式為什麼用程式表示？

例如一個骨架可寫成 `a*sin(t) - b*v**3 - c*x*v - d*x*exp(e*x)`。結構選擇是離散的；係數 $a,b,c,d,e$ 是連續的。程式允許中間變數、組件相乘及平滑分段，較容易表達材料彈性區／塑性區。但程式更自由也意味更難保證安全、可識別性與物理合理性。[1]

### 2.3 不是「一個對話越聊越聰明」

每輪從候選島抽例子，組出新提示，產生四個新骨架，再獨立擬合與評分。搜尋並沒有將全部歷史原封不動加入單一長對話，也沒有反向傳播更新 LLM。[1][3]

## 3. 技術細節：資料怎麼真正進入迴圈？

<center>
<img src="https://arxiv.org/html/2404.18400v3/fig_table_file/fig2_prompt_v10.jpg" alt="圖 2：振盪器的初始提示與係數評估函式" width="95%" style="background:#fff">
<br>
<b>圖 2：振盪器的初始提示與係數評估函式（原論文 Figure 2）</b>
</center>

> [!NOTE]
> **圖 2 怎麼看**：由上到下是 instruction、problem specification、evaluation/optimization、initial equation example。右側放大顯示 NumPy+BFGS 與 Torch+Adam 替代路徑；不是兩個優化器每輪都一起用。[1]


### 3.1 提示結構與候選生成

提示含四部分：任務指令、科學問題／變數描述、評估與係數優化函式、從經驗庫選出的方程式例子。第 2 圖不是單純丟 CSV 給 LLM；生成的程式在外部被執行，數據透過誤差評估影響下一輪可見的範例。論文設定每提示 $b=4$、生成溫度 0.8、每次 $k=2$ 個範例；係數向量最多 10 維。[1]

### 3.2 兩層優化

對固定骨架 $f$，數值優化求：

$$\mathbf p^*=\arg\min_{\mathbf p}\frac1n\sum_i(f(\mathbf x_i,\mathbf p)-y_i)^2.$$

然後回傳 **負 MSE**：

$$s(f,\mathcal D)=-\operatorname{MSE}(f(\mathbf X,\mathbf p^*),\mathbf y).$$

所以分數越接近 0 越好；負 0.01 優於負 0.2。BFGS 是常見的準牛頓數值優化，Adam 則用自動微分的梯度調整係數。兩者調的都是該方程式參數，不是語言模型。[1][3]

報表使用 NMSE，但**搜尋模板回傳的是未標準化負 MSE**。在同一個固定資料集上，若採常見 $\mathrm{MSE}/\operatorname{Var}(y)$ 定義，兩者排序相同；跨資料集或不同 split 則不能直接比較原始 MSE。[3]

> [!WARNING]
> **NMSE 定義與重現邊界**：v3 表格稱 Normalized MSE，但本文與所檢查的主搜尋路徑沒有給出可完整重現表 1 分母／最終測試流程的實作。不能把「負 MSE 的搜尋分數」當成「已重現 NMSE」。本動畫若顯示 NMSE，明示採 `MSE / population variance(y)` 的教學定義；不是宣稱重新得到論文實驗分數。[1][3]

### 3.3 多島經驗庫與兩階段抽樣

十個島各保留候選族群。先均勻選一島，再根據分數以 Boltzmann／softmax 抽一個分數群集：

$$P_i=\frac{\exp(s_i/\tau_c)}{\sum_j\exp(s_j/\tau_c)}.$$

同分群集內，較短程式被優先抽取：釋出實作用正的字元長度 $l_i$，令 $\tilde l_i=(l_i-\min l)/(\max l+10^{-6})$，機率正比 $\exp(-\tilde l_i)$。這是抽範例的偏好，不是加入 MSE 的明確複雜度懲罰。每四小時重置較差的一半島，由存活島的最佳程式重新開始。[1][3]

> [!WARNING]
> **文字、公式、程式有差異**：第 2.4 節先定義 `Score=-MSE`，更新集合卻寫 `s=-Score` 再最大化，符號不一致；Algorithm 1 與模板則直接最大化負 MSE。附錄 B 把長度描述成「負長度」，與程式正字元長度也不同。更重要的是，`buffer.py` 先登錄所有有效候選，再另行追蹤最佳值，不是僅有打破最佳紀錄的候選才進庫。解讀重現行為應以可定位的實作為準。[1][3]

### 3.4 以 Oscillation 2 看出「找得準」不一定「找出同一條真式」

真式為：

$$\dot v=0.3\sin(t)-0.5v^3-xv-5x\exp(0.5x).$$

附錄 G 的 GPT-3.5 結果則包含：

$$\dot v\approx0.3\sin(t)-0.5v^3-xv+5(1-\exp(x)).$$

兩個恢復力項在 $x=0$ 的低階展開相近，但不是代數相等。由資料所涵蓋的區域、不同誤差指標與測試設計，決定這種替代式能有多好的分數。[1]

依兩式直接計算，$x=0.2$ 時恢復力相對差約 0.167%；$x=2$ 時真項 −27.182818、替代項 −31.945280，相對差 17.52%；$x=4$ 已達 81.34%。這些是**公式計算**，不是作者新增實驗，也不是整個加速度的相對誤差。

> [!WARNING]
> 附錄 G 用「$x\in(-2,2)$ 大致是資料位移範圍」說明近似，但釋出 Oscillation 2 CSV 的訓練位移約 [−0.2056, 0.1749]、OOD 約 [−0.4032, 0.5354]，實際更窄。故「低 NMSE」不能讀成在整個 (−2,2)、更遠位移或全新初始條件皆恢復真實動力學；也不能把點上加速度預測，自動升格成長時間軌跡穩定性保證。[1][3]

### 3.5 釋出程式靜態稽核

檢查官方程式 commit `41c212312df6c16d936c9cb395356a62774c47e3`。僅讀取原始碼與 CSV，沒有執行任意生成式程式、啟動模型或重跑搜尋。[3]

<center><b>表 3：論文敘述與釋出路徑對照</b></center>

| 檢查項目 | 檔案與實際行為 | 判讀 |
|---|---|---|
| BFGS 擬合 | `specs/specification_oscillator2_numpy.txt:21–37`，初值全 1、`minimize(...,method='BFGS')`，僅回傳 `-result.fun`，拒絕 NaN／Inf | 真正擬合係數；沒有保存最佳係數供下游測試的完整公開流程；未檢查 `result.success` |
| Adam 擬合 | `specs/specification_oscillator2_torch.txt:22–47`，LR=1e-4、10000 步、10 個參數 | 對方程式做梯度；30 秒外部 timeout 可能提前使整個候選失敗，不保證跑滿 |
| 訓練／測試 | `main.py:43–51` 只載入 `train.csv`；原始 ID／OOD CSV 有釋出 | 主路徑未見把測試回饋送入搜尋；但完整表格挑選、重新擬合與 NMSE 計算腳本未在此路徑確認，不能證明實驗完全無洩漏 |
| 預算 | `main.py:32` 10000 samples；`Config` 每提示 4；sampler 每候選增加計數，批次前檢查上限 | 約對應 2500 提示輪；計數含初始值與批次邊界，不宜宣稱恰好 10000 個有效方程式 |
| 併行 | `config.py:59–60` 預設 sampler=1、evaluator=1；`sampler.py:83–95` 逐候選呼叫 | 與論文 4 evaluators 不同；建立多個物件本身不代表並行排程 |
| 溫度 | API payload 有 model、messages、max_tokens=512，沒傳 temperature；local request 傳 None | 不能從此版本斷言實際 API 使用論文 0.8 |
| 抽樣週期 | `config.py:43` 30000；附錄 B 的 N=10000 | 預設不同，重現需明示 |
| 緩衝區 | `buffer.py:123–128,208–220` 所有有效程式登錄；最高分另存 | 論文簡化演算法不是逐行等價實作 |
| 評估安全 | `evaluator.py:152–165,202–206` 用 multiprocessing timeout，對完整字串 `exec` | 有時間限制不等於安全沙箱；未見論文宣稱的 2GB 硬限制、檔案／網路隔離或系統呼叫封鎖 |
| 結果重現 | 倉庫含資料與搜尋管線；未見完整 baseline／報表 NMSE 重建程式 | 本筆記重算的是公開表格比例與資料範圍，不是聲稱完整重現論文 |

> [!WARNING]
> **勿直接在工作電腦執行生成程式**：`LocalSandbox` 的 docstring 宣稱 secure，不能替代隔離實作證據。正式應用需容器／低權限帳號、唯讀資料、網路封鎖、CPU／記憶體限制，並以白名單 AST／運算子或受限 DSL 降低風險。這是部署建議，不是本論文已驗證功能。[3]

## 4. 實驗設定：四個任務、不同意思的 OOD

<center><b>表 4：四個資料集與釋出 CSV 實際筆數</b></center>

| 任務 | 輸入 → 目標 | train／ID／OOD 筆數 | OOD 的意義 |
|---|---|---|---|
| Oscillation 1 | 位移 x、速度 v → 加速度 a | 10000／10000／10000 | 同一模擬軌跡較早時間區段；不是全新設備或任意新初始條件 |
| Oscillation 2 | 時間 t、位移 x、速度 v → a | 10000／10000／10000 | OOD t∈[0,20)，訓練約 [30,50]，ID 與訓練時間幾乎交錯 |
| E. coli growth | 菌量 B、基質 S、溫度 T、pH → 生長率 | 7500／2500／15000 | 合成式的更廣變數範圍，不是外部實驗室的獨立生物實測 |
| Stress–strain | 應變、溫度 → 應力 | 2161／1442／738 | 保留 200°C 曲線，位於原實驗 20–300°C 範圍之內；是留一條件測試，不是超出溫度範圍外插 |

資料筆數與範圍由釋出 CSV 計算；問題設定依附錄 D。細菌資料 train/ID 的 B、S 約 1–5、T 約 1–40、pH 約 2–12；OOD 約擴至 B、S 1–10、T 1–60、pH 1–14，但 OOD 的每一列不必在每個維度都落於訓練範圍外。[1][3]

Stress CSV 的溫度已縮放為約 0.0667–1，OOD=0.666666667，不能把 CSV 數字直接解讀成攝氏；應力／應變也不是原圖 MPa／原始應變的直接值。原圖 16 才是實驗物理尺度。[1][3]

<center><b>表 5：搜尋與比較設定</b></center>

| 項目 | v3 報告的設定 |
|---|---|
| 主模型 | GPT-3.5-turbo、Mixtral-8x7B；附錄另列 GPT-4o 的兩個振盪器結果 |
| 自託管硬體 | Mixtral：4 張 NVIDIA RTX 8000，各 48GB；GPT 系列用 API |
| LLM-SR 搜尋 | 約 2500 輪、每提示 4 個骨架、k=2、溫度 0.8、最多 10 係數 |
| 評估 | NumPy+BFGS 或 Torch+Adam；論文寫 4 evaluators、30 秒／2GB 限制；程式差異見表 3 |
| 群體 | 10 島，每 4 小時重置較差一半；同分群集偏好短程式 |
| 基線 | GPlearn、PySR、DSR、uDSR、NeSymReS、E2E；附錄增 LMX、改作方程式任務的 FunSearch |
| 基線預算 | 搜尋型超過 200 萬 iterations；附錄 A 說 baseline 各 5 次複跑，報最好結果；未提供與 LLM-SR 全部方法一致的平均±標準差 |
| 評估口徑 | 表格 NMSE 越低越好，分 ID/OOD；不代表符號結構完全恢復率 |

> [!WARNING]
> **不能把 200 萬÷2500=800 直接寫成 800 倍速度或成本優勢**：不同方法的 iteration 可能是 generation、候選或批次，LLM 每輪另含四次骨架生成及連續擬合。論文缺乏同硬體牆鐘、token 與美元總成本完整對照。即使僅取 2500×4×30 秒=83.33 小時，也只是每候選都 timeout 且串行的名義上界算例，不是實測 runtime；實際會因並行、早停、失敗與 API 等待改變。[1][3]

### 4.1 自訂方程式與原始參數

<center><b>表 6：振盪器參數（原 Table 2）</b></center>

| Dataset | Time range | initial values | F | $\alpha$ | $\beta$ | $\delta$ | $\gamma$ | $\omega$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Oscillator 1 | (0, 50) | {x=0.5, v=0.5} | 0.8 | 0.5 | 0.2 | _ | 0.5 | 1.0 |
| Oscillator 2 | (0, 50) | {x=0.5, v=0.5} | 0.3 | 0.5 | 1.0 | 5.0 | 0.5 | 1.0 |

數據來源：v3 原論文。[1]


Oscillation 1 的真式為 $\dot v=0.8\sin(x)-0.5v^3-0.2x^3-0.5xv-x\cos(x)$。注意第一個 sine 的輸入是**位移 x，不是時間 t**；不要依常見驅動振盪器直覺改寫。兩題初值皆 x=v=0.5，模擬 t=0–50。[1]

細菌式是四個效應相乘：菌量 B、Monod 基質項 $S/(K_s+S)$、自訂 tanh／四次衰減的溫度項、指數與 sine 平方的 pH 項。這是刻意組合的合成 benchmark，不是本文在實驗室發現的新生物定律。[1]

## 5. 結果：多數比較很強，但不能略過失敗格

### 5.1 主表完整數字

<center><b>表 7：四題的完整 ID／OOD NMSE（原 Table 1）</b></center>

| 模型 | Osc1 ID | Osc1 OOD | Osc2 ID | Osc2 OOD | E.coli ID | E.coli OOD | Stress ID | Stress OOD |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| GPlearn | 0.0155 | 0.5567 | 0.7551 | 3.188 | 1.081 | 1.039 | 0.1063 | 0.4091 |
| NeSymReS | 0.0047 | 0.5377 | 0.2488 | 0.6472 | N/A | N/A | 0.7928 | 0.6377 |
| E2E | 0.0082 | 0.3722 | 0.1401 | 0.1911 | 0.6321 | 1.4467 | 0.2262 | 0.5867 |
| DSR | 0.0087 | 0.2454 | 0.0580 | 0.1945 | 0.9451 | 2.4291 | 0.3326 | 1.108 |
| uDSR | 0.0003 | 0.0007 | 0.0032 | 0.0015 | 0.3322 | 5.4584 | 0.0502 | 0.1761 |
| PySR ( Cranmer, 2023 ) | 0.0009 | 0.3106 | 0.0002 | 0.0098 | 0.0376 | 1.0141 | 0.0331 | 0.1304 |
| LLM-SR ( Mixtral ) | 7.89e-8 | 0.0002 | 0.0030 | 0.0291 | 0.0026 | 0.0037 | 0.0162 | 0.0946 |
| LLM-SR ( GPT-3.5 ) | 4.65e-7 | 0.0005 | 2.12e-7 | 3.81e-5 | 0.0214 | 0.0264 | 0.0210 | 0.0516 |

數據來源：v3 原論文。[1]


> [!NOTE]
> 原 Table 1 所有值以原文數字轉錄；NeSymReS 的 E. coli 欄為 N/A，因四輸入超出模型支援的至多三變數設定。這裡 N/A 橫跨 ID/OOD，不是把後面的材料數值錯移到細菌欄。[1]

### 5.2 以每題最強非 LLM 基線比較，而非挑弱對手

<center><b>表 8：OOD 重新計算；「低幾倍」為 baseline NMSE ÷ 方法 NMSE</b></center>

| 任務 | 最佳非 LLM 基線 OOD | GPT-3.5 LLM-SR OOD | Mixtral LLM-SR OOD | 解讀 |
|---|---:|---:|---:|---|
| Oscillation 1 | uDSR 0.0007 | 0.0005 | 0.0002 | 兩者較好，但收益幅度與其他題不同 |
| Oscillation 2 | uDSR 0.0015 | 0.0000381 | 0.0291 | GPT-3.5 誤差低 39.37 倍；Mixtral 反而高 19.4 倍 |
| E. coli | PySR 1.0141 | 0.0264 | 0.0037 | 各低 38.41 與 274.08 倍；這是特定合成題與特定 backbone，不是平均總收益 |
| Stress–strain | PySR 0.1304 | 0.0516 | 0.0946 | GPT-3.5 相對降低 60.43%；不能等同外插未知溫度成功 |

以上比例為由原 Table 1 重算，不是新 benchmark 實驗。

> [!WARNING]
> **「兩種 backbone 都一致優於基線」過度概括**：Mixtral 在 Oscillation 2 的 ID 0.0030、OOD 0.0291，分別輸給 PySR 的 0.0002 與 uDSR 的 0.0015。不能將每題不同 backbone 的最佳格拼成單一模型結果；選完最好格的 oracle 比較必須另行標示。[1]

<center>
<img src="https://arxiv.org/html/2404.18400v3/fig_table_file/compare_curves_id_final.jpg" alt="圖 3：四個問題的最佳誤差搜尋軌跡" width="95%" style="background:#fff">
<br>
<b>圖 3：四個問題的最佳誤差搜尋軌跡（原論文 Figure 3）</b>
</center>

> [!NOTE]
> **圖 3 怎麼看**：橫軸為 iterations，縱軸 Normalized MSE 為對數刻度；紅色實／虛線分 GPT-3.5／Mixtral，綠色 PySR、藍色 DSR、紫色 uDSR。最佳值曲線只會下降，不顯示候選失敗率或多 seed 分布；不同方法 iteration 單位也未必等價。[1]


### 5.3 誤差低，不代表符號完全相同

<center>
<img src="https://arxiv.org/html/2404.18400v3/osc_equations-v14.png" alt="圖 4：兩個振盪器的真式、LLM 程式與基線" width="95%" style="background:#fff">
<br>
<b>圖 4：兩個振盪器的真式、LLM 程式與基線（原論文 Figure 4）</b>
</center>

> [!NOTE]
> **圖 4 怎麼看**：兩列分 Oscillation 1／2；左是真式和相圖，中間是 LLM-SR 程式骨架及擬合後簡式，右是其他 SR。綠底標出恢復的真式項，不代表未標綠的替代項沒有用，也不等於完整符號式精確相等。[1]

<center>
<img src="https://arxiv.org/html/2404.18400v3/fig_table_file/qual_bacterial4.jpg" alt="圖 5：細菌生長率在四種變數下的分布" width="95%" style="background:#fff">
<br>
<b>圖 5：細菌生長率在四種變數下的分布（原論文 Figure 5）</b>
</center>

> [!NOTE]
> **圖 5 怎麼看**：列由上到下是藍色真值、紅色 LLM-SR、綠色 PySR、紫色 uDSR；四欄橫軸是菌量、基質、溫度、pH，縱軸生長率。灰色區域與黑點標記 ID，其餘延伸區域才用來看 OOD 偏移；這是合成式資料，不是培養皿照片。[1]


Oscillation 2 的近似恢復力就是例子。E. coli 的分布圖則說明低 ID 誤差未必預測得出更廣變數範圍；LLM 的溫度／pH 單峰先驗有助結構，但仍不能靠圖形證明因果或唯一性。[1]

### 5.4 消融：資料擬合與迭代都不是裝飾

<center>
<img src="https://arxiv.org/html/2404.18400v3/fig_table_file/ablation_all_nov24_v3.png" alt="圖 6：Oscillation 2 的元件消融" width="95%" style="background:#fff">
<br>
<b>圖 6：Oscillation 2 的元件消融（原論文 Figure 6）</b>
</center>

> [!NOTE]
> **圖 6 怎麼看**：縱軸對數 NMSE，實色柱 ID、斜線柱 OOD；各組依序移除 prior、program、iterative refinement、skeleton+optimizer，最後比較 Torch+Adam 與 NumPy+BFGS。每柱的高度須連同不同 backbone／設定理解，不能由對數高度直接讀出線性百分比。[1]


Oscillation 2 中，移除迭代改良的 ID/OOD 約 0.101／0.181；要求 LLM 直接連係數一起生成、移除 skeleton+optimizer，約 0.3780／0.3751。完整 GPT-3.5 為 2.12e-7／3.81e-5。這些比較支持「離散結構＋數值擬合」分工的重要性，但仍缺同總成本、多 seed 誤差條件下的充分消融。[1]

<center><b>表 9：多島與對應抽樣的聯合消融（原 Table 3）</b></center>

| 模型 | Stress ID | Stress OOD | Osc2 ID | Osc2 OOD |
| --- | --- | --- | --- | --- |
| LLM-SR (w/o multi-island & sampling) | 0.0257 | 0.1010 | 6.23e-6 | 0.0008 |
| LLM-SR | 0.0210 | 0.0516 | 2.12e-7 | 3.81e-5 |

數據來源：v3 原論文。[1]


多島加對應抽樣讓 stress OOD 從 0.1010 降至 0.0516，相對降低 48.91%；Oscillation 2 OOD 從 0.0008 降至 3.81e-5，約低 21.00 倍。但此消融**同時改掉島數與 top-k／隨機抽樣策略**，不能把全部收益純歸因島數。[1]

<center><b>表 10：LLM 搜尋方法與元件比較（原 Table 4）</b></center>

| 模型 | Osc1 ID | Osc1 OOD | Osc2 ID | Osc2 OOD |
| --- | --- | --- | --- | --- |
| DSR | 0.0087 | 0.2454 | 0.0580 | 0.1945 |
| uDSR | 0.0003 | 0.0007 | 0.0032 | 0.0015 |
| PySR | 0.0009 | 0.3106 | 0.0002 | 0.0098 |
| LMX | 0.5031 | 48.93 | 1.004 | 0.9371 |
| FunSearch | 0.4840 | 8.059 | 0.7234 | 0.5861 |
| LLM-SR | 0.1371 | 0.6764 | 0.3780 | 0.3751 |
| LLM-SR (w/o Prior) | 0.0001 | 0.0029 | 4.65e-5 | 0.0071 |
| LLM-SR (w/o Program) | 0.0001 | 0.0035 | 3.22e-5 | 0.0043 |
| LLM-SR | 4.65e-7 | 0.0005 | 2.12e-7 | 3.81e-5 |

數據來源：v3 原論文。[1]


表中 LMX、FunSearch 都用 GPT-3.5、相同名義迭代數，是比「不同 backbone、不同輪數」更接近的比較；但任務適配、提示、數值優化與候選成本仍不完全相同。原版 FunSearch 與本文改編版也不應混為同一標準配置。[1]

<center><b>表 11：三種 LLM backbone（原 Table 5）</b></center>

| 模型 | Osc1 ID | Osc1 OOD | Osc2 ID | Osc2 OOD |
| --- | --- | --- | --- | --- |
| LLM-SR ( Mixtral ) | 7.89e-8 | 0.0002 | 0.0030 | 0.0291 |
| LLM-SR ( GPT-3.5 ) | 4.65e-7 | 0.0005 | 2.12e-7 | 3.81e-5 |
| LLM-SR ( GPT-4o ) | 7.29e-6 | 6.75e-5 | 4.27e-10 | 1.29e-6 |

數據來源：v3 原論文。[1]


> [!WARNING]
> **更大模型不處處更好**：GPT-4o 在 Oscillation 1 的 ID 7.29e-6，比 GPT-3.5 的 4.65e-7 高約 15.68 倍，也輸給 Mixtral；OOD 才較好。附錄只測兩個振盪器，不可推論 GPT-4o 在四題全勝。[1]

### 5.5 附錄 A–G 圖像導讀與限制

以下保留全部附錄圖，方便回頭查證。Prompt 圖是輸入／輸出的示例，不是執行證據；曲線、方程式與實驗數字也要分開閱讀。沒有精確數字表支援的噪音／困惑度圖，僅解釋趨勢，不自行估讀成精確實驗值。[1]

<center>
<img src="https://arxiv.org/html/2404.18400v3/spec_prompts_vert5.png" alt="圖 7：細菌與材料任務的文字規格範例" width="95%" style="background:#fff">
<br>
<b>圖 7：細菌與材料任務的文字規格範例（原論文 Figure 7）</b>
</center>

> [!NOTE]
> **圖 7 怎麼看**：上下框分細菌生長與材料應力應變，顯示變數說明和初始線性骨架；為易讀省略 evaluation，不代表正式提示不含它。文字背景讓模型利用領域先驗，但未提供完整自訂真式。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/fig_table_file/spec_torch.jpg" alt="圖 8：PyTorch 版本提示與 Adam 擬合" width="95%" style="background:#fff">
<br>
<b>圖 8：PyTorch 版本提示與 Adam 擬合（原論文 Figure 8）</b>
</center>

> [!NOTE]
> **圖 8 怎麼看**：圖中 equation 以 tensor 運算表達，evaluation 用 Adam 調 params。可微條件讓梯度通到係數；不是通到遠端 LLM，也不代表任何 if-else 都能直接可微。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/fig_table_file/perplexity_feyn.png" alt="圖 9：Feynman 與新任務的 Mixtral 困惑度" width="95%" style="background:#fff">
<br>
<b>圖 9：Feynman 與新任務的 Mixtral 困惑度（原論文 Figure 9）</b>
</center>

> [!NOTE]
> **圖 9 怎麼看**：橫軸任務，縱軸 equation conditioned on scientific context 的 median perplexity。Feynman 橘柱較低，三個新合成題藍柱較高；材料題因沒有單一真式未納入。低困惑度是背誦風險線索，不能單獨證明訓練集污染。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/perplexity_qual2.svg" alt="圖 10：困惑度實驗使用的上下文與方程式" width="95%" style="background:#fff">
<br>
<b>圖 10：困惑度實驗使用的上下文與方程式（原論文 Figure 10）</b>
</center>

> [!NOTE]
> **圖 10 怎麼看**：左欄 Feynman、右欄自訂 benchmark；每格紫色是科學 context、藍色是被計算條件機率的 equation。這裡比較的是特定文字化方式，不是對不同答案表達完全不變的污染指標。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/fig_table_file/feynman_curve4.png" alt="圖 11：著名公式與新題的誤差下降速度" width="95%" style="background:#fff">
<br>
<b>圖 11：著名公式與新題的誤差下降速度（原論文 Figure 11）</b>
</center>

> [!NOTE]
> **圖 11 怎麼看**：橫軸迭代、縱軸對數 NMSE；黃色 Feynman 在很少步數迅速下降，藍色不同線型是四個新任務。說明背誦可能讓評測過於容易，不證明新題已完全無洩漏。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/feyn_prompts1.png" alt="圖 12：Feynman I.37.4 的單次回答" width="95%" style="background:#fff">
<br>
<b>圖 12：Feynman I.37.4 的單次回答（原論文 Figure 12）</b>
</center>

> [!NOTE]
> **圖 12 怎麼看**：上框輸入描述光學干涉變數，下框 LLM 一次就產生接近標準干涉強度公式。這是 qualitative recitation 示例，不是對全部 Feynman 題目的恢復率統計。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/feyn_prompts2.png" alt="圖 13：Feynman I.26.2 的單次回答" width="95%" style="background:#fff">
<br>
<b>圖 13：Feynman I.26.2 的單次回答（原論文 Figure 13）</b>
</center>

> [!NOTE]
> **圖 13 怎麼看**：以折射角、折射率與另一角度描述問題後，LLM 產生含 arcsin 和 sin 的熟悉關係。可由語意線索召回答案，不等於從新資料辨識未知機制。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/fig_table_file/oscillation_phase_v2.jpg" alt="圖 14：兩個自訂振盪器的相平面" width="95%" style="background:#fff">
<br>
<b>圖 14：兩個自訂振盪器的相平面（原論文 Figure 14）</b>
</center>

> [!NOTE]
> **圖 14 怎麼看**：橫軸位移 x、縱軸速度 v；藍色虛線是同一系統模擬軌跡，紅星 t0 是初始狀態。左／右分 oscillator 1／2；圈形不能當成隨機獨立樣本或新初始條件實驗。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/fig_table_file/pH_dependency_v3.png" alt="圖 15：自訂生長反應與文獻形狀對照" width="95%" style="background:#fff">
<br>
<b>圖 15：自訂生長反應與文獻形狀對照（原論文 Figure 15）</b>
</center>

> [!NOTE]
> **圖 15 怎麼看**：左橫軸溫度、右橫軸 pH，縱軸為各效應因子；彩線是既有模型，黑虛線 custom model 較尖銳。這是 benchmark 設計的對照，不是本文發現細菌新生理機制。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/fig_table_file/stress_gt_v2.png" alt="圖 16：6061-T651 鋁材的實測應力應變曲線" width="95%" style="background:#fff">
<br>
<b>圖 16：6061-T651 鋁材的實測應力應變曲線（原論文 Figure 16）</b>
</center>

> [!NOTE]
> **圖 16 怎麼看**：橫軸 strain、縱軸 stress MPa，顏色分 20、100、150、200、250、300°C。隨溫度改變的曲線形狀讓多行／平滑分段程式有用；留出 200°C 仍在量測溫度包絡內。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/more-ablations2.png" alt="圖 17：四題的領域背景與程式表示消融" width="95%" style="background:#fff">
<br>
<b>圖 17：四題的領域背景與程式表示消融（原論文 Figure 17）</b>
</center>

> [!NOTE]
> **圖 17 怎麼看**：四個子圖分四題，柱組為 PySR、w/o Prior、w/o Program、完整 LLM-SR，實色／斜線分 ID/OOD；以對數刻度呈現。此處是 GPT-3.5 的消融，不能拿去替 Mixtral 在 Oscillation 2 的失利背書。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/fig_table_file/noise_robustness.png" alt="圖 18：加入高斯噪音後的誤差" width="95%" style="background:#fff">
<br>
<b>圖 18：加入高斯噪音後的誤差（原論文 Figure 18）</b>
</center>

> [!NOTE]
> **圖 18 怎麼看**：橫軸 noise σ∈{0,0.01,0.05,0.1}、縱軸線性 NMSE；紅色 LLM-SR、綠色 PySR，實線 ID、虛線 OOD。整體 LLM-SR 較穩健，但缺精確附表與多 seed 誤差棒，不把圖估讀值當精確測量。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/osc2_gpt_curve.png" alt="圖 19：Oscillation 2 搜尋中的簡式演變" width="95%" style="background:#fff">
<br>
<b>圖 19：Oscillation 2 搜尋中的簡式演變（原論文 Figure 19）</b>
</center>

> [!NOTE]
> **圖 19 怎麼看**：橫軸迭代、縱軸 NMSE；紅色最佳軌跡逐步找到驅動、阻尼與交叉項，綠底是與真式相同的項。最後恢復力仍為替代近似 5(1−exp(x))，不是完整真式 −5x exp(0.5x)。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/osc1_mixtral_curve_v2.png" alt="圖 20：Mixtral 在 Oscillation 1 的骨架演變" width="95%" style="background:#fff">
<br>
<b>圖 20：Mixtral 在 Oscillation 1 的骨架演變（原論文 Figure 20）</b>
</center>

> [!NOTE]
> **圖 20 怎麼看**：中間誤差曲線以引線連到不同時期程式，程式內參數仍是 placeholders；各框不是擬合後精確係數。其用途是看如何組合 driving／damping／restoring 項。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/bg22_gpt_curve_v2.png" alt="圖 21：GPT-3.5 的細菌生長骨架演變" width="95%" style="background:#fff">
<br>
<b>圖 21：GPT-3.5 的細菌生長骨架演變（原論文 Figure 21）</b>
</center>

> [!NOTE]
> **圖 21 怎麼看**：誤差下降過程逐步修改溫度與 pH 反應，B、Monod 型基質效應較早形成。這呈現先驗可縮小結構空間，但最後是否正確仍須看資料與 OOD。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/stress_mixtral_curve_v2.png" alt="圖 22：Mixtral 的材料骨架演變" width="95%" style="background:#fff">
<br>
<b>圖 22：Mixtral 的材料骨架演變（原論文 Figure 22）</b>
</center>

> [!NOTE]
> **圖 22 怎麼看**：程式將彈性／塑性相關部分組合，藉 sigmoid 等平滑轉換近似分段；連線對應當時的最佳誤差。程式可表達這些結構，不代表所有生成分段模型都符合材料力學。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/bg_equations_vert.png" alt="圖 23：細菌生長的最終程式與基線公式" width="95%" style="background:#fff">
<br>
<b>圖 23：細菌生長的最終程式與基線公式（原論文 Figure 23）</b>
</center>

> [!NOTE]
> **圖 23 怎麼看**：上方為不同 LLM-SR backbone 的多行程式，下方 DSR／uDSR／PySR 表達式。篇幅長短不是泛化證明；要同時回看表 1 的 ID／OOD 誤差。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/stress_equations_vert.png" alt="圖 24：材料應力應變的最終方程式" width="95%" style="background:#fff">
<br>
<b>圖 24：材料應力應變的最終方程式（原論文 Figure 24）</b>
</center>

> [!NOTE]
> **圖 24 怎麼看**：多行程式展示平滑非線性／分段構成，底部列其他 SR 公式。它是資料驅動經驗模型，不是證明唯一的本構定律，也未驗證任意載入路徑。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/qual_osc2.png" alt="圖 25：振盪器加速度的視覺比較" width="95%" style="background:#fff">
<br>
<b>圖 25：振盪器加速度的視覺比較（原論文 Figure 25）</b>
</center>

> [!NOTE]
> **圖 25 怎麼看**：三個主圖橫軸分 t、x、v，縱軸 dv/dt；藍真值、紅虛線 LLM-SR、綠點畫 PySR、紫點 uDSR。右側放大早期時間區段，顯示微小差異；不能從重疊曲線推論長期積分穩定性。[1]


<center>
<img src="https://arxiv.org/html/2404.18400v3/qual_stress.png" alt="圖 26：材料曲線的定性比較與 300°C 放大" width="95%" style="background:#fff">
<br>
<b>圖 26：材料曲線的定性比較與 300°C 放大（原論文 Figure 26）</b>
</center>

> [!NOTE]
> **圖 26 怎麼看**：左為各溫度的 stress–strain，灰線真值、紅虛線 LLM-SR；右放大 300°C 並比較 PySR、uDSR。300°C 放大不是保留的 200°C OOD 曲線，因此此圖不能單獨證明表 1 的 OOD 改善。[1]


## 6. 總結：可借用的是分工，不是「LLM 已找到真理」

LLM-SR 的貢獻是把領域語意、可執行結構、連續係數優化及進化搜尋接在一起。四題展示有希望的效果，尤其部分 OOD 格差距很大；但模型依賴、同軌跡測試、近似式不可識別性、缺乏完整成本與多 seed 分布，限制了結論的外推範圍。[1]

> [!TIP]
> **可以借用**：在工業代理模型／設備方程式辨識，把 LLM 限定成「受物理約束的結構提案器」，讓獨立數值 solver 擬合係數。先比較線性／物理既有模型／PySR，再做新工況或新設備 hold-out；同時記錄 train、validation、真正外推誤差、token、牆鐘與無效程式率。只在候選改善且符合量綱、單調性／穩定性條件時留下。這是本筆記提出的轉用方案，不是論文已在工廠完成的驗證。

與 [TextGrad](https://hackmd.io/@Yuan1108/S1PMaJzCGl) 相比，TextGrad 以文字評語改生成式系統的文字變數；LLM-SR 有明確的數值資料誤差與係數擬合。與 [Dream-RSI](https://hackmd.io/@Yuan1108/rJ_uIzoRfe) 相比，LLM-SR 改方程式候選，Dream-RSI 改探索控制程式。共通點是「固定生成模型＋外部回饋＋搜尋」，不代表三者優化同一對象。

## 7. 我的筆記

### 重點筆記
>

### 可以借用的地方

- **材料曲線代理模型**：先以簡單物理分段結構當骨架，再讓數值擬合調轉折點、斜率與溫度依賴，保持量綱可檢查。
- **設備動態辨識**：LLM 只能從允許的力／流量／阻尼項提案；以新設備、新初值、新負載測試，不只切同一軌跡的時間段。
- **RAG 的有限轉用**：可用檢索補領域約束或單位，但 RAG 是論文未來方向，不是現有方法已用的核心模組。檢索出精確目標方程式時，必須另外標記答案先驗／污染風險。
- **最佳化研究**：把離散結構搜尋與連續參數估計解耦，候選群保留多樣性；另設獨立 validation，避免一直用同批資料搜尋而產生適應性過擬合。

### 疑問

1. 在相同牆鐘／美元／token 預算、多個隨機種子下，收益是否仍在？
2. 若提供無關或錯誤任務描述，效果如何？目前 w/o Prior 是移除說明，不是安慰劑／誤導先驗測試。
3. 能否以額外新初始條件／更寬位移區域，區分 $-5x e^{0.5x}$ 與 $5(1-e^x)$？
4. 能否將 sandbox、最佳係數保存、NMSE 分母、baseline 設定、完整 held-out 選模流程一起釋出？
5. 資料只含有限軌跡時，如何避免可解釋但錯誤的因果／物理敘述？

### 相關

- [[Symbolic regression|符號迴歸]]：同時找結構與數值係數。
- [[Evolutionary program search|進化式程式搜尋]]：以可執行評估推動候選族群更新。
- [[In-context learning|情境學習]]：把已評估例子放入提示，不更新模型權重。
- [[Neuro-symbolic AI|神經符號 AI]]：把神經提案能力接到可檢查符號／程式。
- [[【Optimizing generative AI by backpropagating language model feedback】]]
- [[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]

## Sources

[1] https://arxiv.org/html/2404.18400v3
[2] https://arxiv.org/abs/2404.18400
[3] https://github.com/deep-symbolic-mathematics/LLM-SR
