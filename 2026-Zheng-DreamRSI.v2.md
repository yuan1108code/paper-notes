---
title: "【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】"
tags: [paper, agent, recursive-self-improvement, exploration, status-inbox]
concepts: ["[[World model]]", "[[Replay simulator]]", "[[Pareto hypervolume]]", "[[Recursive self-improvement]]"]
papers: ["[[【Optimizing generative AI by backpropagating language model feedback】]]", "[[【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】]]", "[[【Recursive Self-Improvement in AI：From Bounded Self-Refinement to Autonomous Research Loops】]]"]
---

# 【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 作者 | Tong Zheng, Xidong Wu, Zheng Zhang, Zhankui He, Chaoyi Zhang, Benjamin Coleman, Ruoqiao Wei, Di Bai, Haolin Liu, Rui Liu, Xue Wang, Yue Zhuan, Wang-Cheng Kang, Renkai Xiang, Heng Huang, Xinwu Cheng, Yunsong Guo（17 位） |
| 機構 | Google、Google DeepMind、University of Maryland, College Park、University of Virginia |
| 發表 | arXiv 預印本，cs.CL；未在本來源確認正式會議或期刊錄取 |
| 版本 | 本筆記依 v2（2026-10-06），首次 v1 為 2026-09-14；主文 11 頁，含參考文獻與附錄共 43 頁。[2] |
| 連結 | [arXiv](https://arxiv.org/abs/2609.14858)／[PDF v2](https://arxiv.org/pdf/2609.14858v2)／[HTML v2](https://arxiv.org/html/2609.14858v2)／[作者入口倉庫](https://github.com/zhengkid/Dream-RSI)／[實作倉庫](https://github.com/google-research/dream-rsi) |
| 引用（IEEE） | T. Zheng et al., "Dream-RSI: Recursive self-improvement through evolving worlds," 2026, *arXiv:2609.14858*, doi: 10.48550/arXiv.2609.14858. |
| 動畫解說 | [papers.poyuanchu.com/2026-zheng-dreamrsi/](https://papers.poyuanchu.com/2026-zheng-dreamrsi/) |
| 論文類型 | 方法與系統：用歷史重播改良探索控制程式，不是訓練基礎模型權重 |
| 狀態 | 已整理全文與附錄；未重跑昂貴的發現實驗 |
| 與我的相關度 | 4 / 5：可借用於 RAG／工業 AI 的候選管線搜尋、優化器外層預算分配；不直接提供新的檢索模型或時間序列預測器 |
| 加入日期 | 2026-11-18 |

> [!NOTE]
> **一句話**：讓 AI 先把真正試過的探索過程存成一張地圖，再在這張固定地圖裡反覆試「先走哪條路、並行幾條、何時停」，只把最有希望的新控制程式拿回真實環境試；**改的是搜尋策略程式碼，不是 LLM 權重**。[1]

## 1. 背景：會寫程式，不等於會分配探索預算

### 1.1 任務是長時間的「提出、執行、評分、改良」

科學與演算法發現可以想成一間研發實驗室：代理人寫一版解法，評估器檢查正確性和效能，再決定要沿著這個方向改，還是另開一條路。當一次研究要上百到上千次嘗試，浪費的往往不只是單次答案品質，而是把大量預算投入已經停滯的方向。[1]

**探索策略**就是分配工作的人：要開多少分支、優先延伸誰、同時跑幾份、何時停止。**發現代理人**才是實際寫候選程式的人。把兩者拆開後，才有辦法明確優化「研發管理方式」。[1]

### 1.2 兩種既有做法各有代價

- 固定策略容易實作，但無法從跨輪經驗改變分支深度、停止與並行方式。
- 直接在線上試新策略，需要先跑完很長一段探索，才能知道它是否值得。若每個候選控制器都要重新呼叫寫程式代理人與昂貴評估器，外層搜尋會非常貴。
- 把歷史寫成文字摘要可以提醒代理人，但不是直接測試另一種預算分配規則。[1]

Dream-RSI 的切入點不是「換一個更大的模型」，而是讓已花錢取得的執行紀錄能反覆提供策略層回饋。

## 2. 核心想法：歷史不是只有記憶，也可以是可重播世界

<center>
<img src="https://arxiv.org/html/2609.14858v2/Dream-RSI-latest.png" alt="Dream-RSI 線上探索、歷史世界與離線策略改良迴圈" width="95%">
<br>
<b>圖 1：三階段的遞迴自我改良（原論文 Figure 1）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**：左邊是控制程式加固定代理人進行線上探索，產生樹狀歷史；中間把樹整理成模擬器，累積進右側的 simulator pool。右下放大框是「提出新版策略 → 在池中重播 → 得到成本品質曲線及軌跡回饋 → 再改程式」。最上方箭頭回到左上，代表選好的策略才回線上執行；不是在離線世界生成新的科學結果。[1]

### 2.1 線上探索：真的做實驗

控制器選定父節點後，代理人恢復該節點的工作目錄，讀既有資訊，提出／實作下一個候選，再交由固定評估器測量。每個節點保存程式、檔案系統快照、執行歷史與分數。因此後來的紀錄不只是成功案例，也包括失敗原因與修復歷程。[1]

### 2.2 離線 dreaming：重排已知的嘗試，不預言未知結果

<center>
<img src="https://arxiv.org/html/2609.14858v2/HRS-latest.png" alt="兩個替代策略沿同一個已記錄探索樹重播" width="95%">
<br>
<b>圖 2：同一張歷史地圖可測試不同控制器（原論文 Figure 2）</b>
</center>

> [!NOTE]
> **圖 2 怎麼看**：上方選中的策略、代理人與歷史共同產生中間那棵樹。左側藍色、紫色的替代策略各自揭露不同的分支／深度，右側比較品質、呼叫成本與序列延遲。淡色區塊表示不同走法覆蓋的既有節點，不能解讀成新生成的分支。[1]

> [!WARNING]
> **不是一般生成式世界模型**：Dream-RSI 不學一個能預測任何動作後果的神經動態模型，只重播已實現的 parent–child 結果。線上代理人即使從同一工作目錄重新生成，也可能得到別的結果；重播沒有把這種分布完整建模。因此「歷史重播分數較好」不等於「未來探索一定較好」，更不是無限自我改良保證。[1]

## 3. 技術細節

### 3.1 共用的動作介面與可見範圍

令樹為 $\mathcal T$、根節點為 $r$，可選集合是根與目前葉節點。根可以開新方向，葉可以沿已有方向往下修；最多有 $W$ 個 worker 同時工作。每次動作是一組合法、不重複的候選父節點，空集合代表停止。[1]

線上與離線的差別是「下一個節點怎麼來」：

<center><b>表 2：線上執行與歷史重播的差別（依第 3 節）</b></center>

| 面向 | 線上探索 | 離線重播 |
|---|---|---|
| 下一個結果 | 真正呼叫代理人與評估器，帶隨機性 | 讀取已保存的觀測，固定且可重現 |
| 根節點 | 產生一個新方向的嘗試 | 揭露最早尚未揭露的根分支 |
| 非根節點 | 沿工作目錄產生新候選 | 揭露它唯一的已記錄下一步 |
| 決策可見內容 | 已完成且允許讀取的歷史 | 當前已揭露的 prefix；不可偷看未揭露分數 |
| 能否超出歷史 | 可以，這正是新增資料的來源 | 不可以；沒有後續就不會憑空補值 |
| 成本 | 代理人生成、執行、評估與等待 | 主要是讀檔與策略執行；策略改寫仍須 LLM |

主文數學動作把根視為集合中的單一元素；附錄／實作改以不同 root cell 表示尚未開啟方向，因此一批可以開多個 root。這是抽象層次與 cell 介面的差別，不能把主文的集合直接當完整 API。[1][4]

### 3.2 用品質—成本的面積評分，而非只看最高分

每個歷史世界 $i$ 有 $B_i$ 個已評估節點，最佳紀錄分數為 $S_i>0$。在五個預算偏好 $\rho\in\{0,0.25,0.5,0.75,1\}$ 下各跑一次策略。每次重播產生呼叫數 $n_{i,\rho}$、最好分數 $s_{i,\rho}$、序列輪數 $k^*_{i,\rho}$，品質正規化為：[1]

$$a_{i,\rho}=\operatorname{clip}(s_{i,\rho}/S_i,0,1).$$

**Pareto frontier** 是「沒有另一個點同時更便宜又不更差」的點集合。把不同預算偏好的點去重、去掉被支配點，依呼叫數由小到大排序為 $(\bar n_j,\bar a_j)$，令 $\bar a_0=0$，原論文式 (1) 為：[1]

$$V_i=\frac1{B_i}\sum_j(\bar a_j-\bar a_{j-1})(B_i-\bar n_j)-\lambda\frac1{|\mathcal R|}\sum_{\rho:n_{i,\rho}>0}\frac{k^*_{i,\rho}}{n_{i,\rho}}.$$

- 第一項是正規化 **hypervolume**：越早花少量呼叫達到高品質，面積越大。不是單條時間曲線任意梯形積分。
- 第二項是並行懲罰：同樣四次呼叫，分四輪做，$k/n=1$；兩輪各做兩次，$k/n=0.5$。這獎勵有用的並行，不代表等比例的實際牆鐘加速。
- $\lambda$ 是品質成本與序列延遲的權衡。實作預設為 0.05；這是程式預設，不是論文逐個實驗已驗證採用的完整超參數證據。
- 以上依賴分數「越大越好且 $S_i>0$」。原始數學任務若是越小越好，必須由任務評估器轉成適合的搜尋分數。[1][4]

**手算教學例，不是論文實驗**：令 $B=10$，五個偏好得到 $(n,a,k)=(0,0,0),(2,0.5,1),(4,0.8,2),(6,1,3),(8,1,4)$。最後一點被 $(6,1)$ 支配；面積為 $0.5(10-2)/10+0.3(10-4)/10+0.2(10-6)/10=0.66$。原式把零呼叫項視為零，故懲罰平均 $0.4$；$\lambda=0.05$ 時 $V=0.64$。動畫的計算器以同一公式即時計算。

> [!WARNING]
> **論文式 (1) 與釋出程式的零呼叫語意不同**：`heuristic_search/eval.py` 的 `_pareto_summary` 只把 `budget>0` 的 ratio 放進平均分母，原式則除以固定的五個偏好數、零呼叫貢獻零。上述教學例在程式語意下 penalty=0.5、reward=0.635，而非 0.64。此外程式排除 `n_evaluated<2` 或無正最佳分數的 world。一般非退化情況一致，但碰到零呼叫／退化世界時不能混用兩種數字。[4]

### 3.3 每輪怎麼選策略，以及保證到哪裡為止

每次離線階段包含目前策略與三次改寫，共 $M=4$ 個版本；每個都在目前累積的所有世界與五個偏好上評分，取跨世界平均最好者。因舊策略也是候選，**被選版本在同一個固定世界池的平均重播分數不會比舊版差**。這是有限候選的選擇性質，不是新線上回合的單調改善定理。[1]

附錄 D.2 更要求策略只用已見前綴重建每條分支的趨勢、成功錨點、退步與錯誤修復，而非只看最新分數。編譯／形狀／數值檢查失敗不應直接等同「方向沒救」；批次需兼顧利用、探索、最多一個有依據的修復機會。這些是 policy-development prompt 的設計規範，並不是已獨立消融證明有效的每一項機制。[1]

附錄使用 `beta` 當預算偏好：一次 episode 內固定，離線掃五個值，跨輪才依歷史證據挑下一輪預設。`plan_grid` 在下一回合前規劃寬度與深度，不能偷看當前回合未來結果。超出凍結世界結構支援的 grid 不能獲得重播獎勵。[1]

### 3.4 附錄的實際發現了什麼

- **Lasso solver（E.1）**：用 strong-rule screening 篩候選特徵，維持互斥 active-set 清單，按需要建立 Gram matrix；用 Cauchy–Schwarz 上界省掉一定不會違反 KKT 條件的精確梯度計算。當需重算的量超過 $0.3p$ 或 reset 間隔達 8，就 full refresh。另有記憶體對齊、SIMD、OpenMP 等硬體優化。這些是被發現的解法，不等於探索控制器本身。[1]
- **ConvDiv kernel（E.2）**：融合卷積與 pooling，讓中間值留在暫存器，最後才做縮放與 bias reduction；避免物化完整中間 feature maps。程式針對輸入形狀、GPU、contiguous 與 channel-sum 做 guard，不符合就退回 PyTorch，不能宣稱是對任何 3D convolution 的通用替代品。權重和 bias 有快取，訓練／權重更新情境的快取失效不是這個推論基準所證明的範圍。[1]

## 4. 實驗設定

### 4.1 四領域、九任務，並非九個模型

4 個 kernel + 3 個數學任務 + 1 個 Lasso + 1 個 harness = 9 個任務；Lasso 換兩種 backbone，因而 Table 3 有 10 列設定，不是任務數互相矛盾。[1]

<center><b>表 3：搜尋設定（原論文 Table 3；B 是每輪上限，不是實際用量）</b></center>

| 任務 | 發現模型 | 輪數 T | 工作者 W | 每輪上限 B | 離線版本 M |
|---|---|---:|---:|---:|---:|
| Lasso | Gemini-3.1-Pro | 5 | 10 | 110（10×11） | 4 |
| Lasso | Gemini-3.7-Flash | 5 | 32 | 640（32×20） | 4 |
| VGG16 / LayerNorm / ConvDiv / ConvMax（各自） | Gemini-3.1-Pro | 9 | 10 | 110（10×11） | 4 |
| Sum–Difference / Autocorrelation / Circle Packing（各自） | Gemini-3.1-Pro | 10 | 10 | 110（10×11） | 4 |
| Harness | Claude Opus 4.8 + Claude Code | 9 | 4 | 16（4×4） | 4 |

科學發現的主要受控對照是 Recursive Fixed Exploration：相同發現代理人、評估器、初始化、每輪上限與起始策略；兩者第一輪相同，其後只有 Dream-RSI 改策略。固定策略也能讀歷史，不能把這個 baseline 誤寫成「完全沒有記憶」。[1][4]

### 4.2 評估器、硬體與真正的泛化測試

- Lasso 用與 SimpleTES 相同的 17 個合成 instance 搜尋，另在 6 個 held-out 資料集測 solver 執行時間；平台是 AMD EPYC 7B12、8 vCPU、32 GB RAM 的 KVM Linux。每個 lambda 的目標函數不得比 sklearn 高超過 $10^{-6}$，錯誤候選的搜尋分數為零；搜尋目標是計時集合幾何平均的倒數，結果表則列六資料集算術平均 runtime，兩者不是同一指標。[1]
- Kernel 在 NVIDIA L40S 上測 correctness 與 inverse runtime（1/ms）；PACEvolve 每個任務最多 48 小時。呼叫數、48 小時與 GPU 實際執行時間不能當同一資源單位。[1]
- Sum–Difference 最大化集合和差比值的對數比；Circle Packing 最大化方形內互不重疊圓的半徑和；Autocorrelation 越低越好。附錄列三種不等式，釋出主設定實際使用 `autocorrelation_third`。[1][4]
- Harness 以 Terminus-Kira 為 seed，每個候選跑 Terminal-Bench 2.1 全部 89 題、每題兩次。發現、策略開發與評估都使用 Opus 4.8；沒有固定探索的完整受控重跑，作者估其額外成本超過 10 萬美元。[1]

> [!WARNING]
> **成本帳只算 discovery-agent calls**：主圖的 Compute 不是總 token、美元、牆鐘或能耗。重播讀取便宜，不代表寫新策略的 LLM 呼叫、歷史讀取與評估都免費；文章沒有提供足夠完整成本表，無法把「162× 少呼叫」換成「總花費省 162×」。[1]

## 5. 結果與核對

### 5.1 Lasso：平均變好，但不是所有資料集都變好

<center><b>表 4：六個 held-out 資料集 runtime（ms，低較好；原論文 Figure 4a 的精確表格）</b></center>

| 方法／模型 | Calls | Gisette | RCV1 | DNA | Leukemia | Colon | Duke Breast | Avg. |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| sklearn | — | 11275.2 | 252881.7 | 93.8 | 227.2 | 229.8 | 374.0 | 44180.3 |
| glmnet | — | 9063.6 | 73072.8 | 351.9 | 45.0 | 24.2 | 47.7 | 13767.5 |
| SimpleTES / GPT-OSS-120B（原報） | 51200 | 3141.9 | 19625.6 | 15.9 | 15.5 | 11.6 | 18.1 | 3804.8 |
| SimpleTES / GPT-OSS-120B（作者重現） | 51200 | 8651.0 | 41143.1 | 37.6 | 28.2 | 19.5 | 31.1 | 8318.4 |
| Fixed / Pro | 550 | 1861.8 | 19550.1 | 41.5 | 26.1 | 14.5 | 28.4 | 3587.1 |
| Dream-RSI / Pro | 317 | 2841.0 | 14616.0 | 49.9 | 30.2 | 16.4 | 32.5 | 2931.0 |
| Fixed / Flash | 3200 | 1133.1 | 13873.0 | 29.8 | 24.1 | 15.7 | 24.4 | 2516.7 |
| Dream-RSI / Flash | 1879 | 1091.9 | 12923.4 | 31.4 | 21.0 | 12.2 | 23.6 | 2350.6 |

以上各列平均值可由六個列值重算，在表格顯示精度下吻合。相對固定探索，Pro calls 減少 42.36%、平均 runtime 降低 18.29%；Flash calls 減少 41.28%、平均 runtime 降低 6.60%。51200/317=161.51，四捨五入為約 162×；51200/1879=27.25，為約 27×。這核對的是原文的呼叫口徑，不是相同模型的因果比較。[1]

> [!WARNING]
> Pro 的平均收益主要來自大而慢的 RCV1：另五個資料集都比固定策略慢。Flash 則六個中五個變快，DNA 變慢。原報 SimpleTES 在幾個小資料集也仍然更快；跨模型比較有 GPT-OSS 與 Gemini 的混雜，不能把全部差異歸功於重播策略。[1]

**原 Figure 4 怎麼看（文字重現）**：上方精確 runtime 表如表 4；下方兩條遞迴曲線分別是 Pro 與 Flash，橫軸累積 discovery compute、縱軸平均 runtime（越低越好），標記 1–5 表示輪次。它展示品質與呼叫的取捨，不是總成本或多個隨機種子的信賴區間。[1]

### 5.2 數學任務：兩項持平／領先，Autocorrelation 不是最好

<center><b>表 5：數學結果（原論文 Table 1 的全部方法；— 表示未報）</b></center>

| 方法 | 模型 | Sum–Difference ↑ | Autocorrelation ↓ | Circle Packing ↑ |
|---|---|---:|---:|---:|
| AlphaEvolve | Gemini-2.0 Pro + Flash | — | 1.455700 | 2.635862 |
| AlphaEvolveV2 | Gemini-2.0 Pro + Flash | 1.121936 | — | 2.635983 |
| OpenEvolve | 未列 | — | 1.460000 | — |
| CodeEvolve | 未列 | — | — | 2.635980 |
| ShinkaEvolve | Mixed | — | 1.457800 | 2.635982 |
| TTS-Discovery | Qwen3-8B | — | — | 2.635983 |
| ThetaEvolve | Distilled-Qwen3-8B | — | 1.493000 | 2.635983 |
| EvoX | Gemini-3.0-Pro | — | 1.458900 | 2.635900 |
| SimpleTES | GPT-OSS-120B | 1.143975 | 1.453675 | 2.635983 |
| Fixed | Gemini-3.1-Pro | 1.144047 | 1.456001 | 2.635983 |
| Dream-RSI | Gemini-3.1-Pro | 1.145427 | 1.456375 | 2.635983 |

Sum–Difference 比固定策略高 0.001380（約 0.1206%），Circle Packing 同為 2.635983；Autocorrelation 比固定策略高 0.000374，因為越低越好，這是變差。SimpleTES 的 1.453675 也優於 Dream-RSI。不能把「competitive」改寫成所有任務 SOTA。[1]

### 5.3 Kernel：先分清「更少呼叫」與「更高效能」

<center><b>表 6：原 Figure 3 的作者標註，不是從精確數表重算</b></center>

| 任務 | 比較口徑 | 作者標註 |
|---|---|---:|
| VGG16 | 可比效能所需 discovery calls | 少 2.43× |
| LayerNorm | 可比效能所需 discovery calls | 少 1.79× |
| ConvDiv | 可比 discovery budget 下的 inverse runtime | 高 2.09× |
| ConvMax | 可比 discovery budget 下的 inverse runtime | 高 1.44× |

**原 Figure 3 怎麼看（文字重現）**：橫軸是 generations、縱軸為 1/ms（越高越快）；藍實線 Dream-RSI、灰虛線 Recursive Fixed Exploration、橘虛線 PACEvolve。VGG16／LayerNorm 比的是到達相近高度時的橫向距離；ConvDiv／ConvMax 比的是相近橫座標的縱向差。PACEvolve 的終點是另外的 48 小時實驗，不可把 2.09× 誤當對 PACEvolve 的倍率。[1]

> [!WARNING]
> HTML／附錄沒有列出足以獨立重算 Figure 3 四個倍率的同品質／同預算精確座標表。本筆記視覺核對曲線與標註，但將 2.43、1.79、2.09、1.44 保留為「作者圖上宣稱」，不冒充已由原始數據重現。直接拿兩條曲線各自最後的值相除，也不等於同預算比較。[1]

原 Figure 8 是 PACEvolve 在四任務 48 小時內的 best-so-far 曲線：VGG16 很快停滯，其他曲線階梯式進步。它補足其搜尋過程，但仍不補齊上述 Dream-RSI 倍率的精確配對座標。[1]

### 5.4 Harness：提升 8.43 個百分點，沒有固定探索的完整對照

<center><b>表 7：Harness full-suite 結果（原 Figure 5）</b></center>

| 方法 | TB2.1 分數 |
|---|---:|
| Seed Terminus-Kira | 0.7247 |
| Claude Code（圖示精度） | 0.75 |
| Meta-Harness（圖示精度） | 0.79 |
| Dream-RSI | 0.8090 |

0.8090−0.7247=0.0843，即 **8.43 個百分點**；相對 seed 約 +11.63%。圖 5b 將 seed 四捨五入顯示為 0.72，因此不要用 0.81−0.72 算成 9 點。Claude Code、Meta-Harness 只有兩位小數，不能憑它們算出更高精度差值。[1]

**原 Figure 5 怎麼看（文字重現）**：上圖是逐次探索的 best-so-far 分數，只保留歷來最好，自然不下降；這不能證明每一輪候選都更好。下圖是不同 harness 的最終分數，並不是重播策略與固定策略的受控消融。[1]

> [!WARNING]
> 搜尋候選即在全套 89 題、每題兩次上評估，需警覺反覆選擇造成對 benchmark 的適應；這不等同有一個獨立、完全未參與選擇的外部測試集。論文未提供足夠的多種子變異、完整成本與固定探索對照來隔離所有因素。[1]

### 5.5 歷史累積與 guidance 消融

<center><b>表 8：ConvDiv 歷史池消融（原 Table 2）</b></center>

| 設定 | Performance ↑ | Discovery calls ↓ |
|---|---:|---:|
| 全部歷史 | 1.90 | 786 |
| 僅目前歷史 | 1.59 | 430 |

全部歷史效能高約 19.50%，但也多用約 82.79% 呼叫。這顯示一種品質—成本取捨，**不是同預算下已證明累積歷史較好**。[1]

原 Figure 6 的 ConvDiv 曲線：藍色 Dream-RSI、灰虛線 Fixed、橘色 Dream-RSI + Guidance、綠色 Fixed + Guidance。兩組加文字方向摘要的最終表現都較差，支持「文字指引可能限制探索」的解釋；只是一個任務與這個 guidance 設計，不能概括成所有記憶摘要都有害。[1]

<center><b>表 9：探索行為跨輪變化（原 Figure 7 的明列數字）</b></center>

| Round | E0 | E1 | E2 | E3 | E4 | E5 | E6 | E7 | E8 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Round-best 1/ms | 0.427 | 0.625 | 0.855 | 1.403 | 1.488 | 1.499 | 1.770 | 1.880 | 1.898 |
| Evaluated attempts | 110 | 110 | 87 | 80 | 50 | 92 | 80 | 91 | 86 |

九輪 attempts 相加為 786，與 Table 2 一致；1.898 四捨五入為 1.90。作者觀察到前段減少探索、進步停滯後又增加投入；這是關聯性的探索軌跡，不是每次調整 beta 都導致收益的因果證明。[1]

### 5.6 釋出程式核對與可重現範圍

入口倉庫指向 `google-research/dream-rsi`，後者包含策略、TraceEnv、離線目標與啟動腳本。本筆記做靜態閱讀，未安裝／執行作者的下載腳本，也未呼叫昂貴模型或 GPU 工作。[3][4]

<center><b>表 10：主張與目前釋出內容的核對</b></center>

| 項目 | 檢查結果 |
|---|---|
| 改的是控制程式 | README 與 policy／learning 結構一致；不是梯度微調模型 |
| 五點預算掃描 | `eval.py` 預設 `0.0,0.25,0.5,0.75,1.0`；名稱為 beta |
| 品質成本目標 | `_trace_auc` 以不同預算的 nondominated points 算正規化面積 |
| 並行懲罰 | `_pareto_summary` 使用 effective sequential rounds / calls；零呼叫分母與式 (1) 有差異 |
| 9 任務全可一鍵重現嗎 | README 明列未包含 harness optimization、Current History Only 消融的開關 |
| 外部任務依賴 | Math／Lasso 評估器來自 SimpleTES 的固定 commit，另需下載；Kernel 需 GPU server |
| 入口 README 時效 | badge 還寫 8 tasks / 3 domains，與 v2 的 9 tasks / 4 domains 不同；以 v2 為本筆記數量口徑 |

## 6. 總結：最值得學的是把昂貴歷史變成策略評分器

1. 把「候選解法」和「決定下一步試哪裡的控制器」分開，才能在不同層次優化。
2. 固定的歷史地圖讓很多策略便宜地比較，省掉每個候選都完整線上重跑的成本。
3. 真正的新發現仍然靠線上代理人與評估器；重播只能篩選，不能創造未記錄結果。
4. 實證顯示若干任務有品質—calls 優勢，但不是所有資料集、所有目標、所有成本口徑都贏。
5. 平均重播分數不退步的選擇保證，必須與未來任務泛化分開看。[1]

> [!TIP]
> **可以借用**：對 RAG／工業 AI 的多候選管線搜尋，先記錄每個版本的父版本、修改、成本、錯誤、正確性與評估結果，再離線比較「何時繼續修、何時換方向」。但若新控制器會改變檢索問題、工具參數或外部資料，就可能離開舊歷史支援範圍，不能把舊節點的分數當成新動作的真實答案；最後仍要用獨立新資料與真實成本驗證。

與 [TextGrad](https://papers.poyuanchu.com/2025-yuksekgonul-textgrad/) 的關係是「都讓 LLM 改系統層可執行／文字物件而非模型權重」。差異是 TextGrad 依文字評語沿計算圖回傳修改建議，Dream-RSI 則以歷史重播評分探索控制程式；兩者不是同一演算法。[1]

## 7. 我的筆記

### 重點筆記
>

### 可以借用的地方
> 建立可追溯的實驗樹，而不只保留最終成功版本。將 correctness failure、工具／環境失敗與「有效但沒進步」分開標記，避免用一次程式 bug 就封殺一個可能有價值的方向。搜尋品質、calls、token／美元與序列延遲分欄記錄。

### 疑問
> 以新世界作為 held-out policy-selection set，重播分數與下一輪線上收益有多高相關？計入 dreaming 的模型成本後，節省是否仍成立？若模型、提示詞、工具或資料分布改變，過去世界應如何降權？prefix-only 限制除了提示詞規範之外，有沒有足夠的 sandbox／資訊隔離？

### 相關
- 綜述：[[【Recursive Self-Improvement in AI：From Bounded Self-Refinement to Autonomous Research Loops】]]（依改進對象、驗證訊號與人工參與程度整理自我改進研究）。
- 概念：[[World model|世界模型]]、[[Replay simulator|歷史重播模擬器]]、[[Pareto hypervolume|Pareto 超體積]]。
- 相關論文：[[【Optimizing generative AI by backpropagating language model feedback】]]；[完整筆記／動畫入口](https://papers.poyuanchu.com/2025-yuksekgonul-textgrad/)。
- 方法來源與程式來源見下列 Sources；教學示意數據不屬於作者的實驗結果。

## Sources

[1] https://arxiv.org/html/2609.14858v2
[2] https://arxiv.org/abs/2609.14858
[3] https://github.com/zhengkid/Dream-RSI
[4] https://github.com/google-research/dream-rsi


### 延伸閱讀：方程式搜尋

- [[【LLM-SR：Scientific Equation Discovery via Programming with Large Language Models】]]：同樣結合固定 LLM 與外部回饋；LLM-SR 改方程式骨架並擬合數值係數，不是改提示文字或探索控制器。
