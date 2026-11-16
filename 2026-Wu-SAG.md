---
title: "【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】"
tags: [paper, llm, rag, multi-hop-qa, graph-rag, status-inbox]
concepts: ["[[Retrieval-augmented generation]]", "[[Multi-hop question answering]]", "[[GraphRAG]]", "[[Hypergraph]]", "[[Personalized PageRank]]", "[[Recall@k]]", "[[BM25]]", "[[Graph database]]", "[[LLM-as-a-judge]]"]
papers: ["[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]", "[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]", "[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]", "[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]", "[[【A Survey on LLM-as-a-Judge】]]"]
---

# 【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 作者 | Yuchao Wu、Junqin Li、XingCheng Liang、Yongjie Chen、Yinghao Liang、Linyuan Mo、Guanxian Li（Zleap AI，公司團隊） |
| 發表 | arXiv 2606.15971（v1 2026/6/14，v2 2026/8/25，本筆記依 v2）；cs.CL；尚未標註會議 |
| 連結 | [arXiv](https://arxiv.org/abs/2606.15971)・[PDF](https://arxiv.org/pdf/2606.15971)・[HTML](https://arxiv.org/html/2606.15971v2)・[評測程式（SAG-Benchmark）](https://github.com/Zleap-AI/SAG-Benchmark)・[工程版（SAG）](https://github.com/Zleap-AI/SAG) |
| 引用（IEEE） | Y. Wu et al., "SAG: SQL-retrieval augmented generation with query-time dynamic hyperedges," 2026, *arXiv:2606.15971*. |
| 動畫解說 | [papers.poyuanchu.com/2026-wu-sag/](https://papers.poyuanchu.com/2026-wu-sag/) |
| 論文類型 | 方法論文：不建全域知識圖譜的結構化檢索架構 |
| 狀態 | 待讀 |
| 與我的相關度 | 5 / 5（多步驟檢索；只用 MySQL + Elasticsearch 就能做，資料一直增加也不用重建圖，很適合企業文件或災防資料） |
| 加入日期 | 2026-11-12 |

> [!NOTE]
> **一句話**：GraphRAG 這類方法要先把整個語料庫建成一張大知識圖，又貴又難更新。SAG 改成**每個段落只抽一句「事件」和它提到的「實體」，存進 SQL 表**。查詢時用 SQL JOIN 找「和種子段落共用同一個實體」的其他段落，**臨時串出這題需要的關聯**，最後還是把原始段落交給 LLM。在 MuSiQue 上 Recall@5 達到 80.36%。

---

## 1. 背景：向量檢索不會串，知識圖譜太重

### 向量檢索（一般 RAG）

一般 RAG 把每個段落轉成向量，拿問題的向量去找最相似的前 k 段。每一段**各自打分、互不相干**，所以：

- 答案在單一段落時很好用
- 多步驟問題（multi-hop QA，要串兩份以上文件才能回答）就不行：第二份文件常常和問題沒什麼字面相似，漏掉一份，整條證據鏈就斷了

### 圖譜式 RAG

為了補上「段落之間的關聯」，GraphRAG、HippoRAG、LightRAG 等方法**在建索引時就用 LLM 把整個語料庫轉成知識圖譜**（節點是實體、邊是關係），查詢時沿著圖走。問題是：

- **語意被拆碎**：一句「A 公司在 D 日以 X 元收購 B 公司」要拆成好幾個三元組（主詞–關係–受詞），「這些是同一件收購」的邊界就不見了
- **建圖很貴**：要對全部語料呼叫 LLM，還要維護一張全域的圖
- **難以增量更新**：新文件進來可能要重算或合併節點

超圖方法（HyperGraphRAG、HyperRAG）用**超邊**（hyperedge，一條邊可以同時連好幾個節點）保留多元關係，但仍然要事先建好一整張語料庫級的超圖。

<center>
<img src="https://arxiv.org/html/2606.15971v2/Figure1.svg" alt="圖 1" width="90%">
<br>
<b>圖 1：三種 RAG 架構的比較（原論文 Figure 1）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**
> 三列由上到下是三種架構，每列左邊是流程，右邊是作者給的四項評分（綠＝好、橘＝中、紅＝差，三格代表程度）。
> - **NaiveRAG（綠色）**：問題 → 向量化 → 語意向量搜尋 → 取前 k 段。可擴充性、成本、速度都好，但精準度低；限制是「只靠相似度、檢索很淺」
> - **GraphRAG（紫色）**：問題 → LLM 抽三元組 → 存進知識圖（Neo4j 圖示）→ LLM → 段落。精準度高，但可擴充性、成本、速度都差；限制是「慢、貴、難增量更新」
> - **SAG（藍色）**：問題 → LLM 找實體與事件 → **SQL**（資料庫圖示）→ 超邊（方塊是事件、圓圈是實體）→ LLM → 段落。下方標「Dynamic associations」（動態關聯）。可擴充性好、精準度高，成本和速度中等
> - 這四項評分是作者的定性主張，不是實驗數字（實際速度見第 5.4 節，SAG 其實比 HippoRAG 2、GraphRAG 慢）

---

## 2. 核心想法：事件當超邊，實體當 JOIN 的鍵

SAG 的觀察是：**多步驟檢索需要的關聯，本來就藏在「段落講了什麼事」和「不同段落共用了哪些實體」裡**，不一定要事先建成一張圖。

### 三層結構

- **段落**（chunk）：原始文字，最後交給 LLM 讀的證據
- **事件**（event）：每個段落用 LLM 濃縮成**一句完整的事實陳述**，一對一對應
- **實體**（entity）：事件裡提到的人、地、組織、時間等，一個事件對應多個實體

每個事件連著它的所有實體，就等於一條**超邊**：一條「邊」同時把好幾個實體綁在一起，保留了整件事的多元關係，不用拆成三元組。作者稱為**潛在超邊**（latent hyperedge），因為它沒有被真的建成一張圖，只是 SQL 表裡的幾列：

<center><b>表 2：事件–實體表的樣子（示意）</b></center>

| event_id | entity |
|---|---|
| e1（Titian 的畫作《The Bacchanal of the Andrians》…） | The Bacchanal of the Andrians |
| e1 | Titian |
| e2（《The Martyrdom of Saint Lawrence》是 Titian 在威尼斯的作品…） | The Martyrdom of Saint Lawrence |
| e2 | Venice |
| e3（黑死病在威尼斯爆發過…） | Black Death |
| e3 | Venice |

用 event_id 分組就能還原每條超邊的完整實體集（論文 Proposition 1：無損儲存）；用 entity 分組就知道哪些事件共用了同一個實體，例如 e2 和 e3 都有 Venice。

### 查詢時才串關聯

查詢時，從種子事件出發：**事件 → 它的實體 → 共用這些實體的其他事件**，兩次 SQL JOIN 就是一輪擴展。這張「鄰域」只為這個問題存在，查完就丟，所以叫**查詢時動態超邊**（query-time dynamic hyperedges）。

<center><b>表 3：SAG 和其他做法的差別</b></center>

| | 向量 RAG | GraphRAG／HippoRAG | 超圖 RAG | SAG |
|---|---|---|---|---|
| 索引單位 | 段落向量 | 實體與三元組 | 實體與超邊 | 事件 + 實體（SQL 列） |
| 段落之間的關聯 | 沒有 | 建索引時固定 | 建索引時固定 | **查詢時用 JOIN 臨時串** |
| 新文件進來 | 加向量 | 可能要合併、重算 | 可能要合併、重算 | **只要新增幾列** |
| 交給 LLM 的證據 | 原段落 | 多半是摘要、實體、關係 | 實體、關係 CSV | **原段落** |

> [!TIP]
> 「用共同實體當 JOIN 的鍵」是很樸素的想法，但很實用：企業本來就有 SQL 資料庫和搜尋引擎（Elasticsearch），不必再導入圖資料庫。

---

## 3. 技術細節

<center>
<img src="https://arxiv.org/html/2606.15971v2/Figure2.png" alt="圖 2" width="95%">
<br>
<b>圖 2：SAG 整體架構（原論文 Figure 2）</b>
</center>

> [!NOTE]
> **圖 2 怎麼看**
> - **上半（1）Upload，離線建索引**：語料庫 → Extract（LLM 抽取）→ 每個 Chunk 用虛線一對一對到 Event（One-to-One），每個 Event 再用實線連到多個 Entity（橘色圓點，Many-to-Many Hyperedge）→ 存進 SQL 和向量索引
> - **中段（2）Search，線上檢索**，兩條路徑：
>   - 上路：Query → LLM 找出實體（三個藍色圓點）→ Entity recall → SQL 找出含這些實體的事件
>   - 下路：Query → 直接用向量找事件，相似度 > 0.4 的都留下
>   - 兩路的事件合成 **Event set**，右邊的循環是擴展：Event set →（SQL）→ Entity →（SQL）→ Event →（Join）回到 Event set
> - **下半，輸出**：
>   - 上方：Event set → 相似度篩選 → 取前 100 個事件 → **LLM 一次讀完，挑最多 5 個**，換回原段落（Chunk 1–5）
>   - 下方：Query → 一般向量檢索段落，**補滿到 10 段**（Fill to 10）
>   - 兩邊 Concat（接起來）→ 最終 10 段

### 3.1 離線：建事件–實體索引

- 每個段落**呼叫一次 LLM**（Qwen3.6-Flash），輸出一個事件（標題 + 內容）和一串帶類型的實體
- 實體有 **11 種類型**：時間、地點、人物、組織、群體、主題、作品、產品、動作、數值、標籤
- 提示詞強調：**不論段落講幾件事，都整合成一個事件**；忠於原文，不推論；同一實體的全名、縮寫、別名都要抽
- 寫進 MySQL（事件–實體多對多表）和 Elasticsearch（向量 + 全文索引）
- **刻意不做實體消歧**（判斷兩個名字是不是同一個東西），只做字串正規化和去重，這樣每個段落可以獨立處理、只追加不重算

### 3.2 種子檢索：兩條路徑

<center><b>表 4：兩條種子路徑</b></center>

| 路徑 | 做法 | 門檻 | 補哪種失敗 |
|---|---|---|---|
| A：實體導向 | LLM 找出問題裡的實體 → 在實體向量索引找相似實體（相似度 ≥ 0.9）→ SQL 取出含這些實體的所有事件 | 高（0.9），因為實體名稱很短，放寬容易把不同東西混在一起 | 問題和段落字面不像，但有共同的名字 |
| B：事件向量 | 問題向量直接比對事件向量 | 低（0.4），先求多找，精準度交給後面排序 | 問題沒有明確實體，或實體名稱寫法不同 |

$$R = R_{ent} \cup R_{evt}$$

### 3.3 擴展、粗排、精選

1. **擴展**：從種子事件 $R$ 用反向 JOIN 取出它們的實體，沒走過的實體成為「前沿」，**先修剪到最多 50 個**，再 JOIN 回其他沒看過的事件。這是一輪，預設只做 **1 輪**（$L = 1$）
2. **候選池**：$C = R \cup X$（種子 + 擴展得到的事件）
3. **粗排**：用向量相似度從 $C$ 取前 $K_{cand} = 100$ 個事件
4. **結構路徑**：LLM 一次讀這 100 個事件，**挑出最多 5 個「合起來能推出答案」的事件**，換回原段落。作者強調這是「整體挑選」，不是逐一打分的重新排序：模型能看到事件之間的關係
5. **語意路徑**：一般向量檢索段落，去掉重複的，**補滿剩下的名額**，總共輸出 10 段

$$D_{out} = D_{evt} \cup \text{TopK}_{10-|D_{evt}|}(R_{chunk} \setminus D_{evt})$$

> [!NOTE]
> 擴展一輪在二部圖（實體與事件兩種節點）上等於走兩步：事件 → 實體 → 事件。所以 $L$ 輪最多觸及距離種子 $2L$ 步以內的事件（Proposition 2）。「動態」指的是這個鄰域依每題的種子臨時組出來，不是在查詢時新建超邊。

### 3.4 限定範圍（大型語料庫用）

主實驗是在整個語料庫上做。語料庫很大時，可以**先用向量檢索選出前 $K_{pool}$ 個段落**，之後所有事件、實體、JOIN 都只在這個範圍內做。昂貴的 LLM 只看 $K_{cand}$ 個事件，所以**查詢成本不會隨語料庫變大而等比例增加**。

---

## 4. 實驗設定

<center><b>表 5：實驗設定</b></center>

| 項目 | 設定 |
|---|---|
| 資料集 | MuSiQue（最多 4 跳，刻意去掉單跳捷徑，最難）、2WikiMultiHopQA、HotpotQA（2 跳）；各抽 1,000 題 |
| 語料庫 | 照 HippoRAG 2 的設定，把支持段落和干擾段落混在一起：MuSiQue 11,656 段、2Wiki 6,119 段、HotpotQA 9,811 段 |
| 其他 | 語料增長實驗另用 NQ（1,000 題、9,633 段）；附錄另測 NarrativeQA（293 題） |
| 比較方法 | 簡單檢索：BM25、Contriever；大型向量模型：BGE-Large-EN-v1.5、GTE-Qwen2-7B、GritLM-7B、NV-Embed-v2；結構式：GraphRAG、LightRAG、HippoRAG 2、HyperGraphRAG、HyperRAG |
| 統一設定 | 所有結構式方法都用 **BGE-Large-EN-v1.5** 當檢索器、**Qwen3.6-Flash** 當閱讀器（以及抽取、重新排序用的 LLM） |
| SAG 參數 | $L = 1$、種子上限 50、前沿上限 50、$K_{cand} = 100$、輸出 10 段（結構路徑最多 5 段） |
| 指標 | 檢索：**Recall@5**（前 5 段找回多少比例的支持段落）；問答：EM、F1；另有 4 項 LLM 評審指標（Qwen3.7-Plus 評分） |

> [!NOTE]
> 除了 HippoRAG 2，其他結構式方法直接回傳答案或摘要，不是排序過的段落清單，所以**沒有 Recall@5**，只能比問答 F1。

---

## 5. 結果

### 5.1 主要結果

<center><b>表 6：主要結果（原論文 Table 1；Recall@5 與 F1，%）</b></center>

| 方法 | MuSiQue R@5 | F1 | 2Wiki R@5 | F1 | HotpotQA R@5 | F1 | 平均 R@5 | 平均 F1 |
|---|---|---|---|---|---|---|---|---|
| BM25 | 42.11 | 35.19 | 63.78 | 56.26 | 73.05 | 64.64 | 59.65 | 52.03 |
| Contriever | 45.32 | 34.39 | 59.25 | 50.62 | 74.85 | 64.06 | 59.81 | 49.69 |
| BGE-Large-EN-v1.5 | 56.32 | 42.80 | 69.83 | 61.49 | 89.20 | 74.81 | 71.78 | 59.70 |
| GTE-Qwen2-7B | 61.92 | 44.60 | 74.18 | 64.91 | 88.45 | 73.04 | 74.85 | 60.85 |
| GritLM-7B | 66.22 | 49.30 | 75.62 | 66.12 | 93.40 | 73.22 | 78.41 | 62.88 |
| NV-Embed-v2 | 68.84 | 52.88 | 77.00 | 68.45 | 95.25 | 78.00 | 80.36 | 66.44 |
| GraphRAG | – | 54.14 | – | 72.08 | – | 76.29 | – | 67.50 |
| LightRAG | – | 46.66 | – | 65.75 | – | 73.09 | – | 61.83 |
| HyperGraphRAG | – | 49.20 | – | 63.44 | – | 74.70 | – | 62.45 |
| HyperRAG | – | 51.80 | – | 76.89 | – | 77.19 | – | 68.63 |
| HippoRAG 2 | 65.13 | 47.49 | 90.35 | 74.34 | 94.35 | 77.24 | 83.28 | 66.36 |
| **SAG** | **80.36** | **61.15** | **93.34** | **78.06** | **96.50** | **79.66** | **90.07** | **72.96** |

（我用各資料集數字重算平均，和表上的平均欄、附錄 Table 5 的 EM 平均都一致。）

重點：

- **檢索**：三個資料集 Recall@5 都第一。差距**跳數越多越大**：MuSiQue 比 HippoRAG 2 高 15.23 點、比最強的 NV-Embed-v2 高 11.52 點；HotpotQA 只比 HippoRAG 2 高 2.15 點，但 Recall@2 高 13.20 點（91.55 vs. 78.35）
- **作者的解釋**：HippoRAG 2 用個人化 PageRank 在圖上傳遞分數，每走一步分數就打折（阻尼），四跳之外的證據分數被壓得很低；SQL JOIN 沒有這種衰減，幾步外的事件一樣會進候選池
- **問答**：F1 三個資料集都第一。MuSiQue 61.15，比 GraphRAG 高 7.01 點；2Wiki 比 HyperRAG 高 1.17 點；HotpotQA 比 NV-Embed-v2 高 1.66 點
- **2Wiki 的插曲**：初版 SAG 等到最後才修剪，高相似但無關的擴展事件擠掉了真正的證據，Recall@5 只有 88.00%，輸給 HippoRAG 2 的 90.35%。改成擴展前、擴展中就修剪後升到 93.34%。作者說是靠可追蹤的中間狀態找到問題的

<center><b>表 7：完整檢索結果（原論文 Table 4，%）</b></center>

| 方法 | MuSiQue R@1 | R@2 | R@5 | R@10 | 2Wiki R@2 | R@5 | HotpotQA R@2 | R@5 |
|---|---|---|---|---|---|---|---|---|
| NV-Embed-v2 | 34.02 | 51.08 | 68.84 | 77.27 | 68.93 | 77.00 | 85.55 | 95.25 |
| HippoRAG 2 | 30.65 | 49.52 | 65.13 | 73.76 | 76.55 | 90.35 | 78.35 | 94.35 |
| **SAG** | **36.82** | **63.62** | **80.36** | **83.37** | **83.93** | **93.34** | **91.55** | **96.50** |

### 5.2 LLM 評審

<center><b>表 8：LLM 評審結果（原論文 Table 9，Qwen3.7-Plus 評分，MuSiQue／2Wiki／HotpotQA）</b></center>

| 方法 | 答案正確度 | 涵蓋度 | 上下文相關度 | 證據召回 |
|---|---|---|---|---|
| GraphRAG | 56.56／79.32／84.91 | 46.60／77.97／62.93 | 59.17／73.02／89.86 | 52.31／71.74／77.88 |
| LightRAG | 53.07／72.52／80.98 | 43.54／71.15／60.88 | 47.55／63.65／83.57 | 63.91／75.89／87.49 |
| HyperGraphRAG | 58.03／80.31／85.00 | 48.52／79.73／63.78 | **65.51**／80.27／89.55 | 73.24／83.35／91.63 |
| HyperRAG | 56.01／84.93／85.32 | 48.06／84.23／63.16 | 58.74／**80.30**／**91.13** | 68.23／87.90／92.37 |
| HippoRAG 2 | 50.09／83.93／84.91 | 43.20／82.42／63.31 | 45.75／74.58／86.65 | 64.77／88.45／93.65 |
| **SAG** | **63.69／85.26／87.95** | **54.73／85.07／64.90** | 59.54／76.02／89.50 | **78.87／89.44／95.87** |

SAG 在答案正確度、涵蓋度、證據召回三項都第一。上下文相關度輸給 HyperGraphRAG、HyperRAG（差 1.63–5.97 點），作者的解釋是：SAG 和 HippoRAG 2 只給評審前 5 段原文，其他方法給的是摘要、實體、關係的組合，資訊比較廣（附錄 Table 12）。

### 5.3 消融：哪個環節最重要（MuSiQue）

<center><b>表 9：消融與參數掃描（原論文 Table 6、7，MuSiQue Recall@5）</b></center>

| 變體 | R@2 | R@5 | R@10 | 與預設差（R@5） |
|---|---|---|---|---|
| **SAG（預設）** | 63.62 | 80.36 | 83.37 | — |
| 改用三元組索引 | 61.83 | 77.61 | 81.54 | −2.75 |
| 不擴展（$L=0$） | 57.75 | 69.41 | 74.76 | −10.95 |
| 精選改用 Qwen3-Reranker-8B（逐一打分） | 48.97 | 67.11 | 76.51 | **−13.25** |
| 只用語意路徑（$K_{event}=0$） | 41.56 | 56.23 | 63.77 | −24.13 |
| 擴展 $L=2$／3／4 | — | 80.47／80.40／80.71 | — | 差不到 0.35 |
| $K_{cand}=50$ | 62.80 | 79.35 | 81.69 | −1.01 |
| $K_{cand}=200$ | 65.12 | 81.06 | 84.47 | +0.70 |
| $K_{cand}=500$ | 64.72 | **82.24** | **86.56** | +1.88 |
| $K_{event}=2$／4／6／8／10 | — | 73.52／79.80／79.93／80.14／79.80 | — | 5 以上不再進步 |

各環節的角色：

- **LLM 整體精選影響最大**：換成逐一打分的重新排序模型，Recall@5 少 13.25 點。作者的說法是 LLM 一次看全部候選，能看出「哪幾個事件合起來才是完整的證據鏈」
- **擴展第二重要**：關掉少 10.95 點；但**一輪就夠**，多做幾輪差不到 0.35 點
- **超邊 vs. 三元組影響最小**：換成三元組只少 2.75 點，而且三元組版仍比 HippoRAG 2 高 12.48 點。作者自己也承認 SAG 的優勢**不能只歸功於超邊表示法**
- **兩條路徑互補**：只用語意路徑只有 56.23%（等於 BGE 單獨檢索的程度），結構路徑最多挑 5 個是最佳點

> [!WARNING]
> 換成重新排序模型的版本（67.11%）雖然仍高於 HippoRAG 2（65.13%），但**低於什麼結構都不用的 NV-Embed-v2 單純向量檢索（68.84%）**；不擴展的版本（69.41%）也只和 NV-Embed-v2 差不多。論文只拿 HippoRAG 2 比較。換句話說，SAG 的大幅領先**很大一部分來自「LLM 一次讀 100 個候選再挑」這一步**，而 BGE、NV-Embed 等向量基準完全沒有 LLM 重新排序。

### 5.4 成本與速度（MuSiQue）

<center><b>表 10：建索引與查詢成本（原論文 Table 8）</b></center>

| 方法 | F1 | 建索引時間（小時） | 建索引 token（百萬） | 檢索（秒／題） | 問答（秒／題） | 總時間（秒／題） | Token／題 |
|---|---|---|---|---|---|---|---|
| RAG（BGE） | 42.80 | 0 | 0 | 0.10 | 1.75 | **1.85** | **1,727** |
| GraphRAG | 54.14 | 10.15 | 125.12 | 0.23 | 12.56 | 12.79 | 11,438 |
| LightRAG | 46.66 | 25.94 | 103.14 | 11.97 | 18.13 | 30.10 | 9,974 |
| HyperGraphRAG | 49.20 | 16.21 | 216.11 | 4.45 | 23.54 | 27.99 | 32,536 |
| HippoRAG 2 | 47.49 | **2.30** | **15.93** | 0.66 | 8.50 | 9.16 | 4,788 |
| SAG（$K_{cand}=50$） | 60.41 | 4.57 | 28.26 | 15.93 | 6.75 | 22.68 | 8,236 |
| SAG（$K_{cand}=100$） | **61.15** | 4.57 | 28.26 | 18.67 | 6.36 | 25.03 | 11,536 |

- **建索引**：比 GraphRAG、LightRAG、HyperGraphRAG 省 55.0–82.4% 時間、72.6–86.9% token；但 **HippoRAG 2 更便宜**（2.30 小時、15.93M token）
- **查詢**：每題 **22.68–25.03 秒**，比 HippoRAG 2（9.16 秒）、GraphRAG（12.79 秒）**慢**；瓶頸是檢索加 LLM 精選（15.93–18.67 秒）。作者承認「在目前的規模下，SAG 沒有明顯的成本或延遲優勢」
- $K_{cand}$ 從 50 加到 100：F1 只多 0.74，token 多 40.1%、延遲多 10.4%。**50 是比較划算的設定**

### 5.5 穩健性

<center>
<img src="https://arxiv.org/html/2606.15971v2/embedding_robustness.svg" alt="圖 3" width="60%">
<br>
<b>圖 3：換不同向量模型時的 Recall@5（原論文 Figure 3a，MuSiQue）</b>
</center>

> [!NOTE]
> **圖 3 怎麼看**
> 三組柱子是三種方法，每組四根代表四個向量模型：**深藍 BGE-v1.5、橘 GTE-Qwen2、綠 GritLM-7B、淺紫 NV-Embed-v2**。
> - 一般向量檢索（Naive）：56.32 → 68.84，向量模型越強越好，差 12.52 點
> - HippoRAG 2：65.13 → 74.55，差 9.42 點
> - **SAG：80.36 → 81.71，只差 1.35 點**，而且用最弱的 BGE 也贏過用最強 NV-Embed 的 HippoRAG 2
> - 作者的解釋：SAG 只在種子和粗排用向量，擴展靠精確的 SQL JOIN；HippoRAG 2 的 PageRank 起點就取決於向量相似度

<center>
<img src="https://arxiv.org/html/2606.15971v2/candidate_pool_scaling.svg" alt="圖 4" width="60%">
<br>
<b>圖 4：限定候選範圍大小與 Recall@5（原論文 Figure 3b，MuSiQue）</b>
</center>

> [!NOTE]
> **圖 4 怎麼看**
> 橫軸是先用向量選出的段落數 $K_{pool}$（100、200、500、1000、2000、全部 11,656），縱軸是 Recall@5，注意縱軸從 74 開始。
> - 100 段：75.74；200 段：76.87；**500 段：79.57**；1,000：79.87；2,000：80.24；全部：80.36
> - 只用 500 段（**不到語料庫的 5%**），就保住全量結果的 99%。這是作者主張「語料庫變大，查詢成本不會跟著變大」的依據

<center>
<img src="https://arxiv.org/html/2606.15971v2/incremental.svg" alt="圖 5" width="60%">
<br>
<b>圖 5：語料庫持續增加時的 Recall@5（原論文 Figure 3c）</b>
</center>

> [!NOTE]
> **圖 5 怎麼看**
> 橫軸是語料庫大小（25% → 100%，評估題目固定）。**綠色是 NQ（單跳），藍色是 MuSiQue（多跳）**；實線圓點是 SAG、虛線方塊是 HippoRAG 2、點線三角是 BGE。
> - NQ：三條線都很接近，SAG 從 76.80 降到 74.81，每個點都略高
> - MuSiQue：**SAG 從 87.90 降到 82.57（−5.33 點）**，HippoRAG 2 從 69.23 降到 60.27（−8.96 點），BGE 約從 62 降到 59（從圖估讀）
> - 語料變多會帶進「實體重疊的干擾段落」，把橋接段落擠掉；SAG 退化得比較慢

### 5.6 語料庫連通性

<center><b>表 11：MuSiQue 上各方法索引的連通程度（原論文 Table 2）</b></center>

| 方法 | 索引關係 | 節點 | 連結 | 平均度數 | 連通分量 | 最大分量 | 孤立點 |
|---|---|---|---|---|---|---|---|
| GraphRAG | 實體–實體 | 128,189 | 106,489 | 1.66 | 66,472 | 45.9% | 51.1% |
| LightRAG | 實體–關係 | 82,321 | 116,555 | 2.83 | 9,277 | 85.3% | 10.3% |
| HyperGraphRAG | 實體–實體 | 186,897 | 272,524 | 2.92 | 4,600 | 91.6% | 0.3% |
| HippoRAG 2 | 段落–實體 | 97,043 | 570,458 | 11.76 | **96** | **99.7%** | 0.1% |
| SAG | 事件–實體 | 95,341 | 126,540 | 2.65 | 400 | 96.7% | **0.001%** |

- 連通分量越少、最大分量越大，代表索引越不零碎、越多段落「走得到」
- GraphRAG 的分量數是 SAG 的 166 倍，**超過一半的節點是孤立的**
- HippoRAG 2 最連通，但連結數是 SAG 的 4.5 倍，Recall@5 卻低很多。作者的結論是「連結多不等於好」，SAG 是用中等的度數達到高連通
- 這幾種圖的節點、邊意義不同，**只能看零碎程度，不能直接比大小**

### 5.7 成功與失敗的例子

<center>
<img src="https://arxiv.org/html/2606.15971v2/figures/good-query.png" alt="圖 6" width="50%">
<br>
<b>圖 6：擴展成功的例子（原論文 Figure 4）</b>
</center>

> [!NOTE]
> **圖 6 怎麼看**
> 問題：「畫《The Bacchanal of the Andrians》的畫家，死在哪個城市？那個城市發生過幾次瘟疫？」標準文件是《The Bacchanal of the Andrians》、Black Death（黑死病）、《The Martyrdom of Saint Lawrence》（Titian）。
> - 中間藍框是問題，**橘色圈是實體**、**白色圈是一般事件**、**藍色圈是進了前 5 的事件**、**綠框藍圈是進了前 5 的標準文件**
> - 種子路徑直接找到 Black Death 和《The Bacchanal of the Andrians》（左下兩個綠色）
> - 從 Black Death 事件找到共同實體 **Venice**（橘色），再沿 Venice 擴展，找到第三份標準文件《The Martyrdom of Saint Lawrence》（左上綠色）
> - 右側一排白色的 Black Death 等是相似但沒被選中的事件

<center>
<img src="https://arxiv.org/html/2606.15971v2/figures/bad-query.png" alt="圖 7" width="50%">
<br>
<b>圖 7：擴展失敗的例子（原論文 Figure 5）</b>
</center>

> [!NOTE]
> **圖 7 怎麼看**
> 問題：「Greenwood Laboratory School 所在的城市，什麼時候成為《The Poor Boob》編劇出生州的首府？」需要 4 份標準文件。
> - 種子找到 The Poor Boob 和 Greenwood Laboratory School（兩個綠色），但**這兩個事件沒有留下可以通往其他證據的共同實體**，所以沒有擴展
> - 編劇 Margaret Mayo 和 Springfield, Illinois 兩份都沒找到，前 5 裡塞進了 Oklahoma、Maine、Washington 等不相關的事件
> - 結論：擴展只有在種子事件裡<b>剛好有一個有辨識度的「橋接實體」</b>時才有效

### 5.8 NarrativeQA（附錄 K）

在小說、劇本的閱讀理解資料集上，只和 HippoRAG 2 比：EM 6.48 → 12.29，F1 22.86 → 31.90。兩者分數都很低，只有一個比較對象，參考價值有限。

---

## 6. 總結

1. 向量檢索每段各自打分，串不起多步驟證據；知識圖譜能串，但建圖貴、會拆碎語意、難更新
2. SAG 每個段落只抽**一個事件 + 一串實體**，存成 SQL 的多對多表，事件等於一條保留多元關係的超邊
3. 查詢時：實體路徑 + 向量路徑找種子 → **用共同實體 JOIN 擴展一輪** → 取前 100 → **LLM 整體挑最多 5 個** → 向量檢索補滿 10 段，交給 LLM 的永遠是原文
4. 三個多步驟資料集的檢索與問答都第一，MuSiQue Recall@5 80.36%；跳數越多領先越多
5. 換向量模型、語料增加時都比 HippoRAG 2 穩；只在前 5% 的段落裡做就能保住 99% 的結果
6. 代價：建索引比 HippoRAG 2 貴，**每題查詢 23–25 秒，比 HippoRAG 2、GraphRAG 慢**

> [!WARNING]
> **限制（作者自己提的）**
> - **不處理別名**：「Apple Inc.」和「Apple」被當成兩個實體，靠別名才能串起的關聯會斷。作者建議加一張輕量的別名表
> - **只能追加，不能修改**：沒有機制更新或淘汰過時的事件，當作代理人的記憶還不夠（要處理會變的事實），未來想做版本化的事件

> [!WARNING]
> **我核對數字時注意到的地方**
> - **「最強基準」前後指不同對象**：摘要說 MuSiQue 領先最強基準 11.52 點，比的是 NV-Embed-v2（68.84）；正文說領先 15.23 點，比的是 HippoRAG 2（65.13）。兩個數字都算得出來，但比較對象不同
> - **比較不完全對等**：SAG 用 LLM 一次讀 100 個候選再挑，向量基準沒有任何重新排序。換成一般的重新排序模型後，SAG 的 Recall@5（67.11）反而**低於單純的 NV-Embed-v2（68.84）**，論文沒提這點
> - **Recall@5 幾乎只反映 LLM 挑的那幾段**：論文的圖 2 把結構路徑的段落接在前面（論文沒明寫排序，這是我從 Concat 順序推斷的），$K_{event}=5$ 時前 5 名多半就是 LLM 挑的結果，語意路徑主要影響第 6–10 名；問答也只用前 5 段（Table 12），所以語意補位對問答的貢獻應該很小
> - **預設不是最好的設定**：$K_{cand}=500$ 時 Recall@5 82.24，比預設 80.36 高，作者選 100 是為了成本
> - **評審和閱讀器同一家**：閱讀器是 Qwen3.6-Flash，評審是 Qwen3.7-Plus，可能有同家族偏好
> - **語料增長實驗的數字和主表不同**：MuSiQue 在 100% 語料時是 82.57，主表是 80.36，因為只用四分之一的題目評估，兩者不能直接比
> - **連通性的「孤立點幾乎為零」只適用 MuSiQue**：HotpotQA 上有 5,787 個孤立點（5.8%）
> - **精選提示詞的例子會湊滿 5 個**：附錄 Figure 7 的模型推理裡寫「We need 5 relations」，把只是佐證的段落也補進來；提示詞還沿用「knowledge graph 的關係描述」這種舊措辭
> - **v2 的版本說明**：作者註明 v2 曾被誤發成另一篇（arXiv:2608.12129，已撤回）；作者是公司團隊，兩個 GitHub 是自家專案

> [!TIP]
> **可以帶走的思路**
> - **不建圖也能做多步驟檢索**：每段抽一句事件 + 實體，存成 SQL 多對多表，查詢時 JOIN 共同實體就能找到橋接段落。現有的 MySQL／PostgreSQL + Elasticsearch 就夠
> - **索引只追加**：新文件進來只要抽取、寫入新列，不用重建任何東西，很適合持續增加的公文、報告、事件紀錄
> - **證據永遠用原文**：結構只拿來「找」，交給 LLM 的是原段落，不會因為摘要或三元組丟資訊
> - **最值錢的是「LLM 整體挑選」**：讓 LLM 一次看一批候選、挑出能湊成證據鏈的幾段，效果遠勝逐一打分的重新排序模型；這招可以單獨加到任何 RAG 上
> - **先用向量縮小範圍**：只在前幾百段裡做結構擴展，保住大部分效果，成本可控

---

## 7. 我的筆記

### 重點筆記
>

### 可以借用的地方
> 災防資料（事件通報、災情報告）本來就是「事件 + 地點 + 時間 + 單位」的形式，很適合 SAG 的事件–實體表；地名、單位名的別名問題要先建一張別名表。可以和 GDP-RAG 合起來：SAG 負責檢索出橋接段落，GDP-RAG 負責決定還缺什麼要再問。

### 疑問
> 中文實體的正規化（簡繁、全半形、機關簡稱）會不會讓 JOIN 斷得更嚴重？每題 20 秒以上的延遲，線上服務能接受嗎？「LLM 整體挑選」單獨加在一般向量檢索後面，能拿到多少效果？

### 相關
- 概念：[[Retrieval-augmented generation|檢索增強生成]]、[[Multi-hop question answering|多步驟問答]]、[[GraphRAG]]、[[Hypergraph|超圖]]、[[Personalized PageRank|個人化 PageRank]]、[[Recall@k]]、[[BM25]]、[[Graph database|圖資料庫]]、[[LLM-as-a-judge|LLM 當評審]]
- 相關論文：[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]（同樣用 HotpotQA、2Wiki、MuSiQue；GDP-RAG 改的是「怎麼規劃子問題」，SAG 改的是「怎麼建索引、怎麼串段落」，兩者可以疊加）
- 相關論文：[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]（SAG 的評測協定沿用自 IRCoT／HippoRAG 2；IRCoT 改檢索流程，SAG 改索引結構）
- 相關論文：[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]（同樣讓 LLM 透過 SQL／圖資料庫取資料，但 PlanRAG 查的是現成的結構化資料庫）
- 相關論文：[[【MemRec：Collaborative Memory-Augmented Agentic Recommender System】]]（MemRec：同樣在圖上臨時取出一小塊鄰域、交給 LLM 精選，但對象是使用者–商品的文字記憶）
- 主要比較對象：HippoRAG 2（Gutiérrez et al., ICML 2025）、GraphRAG（Edge et al., 2024）、LightRAG（EMNLP 2025）、HyperGraphRAG（NeurIPS 2025）、HyperRAG（Nature Communications 2026）
- 資料集：MuSiQue、2WikiMultiHopQA、HotpotQA、NQ、NarrativeQA
- 相關論文：[[【A Survey on LLM-as-a-Judge】]]（本篇用 LLM 判斷答案對錯；綜述整理了評審的偏誤與檢查方法，可以用來檢視這類評估設計）
