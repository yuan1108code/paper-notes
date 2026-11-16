---
title: "【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】"
tags: [paper, llm, rag, decision-making, status-inbox]
concepts: ["[[Retrieval-augmented generation]]", "[[ReAct]]", "[[Graph database]]", "[[Compound AI system]]", "[[Chain-of-thought prompting]]"]
papers: ["[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]", "[[【Optimizing generative AI by backpropagating language model feedback】]]", "[[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]", "[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]", "[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]"]
---

# 【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 作者 | Myeonghwa Lee, Seonho An, Min-Soo Kim（韓國 KAIST 計算學院） |
| 發表 | NAACL 2024；arXiv 2406.12430（2024/6/18） |
| 連結 | [arXiv](https://arxiv.org/abs/2406.12430)・[PDF](https://arxiv.org/pdf/2406.12430)・[GitHub（程式碼與資料集）](https://github.com/myeon9h/PlanRAG) |
| 動畫解說 | [papers.poyuanchu.com/2024-lee-planrag/](https://papers.poyuanchu.com/2024-lee-planrag/) |
| 論文類型 | 新任務 + 新資料集 + 新方法：定義 Decision QA、建立 DQA 資料集、提出 PlanRAG |
| 狀態 | 待讀 |
| 與我的相關度 | 5 / 5（讓 LLM 查資料庫、依商業規則做決策，和智慧決策支援、排班、工業 AI 落地直接相關） |
| 加入日期 | 2026-11-11 |

> [!NOTE]
> **一句話**：讓 LLM 當決策者，從大型資料庫裡分析資料、選出最好的決策。關鍵是**先擬計畫、再查資料**：先列出「要做哪些分析、依什麼順序」，照計畫產生 SQL 或 Cypher 查詢，發現計畫不夠時再**重新規劃**。比只會邊想邊查的迭代式 RAG，正確率高 15.8 和 7.4 個百分點。

---

## 1. 背景：LLM 能不能幫忙做決策？

### 決策的三個步驟

作者用輝瑞藥廠舉例：要降低生產成本又要準時出貨，就得決定（P1）**哪座工廠要開或停**、（P2）**每座工廠要僱多少人**。這類決策通常有三步：

1. **擬計畫**：決定要做哪些分析
2. **查資料**：下查詢、取出需要的數據
3. **做決定**：根據分析結果選出答案

過去幾十年的決策支援系統，主要是讓第 2、3 步比較輕鬆，**最難的第 1 步一直靠人**。這篇想知道：LLM 能不能從頭到尾三步都包辦？

### 現有 RAG 的盲點

RAG 讓 LLM 先檢索再回答；迭代式 RAG（例如 IRCoT、ReAct）可以邊推理邊多次檢索。但它們都是為**知識問答**設計的，擅長「查出某個事實」，**不擅長規劃分析流程**。

論文的觀察：問「怎麼讓 Deccan 的利潤最大」時，現有方法只會先去查「Deccan 是什麼」，而不是先想清楚該算哪些東西。

> [!TIP]
> 這和工業場域的痛點一樣：資料都在資料庫裡，真正難的是**知道該查什麼、怎麼組合成決策**。

---

## 2. 新任務：Decision QA

### 定義

給定三樣東西，輸出最佳決策 $d_{best}$：

- **決策問題 Q**：用文字描述目標，例如「我該把商人放在哪個貿易節點？目標是讓 BAH 在本國節點 Deccan 的利潤最大」
- **商業規則 R**：用文字寫出計算公式，例如利潤怎麼算
- **資料庫 D**：遵循某個資料表結構（schema）S，可能是關聯式資料庫（RDB，用 SQL 查）或圖資料庫（GDB，用 Cypher 查）

資料庫**太大，不能整個塞進提示詞**，LLM 必須自己下查詢取資料。

<center>
<img src="https://arxiv.org/html/2406.12430v1/DQAexample.v28.png" alt="圖 1" width="90%">
<br>
<b>圖 1：Decision QA 的例子（原論文 Figure 1）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**
> 由上到下對應決策的三個步驟，場景取自遊戲《歐陸風雲 IV》的國際貿易。
> - **Step 1（上方）**：使用者問「商人該放哪？目標是 BAH 在本國節點 Deccan 的利潤最大」。灰色圓柱是資料庫，裡面有四張表：國家（Country，橘色表頭）、國家在各節點的貿易力（NodeCountry，黃色）、貿易節點的數值（TradingNode，灰色）、節點之間的貿易流向（TradingFlow，藍色）。帶底線的欄位是主鍵。LLM 的計畫是三步：找出流向 Deccan 的來源節點 → 算每個決策會增加多少流量 → 算利潤增加多少。
> - **Step 2（中間兩張地圖）**：紅點是貿易節點，白色箭頭是貿易流向。
>   - 左圖「決策 1：商人放 Doab」：Doab 往 Deccan 的流量增加 1.62（粉紅標籤），Deccan 利潤 **+0.23**（綠字）
>   - 右圖「決策 2：商人放 Ganges」：流量先往 Doab +0.79，再往 Deccan 只剩 +0.13（青色標籤），利潤只 +0.02
> - **Step 3（下方）**：回答「把商人放在 Doab」。
>
> 重點：**利潤增加多少並沒有存在資料庫裡**，必須依規則從資料算出來。

### 和一般表格問答的差別

- 要依**商業規則**計算，不是查一個現成的值
- 資料庫大很多：Locating 情境平均每張表 2,038.8 列，一般表格問答資料集只有個位數到 30 列

---

## 3. 新資料集：DQA

真實企業資料難以取得，作者改從兩款**模擬經濟與商業的策略遊戲**擷取資料，並自己寫了模擬器，消除遊戲的隨機性、算出每個決策的真實結果當標準答案。

<center><b>表 2：DQA 的兩個情境</b></center>

| 情境 | 來源遊戲 | 問題類型 | 目標 | 題數 |
|---|---|---|---|---|
| Locating（選址） | 歐陸風雲 IV | 商人要放哪個貿易節點（類似「哪座工廠要開」） | 讓指定國家的利潤增加最多 | 200 |
| Building（擴建） | 維多利亞 3 | 要擴建哪座工廠（類似「每座工廠僱多少人」） | 讓指定商品的價格最低 | 101 |

每題都有 RDB 和 GDB 兩個版本，所以共 602 個資料庫。

### Locating 的商業規則

$$TPR(n, c) = \frac{TP_{country}(n, c)}{TP_{total}(n)}$$

$$IV(dest) = 1.05 \cdot \sum_{(src, dest) \in F} flow(src, dest)$$

$$profit(c) = (LV(h) + IV(h)) \cdot TPR(h, c)$$

白話：國家在本國節點 $h$ 的利潤 = （本地產值 + 流入的產值）× 自己貿易力的佔比。在某個節點放商人，會增加從那裡流向本國節點的流量。

### Building 的商業規則

$$CP(g) = BP(g) \cdot \left(1 + 0.75 \cdot \frac{TD(g) - TS(g)}{\max(TD(g), TS(g))}\right)$$

白話：商品現價 = 基本價 ×（1 + 0.75 × 供需落差的比例）。需求大於供給就漲價，供給大於需求就降價。

<center>
<img src="https://arxiv.org/html/2406.12430v1/building_scenario.v16.png" alt="圖 2" width="55%">
<br>
<b>圖 2：Building 情境的例子（原論文 Figure 2）</b>
</center>

> [!NOTE]
> **圖 2 怎麼看**
> 目標：讓家具（紅點）便宜。B1、B2 是家具工廠，B3 供應木材（綠點）和硬木（藍點）。上方灰色圓柱是簡化的四張表。CI 是工廠目前吃進的原料量，CO 是產出的家具量。
> - **(a) 現況**：B1 吃 40 單位木材、產 40 個家具；B2 吃 25 單位硬木、產 50 個家具。
> - **(b) 決策 1：擴建 B1**（打勾）：B1 變兩座廠房，木材吃到 80、家具產到 **80**（綠字）。木材供應有 120，夠用。
> - **(c) 決策 2：擴建 B2**（打叉）：B2 雖然也擴建，但硬木總共只有 30，CI 只從 25 增加到 30，家具只產到 **60**（紅字）。
>
> 重點：表面上兩座工廠都能擴建，要**沿著供應鏈多查幾層**（工廠 → 原料 → 原料供給量）才知道哪個有效。這也是 Building 比 Locating 難的原因。

<center><b>表 3：DQA 資料庫規模</b></center>

| 統計 | Locating | Building |
|---|---|---|
| 題數 | 200 | 101 |
| RDB 每張表平均列數 | 2,038.8 | 579.0 |
| RDB 每張表平均欄數 | 4.5 | 4.5 |
| GDB 平均邊數 | 1,432.3 | 374.7 |
| GDB 平均節點數 | 606.5 | 204.3 |

---

## 4. 方法：PlanRAG

一般 RAG（單次或迭代）只有一種推理：**根據查到的資料推理**。PlanRAG 多了一種：**擬計畫**。兩種推理由**同一個 LLM** 完成，避免多個模型互相配合的副作用。實作上是在 ReAct 提示詞裡加上「Plan」和「Re-plan」兩個指令。

### 三個步驟

1. **規劃（Planning）**：輸入問題 Q、資料表結構 S、商業規則 R，先產生一份**分析計畫**，列出需要做哪些分析、依什麼順序
2. **檢索與回答（Retrieving & Answering）**：輸入 Q、S、R **加上計畫**，照計畫產生 SQL 或 Cypher 查詢，實際在 MySQL 或 Neo4j 執行，再根據結果推理
3. **重新規劃（Re-planning）**：每次查完，LLM 都要評估目前的計畫還行不行。不夠的話，就根據 Q、S、R、目前計畫和查詢結果產生新計畫，補分析或修正方向；夠了就回答

<center>
<img src="https://arxiv.org/html/2406.12430v1/solving_example.v7.png" alt="圖 3" width="95%">
<br>
<b>圖 3：迭代式 RAG 與 PlanRAG 的推理過程比較（原論文 Figure 4）</b>
</center>

> [!NOTE]
> **圖 3 怎麼看**
> 同一題（商人放哪裡），最上方是題目和資料表結構。黃色區塊是檢索、粉紅區塊是規劃、藍色區塊是回答；每個圓柱 DB 圖示代表一次資料庫查詢和它的結果。
> - **左 (a) 迭代式 RAG**：沒有計畫，想到什麼查什麼。第一次查來源節點得到 Doab、Patagonia、Malacca；第二次查錯了，拿到 Deccan 本身的資料，紅字寫「觀察結果好像不對」；第三次修正查詢；最後算出利潤增加 Doab 0.0011、Patagonia 0.0728、Malacca 0.0017，選了 **Patagonia**，答錯（紅色叉叉）。論文把這歸類為「公式用錯，低估了 Doab」。
> - **右 (b) PlanRAG**：先在粉紅框寫好三步計畫：找出 Deccan 的來源節點 → 算每個節點放商人的利潤增加 → 選最大的。接著檢索區塊每一步都標註「（Step 1）」「（Step 2）」「（Step 3）」照計畫執行：查到唯一的來源是 Doab、算出利潤增加約 8.29、選出 Doab，答對（綠色勾勾）。
>
> 差別在於：有計畫時查詢**有條理、有目標**；沒計畫時容易繞路、查錯，也容易漏掉要算的東西。

> [!TIP]
> PlanRAG 本身不複雜，就是**提示詞多了「先規劃」「每步後檢查要不要重新規劃」**。這種「先擬計畫再執行、執行中可修正」的模式，很容易套到現有的代理人系統。

---

## 5. 實驗設定

<center><b>表 4：比較的四種決策者</b></center>

| 名稱 | 做法 |
|---|---|
| SingleRAG-LM | 只查一次資料庫就回答 |
| IterRAG-LM | 迭代式 RAG（以 ReAct 提示詞實作），可以多次查詢 |
| PlanRAG-LM | 本文方法，先規劃、可重新規劃 |
| PlanRAG-LM w/o RP | PlanRAG 但拿掉重新規劃 |

- 全部用 **GPT-4**，溫度 0，以 LangChain 實作；RDB 用 MySQL、GDB 用 Neo4j
- **零樣本、只跑一次**。作者不用示範例子的理由：真實商業情境很難事先知道最佳策略；而且有示範時，模型會連示範裡的資料庫內容一起過度模仿
- 答案和標準答案**語意相同**才算對

---

## 6. 結果

### 6.1 主要結果

<center><b>表 5：DQA 正確率（%，RDB 與 GDB 的平均，原論文 Table 4）</b></center>

| 決策者 | Locating | Building |
|---|---|---|
| SingleRAG-LM | 30.5 | 2.5 |
| IterRAG-LM | 48.5 | 37.6 |
| **PlanRAG-LM** | **64.3** | **45.0** |
| PlanRAG-LM（不重新規劃） | 53.5 | 44.1 |

- PlanRAG 比迭代式 RAG 高 **15.8**（Locating）和 **7.4**（Building）個百分點（論文寫成 15.8%、7.4%，其實是百分點差距）
- **拿掉重新規劃**：Locating 掉 10.8 點，Building 掉 0.9 點，重新規劃確實有用
- 單次 RAG 在 Building 幾乎全錯（2.5%）：Building 需要非常複雜的查詢，一次寫不出來。它在超過 60% 的 Locating 題和 95% 的 Building 題**完全沒查到任何資料**
- Building 進步較少，因為要沿著供應鏈走更多層，**計畫本身就比較難擬**

<center><b>表 6：不同資料庫的正確率（%，原論文 Table 5）</b></center>

| 決策者 | Locating RDB | Locating GDB | Building RDB | Building GDB |
|---|---|---|---|---|
| SingleRAG-LM | 25.5 | 35.5 | 2.0 | 3.0 |
| IterRAG-LM | 37.5 | 59.5 | 34.7 | 40.6 |
| **PlanRAG-LM** | **64.5** | **64.0** | **40.6** | **49.5** |

PlanRAG 在兩種資料庫都最好。Building 用圖資料庫時進步更多，因為多層關係在圖上是「多走幾步」，在 RDB 則要寫很多 JOIN。

### 6.2 簡單題反而進步最多

作者把題目分成兩類：迭代式 RAG **只查一次**就作答的（SR，單次檢索題），和**查了多次**的（MR）。

<center><b>表 7：單次檢索題與多次檢索題的正確率（%，原論文 Figure 5）</b></center>

| 情境 | 題型（佔比） | IterRAG-LM | PlanRAG-LM |
|---|---|---|---|
| Locating | SR（20.1%） | 43.9 | **65.6** |
| Locating | MR（79.9%） | 49.4 | **61.9** |
| Building | SR（14.5%） | 12.5 | **20.2** |
| Building | MR（85.5%） | 40.3 | **47.9** |

SR 題的進步比 MR 大。作者解釋：這些題目**其實不簡單**，是迭代式 RAG **低估了難度**，只查一次就急著回答；PlanRAG 先擬計畫，比較能看出需要查好幾次。

### 6.3 錯在哪裡

**漏掉關鍵分析的比例**（該查或該算的關鍵數值沒處理，原論文 Table 6）：

| 決策者 | Locating | Building |
|---|---|---|
| IterRAG-LM | 3.3% | 33.2% |
| PlanRAG-LM | **1.3%** | **21.8%** |

**失敗原因分類**（原論文 Figure 6，數值為從圖上估讀，佔全部題目的比例）：

<center><b>表 8：失敗原因（約略值）</b></center>

| 錯誤類型 | 意思 | Locating：Iter → Plan | Building：Iter → Plan |
|---|---|---|---|
| CAN | 考慮了錯的候選（例如把 Deccan 的下游節點當候選） | 約 29% → 約 6.5% | 約 5% → 約 0% |
| MIS | 漏掉需要的分析 | 約 1.5% → 約 0% | 約 22% → 約 13.5% |
| DEEP | 資料或公式用錯 | 約 14.5% → 約 23.5% | 約 18% → 約 23.5% |
| QUR | 查詢語法寫錯 | 約 1% → 約 1% | 約 9% → 約 8% |
| OTH | 其他（例如超過長度上限） | 約 5% → 約 5% | 約 8% → 約 10% |

PlanRAG **大幅減少 CAN 和 MIS**：比較懂題目在問什麼、比較不會漏查。DEEP 反而略增，作者解釋：DEEP 只會在沒有 CAN、MIS 錯誤時出現（候選都沒找到，就談不上算錯），所以這是前兩類減少後的副作用。

### 6.4 重新規劃的統計

<center><b>表 9：依重新規劃次數分組（題數與該組正確率，原論文 Table 7）</b></center>

| 重新規劃次數 | Locating | Building |
|---|---|---|
| 0 次 | 376 題（66.8%） | 125 題（57.6%） |
| 1 次 | 0 | 0 |
| 2 次 | 12 題（41.7%） | 24 題（41.7%） |
| 3 次 | 5 題（20.0%） | 23 題（21.7%） |
| 4 次以上 | 7 題（0.0%） | 30 題（13.3%） |
| 合計 | 400 題（64.3%） | 202 題（45.0%） |

- Building 重新規劃的比例高很多（38% 的題目，Locating 只有 6%），代表 Building 的計畫比較難一次擬對
- **重新規劃越多次，正確率越低**：需要一改再改的，通常是模型本來就搞不懂的題目
- 附錄的分類：Locating 最常見的修改是「**增加查詢步驟**」，Building 最常見的是「**把一步拆成好幾個子步驟**」

> [!WARNING]
> 表格裡「1 次重新規劃」的題數是 0，論文沒有解釋原因，可能和計數方式有關（例如第一次規劃也算一次）。

### 6.5 換成其他模型

附錄 Table 11 換成 GPT-3.5、Llama 2（70B、13B）、Phi-2：

- Llama 2 和 Phi-2，兩種方法**一題都答不對**（全部 0%）
- GPT-3.5 上，PlanRAG 反而比迭代式 RAG 差（例如 Locating RDB 0% vs 8.0%）。作者認為 PlanRAG 的提示詞**太複雜**，GPT-3.5 跟不上

> [!WARNING]
> 這代表 PlanRAG 的效果**高度依賴 GPT-4 等級的模型**。地端或小模型目前很難直接套用。

---

## 7. 總結

1. 定義新任務 **Decision QA**：根據問題、商業規則和大型資料庫，選出最佳決策
2. 從兩款策略遊戲建立 **DQA** 資料集：301 題、602 個資料庫（RDB 和 GDB 各一份）
3. 提出 **PlanRAG**：先規劃再檢索，每步後評估要不要重新規劃
4. GPT-4 上比迭代式 RAG 高 15.8 和 7.4 個百分點；主要是減少找錯候選、漏查資料
5. 重新規劃有幫助，但改越多次越可能還是錯；換成較弱的模型就失效

> [!WARNING]
> **限制**
> - 資料來自遊戲，和真實企業資料仍有差距
> - 只用 GPT-4、零樣本、跑一次，沒有多次實驗的變異
> - 只處理 RDB 和 GDB，沒處理向量資料庫或混合式資料
> - 沒有針對查詢生成做微調（例如專門寫 Cypher 的模型）
> - 只用單一 LLM，沒試多個模型分工
> - 論文把百分點差距寫成「%」，引用時要注意

> [!TIP]
> **可以帶走的思路**
> - **決策型 RAG 要先規劃**：問題是「該怎麼做」而不是「是什麼」時，先讓 LLM 列分析計畫，再照計畫查資料
> - **把商業規則寫進提示詞**：利潤、價格這類衍生數值不在資料庫裡，要讓模型知道怎麼算
> - **工業應用**：排班、產線調度、備料決策都符合「規則 + 資料庫 + 選最佳決策」的形式，可以拿來做決策支援原型
> - **評估要分錯誤類型**：CAN、MIS、DEEP、QUR 這種分類，比只看正確率更能看出系統卡在哪
> - **和最佳化方法搭配**：LLM 負責理解問題、擬計畫、查資料；真正的最佳化（例如 QUBO、整數規劃）交給專門的求解器，也許比讓 LLM 自己算更可靠

---

## 8. 我的筆記

### 重點筆記
>

### 可以借用的地方
> 智慧排班引擎：能不能讓 LLM 依 PlanRAG 的方式，先讀勞基法規則和人員資料表、擬分析計畫，再把限制式交給 QUBO 求解器？工研院的工業資料庫決策支援，能不能用 DQA 的錯誤分類來評估？

### 疑問
> 規則很多、很長時，計畫還擬得出來嗎？如果用 TextGrad 優化規劃用的提示詞，能不能讓 GPT-3.5 或地端模型也跑得動？

### 相關
- 概念：[[Retrieval-augmented generation|檢索增強生成]]、[[ReAct]]、[[Graph database|圖資料庫]]、[[Compound AI system|複合 AI 系統]]、[[Chain-of-thought prompting|思維鏈提示]]
- 相關論文：[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]（本文比較的迭代式 RAG 之一）、[[【Optimizing generative AI by backpropagating language model feedback】]]（可以用來優化規劃提示詞）
- 相關論文：[[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]（同樣在意結構化資料：PlanRAG 用查詢語言，PixelRAG 用截圖保留結構）
- 後續論文：[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]（GDP-RAG：同樣先規劃再檢索，但規劃前先檢索一次、只為缺的資訊出子問題）
- 相關論文：[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]（SAG：同樣讓檢索走 SQL，但對象是從非結構化文件抽出的事件–實體表）
- 提示詞基礎：ReAct（Yao et al., ICLR 2023）
- 其他迭代式 RAG：FLARE（Jiang et al., 2023）
- 表格問答資料集：HybridQA、TAT-QA、FinQA、WikiTableQA、TabFact
