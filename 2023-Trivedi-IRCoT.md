---
title: "【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】"
tags: [paper, llm, rag, multi-hop-qa, status-inbox]
concepts: ["[[Retrieval-augmented generation]]", "[[Multi-hop question answering]]", "[[Chain-of-thought prompting]]", "[[BM25]]", "[[Compound AI system]]"]
papers: ["[[【Optimizing generative AI by backpropagating language model feedback】]]", "[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]", "[[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]", "[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]", "[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]"]
---

# 【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 作者 | Harsh Trivedi, Niranjan Balasubramanian（Stony Brook University）、Tushar Khot, Ashish Sabharwal（Allen Institute for AI） |
| 發表 | ACL 2023；arXiv 2212.10509（v2，2023/6/23） |
| 連結 | [arXiv](https://arxiv.org/abs/2212.10509)・[PDF](https://arxiv.org/pdf/2212.10509)・[GitHub（ircot）](https://github.com/stonybrooknlp/ircot) |
| 動畫解說 | [papers.poyuanchu.com/2023-trivedi-ircot/](https://papers.poyuanchu.com/2023-trivedi-ircot/) |
| 論文類型 | 方法論文：提出不需訓練的檢索方法 IRCoT |
| 狀態 | 待讀 |
| 與我的相關度 | 5 / 5（RAG 問答的核心問題：一次檢索不夠時怎麼辦；方法只靠提示詞，可以直接套到現有系統） |
| 加入日期 | 2026-11-11 |

> [!NOTE]
> **一句話**：多步驟的問題，光用題目去搜一次資料不夠，因為**下一步要查什麼，取決於上一步推出了什麼**。IRCoT 讓 LLM **每寫一句推理就拿這句話再去檢索一次**，查到的新資料再幫它寫下一句，推理和檢索交替進行。

---

## 1. 背景：為什麼「搜一次」不夠？

### 思維鏈與檢索各自的限制

- **思維鏈**（chain-of-thought，CoT）：請 LLM 一步一步寫出推理，多步驟題目的正確率會明顯提升。但如果需要的知識**不在模型裡或已經過時**，它就會一本正經地編出錯的事實（幻覺）
- **檢索增強**（RAG）：先用題目去知識庫搜相關段落，再交給 LLM 回答。對「答案就寫在某一段」的簡單題很有效

### 多步驟題目的困境

論文的例子：「**Lost Gravity（雲霄飛車）是在哪個國家製造的？**」

1. 用題目去搜，會找到 Lost Gravity 的維基百科頁面，但上面**只寫了製造商是 Mack Rides，沒寫國家**
2. 要先推出「製造商是 Mack Rides」，再拿「Mack Rides」去搜，才找得到「Mack Rides 是德國公司」

第二段資料和原題目**幾乎沒有共同的字**，只用題目去搜很難找到。

所以作者的觀察是：

- **沒有檢索**，模型推理時容易產生幻覺
- **沒有先推出第一步**，就不知道第二步該查什麼

兩者要**互相帶路**。

> [!TIP]
> 這正是 RAG 系統常見的失敗模式：使用者的問題需要串好幾份文件，但檢索只做一次，第二份文件永遠搜不到。

---

## 2. 核心想法：推理一句、檢索一次

<center>
<img src="https://arxiv.org/html/2212.10509v2/main-figure.png" alt="圖 1" width="95%">
<br>
<b>圖 1：IRCoT 的運作流程（原論文 Figure 2）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**
> 最上方是題目 Q：「Murray Head 最為人所知的 1970 年國際暢銷歌曲，是誰寫的？」
> - **左側大框：流程**。由上往下讀，箭頭往右是「檢索」，往左是「推理」：
>   1. `Retrieve(Q)`：先用題目搜一批段落（**橘色**文件圖示）
>   2. `T1 ← Reason(Q, 橘色段落, ∅)`：根據題目和橘色段落，寫出第一句推理 T1（**黃色框**）：「這首歌是《Super Star》」
>   3. `Retrieve(T1)`：拿 T1 這句話去搜，得到**藍色**段落
>   4. `T2 ← Reason(Q, 橘+藍, T1)`：寫出第二句 T2（**藍色框**）：「《Super Star》是 Andrew Lloyd Webber 和 Tim Rice 寫的」
>   5. `Retrieve(T2)`：拿 T2 再搜，得到**綠色**段落
>   6. `T3`（**綠色框**）出現「So the answer is」，觸發右側橘色的 **Stop**
> - **左側最下方**：三批段落（橘 + 藍 + 綠）合起來，就是最後的檢索結果，交給讀者模型回答。
> - **右上 Retrieve(q)**：檢索就是拿一句查詢 q，經過搜尋引擎（放大鏡），得到幾份文件。
> - **右下 Reason(Q, 段落, T1)**：實際送給 LLM 的提示詞長相。上半是示範例子（Lost Gravity 那題的段落和完整推理），下半是目前收集到的段落、題目 Q 和已經寫好的推理 T1。藍底的那句是模型這次新產生的。

### 流程（論文 3.1 節）

1. **起手**：用題目 Q 檢索 K 個段落
2. **推理步**：把題目、目前收集的所有段落、已寫好的推理句交給 LLM，請它接著寫。**就算它一次寫了好幾句，也只留第一句**
3. **檢索步**：拿剛寫好的那句當查詢，再檢索 K 個段落，加進收集的集合
4. 重複 2、3，直到推理句出現「answer is」，或超過最多步數（**8 步**）
5. 所有收集到的段落（上限 **15 段**）就是檢索結果

### 三個需要的東西

<center><b>表 2：IRCoT 的組成</b></center>

| 元件 | 本文用的 | 說明 |
|---|---|---|
| 基礎檢索器 | BM25（Elasticsearch） | 傳統的關鍵字檢索，給一句查詢回傳 K 個段落 |
| 會寫思維鏈的 LLM | GPT3（code-davinci-002）、Flan-T5 各種大小 | 不需要任何訓練，只用提示詞 |
| 少量示範 | 每個資料集人工寫 20 題的推理過程 | 示範裡放正確的支持段落，加上 M 個（1–3）隨機的干擾段落 |

> [!TIP]
> **完全不用訓練**：IRCoT 只改了「怎麼檢索」，檢索器和 LLM 都是現成的。這是它容易落地的原因。

---

## 3. 讀者：用收集到的段落回答

檢索完之後，另外用一個「讀者」（QA reader）根據段落回答，有兩種提示方式：

- **CoT 提示**：讓模型從頭寫完整的推理，最後一句寫「answer is: …」
- **直接提示**（Direct）：只輸出答案

附錄的比較發現：**Flan-T5 用直接提示比較好，GPT3 用 CoT 提示比較好**，正文實驗就照這樣選。

> [!NOTE]
> IRCoT 在檢索過程中其實已經產生了一條推理鏈，能不能直接拿它的答案、省掉讀者？附錄 F 試過：Flan-T5 少了讀者明顯變差（例如 HotpotQA F1 從 59.1 降到 52.6）；GPT3 有時差不多，但有讀者的版本總是更好或接近。所以都保留獨立的讀者。

---

## 4. 實驗設定

<center><b>表 3：實驗設定</b></center>

| 項目 | 設定 |
|---|---|
| 資料集 | HotpotQA、2WikiMultihopQA、MuSiQue、IIRC，都是多步驟問答，改成「開放領域」設定（自己去語料庫找段落） |
| 語料庫大小 | HotpotQA 約 523 萬段、2WikiMQA 約 43 萬、MuSiQue 約 14 萬、IIRC 約 188 萬 |
| 資料切分 | 每個資料集 100 題調參、500 題測試 |
| 比較的檢索方式 | **NoR**：不檢索，只靠模型記憶；**OneR**：用題目檢索一次（K 選 5–15）；**IRCoT**：每步檢索 K 個（K 選 2、4、6、8） |
| 檢索指標 | 最多 15 段時，正確支持段落被找回來的比例（recall） |
| 問答指標 | 答案 F1（和標準答案的字詞重疊程度） |
| 重複 | 3 組不同的示範例子，回報平均與標準差 |

> [!NOTE]
> 為什麼不用 Recall@k、MAP 這類常見指標？因為 IRCoT 每題的推理步數不同，取回的段落數也不固定，而且段落沒有排名。作者改用「同樣最多 15 段的預算下，最好能找回多少」來比較。

---

## 5. 結果

### 5.1 檢索變好了

<center>
<img src="https://arxiv.org/html/2212.10509v2/main_retrieval_results.png" alt="圖 2" width="95%">
<br>
<b>圖 2：一次檢索（OneR）與 IRCoT 的檢索 recall（原論文 Figure 3）</b>
</center>

> [!NOTE]
> **圖 2 怎麼看**
> 左邊用 Flan-T5-XXL（11B）產生推理，右邊用 GPT3（175B）。每個資料集兩根柱子：**紅色點點是 OneR**、**深藍斜線是 IRCoT**，柱子上的數字是 recall（%），縱軸從 20 開始。
> - OneR 不用 LLM，所以左右兩邊的紅柱幾乎一樣（例如 HotpotQA 都是 61.5）
> - 藍柱全部比紅柱高，而且**右邊（GPT3）拉開得更多**：2WikiMQA 從 68.1 到 90.7，IIRC 從 36.0 到 57.2
> - 意思是：模型越會推理，推理句當查詢就越準

<center><b>表 4：IRCoT 相對於一次檢索的 recall 提升（百分點）</b></center>

| 推理用的模型 | HotpotQA | 2WikiMQA | MuSiQue | IIRC |
|---|---|---|---|---|
| Flan-T5-XXL | +7.9 | +14.3 | +3.5 | +10.2 |
| GPT3 | +11.3 | +22.6 | +12.5 | +21.2 |

### 5.2 問答也變好了

<center>
<img src="https://arxiv.org/html/2212.10509v2/main_qa_results.png" alt="圖 3" width="95%">
<br>
<b>圖 3：三種檢索方式的問答 F1（原論文 Figure 4）</b>
</center>

> [!NOTE]
> **圖 3 怎麼看**
> 每個資料集三根柱子：**橘色格紋是不檢索（NoR）**、**紅色點點是一次檢索（OneR）**、**深藍斜線是 IRCoT**，數字是答案 F1。
> - 幾乎都是「藍 > 紅 > 橘」，代表檢索有用，而且 IRCoT 的檢索更有用
> - 唯一例外是**右邊 GPT3 的 IIRC**：不檢索就有 52.1，反而最高。作者推測 IIRC 需要的知識 GPT3 本來就記得，所以檢索雖然 recall 提升了 21 點，問答卻沒變好
> - 左邊 Flan-T5 不檢索時分數很低（例如 MuSiQue 13.7），代表小模型的記憶有限，更需要好的檢索

<center><b>表 5：問答 F1（數字取自原論文附錄 Table 4，Flan-T5 用直接提示、GPT3 用 CoT 提示）</b></center>

| 模型 | 方法 | HotpotQA | 2WikiMQA | MuSiQue | IIRC |
|---|---|---|---|---|---|
| Flan-T5-XXL | 不檢索 | 25.3 | 32.7 | 13.7 | 28.9 |
| | 一次檢索 | 49.7 | 51.2 | 25.8 | 40.0 |
| | **IRCoT** | **59.1** | **66.5** | **30.8** | **42.5** |
| GPT3 | 不檢索 | 47.5 | 41.2 | 25.2 | **52.1** |
| | 一次檢索 | 53.6 | 54.8 | 29.4 | 49.8 |
| | **IRCoT** | **60.7** | **68.0** | **36.5** | 49.9 |

換資料集也成立：用 A 資料集的示範去測 B 資料集（out-of-distribution），IRCoT 一樣贏過一次檢索和不檢索。

### 5.3 推理更少編造事實

<center>
<img src="https://arxiv.org/html/2212.10509v2/factual_errors.png" alt="圖 4" width="60%">
<br>
<b>圖 4：40 題中，推理鏈至少有一個事實錯誤的題數（原論文 Figure 7）</b>
</center>

> [!NOTE]
> **圖 4 怎麼看**
> 作者每個資料集隨機抽 40 題，人工檢查 GPT3 產生的推理鏈，只要有一句事實是錯的就算一題。柱子越矮越好，顏色同圖 3。
> - 每個資料集都是「橘 > 紅 > 藍」：不檢索最容易編造，IRCoT 最少
> - HotpotQA 從 10 題降到 5 題（**少了 50%**），2WikiMQA 從 23 降到 14（約 **40%**）
> - MuSiQue 三種方法都偏高（28、27、23），這個資料集最難

注意：推理裡的事實全對，答案還是可能錯，反之亦然，所以這個指標和正確率是分開看的。

論文 Table 2 的例子很好說明差別：

- **不檢索**：第一句就錯（把舞者記成另一個人），後面全部跟著錯
- **一次檢索**：第一句對了（那是最接近題目的資訊），但第二句又錯
- **IRCoT**：每一步都重新查，兩句都對

### 5.4 小模型也有效

<center>
<img src="https://arxiv.org/html/2212.10509v2/model_scale_qa_results.png" alt="圖 5" width="95%">
<br>
<b>圖 5：不同模型大小的問答 F1（原論文 Figure 9）</b>
</center>

> [!NOTE]
> **圖 5 怎麼看**
> 三張小圖是三個資料集，橫軸是模型大小（十億參數，對數刻度）：Flan-T5 的 0.3、0.7、3、11，以及 GPT3 的 175。**藍色是 IRCoT，紅色是一次檢索**。
> - 除了最小的 0.3B，藍線都在紅線上面，而且差距大致隨模型變大而拉開
> - **最有意思的比較**：3B 的 Flan-T5-XL 用 IRCoT（HotpotQA 56.7、2WikiMQA 55.1、MuSiQue 32.3），比 175B 的 GPT3 用一次檢索（53.6、54.8、29.4）還好。模型小 58 倍，靠更好的檢索補回來
> - 最小的 0.3B 時，MuSiQue 上 IRCoT（13.3）反而略低於一次檢索（14.8），太小的模型寫不出有用的推理句

### 5.5 和其他系統比較

<center><b>表 6：與其他 LLM 開放領域問答系統的比較（EM ｜ F1，原論文 Table 3）</b></center>

| 系統 | HotpotQA | 2WikiMQA | MuSiQue |
|---|---|---|---|
| ReAct | 35.1 ｜ − | − | − |
| SelfAsk | − | 40.1 ｜ − | − |
| DecomP（2023 新版） | − ｜ 53.5 | − ｜ **70.8** | − ｜ 30.9 |
| DSP | **51.4** ｜ **62.9** | − | − |
| **IRCoT QA** | 49.3 ｜ 60.7 | 57.7 ｜ 68.0 | 26.5 ｜ **36.5** |

IRCoT 在 MuSiQue 仍是最好；HotpotQA 略輸 DSP（F1 差 2.0），2WikiMQA 略輸新版 DecomP（差 2.8）。

> [!WARNING]
> 這些系統用的知識來源、LLM、測試題目子集都不同，**不是公平的直接比較**，作者自己也強調只是排行榜式的參考。

---

## 6. 總結

1. 多步驟問題中，「下一步查什麼」取決於「上一步推出什麼」，所以**只用題目檢索一次不夠**
2. IRCoT 讓推理和檢索交替：**每寫一句推理，就拿這句去檢索**，新資料再幫助下一句推理
3. 不需要任何訓練，只要 BM25 + 會寫思維鏈的 LLM + 少量示範
4. 檢索 recall 最多提升 22.6 點、問答 F1 最多提升 15.3 點，推理中的事實錯誤最多減少 50%
5. 3B 小模型用 IRCoT，能贏過 175B 大模型只檢索一次

> [!WARNING]
> **限制**
> - 需要 LLM 會寫思維鏈，太小的模型（例如 0.3B）效果有限
> - 每寫一句推理就要呼叫一次 LLM，**成本和延遲比一次檢索高很多**
> - 收集的段落全部塞進提示詞，需要模型支援長輸入；作者建議未來可以先重新排序、篩選段落
> - 實驗用的 code-davinci-002 已被 OpenAI 停用，GPT3 部分難以重現（Flan-T5 的實驗可以）
> - 論文正文說最小模型是 0.2B，圖表標示為 0.3B，兩處不一致

> [!TIP]
> **可以帶走的思路**
> - **RAG 系統加「推理後再查」**：問題需要串多份文件時（例如「某條法規適用的設備，上次檢修是什麼時候」），讓 LLM 先寫出第一步推理，再拿推理句去查，而不是只用原問題查一次
> - **推理句是比原問題更好的查詢**：它帶出了原問題裡沒有的關鍵詞（例如 Mack Rides），這點對關鍵字檢索特別有幫助
> - **先用小模型試**：3B 模型加好的檢索就能贏過大模型，地端部署很有吸引力
> - **控制成本**：作者建議的方向是「動態決定什麼時候需要再查」，不是每句都查

---

## 7. 我的筆記

### 重點筆記
>

### 可以借用的地方
> 國家災防中心的 RAG 問答，遇到需要跨文件的題目時，能不能改成 IRCoT 式的「推理一句、檢索一次」？搭配 TextGrad 自動調推理和檢索的提示詞？

### 疑問
> 換成向量檢索（dense retrieval）後，推理句當查詢的優勢還那麼大嗎？中文資料、多份長文件時，15 段的上限夠不夠？

### 相關
- 概念：[[Retrieval-augmented generation|檢索增強生成]]、[[Multi-hop question answering|多步驟問答]]、[[Chain-of-thought prompting|思維鏈提示]]、[[BM25]]、[[Compound AI system|複合 AI 系統]]
- 相關論文：[[【Optimizing generative AI by backpropagating language model feedback】]]（可以用來自動優化 IRCoT 這類多步驟系統的提示詞）
- 後續論文：[[【PlanRAG：A Plan-then-Retrieval Augmented Generation for Generative Large Language Models as Decision Makers】]]（把迭代式 RAG 延伸到決策問題，加上先規劃、再重新規劃）
- 相關論文：[[【PixelRAG：Web Screenshots Beat Text for Retrieval-Augmented Generation】]]（把檢索和閱讀都改成網頁截圖，保留表格與版面）
- 後續論文：[[【Only Ask What You Don't Know：Grounded Delta Planning for Efficient Multi-step RAG】]]（GDP-RAG，計畫式 RAG：先檢索再規劃、只問缺的資訊；IRCoT 是它的比較方法之一）
- 相關論文：[[【SAG：SQL-Retrieval Augmented Generation with Query-Time Dynamic Hyperedges】]]（SAG：沿用 IRCoT／HippoRAG 2 的評測協定，改用事件–實體索引與 SQL JOIN 串起多步驟證據）
- 同期做法：ReAct（Yao et al., 2022）、SelfAsk（Press et al., 2022）、DecomP（Khot et al., ICLR 2023）、DSP（Khattab et al., 2023）
- 資料集：HotpotQA、2WikiMultihopQA、MuSiQue（同一作者）、IIRC
