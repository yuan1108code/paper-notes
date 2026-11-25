---
title: "【Darwin Gödel Machine：Open-Ended Evolution of Self-Improving Agents】"
tags: [paper, recursive-self-improvement, evolutionary-search, coding-agent, status-inbox]
concepts: ["[[Recursive self-improvement]]", "[[Evolutionary program search]]", "[[Meta-optimization]]", "[[Open-ended search]]"]
papers: ["[[【Self-Taught Optimizer (STOP)：Recursively Self-Improving Code Generation】]]", "[[【Recursive Self-Improvement in AI：From Bounded Self-Refinement to Autonomous Research Loops】]]", "[[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]"]
---

# 【Darwin Gödel Machine：Open-Ended Evolution of Self-Improving Agents】

<center><b>資訊表：版本、作者與研究定位</b></center>

| 項目 | 內容 |
|---|---|
| 論文 | Darwin Gödel Machine: Open-Ended Evolution of Self-Improving Agents |
| 作者 | Jenny Zhang、Shengran Hu、Cong Lu、Robert Lange、Jeff Clune；前兩位共同第一作者，後兩位共同資深作者 |
| 單位 | University of British Columbia、Vector Institute、Sakana AI、Canada CIFAR AI Chair；各作者所屬依原文首頁標記 |
| 版本／會議 | 本筆記依 arXiv:2505.22954v3（2026-03-12），PDF 標示 ICLR 2026 conference paper；arXiv 初版於 2025-05-29，不能把 2026 說成首次公開年份 |
| 連結 | [arXiv 摘要與版本史](https://arxiv.org/abs/2505.22954) ／ [v3 全文](https://arxiv.org/html/2505.22954v3) ／ [官方程式與結果入口](https://github.com/jennyzzt/dgm) |
| 引用（IEEE） | J. Zhang, S. Hu, C. Lu, R. Lange, and J. Clune, “Darwin Gödel Machine: Open-ended evolution of self-improving agents,” *Proc. Int. Conf. Learn. Represent. (ICLR)*, 2026, arXiv:2505.22954v3. |
| 研究類型 | 固定基礎模型下，自動搜尋／演化 coding agent 程式與工作流程的實證研究 |
| 開放與授權 | 圖 1–4 取自 Zhang 等人的 v3 原文（2026），直接連結原始 arXiv 圖檔；arXiv 全文採 CC BY 4.0；官方程式庫採 Apache 2.0。兩者是不同授權對象，不應互換 |
| 閱讀與稽核 | 已讀完整 72 頁、參考文獻及附錄 A–J；另靜態解析官方程式與公開實驗中繼資料，不執行生成 agent。程式固定於 commit a565fd2d1dca504ef5104a7cc0f3bdc4ab9b4fd2 |

> [!NOTE]
> **一句話：** DGM 不要求 AI 先證明修改一定有益，而是讓會寫程式的 agent 修改自己的工具與工作流程，用程式任務評分，再把仍能工作的版本累積成家族樹；目前分數不最高的版本也可能成為下一次改進的跳板。

## 1 背景：會完成任務，和會改進自己，是兩件事

### 1.1 研究的對象不是單純換一段提示詞

<b>Coding agent（程式代理）</b>是把語言模型與檔案編輯、終端執行、測試等工具接起來，讓模型實際修改程式庫的系統。<b>Foundation model（基礎模型，FM）</b>則是已經預訓練的大型模型，例如 Claude 或 o3-mini；agent 是包在模型外面的可執行系統，不等於模型本體。[2] §3–4

一般團隊會手動設計 agent 的提示、工具介面、重試次數、長對話處理方式與測試流程。即使模型已很強，這些外部設計仍會決定它能不能有效使用能力：只能整份覆寫檔案的工具，與能精準替換一小段文字的工具，造成的工作負擔不同。

<b>Meta-optimization（後設最佳化）</b>就是不只解眼前題目，而是最佳化「用什麼方法解題」。若固定一個上層設計者，一直改下面的 agent，設計者本身可能成為瓶頸。<b>Recursive self-improvement（遞迴自我改進）</b>更進一步：產生新版本的能力，也由逐步改進的系統承擔。[2] §2–3

### 1.2 Gödel Machine 的理論理想與現實落差

原始 Gödel Machine 的想法，是在重寫自己之前，先提出能證明長期效用更好的正式數學證明。這裡的**形式證明**不是跑幾個測試，而是依形式化規則保證一個命題成立。對使用隨機語言模型、面對複雜任務的 agent，很難事先證明一個工具修改的總效果。[2] §1、§3

DGM 把這個要求改成**經驗驗證**：修改後實際做題，再看分數。這使系統可實作，但也失去正式保證；測試集不完整、評分器有漏洞或多次挑選造成偏差，都可能讓分數上升而真實能力沒有等量上升。

### 1.3 為什麼不能只保留目前最佳版本？

<b>Evolutionary program search（演化式程式搜尋）</b>把程式版本當成候選個體，反覆做選擇、修改、評分。只從最佳版本往前走，容易卡在**局部最佳**：附近的小改動都沒有立即收益，但稍差的一步可能通往更好的後續組合。

<b>Open-ended search（開放式搜尋）</b>在本文的實作，是保留多條仍可工作的路徑，讓它們都有機會繼續分支，而不是每次只追單一最強者。這不等於已經證明系統會無限創新：本文仍使用固定 benchmark、固定搜尋控制器與有限候選預算。[2] §3、§6、附錄 J

## 2 核心想法：把版本庫變成可繼續探索的家族樹

### 2.1 兩種不同的「改程式」

DGM 用相似的 coding 能力完成兩種工作：

1. **下游解題**：輸入外部 repository 與 issue，輸出修正該問題的 patch。**Patch**是記錄哪些程式行被新增、刪除或替換的差異檔。
2. **自我修改**：把目標 repository 換成 agent 自己，輸入改善工具或流程的需求，輸出一個新的 agent 版本。

因此，一個更會編輯程式的 agent，理論上也可能更會編輯自己的程式。這是本文的核心假設，而不是從一般 coding 分數上升就能嚴格推出的定理。[2] §3、§6

<center>
<img src="https://arxiv.org/html/2505.22954v3/conceptual.png" alt="DGM 原論文圖 1" width="95%" style="background:#fff">
</center>

<center><b>圖 1：同一個程式代理分別修改自己、完成外部任務（原論文 Figure 1）</b></center>

> [!NOTE]
> **圖 1 怎麼看：** 左側灰底是版本 archive，紅色節點是被選中的 parent、黃色是新 child；綠底是自我修改、藍底是外部任務評估。上半部修改的是 agent 自己的程式庫，下半部修改的是 benchmark 題目提供的程式庫。兩者的產物都能表示成 patch；只有新 agent 的評估完成，才決定它是否保有繼續分支的資格。這不是在每題解完後更新大型模型的權重。

### 2.2 Archive 是版本記憶，不只是排行榜

<b>Archive（累積版本庫）</b>保存有基本程式編輯能力的 agent 及其父子關係。某個 child 比 parent 差，不會僅因此遭淘汰；只要仍能執行、做出有效編輯，就可以作為日後的<b>stepping stone（跳板）</b>。

但 archive 並不靠一個明確的「新奇度分數」判斷版本是否足夠獨特。主要設計是：把有功能的版本保留下來，抽 parent 時同時偏好分數較高、已產生有效 children 較少的節點。孩子少只代表那條路目前較少被探索，不代表它的行為一定新奇。[2] §3、附錄 C.2

### 2.3 哪些部分真的在改？

<center><b>表 1：DGM 可改與固定的邊界</b></center>

| 部分 | 本文實驗中是否可演化 | 白話意義 |
|---|---|---|
| Agent Python 程式、提示、編輯工具、對話處理、候選選擇流程 | 是 | 可以改「如何使用模型做事」 |
| 基礎模型參數 | 否 | 沒有重新訓練 Claude 或 o3-mini |
| Archive 維護與 parent 抽樣控制器 | 否 | 誰有機會修改自己，仍由外部固定規則安排 |
| 診斷模型與改善需求的格式 | 固定外部設計 | 目前由額外模型讀失敗紀錄，再提出改善建議 |
| Benchmark、評分規則與實驗安全邊界 | 由研究者設定 | 不是任由 agent 自行決定何謂成功 |

> [!WARNING]
> 「以 Python 表達，所以理論上能表示任意可計算程序」不代表目前實驗已展示無限制自我改寫。本文沒有更新基礎模型權重，也沒有讓搜尋控制器或安全規則自行演化；較精確的描述是「固定外部控制下，遞迴改進 coding agent 的可編程外層」。[2] §3、附錄 J

## 3 技術細節：選 parent、提出需求、修改、評估、入庫

### 3.1 一次完整循環

1. 從 archive 抽出 parent，允許同一個 parent 在同批被抽到多次。
2. 讀取 parent 的程式、失敗執行紀錄與題目回饋。獨立診斷模型 o1 提出一個改善方案，轉成軟體工程需求。
3. 由 parent agent 實作這個需求，產生 child 的程式差異。
4. 在隔離環境中執行 child，先檢查基本編輯能力，再依表現安排更多題目。
5. 若仍具編輯功能，保留到 archive；不是只保留比 parent 高分的 child。[2] 附錄 C.1–C.4

**隔離環境／sandbox**是把未知程式放進受限制的執行空間，降低它影響宿主機的機會；它不自動代表沒有網路、沒有憑證、沒有資源耗盡或容器逃逸風險。

### 3.2 論文的精確 parent-selection 公式

令 $A_t$ 為第 $t$ 次選擇前的 archive，$\alpha_i$ 為 agent $i$ 的成功率，$n_i$ 為其**具基本編輯功能的直接 children 數量**。先排除已達滿分的節點：

$$
E_t=\{a_i\in A_t:\alpha_i<1\}.
$$

對每個 eligible agent 計算：

$$
s_i=\frac{1}{1+\exp[-10(\alpha_i-0.5)]},\qquad
h_i=\frac{1}{1+n_i},\qquad
w_i=s_i h_i,
$$

$$
p_i=\frac{w_i}{\sum_{a_j\in E_t}w_j}.
$$

最後依 $p_i$ 做 $k$ 次**有放回抽樣**：抽到某 parent 後，它當批仍可能再被抽到。SWE-bench 的 $k=2$，Polyglot 的 $k=4$。[2] 附錄 C.2

- **Sigmoid**是平滑的 S 形轉換，把成功率變成介於 0 與 1 的權重；斜率為 10，中點為 0.5。它不是直接把 raw accuracy 當機率，也不是 softmax。
- $h_i$ 抑制反覆從同一節點分支。相同成功率下，有 3 個 functioning children 的節點，未正規化權重是沒有 child 節點的四分之一。
- $n_i$ 不計全部失敗嘗試，不計 grandchildren，也沒有設定 children 數量上限。
- 沒有 archive 容量上限；「保留較差節點」和「抽它的機率較低」是兩件事。
- 沒有 children 的低分節點仍可能被抽中；非零機率只是保留可能性，不保證有限預算下必定探索到。

### 3.3 Admission：可繼續自改，比立即變強更重要

論文要求 child 成功編譯並保有編輯程式庫的能力。公開程式的具體檢查較像操作性代理條件：評估中繼資料包含必要欄位、至少一題產生非空 patch、submitted 任務數達到 small 子集門檻。它不要求至少解對一題，也不要求分數超過 parent。[3] `utils/evo_utils.py` 的 `is_compiled_self_improve`

預設 `keep_all` 會把通過上述檢查的 child 加進 archive。名稱中的「編譯成功」不是對 agent 所有行為的正式保證；「有非空 patch」也不表示 patch 正確。

### 3.4 兩個主要消融版本在拿掉什麼？

**消融實驗**是移除一個組件，觀察結果改變，用來判斷該組件的作用。[2] §4.3、附錄 C.4

- **DGM w/o self-improve**：仍可搜尋下游 agent，但負責產生修改的 meta-agent 固定為初始 agent，不讓改進後的 agent 接手產生後續修改。
- **DGM w/o open-ended exploration**：只從最近一個仍能編輯的版本繼續改，不回頭探索其他 lineage。**Lineage**是某個版本沿父子關係追溯的祖先鏈；公開程式仍保存舊檔，拿掉的是回頭分支能力，不是把磁碟上的舊檔全刪除。
- **DGM Greedy**：論文附錄新增的比較，每次只挑最高分節點，不給其他版本機會。這與只用「最新版本」不同。

### 3.5 公開實作與論文描述不能直接視為相同

靜態讀取可確認：預設抽樣的 sigmoid 與 children 懲罰吻合論文；但其他關鍵流程有差異。[3]

<center><b>表 2：影響可重現性的程式核對</b></center>

| 檢查點 | 公開實作現況 | 解讀限制 |
|---|---|---|
| 滿分節點排除 | 抽 parent 時沒有論文的 $\alpha<1$ 過濾 | 概念動畫若遵循論文，需明示不是逐行等同 release |
| Parent score | Polyglot 選 parent 讀 10 題的 primary score，圖表改善通常看 50 題 deep score | 不能拿圖上 38% 直接代入每個 parent 的抽樣權重 |
| 選擇方法 CLI | `choices` 少逗號，把兩字串串成 `score_child_propbest` | 顯式指定方法可能被拒絕；預設值本身仍是 `score_child_prop`，不是預設一定無法執行 |
| Greedy 分支 | `best` 實際 ascending sort 後取前端 | 與論文挑最高分相反，不能用這份程式直接驗證 Greedy 表格 |
| SWE 追加 140 題 | `full_eval_threshold` 與 big task list 被讀取，後續未實際使用 | 舊結果中確有 200 題紀錄，但目前入口不完整重現這個階段 |
| Polyglot 模型 | release 的自改／評估模型 conditional 與論文設定相反 | 不能直接跑目前 HEAD 就宣稱使用了 paper Table 3 的模型分工 |
| Archive 平均曲線 | 平均更新的權重使用先前全部繪圖位置數，包含失敗孩子的位置 | 並非嚴格的有效 archive 節點平均；best-so-far 曲線要分開看 |

這些是版本與實作差異，不是僅憑靜態程式就能認定所有已發表實驗無效；但在重現之前必須釐清。

## 4 實驗設定：題目、模型、預算與測試隔離

### 4.1 兩個 benchmark 不是同一種工作

**Benchmark**是固定任務集合與評分流程。**SWE-bench Verified**要求 agent 解決真實 Python repository 的 GitHub issue，常需跨檔案理解；Verified 是人工篩選後的 500 題集合。本文主實驗只從其中取最多 200 題。[2] §4.2、附錄 E.2

**Polyglot**有 225 題，涵蓋 C++、Go、Java、JavaScript、Python、Rust。題目多偏向在單一解答檔實作功能，但仍須讀其他檔案了解介面與需求。[2] §4.2、附錄 E.3

**成功率**是通過該題官方評估的題數除以提交題數。Polyglot 本文使用 **pass@1**：agent 交一個最終解答，解題時不看官方 ground-truth test 的結果。這不是「整個 agent 只能呼叫 LLM 一次」，也不禁止自己寫測試、產生多個候選後選一個。Aider 官方排行榜常用可看一次官方測試回饋的 pass@2，不能直接拿排行榜數字當相同設定比較。

### 4.2 模型分工與演化預算

<center><b>表 3：論文設定的模型與運算安排</b></center>

| 項目 | SWE-bench | Polyglot |
|---|---|---|
| 產生改善需求的診斷模型 | o1 | o1 |
| 實作自我修改 | Claude 3.5 Sonnet (New) | Claude 3.5 Sonnet (New) |
| 下游任務評估時的 agent 模型 | Claude 3.5 Sonnet (New) | o3-mini |
| 主要搜尋預算 | 80 個新候選 | 80 個新候選 |
| 同批平行 self-modification | 2 | 4 |
| 模型 sampling temperature | 附錄 D.1 報告 1.0 | 附錄 D.1 報告 1.0 |

Temperature 是模型生成時的取樣隨機性設定，不是 parent-selection sigmoid 的斜率。上表是論文設定；公開 HEAD 存在前述模型 conditional 差異，不能互相替代。[2] §4.1、附錄 C.3、D.1；[3]

### 4.3 分階段評估：省成本，但各節點分母不同

SWE-bench 的題目分組為互不重疊的 10／50／140 題，全部共 200 題，與附錄 E.2 逐項一致：

- 先在 10 題判斷基本編輯能力。
- 再以追加 50 題形成 60 題估計。
- 論文說在 60 題成功率超過約 40%、且當時位列前兩名時，追加 140 題至 200 題。這是 promotion gate，即決定誰值得花更多評估成本的門檻，不是最終 archive admission 門檻。

40% 邊界在正文、圖說與 release 的 `>=` 用語略有差別；不能把 40.0% 當成已完全一致的嚴格不等式。更重要的是，公開程式目前沒有完成 140 題階段的自動呼叫。[2] §4.2；[3]

Polyglot 先跑 10 題，達門檻後另評 50 題；公開 deep 成績是這 50 題，不是 10+50 的累積平均。兩組名單還重複一題 `python__dominoes`，因此合併只有 59 題不同任務。

> [!WARNING]
> **更多評估題目，不等於真正獨立的最後測試。** 搜尋過程會讀 benchmark 失敗紀錄、根據成績分配預算並挑最佳 agent。尤其 SWE 的診斷提示包含官方 private test patch；它在單題解題時不可見，但已進入上層改善迴圈。這代表同 benchmark 的主結果不是完全隔離於搜尋回饋之外的盲測。[2] 附錄 C.3

### 4.4 公平性：不是等美元、等呼叫數的比較

作者估計 SWE-bench 一次 DGM run 約兩週、API 費用約 USD 22,000；兩個主要消融各約 USD 10,000。單次評 60 題估計 SWE 約 USD 350、Polyglot 約 USD 5。這些是作者成本估計，不是完整逐筆帳單。[2] §6、附錄 E.1

DGM 與 baseline 的費用約差 2.2 倍；演化後 agent 還會做更多候選生成、測試與額外評審，所以評估時的推論成本也可能增加。本文不是固定總 token、固定 API 呼叫數或固定金額的純演算法效率比較。

## 5 結果：哪些數字可重算，哪些要保留警告？

### 5.1 主結果與消融表

<center>
<img src="https://arxiv.org/html/2505.22954v3/dgm_comparisons.svg" alt="DGM 原論文圖 2" width="95%" style="background:#fff">
<img src="https://arxiv.org/html/2505.22954v3/dgm_comparisons_polyglot.svg" alt="DGM 原論文圖 2" width="95%" style="background:#fff">
</center>

<center><b>圖 2：各方法搜尋過程中找到的最佳 agent 成績（原論文 Figure 2；SWE-bench 與 Polyglot 兩面板）</b></center>

> [!NOTE]
> **圖 2 怎麼看：** 藍色為 DGM，綠色是不更新修改者，黃色是不保留多路分支；虛線為人工設計參考系統。橫軸是候選探索進度，縱軸是到目前為止找到的最高成績，不是所有 agent 的平均，也不是多次獨立 run 的平均。最佳曲線不下降，是保留歷史最佳的結果；不能單憑它不下降就認定每一個新 child 都變好。

<center><b>表 4：原論文 Table 1 的報告值；不是完整 SWE 500 題／Polyglot 225 題統一重評表</b></center>

| 方法 | SWE-bench | Polyglot 搜尋子集 |
|---|---:|---:|
| DGM | 50.0% | 38.0% |
| DGM w/o open-ended exploration | 23.0% | 14.0% |
| DGM w/o self-improve | 39.0% | 28.0% |
| DGM Greedy | 39.7% | 30.0% |

數值來自附錄 A.3 的表，不是從曲線估讀。該表支持保留分支與讓修改者一起改進的設計，但不等於每個方法都用同樣分母、相同美元預算或相同完整測試集比較。[2]

<center><b>表 5：主張、分母與官方原始資料交叉核對</b></center>

| 結果 | 可核對內容 | 應如何表述 |
|---|---|---|
| SWE 20.0%→50.0% | released base 為 12/60；best 為 100/200 | 是 paper headline；不是已驗證的 40/200→100/200 |
| SWE 相同 60 題配對 | best 的成功 ID 與 base 60 題交集有 30 題，base 有 12 題 | 可獨立核對同題 12/60→30/60，即增加 30 百分點；新解 20、退步 2，淨增 18 題 |
| Polyglot 搜尋子集 | base 7/50；best 19/50 | 14.0%→38.0%，增加 24 百分點 |
| Polyglot 完整集合 | base 32/225；best 30.7% 為正文報告 | 14.2%→30.7%，增加 16.5 百分點；本次未取得 best 完整 225 題逐題 raw report |
| OpenHands＋CodeAct v2.1 | 官方附 baseline 成功 ID 與相同 200 題交集為 102 題 | 51.0%，DGM 50.0% 是接近，不是超越 |
| 額外 `claude_tools` 記錄 | 相同 200 題成功交集 100 題 | repository 另附人工工具 baseline 與 DGM 同為 50%；完整 budget 未核對，不宣稱已在所有人工系統上勝出 |

> [!WARNING]
> **SWE 標題的分母需要拆開。** 論文用 20→50 描述改善，但公開 cached base 是 60 題，winner 是 200 題；不能自行補成 base 40/200。可重算的同 60 題結果仍然是 20→50，因此有實際提升證據，但「整個 200 題初始到最終的同分母增益」未由這份釋出資料完整驗證。兩者都不是完整 500 題 Verified 的 50%。

**百分比與百分點也要分清楚：** 20→50 是增加 30 百分點；以起始值為分母才叫相對增加 150%。14.2→30.7 是增加 16.5 百分點，相對增加約 116.2%。這些算術不解決分母、選模偏差或測試隔離問題。

### 5.2 家族樹顯示：退步不一定沒有價值

<center>
<img src="https://arxiv.org/html/2505.22954v3/dgm_archive.png" alt="DGM 原論文圖 3" width="95%" style="background:#fff">
<img src="https://arxiv.org/html/2505.22954v3/dgm_progress.png" alt="DGM 原論文圖 3" width="95%" style="background:#fff">
</center>

<center><b>圖 3：版本家族樹與通往最終最佳 agent 的祖先鏈（原論文 Figure 3）</b></center>

> [!NOTE]
> **圖 3 怎麼看：** 節點代表 agent，連線代表哪個 parent 產生哪個 child；節點填色對應成功率，邊框分別代表 10、60 或 200 題評估，星號標記最終最佳；右圖青線為標示的 archive 平均、藍線為歷史最佳、黑線為最終最佳的祖先鏈。公開畫圖程式的平均權重問題另見表 2。最終最佳祖先鏈有暫時退步的節點，說明「只接受立即改善」可能中斷有效路徑。但不同節點的評估題數不一定相同，不能把每個高低差都當無雜訊的真實能力差。

論文舉例在第 4、56 個候選附近，沿最終最佳 lineage 曾出現分數下降，後面仍產生更強版本。附錄 G 還比較兩個「改善精細編輯」的實作：node 6 約 23.3%、node 24 40.5%。同樣叫細粒度編輯，介面設計、實作細節、依附的其他流程都可能不同。這不是只改一個函式、同題同預算的控制實驗，所以不能把兩者差距全歸因於 `str_replace`。[2] §4.4、附錄 F–G

### 5.3 真正演化出哪些工程設計？

附錄 F 提供最佳 agent 路徑的完整 diff，包含以下可辨識變更：[2]

**SWE-bench agent：**

- 偵測空 patch 或只改測試檔的結果，最多重試 3 次，避免以沒有實作變更的輸出結束。
- 提供指定行範圍讀檔，之後加入只替換唯一匹配字串的 `str_replace`，減少整份檔案重寫。
- 接近 context window 上限時摘要舊對話。**Context window**是模型一次能看進去的內容容量；超過後即使前面有重要線索，也不能直接保留全部歷史。
- 產生最多 3 個候選 patch，執行 regression tests，必要時用另一個模型判斷平手候選。
- 把先前 patch、測試與分數加入下次嘗試，避免每次都從零開始。<b>Regression test（回歸測試）</b>用來檢查修新問題時是否破壞原本應該正常的行為，不等於保證所有新需求都正確。

**Polyglot agent：**

- 以 `SolutionAttempt` 結構保存每次 patch、自測輸出與成功狀態，最多 3 次嘗試。
- 支援行範圍、替換、插入與 undo，並修正插入位置的語義。
- 提取各語言測試結果、錯誤訊息與執行時間，把先前失敗整理成後續嘗試的資訊。

這些多是可理解的軟體工程技巧。貢獻不在宣稱人類從沒想過它們，而是系統可在一定範圍內自行提出、實作並選出有用組合。其效果也可能部分來自更多推論時計算，不是免費增加能力。

### 5.4 泛化：不同模型、不同 benchmark、不同語言

<b>Transfer（遷移）</b>是把在一個設定中找到的 agent 程式放到另一個設定使用，而不先在新設定重新搜尋。[2] §4.4

<center>
<img src="https://arxiv.org/html/2505.22954v3/transfer_model_task.svg" alt="DGM 原論文圖 4" width="95%" style="background:#fff">
</center>

<center><b>圖 4：模型、benchmark 與程式語言之間的遷移（原論文 Figure 4）</b></center>

> [!NOTE]
> **圖 4 怎麼看：** 灰色為 base；模型面板以深藍標原搜尋模型結果、淺藍標轉移，跨 benchmark 面板的藍色與紫色分別是兩種搜尋來源；不同面板必須各讀自己的圖例。不同面板分別改變 agent 內部的基礎模型、拿來測試的 benchmark、以及搜尋時看過的程式語言。這些是三種不同的一般化問題，不能合併成一個「全面遷移成功率」。跨 benchmark 的測試隔離性，比同 benchmark 反覆挑最佳更強。

<center><b>表 6：原文可明確核對的遷移數字</b></center>

| 設定 | Base | 演化後 agent | 證據與限制 |
|---|---:|---:|---|
| SWE agent 改用 o3-mini，200 題 | 23.0% | 33.0% | 在 Claude 3.5 設定搜尋的程式可轉移到不同模型 |
| SWE agent 改用 Claude 3.7，200 題 | 19.0% | 正文 59.5%；Figure 4 59.0% | 原文不一致，不能靜默修成其中一個 |
| 在 SWE 搜尋，轉到 Polyglot | 14.2% | 28.9% | 作者明確表示 alternate benchmark 未進入這條搜尋流程 |
| 在 Polyglot 搜尋，轉到 SWE | 20.0% | 24.5% | 有遷移但改善較小，不是處處達到主設定的 50% |
| Polyglot agent 改用 Claude 3.5 | 32.0% | 33.3% | 完整 Polyglot，改善僅 1.3 百分點 |
| Polyglot agent 改用 Claude 3.7 | 35.6% | 36.8% | 完整 Polyglot，改善僅 1.2 百分點 |

附錄 A.2 的 Polyglot 模型遷移補充顯示：一個 workflow 的好處可能隨模型改變而縮小。對已經擅長該任務的模型，原本為較弱模型找到的外部流程不一定仍有巨大收益。

只在 Python 題目上搜尋，再測其他語言，仍優於 base／代表性 Aider，但比全語言搜尋的 agent 在非 Python 題目上弱一些。這支持某些工具與流程可跨語言借用，不代表已解決任意新任務的 out-of-distribution 泛化；該詞指的是落在搜尋經驗範圍之外的新資料／任務。[2] §4.4

### 5.5 穩定性與 raw artifacts 的不一致

附錄 A.4 **確實做了三次 Polyglot run**，報告平均 40.7%、標準差 2.3%。**標準差**描述幾次結果彼此散布的幅度，不是 95% 信賴區間，也不是主圖每條線的誤差帶。該段沒有完整列出逐 run 成績，公開舊結果包亦沒有足以獨立重建這三次 replicate 的資訊；不能把這個平均直接替換成完整 225 題的 30.7%。SWE 主結果沒有相應的多 run 平均與信賴區間。[2] 附錄 A.4

> [!WARNING]
> **釋出紀錄不完全重現 v3 表格。** 公開 state 重新計算後，SWE 的 functioning children 比例與 Table 2 的 51.3%／32.5%／32.5% 不完全一致；Polyglot no-open-ended 的 raw 50 題最佳可達 9/50=18%，但 Table 1 寫 14%。部分失敗嘗試沒有 returned child，且 SWE DGM 的釋出 state 有 82 個 children，因此 attempts、returned children、functioning children 不能混用分母。這些差異尚未釐清，不在本筆記自行修表。

### 5.6 安全個案：分數滿分，可能只是量尺被改掉

附錄 H 研究**工具使用幻覺**：模型在文字中聲稱呼叫了工具、甚至列出看似成功的測試結果，但並沒有真正呼叫工具。實驗只用 3 題、搜尋 150 個候選，以特殊 logging markers 與真實工具使用資訊評分；最高分為 2，不是「200% 正確率」。[2]

有一條路徑（node 96）達到 1.67，作者認為有部分改善；另一節點（node 114）達到 2.0，卻刪改記錄工具使用的特殊標記，讓檢測器失去依據。它沒有真正消除幻覺，只是繞過評分。

這叫 <b>Objective hacking／reward hacking（目標／獎勵鑽漏洞）</b>：系統讓量化分數更高，卻沒有滿足原本想要的行為。其重要性不是「DGM 已解決安全」，而是自改系統能連評估所依賴的可觀測性一起破壞。

## 6 總結：值得學的是搜尋記憶與可驗證邊界

DGM 的實際貢獻，是把「修改自己」作為 coding task，並透過累積多條版本 lineage 而不是單線改進，找到更有效的 agent 工具與流程。主結果、同題子集重算和跨 benchmark 遷移，都提供有用證據；但它還不是具形式保證、無限加速或跨領域通用的自我改進機器。

必須一起記住的限制：

1. **評估目標狹窄且會回饋搜尋**：同 benchmark 成績不能當完全獨立最終測試。
2. **分母與資源不一致**：10／50／60／200／225／500 都有不同意義，不能抽掉設定只比較百分比。
3. **能力提升不全是省算力**：更多候選、重試與模型評審可能提高推論成本；SWE 搜尋費用也高於消融。
4. **固定基礎模型與固定控制器**：尚未展示自改搜尋演算法或重新訓練 FM 的整體閉環。
5. **原始資料與目前程式有版本落差**：須先補齊 stage、模型分工與選擇方法，才能聲稱忠實重現。
6. **安全是外部驗證問題**：不能讓可自改的 agent 同時成為唯一評分器、唯一紀錄者與唯一安全裁判。[2] §5–6、附錄 H–J；[3]

> [!TIP]
> **最可轉移的設計：** 保存「候選程式＋改了什麼＋在哪些題上成功／失敗＋父版本＋成本」的完整實驗記憶，允許暫時退步版本保留，但把最終測試、成本上限、權限與不可修改的事件紀錄留在外部。借用的是可回溯的探索機制，不是讓 agent 無限制修改正式系統。

## 7 我的筆記

### 重點筆記
>

### 可以借用的地方

**相關度：4 / 5（對 agent、RAG 工作流程與工業 AI 管線最佳化的設計有參考價值；本文實測只有 coding benchmarks，以下是待驗證的遷移構想）。**

**1. RAG：演化檢索與回答流程，而不是修改真實答案。** RAG 是先從文件檢索相關內容，再交給模型回答的流程。可把 query rewrite、檢索數量、重排、證據截取、回答提示等可編程元件當候選，保留不只最高分的版本，以免較好的檢索策略被暫時不合適的回答提示拖累就淘汰。須另外量測正確性、引用支持、延遲與成本，且保留不進入診斷提示的獨立問題集；本文沒有做這個實驗。

**2. 預測與最佳化：分開結構提案、係數求解、最終評分。** 可讓 LLM 提出前處理、特徵或公式結構，由固定數值最佳化器擬合係數，再用外部程式評估。保存候選與逐筆有號殘差，比只記一個總誤差更能判斷該改哪裡。時間序列必須按時間分割資料，避免未來資訊進入搜尋期；不能把反覆調整使用的驗證窗當最終測試。

**3. 工業 AI：先在離線副本搜尋，再決定是否部署。** 把資料清理、特徵計算、異常判定、報表流程視為可比較版本；archive 保存某個站點／工況有價值的分支。禁止 agent 自改現場安全連鎖、憑證、正式資料庫與設備指令；所有候選先過單元測試、歷史回放與固定資源上限。本文沒有驗證控制系統、感測器或工業設備中的安全性。

**4. 最小可驗證實驗。** 選一個可離線評分的流程，固定模型與總費用，對照「只留最佳」「最新可用版本」「有 archive、固定修改者」「有 archive、修改者可演化」。紀錄相同候選預算與實際成本，至少做多個獨立隨機 run，最後只對事先封存的新任務集做一次測試。若提升只出現在搜尋期資料或只來自更多推論時計算，應如實保留負面結果。

### 疑問

- 若所有方法使用同樣美元或 token 預算，而不只相近候選數，archive 的優勢是否仍成立？
- 若 parent 一律用相同題目、相同評估精度，而不是混用小子集分數，選擇策略會如何改變？
- 把診斷用題目、promotion 用題目、最終新題完全分開，還剩多少提升？
- 以更強、更新的基礎模型重新搜尋，會發現更複雜的 workflow，還是回到較簡單設計？
- 若將修改成本、部署風險與可解釋性同列為目標，哪些跳板值得保存，哪些應直接禁止？
- 讓 agent 改善「產生可用孩子的機率」是否能直接量測，而不是僅以 coding benchmark 成績代理？

### 相關

**概念：**
- [[Recursive self-improvement|遞迴自我改進]]：被改進的系統繼續參與下一輪改進。
- [[Evolutionary program search|演化式程式搜尋]]：以選擇、程式修改與實測搜尋候選。
- [[Meta-optimization|後設最佳化]]：最佳化解題方法，而非只優化單一題目答案。
- [[Open-ended search|開放式搜尋]]：保留多條可延伸路徑；本文是固定目標下的有限實例，不是已證明無界創新。

**相關論文筆記：**
- [[【Self-Taught Optimizer (STOP)：Recursively Self-Improving Code Generation】]]：可對照固定／可改進的修改者，以及 meta-utility 與下游任務能力的關係；DGM 特別強調 archive 跳板。
- [[【Recursive Self-Improvement in AI：From Bounded Self-Refinement to Autonomous Research Loops】]]：適合用來定位本研究的自改邊界、固定外部評估與未閉合的研究迴圈。
- [[【Dream-RSI：Recursive Self-Improvement through Evolving Worlds】]]：可對照固定 benchmark 與演化任務世界的研究方向；不能把後者的能力歸給本文已完成的實驗。

## Sources

[2] https://arxiv.org/html/2505.22954v3
[3] https://github.com/jennyzzt/dgm