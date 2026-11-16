---
title: "【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】"
tags: [paper, llm, rag, multi-hop-qa, planning, status-inbox]
concepts: ["[[Retrieval-augmented generation]]", "[[Multi-hop question answering]]", "[[Query decomposition]]", "[[Cost-of-pass]]", "[[ReAct]]", "[[Chain-of-thought prompting]]", "[[LLM-as-a-judge]]"]
papers: ["[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]", "[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]", "[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]", "[[【A Survey on LLM-as-a-Judge】]]"]
---

# 【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 作者 | Wei-Chieh Chou、Xuanjun Chen（共同第一作者）、Jian-Ren Lin、Claire Lin、Hung-yi Lee（李宏毅）、Jyh-Shing Roger Jang（張智星），全部來自國立臺灣大學 |
| 發表 | COLM 2026（arXiv 註記 Accepted by COLM 2026）；arXiv 2606.22681（v1 2026/6/21，v2 2026/9/27，兩版內容相同，只改致謝與排版） |
| 連結 | [arXiv](https://arxiv.org/abs/2606.22681)・[PDF](https://arxiv.org/pdf/2606.22681)・[HTML](https://arxiv.org/html/2606.22681v1) |
| 動畫解說 | [papers.poyuanchu.com/2026-chou-gdprag/](https://papers.poyuanchu.com/2026-chou-gdprag/) |
| 論文類型 | 方法論文：不需訓練的多步驟 RAG 規劃方法 GDP-RAG |
| 狀態 | 待讀 |
| 與我的相關度 | 5 / 5（多步驟 RAG 的成本與正確率；核心只是「先檢索、再規劃」加一段提示詞，可直接套到現有 RAG 系統） |
| 加入日期 | 2026-11-12 |

> [!NOTE]
> **一句話**：多步驟 RAG 常見的做法是先讓 LLM 把問題拆成好幾個子問題，再一個個去查。但 LLM 是**憑記憶在拆**，常拆出多餘的步驟，也會漏掉步驟。GDP-RAG 改成**先用原問題檢索一次，把已經查得到的事實記在 Thought 裡，只針對「還不知道的部分」（delta，∆）出子問題**。結果是步驟變少、成本下降，正確率反而最高。

---

## 1. 背景：多步驟 RAG 的兩條路

### 什麼是多步驟問答

**多步驟問答**（multi-hop QA）是答案不在單一段落、要串起兩份以上文件才能回答的問題。例如「Joseph D. Stewart 擔任校長的那所學院在哪裡？」要先查到學院是美國商船學院，再查這所學院在 Kings Point, New York。只用原題目檢索一次，常常找不到第二份文件。

所以現在的 RAG 系統大多會做**問題拆解**（query decomposition）：把大問題拆成幾個子問題（subquery），每個子問題各自檢索、回答，最後再組合。依照「什麼時候拆」，分成兩派：

<center><b>表 2：兩種多步驟 RAG 做法</b></center>

| | 逐步式（step-wise） | 計畫式（plan-based） |
|---|---|---|
| 做法 | 走一步看一步：每查完一次，再決定下一個子問題 | 一開始就把全部子問題規劃好，再依序執行 |
| 代表方法 | ReAct、Self-Ask、IRCoT、KnowTrace、Search-o1 | ReWOO、PlanRAG、PAR-RAG |
| 優點 | 有彈性，能根據新資料調整 | 結構穩定，步數在規劃時就決定 |
| 缺點 | 檢索到的文件越堆越多，超過模型能有效讀的長度（lost-in-the-middle，長輸入中間的資訊容易被忽略）；前一步錯，後面跟著錯；要靠 LLM 自己決定何時停 | 規劃時**沒看過任何文件**，只能憑模型記憶拆題 → 多拆或少拆；每一步都完整執行，成本高 |

### 計畫式的兩個毛病

論文把計畫式方法（文中稱 **Direct Planning**，直接規劃）的問題歸納成兩種：

- **過度規劃**（over-planning）：拆出不必要的步驟。例如「Vanderbilt 和 Emory 哪間大學先成立？」直接規劃會拆成「Vanderbilt 哪年成立」「Emory 哪年成立」「哪間比較早」三步，第三步根本不用檢索
- **規劃不足**（under-planning）：漏掉需要的步驟。子問題寫得太窄，某份關鍵文件永遠沒被查到

再加上計畫式方法通常每一步都要跑完整的「檢索 → 回答 → 驗證」流程，步數一多，成本就直線上升。

> [!TIP]
> 這和實務上很常見的狀況一樣：使用者的問題，其實第一次檢索就已經找到大半答案了，但系統還是照著拆好的步驟一題題查，花了好幾倍的 token。

---

## 2. 核心想法：先檢索，只問不知道的

<center>
<img src="https://arxiv.org/html/2606.22681v1/GDP-RAG.svg" alt="圖 1" width="95%">
<br>
<b>圖 1：直接規劃 vs. 有根據的差量規劃（原論文 Figure 1）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**
> - **左框（A）直接規劃**：問題 → 規劃模組（機器人圖示）→ 子問題路徑，規劃時沒有看任何文件。三種情況：
>   - Case 1：理想路徑 A → B → C → D
>   - Case 2：過度規劃，多出一個**紅色的 X**（多餘的子問題）
>   - Case 3：規劃不足，路徑變成**紅色的** A → C → D，漏掉 B
> - **右上框（B）有根據的差量規劃**：問題先送進檢索器（放大鏡），**預先檢索**拿到 Doc A、B，再一起交給規劃模組。思考泡泡寫著「A → B 已經解決，只需要 C → D」，所以輸出的子問題路徑只有**綠色的** C → D；右下打勾的問號表示 A → B 已經被預先檢索的資料解掉（推理步驟壓縮）
> - **右下虛線框**：三個好處。推理步驟壓縮（跳過已解決的步驟）、過度規劃修剪（跳過多餘的 X）、規劃不足修補（補回漏掉的 B）

### 三個設計

GDP-RAG（Grounded Delta Planning RAG）的名字可以拆開看：**Grounded**（規劃前先用檢索結果打底）、**Delta**（只規劃還缺的那一塊）。實作上只有三個改動：

1. **預先檢索**（preliminary retrieval）：規劃前，先拿**原問題**做一次粗略檢索，得到文件集 $D_{rel}$。規劃器因此知道「手上已經有什麼」
2. **缺口導向的問題拆解**（gap-aware query decomposition）：規劃提示詞多兩條規則：（1）先找出文件裡已經有的資訊；（2）**只為文件裡沒有的資訊建立步驟**
3. **骨架軌跡**（skeletal trajectory）：每個步驟除了子問題 $q_i$，還要寫一個 **Thought** $\theta_i$，記下「從預先檢索已經知道了什麼、這一步還要找什麼」。Thought 會一路保存到最後回答的階段

$$T_{skel} = \text{Plan}(Q, D_{rel}) = \langle(\theta_1, q_1), \ldots, (\theta_n, q_n)\rangle$$

Thought 有兩個作用：

- 規劃器必須**替每個子問題說明理由**，不需要的步驟就比較難寫出來
- 預先檢索找到的證據不會在執行過程中弄丟，最後回答時還看得到（前作 PAR-RAG 也會寫 Thought，但寫完就丟掉）

> [!TIP]
> 三個設計各管一件事（第 6 節消融實驗）：**預先檢索**讓步驟變少；**缺口規則**是步驟變少的主因；**Thought** 是正確率的主因。

### 兩段提示詞的差別

<center><b>圖 2：直接規劃的提示詞（依原論文 Figure 5 以文字重現）</b></center>

```text
Think carefully, build a step-by-step plan for this question: {question} using JSON format.
  For each step in the plan, generate a sub-question.

# Format instructions
Use the following Strict format, only choose an action from the list:[Retrieve, Answer]:
[
  {{
    "Thought":"[your thought about the current step]",
    "Question":"[the question you generated for the current step]",
    "Action": "[the action you chose]"
  }}
]
Return only the JSON array, no explanation.
```

<center><b>圖 3：GDP-RAG 的規劃提示詞（依原論文 Figure 6 以文字重現）</b></center>

```text
Think carefully, build a step-by-step plan for this question: {question} using JSON format.
  For each step in the plan, generate a sub-question.
{formatted_passages}

IMPORTANT: Before creating steps, analyze what information you already have:
1. Identify what specific information is already available in the passages
2. Only create steps for information that is NOT already in the passages
For each step in the plan:
- Write down known information from the passages in "Thought" so it is carried forward
 - Only ask for information that is genuinely missing
# Format instructions
[
  {{
    "Thought": "[summarize what is already known from the passages and explain what this step still needs to find]",
    "Question": "[the sub-question for the missing information]",
    "Action": "Retrieve"
  }}
]
Return only the JSON array, no explanation.
```

> [!NOTE]
> 差別只有三處：多了 `{formatted_passages}`（預先檢索的段落）、多了 IMPORTANT 那兩條規則、Thought 的說明從「你對這步的想法」改成「總結已知資訊並說明還缺什麼」。Action 固定為 Retrieve，因為留下來的每一步都是真的缺資料。

---

## 3. 技術細節：三個階段

<center>
<img src="https://arxiv.org/html/2606.22681v1/overview_new.svg" alt="圖 4" width="95%">
<br>
<b>圖 4：GDP-RAG 整體架構（原論文 Figure 2）</b>
</center>

> [!NOTE]
> **圖 4 怎麼看**（由左到右）
> - **左欄 Planning（米黃底）**：由下往上讀。問題 → 預先檢索（放大鏡）→ 文件 + 問題一起進「缺口導向問題拆解」→ 產出**骨架軌跡**（粉紅色的條列）。上方箭頭「Extract next step (N steps)」表示逐步取出子問題交給中間
> - **中欄 Trajectory Generation（淺藍底）**：每一步跑三個模組，由上往下：
>   - **Act**：子問題 + 文件 + 軌跡記憶 → 暫時答案（Subanswer）
>   - **Review**：子問題 + 暫時答案 + 文件 + 軌跡記憶 → 修正後的答案（Refined Subanswer）
>   - **Update**：子問題 + 篩選後的文件 + 修正後答案 → 存進右邊的軌跡記憶（Save），並沿最下方的箭頭「Refine next step」改寫下一個子問題
> - **中欄右側 Trajectory Memory**：每一步存一筆（Trajectory 1、2 … N），三個綠勾代表子問題、文件、答案都完成；Trajectory N 的紅叉代表這步還沒跑完。Act 和 Review 都會讀取（Utilize）記憶
> - **右欄 Answering（淺綠底）**：原問題 + 完整的軌跡記憶 → 最終答案

### 3.1 問題定義

給定問題 $Q$ 和文件庫 $D$，要建立一條推理軌跡 $T = \langle(q_1, d_1, a_1), \ldots, (q_N, d_N, a_N)\rangle$，每一步是（子問題、檢索到的文件、中間答案），最後 $A = \text{Answer}(Q, T)$。目標是**答案品質最高，同時步數 $N$ 最少**。GDP-RAG 把每步擴充成 $(\theta_t, q_t, d_t, a_t)$，多了 Thought。

### 3.2 規劃階段

如第 2 節：預先檢索 → 缺口導向拆解 → 骨架軌跡。

### 3.3 軌跡生成階段：Act → Review → Update

這一段**沿用 PAR-RAG** 的 Act-Review-Update 循環，軌跡記憶 $M_t$ 存到第 $t$ 步為止的所有 $(\theta, q, d, a)$。

<center><b>表 3：每一步的三個模組</b></center>

| 模組 | 輸入 | 做什麼 | 公式 |
|---|---|---|---|
| Act | 子問題、文件庫、記憶 | 用 $q_t$ 檢索，寫出**附引用**的暫時答案 $\hat{a}_t$ | $\hat{a}_t = \text{Act}(q_t, D, M_{t-1})$ |
| Review | 子問題、暫時答案、文件庫、記憶 | 把 $q_t$ 和 $\hat{a}_t$ 接起來當新查詢，**再檢索一次**，用新文件核對、修正答案 | $a_t = \text{Review}(q_t, \hat{a}_t, D, M_{t-1})$ |
| Update | 子問題、答案、文件、記憶 | 把這步寫進記憶（只存真正支持答案的引用文件），並根據新發現**改寫下一個子問題** | $(M_t, q_{t+1}) = \text{Update}(q_t, a_t, d_t, M_{t-1})$ |

和 PAR-RAG 的差別（附錄 A.2）：

- PAR-RAG 規劃時每步標 Retrieve 或 Answer，標 Answer 的步驟直接用記憶回答、不檢索
- GDP-RAG 規劃時已經把多餘的步驟剪掉，所以**每一步都跑完整循環**，不讓任何中間答案只靠模型記憶
- 記憶同時提供給三個模組，Act 和 Review 引用的文件都會存進去

### 3.4 回答階段

把原問題和完整的軌跡（含 Thought、子問題、引用文件、驗證過的答案）一起交給 LLM，寫出最終答案。

<center><b>演算法 1：GDP-RAG 流程（依原論文 Algorithm 1 以文字重現）</b></center>

```text
D_rel  = Retrieve(Q, D)                 # 預先檢索
T_skel = Plan(Q, D_rel)                 # 缺口導向拆解
for t = 1..n:
    (θ_t, q_t) ← T_skel
    d_t   = Retrieve(q_t, D)
    â_t   = Act(q_t, d_t, M_{t-1})       # 暫時答案
    d*_t  = Retrieve(q_t + â_t, D)       # 用「問題+答案」再查一次
    a_t   = Review(q_t, â_t, d*_t, M_{t-1})
    (M_t, q_{t+1}) = Update(q_t, a_t, d_t ∪ d*_t, M_{t-1})
T = M_n
A = Answer(Q, T)
```

---

## 4. 實驗設定

<center><b>表 4：實驗設定</b></center>

| 項目 | 設定 |
|---|---|
| 資料集 | HotpotQA、2WikiMultiHopQA（2Wiki）、MuSiQue；表格中分別記為 l1、l2、l3 |
| 篩題 | 先讓 GPT-4.1-mini **不檢索直接回答**，答對（Substring Match）的題目丟掉，只留需要檢索的題目。丟掉比例：HotpotQA 37.62%、2Wiki 36.20%、MuSiQue 11.05% |
| 題數 | 每個資料集 600 題，依步數平均抽樣：HotpotQA、MuSiQue 為 2／3／4 跳各 200 題；2Wiki 沒有 3 跳，為 300／0／300。HotpotQA 本身都是兩跳，這裡的「跳數」指支持文件數 |
| LLM | 所有方法都用 GPT-4.1-mini（規劃與生成）；另測 GPT-4.1-nano、GPT-5.5（只測規劃） |
| 檢索 | BAAI/bge-m3 向量檢索取前 30 段 → BAAI reranker（重新排序模型）取前 10 段 |
| 比較方法 | 逐步式：IRCoT、KnowTrace（每步把知識整理成知識圖）、Search-o1（邊搜邊推理，維護摘要）；計畫式：PAR-RAG、Godbole et al.（先依初始檢索寫大綱再出查詢，原本用於寫文章，作者只保留它的規劃部分，接上 GDP-RAG 的執行與回答模組）；另有 Standard RAG（檢索一次就回答） |
| 公平性 | 全部不需訓練；KnowTrace 只用推論時的版本，不做自我微調；逐步式方法最多 10 步 |
| 硬體 | 單張 RTX 3090（只跑檢索模型，LLM 走 API） |

### 評估指標

- **正確率 Acc**：EM、F1、SM 三者的平均
  - **EM**（Exact Match）：答案和標準答案完全一樣才算對
  - **F1**：字詞層級的重疊程度，答案多講廢話會扣分
  - **SM**（Substring Match）：標準答案出現在回答裡就算對，最寬鬆
- **成本**：每題花多少美分（¢），以 GPT-4.1-mini 價格計算（輸入每百萬 token 0.40 美元、輸出 1.60 美元），以及總 token 數
- **Cost-of-pass**（主要指標）：**得到一個正確答案的期望成本** $= C / \text{Acc}$。又貴又不準的方法分數會很差，便宜但常錯的方法也不會太好看

<center><b>表 5：三種正確率指標的例子（原論文 Table 5）</b></center>

| 預測（標準答案：Bob） | EM | F1 | SM |
|---|---|---|---|
| Bob. | 100% | 100% | 100% |
| Bob's father is older. | 0% | 33% | 100% |
| Bob's father is older than John's. | 0% | 22% | 100% |

---

## 5. 結果

### 5.1 主要結果

<center><b>表 6：主要結果（原論文 Table 1；all 為三個資料集的平均）</b></center>

| 方法 | Cost-of-pass ↓ | Acc（%）↑ | HotpotQA | 2Wiki | MuSiQue | 成本（¢）↓ | Tokens ↓ |
|---|---|---|---|---|---|---|---|
| Standard RAG | **0.14** | 37.86 | 56.41 | 31.89 | 25.26 | **0.04** | **1,077** |
| IRCoT | 0.17 | 44.56 | 54.11 | 52.99 | 26.58 | 0.06 | 1,284 |
| KnowTrace | 1.57 | 51.65 | 55.66 | 68.31 | 30.99 | 0.61 | 13,519 |
| Search-o1 | 0.49 | 50.73 | 51.03 | **72.66** | 28.51 | 0.21 | 4,095 |
| PAR-RAG | 0.65 | 58.20 | 62.79 | 69.10 | 42.70 | 0.34 | 7,112 |
| Godbole et al. | 1.12 | 46.28 | 59.42 | 44.83 | 34.59 | 0.45 | 9,606 |
| **GDP-RAG** | 0.51 | **60.63** | **64.75** | 72.62 | **44.53** | 0.27 | 5,717 |

重點：

- **正確率最高**：60.63%，比最強的 PAR-RAG 高 2.43 個百分點；HotpotQA、MuSiQue 第一，2Wiki 以 0.04 點之差輸 Search-o1
- **比 PAR-RAG 便宜**：成本 0.27 vs. 0.34 ¢，token 5,717 vs. 7,112；cost-of-pass 0.51 vs. 0.65（低 21.5%，論文寫 22%）；比 KnowTrace 的 1.57 低 67.5%（論文寫 68%）
- **比它便宜的三個方法都準很多**：Standard RAG、IRCoT、Search-o1 的正確率低 10–23 個百分點
- **顯著性**：對 PAR-RAG 的 EM、F1 提升 p < 0.001；SM 幾乎打平（p = .48）

<center>
<img src="https://arxiv.org/html/2606.22681v1/efficiency_frontier_cost_all.svg" alt="圖 5" width="75%">
<br>
<b>圖 5：正確率與成本的取捨（原論文 Figure 3a）</b>
</center>

> [!NOTE]
> **圖 5 怎麼看**
> 橫軸是正確率（越右越好），縱軸是每題成本（越低越好），箭頭「Better」指向**右下角**。
> - 左下角：Standard RAG（棕色叉）、IRCoT（綠色菱形），很便宜但不準
> - 上方：KnowTrace（紫色三角，0.61 ¢）、Godbole et al.（青綠十字，0.45 ¢），又貴又沒有特別準
> - 中間：Search-o1（紅色倒三角），便宜一些、正確率約 51%
> - 右側：PAR-RAG（橘色方塊）和 **GDP-RAG（藍色圓點）**。藍點在橘點的右下方：更準也更便宜。沒有任何方法同時比 GDP-RAG 更準又更便宜（帕雷托前緣：在兩個目標上都沒有被別人全面超越）

<center><b>表 7：各資料集的 EM／F1／SM（原論文 Table 6，%）</b></center>

| 方法 | HotpotQA EM | F1 | SM | 2Wiki EM | F1 | SM | MuSiQue EM | F1 | SM |
|---|---|---|---|---|---|---|---|---|---|
| IRCoT | 41.67 | 58.83 | 61.83 | 44.00 | 51.81 | 63.16 | 16.33 | 29.24 | 34.17 |
| KnowTrace | 41.50 | 59.47 | 66.00 | 55.16 | 69.93 | **79.83** | 23.33 | 34.65 | 35.00 |
| PAR-RAG | 47.67 | 68.03 | **72.67** | 60.16 | 69.48 | 77.66 | 30.17 | 46.93 | **51.00** |
| Godbole et al. | 47.00 | 64.58 | 66.67 | 33.00 | 40.32 | 61.16 | 25.17 | 39.44 | 39.17 |
| **GDP-RAG** | **52.33** | **70.40** | 71.50 | **66.67** | **73.53** | 77.67 | **34.67** | **48.43** | 50.50 |

GDP-RAG 的優勢主要在 **EM**（每個資料集都領先 PAR-RAG 4.5–6.5 點），SM 反而在 HotpotQA、MuSiQue 略輸。意思是兩者「答案有沒有出現在回答裡」差不多，但 GDP-RAG 的答案更精準、更少多餘的字。（我用表 7 重算三項平均，和表 6 的 Acc 一致。）

### 5.2 消融實驗：每個設計拿掉會怎樣

<center><b>表 8：消融實驗（原論文 Table 2；步數取自正文與 Figure 3b）</b></center>

| 變體 | 拿掉什麼 | Cost-of-pass | Acc（%） | 成本（¢） | Tokens | 平均步數 |
|---|---|---|---|---|---|---|
| **GDP-RAG** | — | 0.51 | 60.63 | 0.27 | 5,717 | **1.64** |
| GP | 缺口規則（仍有段落和 Thought） | 0.93 | **61.34** | 0.50 | 10,512 | 3.36 |
| NP | 預先檢索的段落 | 0.59 | 60.57 | 0.33 | 6,857 | 2.51 |
| RP | 段落換成 10 篇隨機文件 | 0.69 | 59.84 | 0.38 | 7,962 | 2.61 |
| NT | Thought | 0.60 | 56.80 | 0.29 | 6,491 | 約 2.0（從圖估讀） |
| NR | Review（不做第二次檢索驗證） | **0.41** | 59.41 | **0.21** | **4,512** | — |
| NU | Update（不改寫下一個子問題） | 0.51 | 59.77 | 0.26 | 5,511 | — |

<center>
<img src="https://arxiv.org/html/2606.22681v1/planning_ablation_score_steps_main.svg" alt="圖 6" width="70%">
<br>
<b>圖 6：規劃消融的步數與正確率（原論文 Figure 3b）</b>
</center>

> [!NOTE]
> **圖 6 怎麼看**
> 橫軸是正確率，縱軸是平均步數，箭頭「Better」指向右下（步數少、正確率高）。
> - **藍色圓點 GDP-RAG** 在最右下：約 1.64 步、60.63%
> - **紫色三角 GP**（拿掉缺口規則）在最右上：最準（61.34%）但步數翻倍（3.36）
> - **綠色方塊 NP**、**橘色菱形 RP** 在中間偏上：步數約 2.5–2.6，RP 比 NP 更左上，**隨機段落比沒有段落還糟**
> - **棕色十字 NT**（拿掉 Thought）在最左邊：步數約 2，但正確率掉到 56.80%

每個設計的角色：

- **預先檢索**：拿掉後（NP）正確率幾乎不變（60.57%），但成本多 22%、步數從 1.64 變 2.51。換成隨機段落（RP）兩邊都更差，**不相關的段落比沒有段落還糟**
- **缺口規則**：拿掉後（GP）正確率多 0.71 個百分點，但成本多 85%、步數 3.36。連沒有段落的 NP（2.51 步）都比 GP 少步，代表<b>「只問缺的」這條規則本身就是省步數的主因</b>
- **Thought**：拿掉後（NT）正確率掉最多（56.80%），成本差不多。**Thought 管的是正確率，不是步數**
- **Review**：拿掉後（NR）cost-of-pass 降 20%、token 降 21%，正確率只少 1.22 點。**Review 是執行階段最貴的部分，貢獻卻不大**，所以 NR 是另一個「省錢版」選擇
- **Update**：拿掉後（NU）只省約 4% 成本，正確率少 0.86 點。每步只多一次 LLM 呼叫，建議保留

顯著性（附錄 Table 7）也呼應這個分工：NT、NR、NU 對完整版的差距顯著（EM 的 p 分別為 <.001、.007、.03），GP、NP 不顯著。也就是**預先檢索和缺口規則不會讓答案更準，它們的功能是省步數**。

### 5.3 規劃真的變精簡了嗎

作者拿直接規劃和差量規劃產生的步驟，連同標準支持文件（gold documents）一起交給 LLM 判斷每一步對應到哪份文件（LLM 當評審），人工抽查 10% 的一致率是 82%。

<center>
<img src="https://arxiv.org/html/2606.22681v1/planning_stats.svg" alt="圖 7" width="90%">
<br>
<b>圖 7：三種規劃改善的統計（原論文 Figure 4a）</b>
</center>

> [!NOTE]
> **圖 7 怎麼看**
> 三組長條，**淺灰藍是直接規劃（Direct）**、**深藍是 GDP-RAG（Ours）**，柱上數字是全部題目加總的次數，越低越好：
> - **推理步驟壓縮**（左）：對到標準文件的有效步驟，3,561 → 1,388（少 61.0%）。少掉的步驟是因為那份文件預先檢索時已經拿到了
> - **過度規劃修剪**（中）：對不到任何標準文件的多餘步驟，2,823 → 650（少 77.0%）
> - **規劃不足修補**（右）：沒被涵蓋到的標準文件，1,839 → 913（少 50.4%）
> - 柱上有誤差線，但論文沒說明代表什麼

<center><b>表 9：預先檢索的 recall@10（原論文 Table 11）</b></center>

| 資料集 | 2 跳 | 3 跳 | 4 跳 | 平均 |
|---|---|---|---|---|
| HotpotQA | 0.86 | 0.77 | 0.71 | 0.78 |
| 2Wiki | 0.66 | — | 0.50 | 0.58 |
| MuSiQue | 0.70 | 0.44 | 0.24 | 0.46 |

recall@10 是「前 10 段裡找回了多少比例的標準文件」。跳數越多，原問題一次檢索能涵蓋的就越少，**能壓縮的步驟也就越少**。作者指出平均 recall 0.61 剛好和 61.0% 的壓縮率接近。

<center>
<img src="https://arxiv.org/html/2606.22681v1/complexity_steps.svg" alt="圖 8" width="90%">
<br>
<b>圖 8：不同跳數下的平均步數（原論文 Figure 4b）</b>
</center>

> [!NOTE]
> **圖 8 怎麼看**
> 三張小圖是三個資料集，橫軸是跳數，縱軸是平均步數。**淺灰藍圓點是直接規劃**、**深藍方塊是 GDP-RAG**（2Wiki 沒有 3 跳，所以只有兩個點）。
> - HotpotQA：直接規劃約 2.9 → 3.3 步，GDP-RAG 幾乎平的，約 1.1 → 1.2 步
> - 2Wiki：直接規劃約 2.6 → 4.8 步，GDP-RAG 約 1.1 → 2.4 步
> - MuSiQue：直接規劃約 2.9 → 3.7 → 4.4 步，GDP-RAG 約 1.2 → 1.8 → 2.7 步
> - 每個點 GDP-RAG 都比直接規劃少，跳數越多差距越大（以上數字皆為從圖估讀的約略值）

### 5.4 換更強的規劃模型，問題會自己消失嗎

<center><b>表 10：換成 GPT-5.5 當規劃器（原論文 Table 13，步數加總）</b></center>

| 步驟類型 | 直接規劃 GPT-4.1-mini | 直接規劃 GPT-5.5 | 差量規劃 GPT-4.1-mini | 差量規劃 GPT-5.5 |
|---|---|---|---|---|
| 有效步驟 | 3,561 | 4,525 | 1,388 | 1,673 |
| 多餘步驟 | 2,823 | 2,609 | 650 | 506 |

更強的模型寫出的子問題對得上更多文件，但**多餘步驟幾乎沒減少**（2,823 → 2,609）。作者的結論：過度規劃是「沒看文件就規劃」造成的結構問題，換大模型解決不了。（這組只測規劃，沒有跑完整問答。）

### 5.5 光把文件塞給舊方法有用嗎

一個合理的懷疑：GDP-RAG 變好，會不會只是因為多看了預先檢索的文件？作者把同樣的 $D_{rel}$ 直接塞給 PAR-RAG，規劃方式不變：

<center><b>表 11：把預先檢索的文件塞給 PAR-RAG（原論文 Table 10）</b></center>

| 方法 | Cost-of-pass | Acc（%） | 成本（¢） |
|---|---|---|---|
| PAR-RAG | 0.65 | 58.20 | 0.34 |
| PAR-RAG + $D_{rel}$（只給回答模組） | 0.73 | 57.32 | 0.37 |
| PAR-RAG + $D_{rel}$（每一步都給） | 0.92 | 55.22 | 0.45 |
| **GDP-RAG** | **0.51** | **60.63** | **0.27** |

塞得越多，**越貴也越不準**。差別在文件怎麼用：GDP-RAG 用它來決定「哪些不用問」，並把重點濃縮進 Thought；直接塞只是增加雜訊。

### 5.6 換小模型

<center><b>表 12：GPT-4.1-mini 與 GPT-4.1-nano 的比較（原論文 Table 8）</b></center>

| 方法 | mini Acc（%） | mini 成本（¢） | nano Acc（%） | nano 成本（¢） |
|---|---|---|---|---|
| Standard RAG | 37.86 | 0.04 | 24.11 | 0.01 |
| IRCoT | 44.56 | 0.06 | 35.78 | 0.01 |
| Godbole et al. | 46.28 | 0.45 | 32.84 | 0.11 |
| KnowTrace | 51.65 | 0.61 | 32.50 | 0.20 |
| PAR-RAG | 58.20 | 0.34 | **41.38** | 0.09 |
| **GDP-RAG** | **60.63** | 0.27 | 40.00 | 0.07 |
| GDP-RAG NR | 59.41 | 0.21 | 37.79 | 0.06 |

換成 nano 後，**PAR-RAG 反而比 GDP-RAG 準**（41.38 vs. 40.00）。差最多的是 2Wiki 4 跳（19.72 vs. 32.60，差 12.88 點）和 MuSiQue 4 跳（差 6.23 點），剛好是 mini 時 GDP-RAG 贏最多的兩組。作者的解釋：GDP-RAG 把計畫壓到只剩缺的事實，**最後把它們組合起來的工作留給回答模型**；強模型做得到，小模型做不到。PAR-RAG 的步驟多，等於有「備用步驟」幫小模型擋錯。

### 5.7 實際例子

<center><b>表 13：三種改善的實例（原論文 Table 14–16）</b></center>

| 類型 | 問題 | 直接規劃 | GDP-RAG |
|---|---|---|---|
| 推理步驟壓縮 | Joseph D. Stewart 擔任校長的學院在哪裡？ | S1 是哪所學院 → S2 在哪裡（2 步） | 預先檢索已拿到兩份標準文件，只問 1 步：「美國商船學院在哪裡？」 |
| 過度規劃修剪 | Vanderbilt 和 Emory 哪間先成立？ | S1 Vanderbilt 哪年 → S2 Emory 哪年 → S3 哪間先（S3 對不到任何文件） | 兩個年份已在文件中（1873、1836），只剩 1 步比較 |
| 規劃不足修補 | 邊上有 Oakenclough 村的森林，另一個名字是什麼？ | 3 步，只涵蓋 2 份標準文件，漏掉「英格蘭的瑞士」那份 | 用原問題預先檢索就拿到全部 3 份，只要 1 步 |

---

## 6. 總結

1. 計畫式 RAG 的規劃器**沒看過文件就拆題**，所以會多拆（浪費）或少拆（漏查）
2. GDP-RAG 先用原問題檢索一次，叫規劃器**只為文件裡沒有的資訊出子問題**，並把已知的證據寫在 Thought 裡帶到最後
3. 執行階段沿用 PAR-RAG 的 Act-Review-Update，但每個留下來的步驟都完整執行、驗證
4. 三個資料集平均正確率 60.63%（最高），cost-of-pass 0.51，比 PAR-RAG 便宜又更準；平均只要 1.64 步
5. 分工清楚：**缺口規則省步數、Thought 保正確率、Review 貴但貢獻小**

> [!WARNING]
> **限制與要小心的地方**
> - **只測短答案的事實型多步驟問答**，作者自己也說開放式生成、長文、多模態要重新定義「缺口」
> - **只用 GPT-4.1 家族**（mini、nano；GPT-5.5 只測規劃）。換成 nano 時 PAR-RAG 反而比較準，方法依賴回答模型夠強
> - **篩題用的是同一個模型**：先把 GPT-4.1-mini 不檢索就答對的題目丟掉。附錄 Table 3 用未篩選的資料重跑，排名不變，但 Table 3、6、8 都沒有 Search-o1
> - **最準的其實是消融版 GP**（61.34%），GDP-RAG 是「外部方法裡最準」。GP 的成本是 GDP-RAG 的 1.85 倍，作者選擇省錢版本作為主方法
> - **規劃不足的比較不對等**（附錄 A.10）：直接規劃只看它的子問題涵蓋多少文件，GDP-RAG 則是「預先檢索 + 子問題」的聯集，GDP-RAG 本來就多了一次檢索可以算
> - **「推理步驟壓縮」的解讀**：圖 7 左邊少掉的是「有效步驟」，作者的論點是這些文件預先檢索已經拿到；但「61.0% 壓縮率剛好等於 0.61 平均 recall」比較像巧合式的對照，兩者的計算單位（步驟 vs. 文件）不同

> [!WARNING]
> **我核對數字時發現的不一致**
> - **百分比寫法**：正文說 GP「accuracy (+0.71%)」，實際是 61.34 − 60.63 = 0.71 **個百分點**
> - **「2Wiki 步數幾乎不變」和圖不符**：第 7.2 節說 HotpotQA 和 2Wiki 上差量規劃的步數「nearly constant」，但圖 8 的 2Wiki 從約 1.1 步升到約 2.4 步（從圖估讀），只有 HotpotQA 是平的
> - **顯著性表的平均差距算不回來**：Table 7 是 1,800 題逐題比較，每個資料集 600 題等量，所以平均差距應該等於 Table 6 三個資料集差距的平均。實際算出來：對 PAR-RAG 的 EM 差 5.22（表寫 +5.72）、F1 差 2.64（表寫 +3.11）、SM 差 −0.55（表寫 +0.06）；對 IRCoT 的 SM 差 13.50（表寫 +19.44）。EM 幾乎每一列都差約 0.5 點，原因論文沒說明
> - **nano 的「六勝兩負」**：Table 9 的 8 組裡，GDP-RAG 有 3 組比 PAR-RAG 低（−0.47、−12.88、−6.23），作者把 −0.47 算成「打平」
> - **Cost-of-pass 的平均方式**：「all」是三個資料集各自 C／Acc 後再平均（我用 Table 1 驗算 0.513，相符），所以 MuSiQue（0.98）的權重很大。若改用「平均成本／平均正確率」，GDP-RAG 是 0.45、PAR-RAG 是 0.59，排名不變
> - 小地方：Standard RAG 正確率 Table 1 寫 37.86、Table 3 寫 37.85；正文的 reranker 名稱「reranker-large-v2-m3」和參考文獻的「bge-reranker-v2-m3」不一致

> [!TIP]
> **可以帶走的思路**
> - **規劃前先檢索一次**：把原問題的檢索結果交給規劃器，叫它「只為文件裡沒有的資訊出子問題」。只是提示詞改動，現有的多步驟 RAG 幾乎都能直接加
> - **把已知證據寫進 Thought、帶到最後**：比把整包文件塞給每個模組有效，也比較省 token（表 11）
> - **用 cost-of-pass 比較系統**：「每個正確答案花多少錢」比只看正確率更貼近上線需求
> - **驗證步驟（Review）要算帳**：它佔大部分成本、只帶來約 1 點正確率。預算緊時可以拿掉，或只在信心低時才做
> - **檢索品質決定能省多少**：預先檢索的 recall 越高，能跳過的步驟越多；隨機或不相關的段落反而有害，所以前面的 reranker 很重要

---

## 7. 我的筆記

### 重點筆記
>

### 可以借用的地方
> 國家災防中心的 RAG 問答：現在如果有拆題流程，能不能先用原問題檢索一次，再叫規劃器只問缺的？用 cost-of-pass 和平均步數比較改動前後。另外可以試 NR 版（不做 Review）當省錢方案。

### 疑問
> 中文資料、文件很長的時候，預先檢索的 recall 夠高嗎？用開源小模型（地端部署）時，會不會像 nano 一樣反而輸給步驟多的 PAR-RAG？Table 7 和 Table 6 的差距為什麼對不起來？

### 相關
- 概念：[[Retrieval-augmented generation|檢索增強生成]]、[[Multi-hop question answering|多步驟問答]]、[[Query decomposition|問題拆解]]、[[Cost-of-pass]]、[[ReAct]]、[[Chain-of-thought prompting|思維鏈提示]]、[[LLM-as-a-judge|LLM 當評審]]
- 前作（本文的比較對象）：[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]（IRCoT，逐步式代表，本文主表中便宜但正確率 44.56%）
- 相關論文：[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]（計畫式 RAG，本文指出它從模型記憶規劃、沒有先看文件）
- 直接前作：PAR-RAG（Zhang et al., 2025），本文沿用它的 Act-Review-Update
- 相關論文：[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]（SAG：改的是索引與檢索——每段抽事件與實體，用 SQL JOIN 共同實體串段落；和 GDP-RAG 用同樣三個資料集，兩者可疊加）
- 其他比較方法：KnowTrace（Li et al., 2025）、Search-o1（Li et al., 2025）、Godbole et al.（EMNLP 2024）
- 資料集：HotpotQA、2WikiMultiHopQA、MuSiQue
- 相關論文：[[【A Survey on LLM-as-a-Judge】]]（本篇用 LLM 判斷答案對錯；綜述整理了評審的偏誤與檢查方法，可以用來檢視這類評估設計）
