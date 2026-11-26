---
title: "【Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers】"
tags: [paper, latent-reasoning, looped-transformer, chain-of-thought, status-inbox]
concepts: ["[[Latent reasoning]]", "[[Looped Transformer]]", "[[Chain-of-thought prompting]]", "[[Knowledge distillation]]", "[[Transformer]]"]
papers: ["[[【Attention is all you need】]]", "[[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]"]
---

# 【Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers】

<center><b>表 A：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 論文 | Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers |
| 作者 | Ying Fan、Anej Svete、Kangwook Lee |
| 單位 | Microsoft Research；ETH Zürich；KRAFTON；Ludo Robotics。Fan 的部分工作於 UW–Madison 完成 |
| 發表狀態 | arXiv 預印本，尚不以已正式接受的會議論文稱呼 |
| 版本 | arXiv:2606.31779v2，2026 年 7 月 13 日；初稿 2026 年 6 月 30 日。本筆記讀完整份 30 頁，含附錄 A–F |
| 連結 | [原文摘要](https://arxiv.org/abs/2606.31779v2)；[PDF](https://arxiv.org/pdf/2606.31779v2)；[HTML](https://arxiv.org/html/2606.31779v2)；[程式碼](https://github.com/yingfan-bot/lotus) |
| 引用（IEEE） | Y. Fan, A. Svete, and K. Lee, “Bridging the gap between latent and explicit reasoning with looped transformers,” 2026, *arXiv:2606.31779*, ver. 2, doi: 10.48550/arXiv.2606.31779. |
| 與研究主題相關度 | 4 / 5：高效率 LLM 推理與中間監督設計可借用；需要另外訓練，不是可直接取代 RAG 檢索的提示詞技巧 |
| 程式稽核範圍 | 公開儲存庫 commit `eb77e2f7909c5006f58ff0ad7cd6629b942caa9e`，僅靜態閱讀；沒有執行作者程式、安裝環境、下載模型或重跑訓練 |
| 圖像來源 | 下列四圖直接引用已核對的 arXiv v2 原圖 URL，白底顯示，不另行重製散布。arXiv non-exclusive distribution license 不等於 CC-BY |

> [!NOTE]
> LOTUS 把「逐字寫出推理」改成「在固定的連續向量工作區反覆計算」，再用每個標準推理 token 直接教導各位置；在 3B 算術模型上接近顯式 CoT 的答題率，但速度增益必須分清思考階段與整體流程。

## 1 背景：潛在推理為何省了文字，卻未必省時間或保住準確率？

### 1.1 任務與基本名詞

研究任務是數學文字題：輸入問題 Q，輸出數字答案 A。思維鏈（chain-of-thought，CoT）是答案之前的中間步驟，例如 `16−3−4=9 → 9×2=18`。本文區分兩種表示：

- **顯式 CoT**：真的把步驟寫成離散 token。訓練時可用 teacher forcing，也就是一次提供正確前文並平行計算所有位置的損失；推論時沒有正確前文，只能一個 token 接一個 token 產生。
- **潛在推理**：把中間計算放在隱藏向量（latent）裡，不必每一步都選出文字 token。這不代表自動得到低延遲：如果仍逐一產生 latent，依序執行模型的瓶頸仍在。

### 1.2 作者指出的兩個問題

第一是**序列瓶頸**。Coconut、CODI、SIM-CoT 仍以前一個 latent 產生下一個 latent。第二是**缺少逐位置的 CoT 對齊**。只教最後答案，或對齊教師的某個隱藏狀態／KV cache，不等於直接要求第 i 步的第 j 個位置對應標準解法的 token。

Coconut 用答案損失與逐步替換課程；CODI 加入教師隱藏狀態蒸餾；SIM-CoT 以輔助解碼器重建步驟。PCCoT 與 KaVa 採 Jacobi 類反覆更新來平行化 latent，但監督分別偏向隱藏狀態或壓縮 KV cache 的間接對齊。作者的問題是：能否同時使用平行工作區與直接、細粒度的文字監督，讓潛在方法在 3B 規模不再大幅落後顯式 CoT？

### 1.3 與其他加速方法的界線

Looped Transformer 重複使用同一組模型權重，增加運算深度，不是每圈新增一套模型。Padding／pause tokens 提供額外計算位置。LOTUS 結合兩者，但不是 speculative decoding：後者仍生成並驗證顯式文字序列；LOTUS 則不在推論時輸出或驗證離散推理鏈。附錄 A 也討論 diffusion、adaptive depth、latent search；這些是相關脈絡，不是本論文已測試的整合功能。

## 2 核心想法：把步驟攤成工作區，再反覆精煉

把問題想成前方的題目板，中間擺 K 個步驟格，每格 c 個連續向量位置，最後才寫答案。不是先寫完第一格才開始第二格，而是在同一輪模型運算中更新所有格，反覆做 R 輪精煉。

- K：可容納的步驟數上限，主設定是 6。
- c：每一步的向量位置數；Llama 算式版本用 25，GPT-2 用 13。
- R：精煉迴圈深度，主設定為 6。它不是答案 token 數，也不等於工作區寬度。
- S、N：每題原始 CoT 的步驟數與總 token 數，隨題目改變；N 不是 Kc。Nstep 則是實際納入步驟損失的非 padding token 數，截斷、課程階段或長尾處理都可能使它與原始 N 不同。
- Kc：向量工作區總位置數；算式 Llama 是 150，自然語言版本是 300。

訓練時，各位置透過原本的 LM head（把隱藏向量轉成詞彙機率的輸出層）對應 gold CoT token；推論時不必把這些向量轉成推理文字，答案直接以連續 latent 為條件產生。可讀出的步驟是診斷工具，不是解答管線的必要中介。

<center>
<img src="https://arxiv.org/html/2606.31779v2/lotus_figure.svg" alt="LOTUS 的迴圈計算與最終答案監督" width="100%" style="background:#fff">
<br>
<b>圖 1：平行 latent 迴圈與答案生成（原論文 Figure 2）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**：藍色長條是共享的基礎模型，黃色位置代表連續隱藏狀態，灰色方塊是文字目標。左圖把迴圈後的表示投影到 gold CoT token；右圖是產生答案的最後 forward。圖是概念圖，不足以決定實作中的 causal mask、BoT 是否重算，以及迴圈總呼叫數；這些細節見第 3 節。

## 3 技術細節：工作區、迴圈、監督與程式實作

### 3.1 輸入與步驟對齊

輸入形式為：

```text
Q | BoT | block 1：lat × c | … | block K：lat × c | EoT | A
```

BoT／EoT 是開始／結束潛在思考的特殊 token。`lat` 是可學習且各位置共用的 token，不是 Kc 個互不相關的參數向量；位置編碼與上下文使各位置的表示不同。K 是固定預算，S 才是每題原有步驟數。作者稱 K=6 覆蓋約 99% 的 GSM8K-Aug 標準解法；超出 K 的步驟會留下需要自回歸完成的尾段。因此「只逐字解碼答案」描述主要覆蓋情形，不是任何長度問題的無條件保證。

### 3.2 有原始輸入注入的遞迴

依式 (2)，先算 prefix cache Cpre，再得到：

$$h^{(0)}=f_\theta(E\mid C_{pre}),\qquad h^{(t)}=f_\theta(E+h^{(t-1)}\mid C_{pre}),\ t=1,\ldots,R.$$

每次重用同一模型，把原始 latent embedding E 加回上一輪狀態，避免把更新誤解為單純 `h ← f(h)`。梯度穿過展開的迴圈，並非只有最後一圈被訓練。

> [!WARNING]
> <b>R=6 不等於只有 6 次 latent-region forward。</b>式 (2) 先有一次 h⁽⁰⁾，再做 1…R 的 refinement；公開程式也先跑 initial forward，再 `range(n_looped_iters)`。因此主設定有 1+6 次工作區模型評估，另有 prefix／suffix 工作。這不推翻實測延遲，但不能把示意中的「6 輪」直接當成總 forward 次數。

### 3.3 兩種互補損失

步驟損失（式 3）為：

$$L_{step}=\frac{1}{N_{step}}\sum_{i=1}^{K}\sum_{j=1}^{c}\mathrm{CE}(f_{head}(h^{(R)}_{i,j}),T_{i,j}).$$

CE 是交叉熵，鼓勵 gold token 的機率提高；式 (3) 將每步 Ti **padding 或截斷至 c tokens**，padding 不計入損失與 Nstep。這個定義不表示公開程式在所有路徑都明確執行相同截斷，實作差異見表 1。直接 LOTUS 在**最終迴圈**後讀出所有步驟，不是每圈都固定一個步驟後不再更新。答案損失 Lans 用 EoT 的輸出預測第一個答案 token，後續用標準 next-token prediction；總損失為：

$$L=L_{ans}+\lambda_{step}L_{step}.$$

算式設定 λstep=0.05，自然語言設定 0.033。這是兩個各自正規化後的損失權重，不可解讀成「只有 5% 的訓練 token 是推理」或「推理只占 5% 運算」。LOTUS+CODI 再加權重 1.0 的教師蒸餾項，是不同模型變體。

### 3.4 PCL 不等於各位置完全不互動

Parallel chain likelihood（PCL）把讀出機率寫成各位置機率的乘積，而非依賴前面 gold token 的自回歸分解。**機率目標逐位置分解，不代表隱藏狀態獨立**：Transformer 在共同工作區計算，且答案損失會回傳跨位置的全局需求。

作者用「Lstep 覆蓋正確 token 的支持集、Lans 選擇能支持答案的聯合狀態」解釋兩者分工。支持集包含關係是合理的形式分析，但並不是已證明模型必然學得正確演算法或全域最優；表 11 才提供實驗上的互補證據。

附錄 E 指出，在相同 latent 條件下，額外給先前 gold token 不會提高最佳可達的條件熵，因此 PCL 與自回歸 NLL 的理論下界不同。不能把兩種分解的 NLL 直接當成同一尺度的排名。

### 3.5 LOTUS-aux：同一推理骨架，不同監督路徑

LOTUS-aux 加入和基礎模型同尺寸的輔助解碼器 gφ。第 t 圈讀第 t 個 block 的 c 個 latent，再以 teacher forcing 預測該步文字；最後一個 latent 預測該步第一個 token，其後由 gold token 預測下一個 token。這是帶自回歸條件的訓練目標，但 teacher forcing 的 token 位置可平行評分。預設 auxiliary schedule 是**每圈讀一個 block**，和直接 LOTUS 的 post-loop 監督不同。

輔助解碼器是基礎模型的完整可訓練副本，兩者共同訓練；推論時移除 auxiliary decoder。這帶來訓練參數與記憶體成本，不能把「推論不多一個模型」說成「訓練零額外成本」。

**Gold／teacher 資訊出現在哪裡？**直接 LOTUS 的 gold CoT 供計算訓練 loss，不是飽和課程推論的輸入；LOTUS-aux 的 gold prefix 只在 teacher-forced 訓練與表 9／10 明列的 TF 診斷中使用；+CODI 的 teacher hidden states 是額外訓練蒸餾訊號。一般答案推論只給問題、特殊 token 與 latent workspace，不讀取 gold CoT、gold answer、teacher checkpoint 或 auxiliary decoder。課程尚未替換的 CoT 尾段是**訓練可見文字**，不能拿來暗示測試時取得答案標籤。

<center>
<img src="https://arxiv.org/html/2606.31779v2/dslt_aux_figure.svg" alt="LOTUS-aux 的 teacher-forced 輔助解碼器" width="70%" style="background:#fff">
<br>
<b>圖 2：用額外解碼器監督第 t 步（原論文 Figure 3）</b>
</center>

> [!NOTE]
> **圖 2 怎麼看**：底端先放該 block 的向量，再接正確文字前綴；頂端是下一個 gold token。這不是推論時先把 latent 全部翻成自然語言，再把文字交回主模型。

### 3.6 靜態程式核對：不能由概念圖推定的細節

以下針對固定 commit 的公開實作，不宣稱完全重現論文執行環境：

<center><b>表 1：論文表述與公開實作的對照</b></center>

| 項目 | 核對結果 | 判讀 |
|---|---|---|
| 共享權重與注入 | `scripts/lotus.py` 629 行是 `original_embeddings + hidden_states` | 沒有除以 2；595 行舊 docstring 的 `/2` 不符合現行運算 |
| Attention | forward 傳入一般 padding mask 給 GPT-2／Llama causal LM，未見改為雙向的自訂 workspace mask | 平行更新不等於每個位置能在同一層看未來位置；不可畫成雙向全連接 |
| BoT／prefix | `loop_start=earliest_latent_start−1`，prefix cache 到 BoT 前，BoT 包含在每圈 region | 與 §3.2「先 cache [Q,BoT]」不完全相同；EoT 不在 loop region |
| 總迴圈評估 | initial region forward + R 次 refinement | 與式 (2) 一致，但正文／圖的簡稱容易少算一次 |
| 步驟 CE 位置 | post-loop `block_start=i*c`，region 的第 0 個位置是 BoT，actual latent 開始於第 1 個位置 | 實作是一格左移的 next-token 式對齊；不完全是式 (3) 的同位置 hᵢⱼ 讀出 |
| padding／suffix 損失 | step CE 忽略 pad id；main CE 忽略 −100，未被替換的 CoT 尾段仍在 labels | 課程未飽和時 main loss 包含可見 CoT+答案，不能全程只叫 answer-only loss |
| 最終條件 | 有效 KV-cache 路徑重用最後一次 loop forward 的 KV，僅處理 suffix | 最後注入到 `inputs_embeds` 的 h 不會在此路徑再完整重算 latent KV；不同於無 cache 全序列重算 |
| 長步驟 | collator 保留每步完整長度；post-loop CE 依 step_len 切 logits，未在該路徑明確限制 step_len≤c | 若預處理沒有保證每步不超過 c，可能跨 block 切到下一區；這是需核對資料的條件性風險，非已證實結果失效 |
| 深度／寬度解耦 | `eval.py` 預設 latent 數為 R*c，另提供 `--n_latent_override` | 重現固定 Kc 的 R sweep 必須額外固定 latent 數；預設改 R 會同時改寬度 |
| 延遲分界 | eval 另列 eot-marker 時間，LOTUS thought timer 在 suffix forward 前結束，answer timer 在 forward 返回後開始 | 公開 phase sum 不能自動視為完整 wall-clock；表 2 是否已包含 marker／suffix 成本需作者執行紀錄確認 |

> [!WARNING]
> 上述是**靜態可見的論文—程式差異與需確認項目**，不是模型重現失敗的結論。特別是 shifted CE、BoT cache 邊界與最終 KV 使用，應在方法動畫與重現規格中註明，不以概念圖替代實際運算。公開訓練設定與論文基本一致，但該 snapshot 未提供每項消融與自然語言設定的完整配置。

## 4 實驗設定：資料、規模、指標與公平比較

### 4.1 資料與模型

使用 GSM8K-Aug 訓練，正文稱約 385k，附錄 F.2 指定 **385,620** 個訓練樣本，是原 GSM8K 訓練題目的擴增解法。域內測試用 GSM8K，latent token 分析明列 1,319 個測試問題；域外用 GSM-Hard、MultiArith、SVAMP。OOD Average 是三個資料集準確率的**等權平均**，不是把所有題目合併後的加權準確率。

模型只有 GPT-2 124M、Llama-3.2-1B-Instruct、Llama-3.2-3B-Instruct 三種。本文的「scale」到 3B，不能擴張成已證實於 7B、70B 或通用大型推理模型。

<center><b>表 B：兩種訓練設定的差異</b></center>

| 設定 | 算式主實驗 | 自然語言壓力測試 |
|---|---|---|
| CoT 表達 | 簡短算式 | 完整句子推理 |
| 訓練資料 | GSM8K-Aug | 自然語言版本 GSM8K-Aug |
| 模型 | 124M、1B、3B | 3B |
| K、R | 6、6 | 6、6 |
| c | GPT-2：13；Llama：25 | 50 |
| latent 位置 | GPT-2：78；Llama：150 | 300 |
| λstep | 0.05 | 0.033 |

兩個 headline speedup 是**兩個分別訓練的設定**，不是同一個 checkpoint 任意切換文字長短就量得的連續區間。

### 4.2 課程與最佳化

由顯式 CoT checkpoint 初始化，分階段每一 epoch 多替換一個 gold step 為 latent block；第 0 階段是標準 CoT，最後到 K=6。尚未替換的步驟保持可見。預設每 block 一圈 refinement，消融再嘗試解耦深度。

全參數 AdamW，30 epochs，總 batch size 128，梯度範數裁切 1.0。GPT-2／1B learning rate 為 10⁻⁴，3B 為 5×10⁻⁵；Llama 用 bf16，GPT-2 fp32。以 GSM8K validation accuracy 選 checkpoint；論文稱最後一個 epoch 也相近。主結果報 3 seeds 的 mean±std；但引入的 baseline 不一定逐列附 seed 變異，不能假定每列都有同一重跑協定。

### 4.3 比較預算與延遲定義

主表 CODI／CODI+SIM-CoT 與 LOTUS 的序列步驟名目預算都是 6，但每步 latent 寬度與額外架構不同，不能稱總 FLOPs 完全相等。附錄 C 的 Coconut 系列用 10 個循序 latent，PCCoT／KaVa 用 24 個 latent、3 次 Jacobi 迭代；其數字引自相關工作。SIM-CoT 還加 rank-128 LoRA，會影響不只 thought 的各階段成本。

延遲使用單張 NVIDIA H100（正文註腳更精確為 H100 NVL）、batch size 1、greedy decoding，報平均每題 ms。表 2 分 query prefill、thought、answer：prefill 建立問題 cache；thought 是文字 CoT 或 latent 更新；answer 是短答案生成。這不是批次吞吐量測試，也沒有能耗、GPU 記憶體或訓練時間的完整比較。表 3 自然語言只報 thought，不提供完整三階段延遲。

## 5 結果：準確率、延遲、消融與 latent 讀出

### 5.1 規模增大時，plain LOTUS 接近但沒有超過域內 CoT

<center>
<img src="https://arxiv.org/html/2606.31779v2/teaser_figure.svg" alt="不同模型大小及自然語言設定的準確率比較" width="100%" style="background:#fff">
<br>
<b>圖 3：跨規模與 CoT 形式的答題率比較（原論文 Figure 1）</b>
</center>

> [!NOTE]
> **圖 3 怎麼看**：左圖橫軸是模型大小，縱軸是 GSM8K accuracy；右圖是 3B 自然語言設定，不是另一組模型大小。LOTUS 在左圖追近 Explicit CoT，不表示每個規模都最高：GPT-2 的 PCCoT 為 49.5%，高於 LOTUS 44.1%。精確值以原表 1、13、14 為準。

<center><b>表 2：跨規模準確率，原論文 Table 1 完整 24 列，單位 %</b></center>

| 模型 | 方法 | GSM8K | GSM-Hard | MultiArith | SVAMP | OOD 平均 |
|---|---|---:|---:|---:|---:|---:|
| GPT-2 | Explicit CoT | 42.7 | 9.0 | 85.0 | 41.6 | 45.2 |
| GPT-2 | No-CoT | 19.1 | 4.3 | 41.1 | 16.4 | 20.6 |
| GPT-2 | CODI | 42.0 | 9.4 | 93.0 | 41.7 | 48.0 |
| GPT-2 | CODI+SIM-CoT | 42.6 | 9.4 | 92.8 | 42.6 | 48.3 |
| GPT-2 | LOTUS | 44.1±0.7 | 9.5±0.2 | 92.4±1.4 | 41.8±0.9 | 47.9 |
| GPT-2 | LOTUS+CODI | 43.1±1.2 | 9.8±0.5 | 90.0±4.3 | 41.6±0.4 | 47.1 |
| GPT-2 | LOTUS-aux | 35.5±4.5 | 8.2±1.2 | 82.0±2.8 | 35.5±1.5 | 41.9 |
| GPT-2 | LOTUS-aux+CODI | 38.1±3.7 | 8.6±0.5 | 85.7±4.1 | 36.1±2.6 | 43.5 |
| 1B | Explicit CoT | 58.4 | 13.9 | 96.7 | 65.7 | 58.8 |
| 1B | No-CoT | 28.8 | 6.3 | 50.3 | 26.7 | 27.8 |
| 1B | CODI | 52.7 | 11.9 | 95.0 | 60.6 | 55.8 |
| 1B | CODI+SIM-CoT | 56.1 | 12.7 | 96.2 | 61.5 | 56.8 |
| 1B | LOTUS | 57.3±0.2 | 12.7±0.5 | 98.3±2.0 | 60.9±0.6 | 57.3 |
| 1B | LOTUS+CODI | 58.3±0.8 | 12.7±0.2 | 98.9±0.6 | 61.1±0.9 | 57.6 |
| 1B | LOTUS-aux | 55.4±0.2 | 12.6±0.2 | 97.8±0.0 | 58.0±0.6 | 56.1 |
| 1B | LOTUS-aux+CODI | 50.6±4.7 | 11.4±0.8 | 95.7±1.2 | 55.0±3.6 | 54.0 |
| 3B | Explicit CoT | 71.5 | 17.0 | 98.3 | 71.0 | 62.1 |
| 3B | No-CoT | 38.3 | 9.5 | 88.7 | 52.9 | 50.4 |
| 3B | CODI | 60.8 | 14.3 | 98.7 | 73.3 | 62.1 |
| 3B | CODI+SIM-CoT | 62.3 | 14.6 | 98.8 | 74.9 | 62.8 |
| 3B | LOTUS | 70.0±0.9 | 16.0±0.5 | 99.9±0.3 | 75.7±0.8 | 63.9±0.3 |
| 3B | LOTUS+CODI | 70.6±0.2 | 16.3±0.3 | 99.6±0.3 | 74.4±1.4 | 63.4±0.3 |
| 3B | LOTUS-aux | 69.9±0.9 | 16.6±0.5 | 99.8±0.3 | 74.6±2.2 | 63.7±0.8 |
| 3B | LOTUS-aux+CODI | 70.5±0.7 | 16.5±0.4 | 99.4±0.6 | 73.9±0.1 | 63.3±0.4 |

plain LOTUS 相對 Explicit CoT 的 GSM8K 差為 GPT-2 +1.4、1B −1.1、3B −1.5 個百分點。3B 比 CODI+SIM-CoT 高 7.7 個百分點；後者比 CoT 低 9.2 點。3B LOTUS 的 OOD 平均高 1.8 點，但個別資料集不是全勝：GSM-Hard 低 1.0 點、MultiArith 高 1.6 點、SVAMP 高 4.7 點。

LOTUS-aux 在 3B 與 LOTUS 接近，在 GPT-2 明顯下降且 seed 標準差大；加 CODI 並非單調改善，1B aux+CODI 的 GSM8K 僅 50.6±4.7。直接 head 的設計較穩健，但不能把單一消融解釋成所有規模的普遍法則。

> [!WARNING]
> <b>「within 1 point」需要變體限定。</b>plain LOTUS 的 70.0 對 71.5，差是 1.5 個百分點；LOTUS+CODI 的 70.6 才縮至 0.9。正文其他段落使用「within 1.5 points」較精確。這些都是百分點，不是相對百分比。
>
> 正文稱 KaVa 的差距由 1.9 到 5.8 點；原表 13 顯示 1.9 是 **1B** 的 58.4−56.5，不是 GPT-2，因為 KaVa 沒有 GPT-2 數字。3B 為 71.5−65.7=5.8。

### 5.2 速度：2.5 倍 thought 與 2.1 倍三階段合計是不同數字

<center><b>表 3：算式設定的延遲，原論文 Table 2，ms／題</b></center>

| 階段 | LOTUS | Explicit CoT | SIM-CoT | CODI |
|---|---:|---:|---:|---:|
| Query prefill | 16.7 | 15.9 | 29.5 | 16.5 |
| Thought | 133.0 | 338.8 | 162.7 | 88.2 |
| Answer | 31.5 | 29.5 | 59.2 | 30.4 |
| 三階段加總 | 181.2 | 384.2 | 251.4 | 135.1 |

獨立重算：338.8／133.0=2.547，對應約 2.5 倍 thought speedup；384.2／181.2=2.120，對應約 2.1 倍三階段合計 speedup。SIM-CoT thought／LOTUS thought=1.223，約 1.2 倍。CODI 本身更快，thought 為 88.2 ms，但 GSM8K accuracy 只有 60.8%，所以 LOTUS 的主張是效率與準確率的取捨，不是最快的 latent 方法。

<center><b>表 4：自然語言 CoT 壓力測試，原論文 Table 3</b></center>

| 方法 | GSM8K accuracy | Thought ms／題 |
|---|---:|---:|
| Explicit CoT | 68.41±0.59% | 963.6 |
| LOTUS | 68.13±0.77% | 140.8 |

差距為 −0.28 個百分點。相較短算式，顯式文字需要更多循序生成，而 latent 只改成較寬的工作區，因此加速更明顯；但這不是用完全相同寬度與 λ 的單一控制實驗。

> [!WARNING]
> <b>原文 6.9 倍與表列數值不能依一般四捨五入相符。</b>963.6／140.8=6.84375，一位小數應為 **6.8 倍**。即使兩個 ms 值各自只是四捨五入到小數一位，可能比值約落在 6.841–6.847，仍無法四捨五入成 6.9。可寫「原文稱 6.9 倍；表值重算約 6.84 倍」。表 3 未列 prefill 與 answer，**不能宣稱自然語言端到端加速 6.9 倍**，也不能把算式表的其他階段補進去。
>
> 表 2 合計可完整相加，但公開 eval 另列 marker 時間，且 cached suffix forward 與 timer 邊界仍需確認。因此 2.1 倍應寫為「論文表列三階段合計」，不是已獨立驗證的應用端到端 wall-clock。

自然語言域外結果（原表 14）：LOTUS 在 GSM-Hard 為 16.27±0.19，低於 CoT 的 18.27±0.85；SVAMP 為 73.40±0.35，高於 CoT 的 71.93±1.62。GSM8K 的 PCCoT、CODI、KaVa 分別為 47.6、55.9、60.0，LOTUS 68.13，但 baseline 引自相關工作，不能把它當作全部重新配對量測的速度比較。作者稱「statistically on par」，但未提供足以重建正式等價檢定的逐題配對結果與完整檢定規格；本筆記僅說平均值接近、變異範圍有重疊，不把不顯著等同已證明等價。

<center><b>附表 C1：不同序列預算的 GSM8K 準確率（原論文 Table 13，%）</b></center>

| 方法 | GPT-2 | Llama-3.2-1B | Llama-3.2-3B |
|---|---:|---:|---:|
| Explicit CoT | 42.7 | 58.4 | 71.5 |
| Coconut | 36.6 | 33.2 | 未報告 |
| Coconut+SIM-CoT | 44.8 | 42.2 | 未報告 |
| PCCoT | 49.5 | 54.2 | 54.7 |
| KaVa | 未報告 | 56.5 | 65.7 |
| LOTUS | 44.1 | 57.3 | 70.0 |

<center><b>附表 C2：自然語言設定的完整比較（原論文 Table 14，%）</b></center>

| 方法 | GSM8K | GSM-Hard | SVAMP |
|---|---:|---:|---:|
| Explicit CoT | 68.41±0.59 | 18.27±0.85 | 71.93±1.62 |
| PCCoT | 47.6 | 11.0 | 65.2 |
| CODI | 55.9 | 13.6 | 70.1 |
| KaVa | 60.0 | 14.8 | 66.1 |
| LOTUS | 68.13±0.77 | 16.27±0.19 | 73.40±0.35 |

兩表的 baselines 來源與序列預算不同，缺值不是 0；附表 C2 沒有 MultiArith，不補造 OOD 三資料集平均。

### 5.3 消融：寬度、深度與監督都重要

<center><b>表 5：監督設計，原論文 Table 4，GSM8K %</b></center>

| 監督 | 分數 |
|---|---:|
| 只 main／answer loss，無 latent 監督 | 63.3 |
| CODI 蒸餾 | 64.4 |
| 直接 head，每圈讀出 | 68.2 |
| 直接 head，最後一圈讀出（LOTUS） | 70.0 |
| Auxiliary decoder，每圈讀出（LOTUS-aux） | 69.9 |
| Auxiliary decoder，最後一圈讀出 | 68.4 |

直接 head 的 post-loop 比 per-iter 高 1.8 點，auxiliary 的 per-iter 比 post-loop 高 1.5 點；最佳 schedule 因路徑不同而異。LOTUS 比無 latent 監督高 6.7 點。作者推測 auxiliary 每圈監督縮短梯度路徑，這是解釋，不是額外的因果驗證。

<center><b>表 6：工作區寬度 c 與推論適應，原論文 Tables 7、8</b></center>

| c | Kc | 各自重訓後 accuracy | 同一 c=25 checkpoint 的推論 accuracy | 後者 thought ms |
|---:|---:|---:|---:|---:|
| 1 | 6 | 49.7 | 51.4 | 110.9 |
| 5 | 30 | 67.5 | 59.4 | 120.3 |
| 10 | 60 | 68.4 | 64.2 | 127.0 |
| 25 | 150 | 70.0 | 70.0 | 133.0 |
| 30 | 180 | 70.0 | 70.5 | 136.0 |
| 50 | 300 | 未列 | 70.5 | 141.2 |

同一 checkpoint 從 c=25 降到 1、5、10，分別掉 18.6、10.6、5.8 點；增加到 30 或 50 只提高 0.5 點。位置數從 6 到 300 是 50 倍，但延遲只增加 30.3 ms，約 1.273 倍；這是指定硬體／batch 下的平行效率，不代表 50 倍寬度完全免費，或 FLOPs／記憶體不增加。

<center><b>表 7：深度 R，原論文 Tables 5、6；兩欄不可混讀</b></center>

| R | 各自重訓，Kc=150 | 同一 R=6 checkpoint 改推論 R，Kc=150 |
|---:|---:|---:|
| 1 | 未列 | 22.7 |
| 2 | 14.6 | 40.0 |
| 3 | 23.2 | 55.0 |
| 4 | 52.6 | 63.5 |
| 5 | 68.1 | 68.7 |
| 6 | 70.0 | 70.0 |
| 7 | 未列 | 69.3 |

深度不是越多永遠越好：訓練範圍內到 6 持續改善，推論多跑第 7 圈反而略降。訓練消融與同 checkpoint 的 inference sweep 回答的是不同問題。公開配置也未完整實現附錄所說可調 r 的全部消融，重現需顯式解耦 R 與 K。

### 5.4 latent 的可讀性：讀得出 gold 訊號，但不是每題都有乾淨推理鏈

<center><b>表 8：gold token 讀出，原論文 Table 9</b></center>

| 讀出方式 | NLL↓ | Gold top-1 | Gold top-5 |
|---|---:|---:|---:|
| LOTUS h⁽ᴿ⁾→主 head，無 gold prefix | 3.07 | 70.9% | 85.8% |
| LOTUS-aux h⁽ᴿ⁾→aux head，無 gold prefix | 9.02 | 12.4% | 25.8% |
| LOTUS-aux decoder，free-running | 6.39 | 64.5% | 76.7% |
| LOTUS-aux decoder，teacher forcing | 1.56 | 84.3% | 93.9% |

Free-running 是讓輔助解碼器吃自己前一步的輸出；teacher forcing 則給正確前綴，後者有推論時不存在的資訊。70.9% 是 gold **token 位置**的 top-1，不是 70.9% 的題目有完整且忠實的解釋，更不是最終答案正確率。NLL 是負對數概似，越小代表對 gold token 越有把握，但跨 PCL／AR 分解只能參考。

附錄 F.1 的四個例子同時保留成功與失敗：

- 鴨蛋題：`(16−3−4)×2=18`；LOTUS 的早期 block 先顯示 `3+4=7`、`16−7=9` 等有效替代分解，但後面仍出現不正確或重複等式。
- 跑步題：`3×3×60=540`；讀出可走 `3×60=180 → 180×3=540`，但之後也有 `540×540=540` 等錯誤文字。答案正確不等於每一行讀出都正確。
- 下載題：已下載部分耗時 40 分鐘、完整重下 100 分鐘，加更新 20，共 160；兩模型最後答 180、120。正確中間數字曾浮現，仍未整合成正確答案。
- 吸塵器題：`5→10→12→18`，gold reference 少列最後逆推一步；兩模型最後答 30、36。資料的 gold trace 本身也可能不完整。

### 5.5 未見替代路徑：訓練題上的數值探針，不是未見題目的多路搜尋證明

附錄 F.2 的 path bank 建自**原 GSM8K 訓練題**：7,444 題、58,155 條驗證路徑，每題最多 10 條，來自 Llama-3.1-8B 的 rejection sampling。所謂 unseen 是「該替代解法未出現在訓練 trace」，不是問題本身沒見過。

先找 G（只在訓練路徑中的中間數字）與 U（只在替代路徑中的中間數字），排除題目、答案與兩路共有數字；要求每邊至少兩個 disjoint intermediates 得 653 題，再要求 U 不與已訓練／題目／答案數字共用 sub-token，剩 341 題。最終包含 G 871、U 974、Random 974 個數字；Random 與 U 的數量與單／多 token 組成都匹配。

指標不是整條解法機率：每個數字在所有 latent 位置滑動對齊，取**最佳位置**的每 token NLL；再先在每題的數字集合內平均，最後跨題平均。Top-k 也是「任一位置找到該數字」的命中率，多 token 需連續匹配。因此 64.0% 不是 974 個數字直接等權加總後的全域分數，也不是完整替代鏈成功率。

<center><b>表 9：替代路徑的中間數字，原論文 Table 10</b></center>

| Reader | G NLL | U NLL | Random NLL | U top-1 | U top-5 |
|---|---:|---:|---:|---:|---:|
| LOTUS，主 head | 0.07 | 4.28 | 8.16 | 15.3% | 64.0% |
| LOTUS-aux，直接 aux head | 3.30 | 6.04 | 9.42 | 14.5% | 49.3% |
| LOTUS-aux，FR decoder | 0.80 | 12.62 | 20.39 | 5.6% | 39.2% |
| LOTUS-aux，TF decoder | 0.18 | 13.17 | 20.83 | 1.9% | 37.4% |

四種讀出都有 G NLL < U NLL < Random NLL；這支持表示對有效替代中間值不是完全隨機，但沒有證明模型同時保存完整多條演算法、進行顯式搜索、或這些讀出值對答案有因果必要性。

<center>
<img src="https://arxiv.org/html/2606.31779v2/multipath_grid6.svg" alt="訓練路徑、未見有效中間數字與隨機控制的讀出機率" width="95%" style="background:#fff">
<br>
<b>圖 4：同一道巴士路程題中的替代中間數字（原論文 Figure 4）</b>
</center>

> [!NOTE]
> **圖 4 怎麼看**：各小圖橫軸是 latent position，縱軸是該數字的讀出機率。上方是訓練路徑數字，中間紅色列是 unseen-only 的 24、18，下方灰色列是 random 68、26。尖峰代表該數字在某個位置有明顯機率，不是模型真的輸出完整替代推理。精確 0.77 與 0.14 取自附錄文字，不由圖形估讀。

巴士例子可文字重現：

```text
訓練路徑：3×2=6；6×12=72；2×2=4；4×9=36；72+36=108
替代路徑：2×12=24；24+24+24=72；2×9=18；18+18=36；72+36=108
```

24 的讀出機率為 0.77、top-1、位置 142；18 為 0.14、top-2、位置 116。這兩個數字既不在訓練 reference chain，也不在題目或最終答案。不過原圖為說明而畫出的 72、36、108 不能直接當成 only-G 集合；附錄給的 onlyG 是 {3,4,6}、onlyU 是 {24,18}。

### 5.6 兩個損失共同作用

原表 11：完整 LOTUS 的 gold NLL／top-1／top-5 為 3.07／70.9%／85.8%；只 Lstep 是 9.29／9.1%／17.1%；只 Lans 是 5.97／9.4%／26.5%。這是 gold-chain readout 的消融，不應誤寫成三者的 answer accuracy。即使只保留 token 對齊 loss，也沒有得到好的測試集 gold 讀出，說明這個架構下兩個目標需要共同作用；不宜只用「逐位置 CE 本來就會重建完整 chain」帶過。

## 6 總結：值得保留的設計與不能跨過的證據界線

LOTUS 的貢獻不是發明「向量思考」這個概念，而是把**固定步驟／token 工作區、共享權重反覆精煉、主 head 的 post-loop gold token CE、最後答案監督**組成一個有效配方。3B 主結果顯示域內差距縮到 1.5 點，加入 CODI 後縮到 0.9 點；域外等權平均高於 CoT，但困難題集仍落後。

效率方面，算式設定 thought 約 2.55 倍、表列三階段合計約 2.12 倍；自然語言設定 thought 由表值重算約 6.84 倍。結果只支持指定數學資料、模型、硬體與 batch 的觀察，沒有證明在通用任務、長程工具使用、RAG 或工業部署上都有同樣提升。

> [!TIP]
> 可以借用的是「讓內部表示有可核對的中間監督，同時以最終任務損失約束整合」的設計原則。若應用在規則計算、排程或最佳化，可比較只監督答案、只監督中間結構、兩者並用；但必須先有可信的步驟標註，並分別衡量推論延遲、訓練成本與最終誤差，不能直接把本論文的 speedup 搬過去。

主要限制包括：只驗證數學題與最高 3B；固定 K/c/R，長鏈需尾段自回歸；訓練依賴大量 CoT trace 與先前 CoT checkpoint；auxiliary 訓練成本高；readout 可讀性不等於 faithful explanation；替代路徑探針來自訓練題；缺少正式等價檢定與應用級 wall-clock、吞吐／記憶體／能耗稽核。公開程式與論文在位置對齊與 cache 語意上的差異，也應納入後續重現。

### v1 → v2：哪些真的變了？

兩份 HTML 的 14 張編號表已逐格比較：**所有數值（含 std 與缺值位置）未改**，不能把 2.5 倍或 6.9 倍解釋成 v2 新實驗的提升。主要差異如下：

- §4.2 自然語言結果敘述中的 `68.6%` 修為 `68.13%`；v1 的表 3／14 原本就列 68.13，因此是敘述校正，不是準確率變低的新結果。
- 附錄 D 新增自然語言訓練設定：c=50、λstep=0.033，以及從相同資料版本的 CoT checkpoint 初始化。這是 v2 才明確揭露的重現細節。
- 附錄 A 更清楚區分 MUX 的詞彙空間疊加 KL 目標與 LOTUS 的逐位置 CE，加入 GRAM 與 speculative decoding 討論；另補致謝、附錄總覽與文字／引用整理。
- 表 4／14 的部分結果加粗，表 12 修改用語；不是數字更新。兩版皆保留 h⁽⁰⁾ 再加 R 輪的式 (2)、`within 1 point` 貢獻敘述與 6.9 倍宣稱；本筆記的變體與四捨五入警語不因升版而消失。

## 7 我的筆記

### 重點筆記

>

### 可以借用的地方

- **中間目標與終端目標分開設計**：把表示對齊與最終答案分工，避免誤把中間指標最佳化視為任務完成。
- **深度與寬度獨立量測**：固定 latent 位置改 R，或固定 R 改 c；比較重新訓練與推論調整，避免混在同一張曲線裡。
- **診斷與執行分離**：LM-head readout 可以幫助觀察，但不必塞回正式推論路徑；部署應另驗證其因果相關性。
- **延遲分母一致**：思考、prefill、marker、suffix、答案、前後處理、批次大小均需標清；不能用 thought 的倍數代替整個服務加速。

### 疑問與後續驗證

1. 使用相同輸入、相同 latency profiler、相同 EOS／marker 分界重新量測時，表 2 三階段合計與完整 wall-clock 差多少？自然語言設定的完整時間為何？
2. 式 (3) 同位置讀出與公開實作 shifted CE 若分別訓練，哪一種對應出版結果？BoT 在 cache 內或 loop 內的差異有多大？
3. 每步是否真的預處理成長度≤c？超長步驟與 S>K 的比例、處理方式及錯誤率是否有獨立統計？
4. 有效替代中間值的 readout 是否對答案有因果作用？若遮蔽或干預這些位置，正確率是否變化？
5. 在非數學任務、未見問題的多路分析，以及更大模型上是否仍有相同收益？提高 batch 後 speedup、吞吐與記憶體怎麼改變？
6. LOTUS-aux 的訓練額外 FLOPs／記憶體／收斂穩定性，以及主 head 的可解碼性—任務表現取捨，是否值得其成本？

### 相關

- 主題相關度：對高效率語言模型推理與可解讀 latent 訓練高；對 RAG、預測或工業控制是設計啟發，沒有直接驗證的遷移成果。
- [[Latent reasoning|潛在推理]]：不經離散文字的中間計算。
- [[Looped Transformer|迴圈 Transformer]]：共享模型參數、以反覆運算增加有效深度。
- [[Chain-of-thought prompting|思維鏈]]：直接 gold token 監督的來源與主要比較基準。
- [[Knowledge distillation|知識蒸餾]]：CODI、KaVa 的間接監督與 LOTUS 的對比。
- [[Transformer]]：平行處理與 causal attention 的架構基礎。
- [[【Attention is all you need】]]：架構基礎對照；LOTUS 的迴圈與監督需另外訓練，不由原始架構自動得到。
- [[【Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions】]]：顯式推理與外部檢索的對照；本文沒有驗證 latent 能替代外部證據。

### 來源（IEEE）

[1] Y. Fan, A. Svete, and K. Lee, “Bridging the gap between latent and explicit reasoning with looped transformers,” 2026, arXiv:2606.31779, ver. 2, doi: 10.48550/arXiv.2606.31779. [Online]. Available: https://arxiv.org/html/2606.31779v2

[2] Y. Fan, A. Svete, and K. Lee, “Bridging the gap between latent and explicit reasoning with looped transformers,” 2026, arXiv:2606.31779, ver. 1. [Online]. Available: https://arxiv.org/html/2606.31779v1

[3] Y. Fan, “LOTUS,” GitHub repository, commit eb77e2f7909c5006f58ff0ad7cd6629b942caa9e. [Online]. Available: https://github.com/yingfan-bot/lotus/tree/eb77e2f7909c5006f58ff0ad7cd6629b942caa9e
