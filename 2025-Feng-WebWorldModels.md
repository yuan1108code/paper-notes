---
title: "【Web World Models】"
tags: [paper, llm, agent, world-model, status-inbox]
concepts: ["[[World model]]", "[[Neuro-symbolic AI]]", "[[Procedural generation]]", "[[Constrained decoding]]", "[[Retrieval-augmented generation]]", "[[Agent memory]]", "[[ReAct]]"]
papers: []
---

# 【Web World Models】

<center><b>表 1：論文基本資訊</b></center>

| 項目 | 內容 |
|---|---|
| 作者 | Jichen Feng（Princeton、UPenn）、Yifan Zhang（Princeton）、Chenggong Zhang（UCLA）、Yifu Lu（Princeton），以上四位共同一作；Shilong Liu、Mengdi Wang（通訊作者，Princeton） |
| 發表 | arXiv 2512.23676 預印本，尚無正式發表。v1 2025/12/29；v2 2026/11/9 只改了排版和一處章節編號（「Section 3」改成「Section 2」），內容相同。本筆記依你給的 v1 |
| 連結 | [arXiv](https://arxiv.org/abs/2512.23676)・[PDF](https://arxiv.org/pdf/2512.23676v1)・[HTML v1](https://arxiv.org/html/2512.23676v1)・[專案頁](https://princeton-ai2-lab.github.io/Web-World-Models/)・[GitHub](https://github.com/Princeton-AI2-Lab/Web-World-Models) |
| 動畫解說 | [papers.poyuanchu.com/2025-feng-wwm/](https://papers.poyuanchu.com/2025-feng-wwm/) |
| 論文類型 | 立場／系統展示論文：提出一個架構概念，用 7 個網頁示範說明，**沒有任何量化實驗** |
| 狀態 | 待讀 |
| 與我的相關度 | 3 / 5（世界模型和遊戲不是我的主題，但「程式算事實、LLM 只負責寫敘述」「LLM 輸出要符合型別」「LLM 掛了要能退回範本」這三個原則，可以直接用在災防資訊系統） |
| 加入日期 | 2026-11-12 |

> [!NOTE]
> **一句話**：讓 LLM 代理人有一個可以長期活動的「世界」，現在只有兩種極端：傳統網站（資料庫固定，內容有限但可靠）和完全由模型生成的世界模型（內容無限但難控制）。這篇提出中間路線 **Web World Model（WWM）**：世界的狀態和規則（「物理」）用一般的網頁程式碼寫死，LLM 只在這個結構上**補內容**（描述、劇情、對話）。作者做了旅遊地圖、銀河探索、卡牌遊戲、煉金沙盒、太陽系、維基百科、無限書架共 7 個示範，歸納出四個設計原則。

---

## 1. 背景：代理人需要一個「世界」

### 什麼是世界模型

**世界模型**（world model）是一個能回答「我做了這個動作，世界會變成什麼樣子」的模型。Ha 與 Schmidhuber（2018）讓代理人在神經網路「夢出來」的環境裡學習；LeCun 主張自主智慧的核心就是會預測的世界模型；近年也有人用 LLM 模擬網頁操作的結果（WebDreamer），或用 LLM 同時當世界模型和推理者（RAP）。

### 兩個極端

<center>
<img src="https://arxiv.org/html/2512.23676v1/wwm-v5.png" alt="圖 1" width="80%">
<br>
<b>圖 1：Web 框架、Web World Model、世界模型的比較（原論文 Figure 1）</b>
</center>

> [!NOTE]
> **圖 1 怎麼看**
> 三欄各有一句對話框和三列打勾：
> - <b>Web 框架</b>（左，資料庫圖示）：「內容受限於資料庫」。無限內容 ✗、以文字／程式為基礎 ✓、可控 ✓
> - <b>Web World Model</b>（中，本篇）：「框架定規則，LLM 填內容」。三項全部 ✓
> - <b>世界模型</b>（右，模型生成整個世界）：「模型生成一切」。無限內容 ✓、以文字／程式為基礎 ✗、可控 ✗
>
> 這張圖是作者的定位圖，不是實驗結果；三項都打勾的中間欄是作者的主張，全文沒有量測「可控性」。

<center><b>表 2：三種做法的比較</b></center>

| | 傳統 Web 框架 | **Web World Model** | 完全生成的世界模型 |
|---|---|---|---|
| 世界狀態存在哪 | 資料庫 | **程式碼定義結構，需要時才生成內容** | 模型的潛在空間 |
| 內容量 | 開發者事先寫好的範圍 | 理論上無限 | 理論上無限 |
| 一致性、可除錯 | 高 | 高（規則在程式裡） | 低：容易幻覺、前後矛盾 |
| 例子 | 一般網站、資料庫 App | 本篇的 7 個示範 | Genie、影片生成式遊戲、Unbounded |

---

## 2. 核心想法：四個設計原則

### 2.1 物理和想像分開（Separation of Concerns）

類比遊戲裡「物理引擎」和「渲染」的分工，把世界狀態拆成兩部分：

$$S_t = (S_t^{\phi},\ S_t^{\psi})$$

- $S^{\phi}$ **物理層**：完全由程式決定。存背包、座標、資源上限，並且強制邏輯一致（鎖住的門不能穿過、錢不夠不能花）
- $S^{\psi}$ **想像層**：由 LLM 隨機生成。環境描述、NPC 對話、美術風格

狀態更新有固定順序：**先由程式算出結果，再讓 LLM 根據新狀態寫敘述**。

$$S_{t+1}^{\phi} = f_{\text{code}}(S_t^{\phi}, a_t), \qquad S_{t+1}^{\psi} \sim \pi_\theta(\cdot \mid S_{t+1}^{\phi})$$

$a_t$ 是使用者的動作，$f_{\text{code}}$ 是程式寫的規則，$\pi_\theta$ 是 LLM。LLM 只能「描述」程式已經決定好的世界，不能改變它。

<center><b>圖 2：WWM 架構（依原論文 Figure 3 以文字重現）</b></center>

```
使用者動作 a_t
   │ 觸發更新
   ▼
┌─────────────────────┐   新狀態 S^φ_{t+1}   ┌──────────────────────┐   敘述 S^ψ_{t+1}
│ 物理層 S^φ           │ ───────────────▶   │ 想像層 S^ψ             │ ─────────────▶ 呈現的世界
│ 確定性的程式           │                    │ 隨機的 LLM              │
│ 背包、地圖、邏輯        │                    │ 描述、對話、氛圍          │
│ 嚴格邏輯              │                    │ 創意層                  │
└─────────────────────┘                    └──────────────────────┘
   ▲                                                                     │
   └──────────────────────────── 迴圈：新狀態 ◀──────────────────────────┘
```

### 2.2 用型別介面當共同語言（Typed Interfaces）

一般深度學習把狀態存成看不懂的向量；WWM 改用**明確的資料型別**，例如：

```typescript
interface Planet { biome: string; hazard: string; }
```

LLM 不是產生像素或向量，而是產生**符合這個型別的 JSON**。如果 LLM 生成一個新道具，就必須填好物理引擎需要的欄位（重量、價格）。作者稱型別介面是「語法過濾器」，可以擋掉結構上的幻覺。實作上用的是 Gemini 的 `responseSchema` 和 OpenRouter 的 `response_format: json_object`，屬於受限解碼（constrained decoding）的應用。

### 2.3 用確定性雜湊產生無限世界（Deterministic Hashing）

無限的宇宙存不進資料庫，所以**需要時才生成**（just-in-time），用的是遊戲界常見的**程序生成**（procedural generation）：

1. 使用者走到座標 $x$，不查資料庫
2. 把座標丟進雜湊函數，得到一個整數種子 $h(x)$
3. 作者說：這個種子「固定了 LLM 的抽樣隨機性」，所以離開再回來，星球還是一樣

$$S_t^{\psi} \equiv S_{t+k}^{\psi} \quad \text{if}\quad \text{location}(t) = \text{location}(t+k)$$

作者稱之為「零儲存成本的物體恆存性（object permanence）」。

<center><b>圖 3：確定性生成（依原論文 Figure 4 以文字重現）</b></center>

```
第 1 次造訪 ─┐
             ├─▶ 座標 (x, y) ─▶ 雜湊 h(x, y) ─▶ 種子（整數）─▶ LLM 生成器（凍結）
第 100 次造訪 ┘                                                   │
                                                                  ▼
                                               一致的世界狀態 S^ψ（每次都相同）
```

> [!WARNING]
> **「種子固定 LLM 輸出」在公開的程式碼裡沒有實作**（詳見第 5.2 節）。種子實際上用在程式這一側：決定星系佈局、從預先寫好的清單挑描述。呼叫 LLM 時，種子只被當成提示詞裡的一段文字（「Optional vibe seed: …」），API 沒有傳入任何 seed 參數；而且雲端 LLM API 本來就不保證同一個 seed 每次輸出完全一樣。物理層確實恆存，想像層則沒有。

### 2.4 優雅降級（Graceful Degradation）

每一格畫面都呼叫 LLM 太貴，所以有一個「保真度滑桿」：

<center><b>表 3：三級保真度</b></center>

| 等級 | 內容來源 | 什麼時候用 |
|---|---|---|
| 高 | LLM 即時生成專屬內容 | 資源足夠時 |
| 中 | 讀取快取 | 延遲受限時 |
| 基本 | 程式套預先寫好的範本 | LLM 無法使用時 |

因為物理層由程式控制，即使 LLM 完全斷線，世界仍然能運作，只是少了豐富的描述。

### 2.5 技術堆疊

TypeScript 提供型別安全（作者稱為「神經–符號契約」），HTTP 串流讓文字即時顯示，serverless 架構讓程序生成的世界不需要管理伺服器。

---

## 3. 七個示範

<center>
<img src="https://arxiv.org/html/2512.23676v1/images/Teaser2.jpg" alt="圖 4" width="75%">
<br>
<b>圖 4：七個 Web World Model 示範（原論文 Figure 2）</b>
</center>

> [!NOTE]
> **圖 4 怎麼看**
> 七個圓形縮圖：Bookshelf（無限書架）、AI Alchemy（煉金沙盒）、AI Spire（卡牌遊戲）、Galaxy Travel（銀河探索）、Cosmic Voyager（太陽系）、World Travel（旅遊地圖）、WWMPedia（即時生成的百科）。是宣傳用的拼貼圖，縮圖太小看不出細節。

<center><b>表 4：七個示範各自怎麼分工</b></center>

| 示範 | 物理層（程式） | 想像層（LLM） | LLM 掛掉時 | 程式碼公開 |
|---|---|---|---|---|
| Infinite Travel Atlas | 地球座標 → 格子 ID → 雜湊；依緯度決定主題 | 旅遊指南、三日行程、配色 | — | ✓ |
| Galaxy Travel Atlas | 星系佈局、航道、星球屬性（雜湊＋雜訊函數） | 任務簡報、地形、天空、危險 | 從清單挑範本描述 | ✓ |
| AI Spire（卡牌） | 戰鬥引擎：HP、能量、牌組、狀態 | 獎勵卡片、遺物、商店、自訂「願望卡」 | 用預存的範例卡 | ✓ |
| AI Alchemy（沙盒） | 細胞自動機：粉末／液體／氣體的運動 | 新元素、新反應規則（快取起來） | — | ✓ |
| Cosmic Voyager | WebGL 太陽系、軌道（腳本，非真實物理） | 側欄介紹、每 30 秒更新的導覽字幕 | 內建的固定說明 | ✓ |
| WWMPedia | 搜尋、開網頁、清理、固定版面 | 大綱、分節內文、參考文獻 | — | ✗ |
| Bookshelf | 分頁、長度上限、帶到下一頁的狀態 | 書單、逐頁故事 | — | ✗ |

### 3.1 Infinite Travel Atlas：無限旅遊地圖

<center>
<img src="https://arxiv.org/html/2512.23676v1/WorldTravel.png" alt="圖 5" width="95%">
<br>
<b>圖 5：旅遊地圖的互動流程（原論文 Figure 5）</b>
</center>

> [!NOTE]
> **圖 5 怎麼看**
> 三格由左到右：(1) 地球上布滿發光點，點一個（藍框）；(2) 右側面板（橘框）由 AI 生成主題指南、氛圍、配色；(3) 以該地主題呈現的多日行程（綠框，例子是奧地利因斯布魯克）。原圖說寫了 (a)–(d) 四個步驟，圖只有三格，第 (b) 步「雜湊取種子」沒有畫出來。

程式先決定可選的主題範圍，LLM 只從中挑一個方向，再生成內容。作者說：「因為規則是程式，改主題清單就等於改了世界的法則，不需要重新訓練模型。」

<center>
<img src="https://arxiv.org/html/2512.23676v1/images/world-travel/demo7.png" alt="圖 6" width="62%">
<br>
<b>圖 6：選擇奈洛比時的畫面（原論文 Figure 21）</b>
</center>

> [!NOTE]
> **圖 6 怎麼看**
> 上方是地球，黃色發光點是肯亞奈洛比；下方琥珀色的「Travel Plan & Guides」面板是生成的內容，標籤為「DESERT BLOOM VIBE」「EXPECT SHIFTING CLOUDS AND LIGHT SHOWERS」「PANORAMIC LOOKOUT」。作者用這張圖說系統「正確地」觸發了沙漠主題。

> [!WARNING]
> **奈洛比的「沙漠」主題是緯度規則決定的，不是地理判斷**
>
> 我讀了公開的程式碼（`worldPromptService.ts`）：地球上預設的 30 個光點寫在 `SEED_COORDS` 裡，主題由 `deriveSeedTheme` **只依緯度**決定：緯度絕對值小於 12° 一律是 `desert`，12–28° 是 `seaside`，28–50° 是 `forest`，50° 以上是 `urban`，66° 以上是 `mountain`。
>
> 奈洛比的光點在南緯 1.29°，所以被歸為沙漠；但作者自己在程式裡給它的標籤是「Savanna rise」（草原高地），奈洛比實際上也是海拔約 1,800 公尺、氣候溫和的高原城市。反過來，標籤寫著「Dune inlet」（沙丘灣）的杜拜在北緯 25.2°，被歸為 `seaside`；開羅（北緯 30°，真正的沙漠氣候）被歸為 `forest`；新加坡（「Equator quay」）是沙漠，巴黎是森林，柏林是城市。30 個預設光點裡只有 3 個是沙漠，偏偏都不在沙漠（奈洛比、新加坡、拉哥斯）。同一張圖上 LLM 寫的「多雲、陣雨」反而比較接近實際。
>
> 論文說「正確地觸發了沙漠主題」並不成立。這也剛好說明 WWM 的取捨：**程式規則保證一致，但規則本身錯了，錯誤也會一致地出現**。

### 3.2 Galaxy Travel Atlas：銀河探索

<center>
<img src="https://arxiv.org/html/2512.23676v1/GalaxyTravel.png" alt="圖 7" width="95%">
<br>
<b>圖 7：銀河探索的互動流程（原論文 Figure 6）</b>
</center>

> [!NOTE]
> **圖 7 怎麼看**
> 和圖 5 同一個模式：(1) 點選星圖上的星球（橘框）；(2) 右上角顯示「Krypton Gate Node · syncing…」，AI 正在生成任務簡報（藍框）；(3) 結構化的任務紀錄（綠框，例子是 Halion Reach Node：任務簡報、航行筆記、星系旅行提示、星球資料）。第 3 格的星球和第 2 格不同，是兩次操作拼在一起。

這是最完整符合四原則的示範：星系佈局、航道連結、星球屬性完全由程式（`universe.ts`）算出，LLM 只負責「替固定的幾何上色」，而且輸出必須符合 `interface Planet`。LLM 失敗時退回範本描述。作者寫道：「代理人的智慧被限制在安全、可驗證的沙盒裡，無法改變銀河的基本規則或幾何。」

### 3.3 AI Spire：卡牌遊戲

<center>
<img src="https://arxiv.org/html/2512.23676v1/images/card-game/Card.png" alt="圖 8" width="80%">
<br>
<b>圖 8：AI Spire 的神經–符號架構（原論文 Figure 7）</b>
</center>

> [!NOTE]
> **圖 8 怎麼看**
> 左上是<b>戰鬥階段</b>（玩家 HP、能量、牌組、手牌、效果；敵人 HP、效果、下一步）。打贏後進右上的<b>獎勵階段</b>：一般獎勵（生成 3 張卡）或「願望」（玩家輸入「產生一張能冰凍的卡」）。兩者都交給綠框的 <b>LLM（Gemini 2.5 Flash）創意生成</b>，輸出結構化 JSON，經過紫框的<b>契約與驗證</b>（共用 TypeScript 介面 ICard、IRelic），再由最下方的<b>符號核心</b>「轉譯效果代碼 → 套用規則 → 更新遊戲狀態」，最後箭頭回到左邊更新牌組。這張圖把 WWM 的流程畫得最清楚。

<center>
<img src="https://arxiv.org/html/2512.23676v1/images/card-game/Card1.png" alt="圖 9" width="78%">
<br>
<b>圖 9：AI Spire 的獎勵畫面（原論文 Figure 8）</b>
</center>

> [!NOTE]
> **圖 9 怎麼看**
> 紅框是精英戰後掉落的遺物「Ephemeral Might」（每回合開始 +5 力量，回合結束失去）；綠框是「願望」輸入欄；藍框是 LLM 生成的三張卡（Chrono-Sync 技能、Volatile Aura 能力、Unleashed Fury 攻擊）；紫框是跳過；右邊橘框是許願生成的「Inferno Cataclysm」（造成極大燃燒傷害）。原圖說把紅框寫成「extra lyrics reward」，應是「relics」（遺物）的筆誤。

重點是「安全的創意」：LLM 可以發明新卡，但效果只能從程式認得的**效果代碼詞彙**中選。例如遺物寫「start combat strength 1」，程式的觸發器就會在戰鬥開始時加 1 點力量。

### 3.4 AI Alchemy：煉金沙盒

<center>
<img src="https://arxiv.org/html/2512.23676v1/images/ai-alchemy/Alchemy.png" alt="圖 10" width="85%">
<br>
<b>圖 10：AI Alchemy 的架構（原論文 Figure 9）</b>
</center>

> [!NOTE]
> **圖 10 怎麼看**
> 左邊使用者介面（水、火、沙等工具列，以及可以輸入「做出能溶解沙子的酸」的創作主控台）→ 綠框<b>符號核心</b>（細胞自動機，格子裡的粒子依物理分類移動）→ 兩個判斷菱形：「偵測到碰撞？」→「已經有規則？」。有規則就直接套用；沒有就交給紫框的 <b>LLM 合成新反應規則</b>（例如水＋火＝蒸氣），存進<b>反應快取</b>後套用。下方的 AI 監督者可以干預世界，右下是「自我演化、擴張的系統」。

<center>
<img src="https://arxiv.org/html/2512.23676v1/images/ai-alchemy/Alchemy1.png" alt="圖 11" width="90%">
<br>
<b>圖 11：AI Alchemy 的介面（原論文 Figure 10）</b>
</center>

> [!NOTE]
> **圖 11 怎麼看**
> 藍框：創作主控台；橘框：可用元素（空、牆、沙、水、火、煙、氫、氧、電、藤、草、樹、奈米機器人、泥……）；綠框：AI 監督者開關；紫框：模擬畫布；紅框：煉金紀錄，列出 LLM 生成的反應，例如「Rich Soil + Cultivated Soil → Enriched Soil」「Sandy Loam + Cultivated Soil → Premium Soil（NEW）」「Plant Sprout + Ideal Loam → Thriving Plant」。可以看到 LLM 會一路造出越來越「高級」的土壤，名稱很多但實際行為差異不明。

快取是這個示範的關鍵：同一組元素第二次碰撞就直接用快取的規則，不再問 LLM。程式也會限制衰變機率、能量消耗等參數，避免某個元素無限增生（公開程式碼裡有一段「Safety Clamp」把機率封頂）。

### 3.5 Cosmic Voyager：太陽系

<center>
<img src="https://arxiv.org/html/2512.23676v1/images/cosmic-voyager/Voyager.png" alt="圖 12" width="78%">
<br>
<b>圖 12：Cosmic Voyager 的架構（原論文 Figure 11）</b>
</center>

> [!NOTE]
> **圖 12 怎麼看**
> 左：使用者介面（軌道檢視、駕駛模式、地表行走三種模式；選星球、縮放）→ 中間綠框：WebGL 3D 太陽系（場景與狀態管理、渲染、導航邏輯）→ 右上紫框標示為「<b>Symbolic Core（Gemini Flash 2.5）</b>」，負責生成側欄介紹與每 30 秒更新的導覽字幕 → 右下輸出到側欄與底部字幕。注意：這裡把 LLM 標成「符號核心」，和全文「程式是符號、LLM 是神經」的用法相反，其他示範的圖（圖 8、圖 10）都是把程式標成 Symbolic Core。

<center>
<img src="https://arxiv.org/html/2512.23676v1/images/cosmic-voyager/Voyager2.png" alt="圖 13" width="85%">
<br>
<b>圖 13：Cosmic Voyager 的三種模式（原論文 Figure 12）</b>
</center>

> [!NOTE]
> **圖 13 怎麼看**
> 左：軌道模式，點太陽後跳出側欄（橘框，距離 0 AU、半徑 109 個地球半徑，以及 AI 寫的摘要），底部綠框是 AI 導覽字幕。中：駕駛模式，自由飛行。右：降落在星球表面行走。作者坦承軌道速度、距離都是為了好看而設定的腳本，不是真實物理。附錄 Figure 36 說地形「由 LLM 生成」，但程式碼裡地形是用星球名稱當種子的程序生成，LLM 只建議一個「崎嶇程度」參數；這反而是比較符合 WWM 精神的做法。

### 3.6 WWMPedia：即時生成的百科

使用者輸入問題，系統即時產生一頁像維基百科的文章（類似 Grokipedia）。物理層是程式：搜尋、開網頁、擷取段落、清理、固定版面（標題、目錄、分節、參考文獻）；想像層是 LLM：依檢索到的證據寫大綱、分節內文和參考文獻。每一節都可以按「explain more」再展開。

<center>
<img src="https://arxiv.org/html/2512.23676v1/pa.png" alt="圖 14a" width="30%">
<img src="https://arxiv.org/html/2512.23676v1/pb.png" alt="圖 14b" width="30%">
<img src="https://arxiv.org/html/2512.23676v1/pc.png" alt="圖 14c" width="30%">
<br>
<b>圖 14：維基百科、Grokipedia、WWMPedia 查「Superconductor」的比較（原論文 Figure 13）</b>
</center>

> [!NOTE]
> **圖 14 怎麼看**
> 三張都是卡通插畫：(a) 戴眼鏡的貓頭鷹在想維基百科的「Superconductor」頁面（對話泡泡裡是真的截圖）；(b) 機器人在電腦前用 Grokipedia，必須從搜尋結果清單選一篇預先寫好的文章；(c) 魔術師拿著發光的書，泡泡裡是 WWMPedia 即時生成的頁面。
>
> 問題是 (c) 泡泡裡的頁面文字模糊、拼字錯亂（標題看起來像「Sopercondastors」），看起來是被圖像生成模型重畫過的，**讀者無法從圖上確認 WWMPedia 實際產出的內容或引用**。圖說提到的「LLM generated」標籤和時間戳記也看不清楚。WWMPedia 的程式碼也沒有公開。

這個示範其實就是 RAG：檢索由程式控制，生成由 LLM 負責。作者把它包裝成「世界是整個開放網路，每一頁是一個局部狀態」。

### 3.7 Bookshelf：無限書架

<center>
<img src="https://arxiv.org/html/2512.23676v1/Bookshelf_entry.png" alt="圖 15" width="88%">
<br>
<b>圖 15：Bookshelf 的書架畫面（原論文 Figure 14a）</b>
</center>

> [!NOTE]
> **圖 15 怎麼看**
> 左：選標籤（橘框）。<b>介面風格</b>（Forest Morning、Silver Glass、Sunset Wood）只改 CSS，由程式決定；<b>文學標籤</b>（反烏托邦、歷史劇、浪漫喜劇、超自然恐怖、推想浪漫）限制類型、語氣、節奏。右：LLM 依標籤生成四本書的卡片（藍框是被選中的《Quiet Frequencies》）。「Refresh unlocked tags」會換掉沒鎖定的標籤。

每翻一頁，系統把（1）目前的標籤限制、（2）精簡的故事狀態、（3）最近一小段文字交給 LLM 續寫。作者的心得：<b>長篇生成主要是狀態管理的問題</b>，要讓帶到下一頁的狀態保持**有型別、而且小**；LLM 負責局部的文字，程式負責風格限制、分頁、目前未解的劇情線。

---

## 4. 實驗設定

**沒有實驗**。全文沒有任何量化評估：沒有使用者研究、沒有一致性或幻覺的量測、沒有延遲或成本數字、沒有和完全生成式世界模型或傳統網站比較。所有「結果」都是截圖（附錄 Figure 15–37 共 23 張）和作者的描述。

用到的模型：AI Spire、AI Alchemy、Cosmic Voyager 用 Gemini 2.5 Flash；旅遊地圖和銀河探索透過 OpenRouter 呼叫（模型由環境變數設定，論文沒說是哪個）；WWMPedia、Bookshelf 沒有交代。

---

## 5. 我核對程式碼的結果

論文沒有數據可以重算，所以我改讀 GitHub 上的公開程式碼（2026/6 最後更新，5 個示範），核對論文的說法。

### 5.1 對得上的部分

- **型別介面**：AI Spire 用 Gemini 的 `responseSchema`，把卡片類型限制在 `ATTACK`／`SKILL`／`POWER` 三個列舉值；旅遊地圖用 `response_format: json_object`
- **物理層用雜湊**：銀河的星球位置、名稱、危險，都由同一個字串雜湊函數（`(hash << 5) − hash + 字元碼`）決定，每次都一樣
- **優雅降級**：銀河探索在 LLM 失敗時，用雜湊從六組預先寫好的清單挑出地形、天空、危險、任務；AI Spire 退回預存的範例卡
- **安全限制**：AI Alchemy 對 LLM 生成的增生機率做封頂

### 5.2 對不上的部分

<center><b>表 5：論文說法與公開程式碼的差異</b></center>

| 論文的說法 | 公開程式碼 |
|---|---|
| 雜湊種子「固定了 LLM 的抽樣隨機性」，同一地點每次輸出相同（式 2.1） | 呼叫 LLM 時沒有傳入 seed 參數，種子只出現在提示詞的文字裡；temperature 沿用預設值。同一顆星球每次開啟，LLM 寫的內容都可能不同 |
| 銀河探索「用依程序種子建立的檔案快取」儲存結果 | 銀河星球的 LLM 內容沒有快取；主題服務有快取，但是存在記憶體、5 分鐘後過期（`THEME_CACHE_TTL = 5 分鐘`），旅遊光點快取 30 分鐘 |
| 「零儲存成本的物體恆存性」 | 真正恆存的是程式這一側（佈局、名稱、範本描述）；LLM 寫的部分要靠快取，等於還是要存 |
| 奈洛比「正確地」觸發沙漠主題 | 主題只看緯度帶，南北緯 12° 內一律是沙漠；杜拜反而是「海濱」 |

> [!WARNING]
> **式 2.1 的物體恆存性只對物理層成立**
>
> 論文自己的式子 $S_{t+1}^{\psi} \sim \pi_\theta(\cdot \mid S_{t+1}^{\phi})$ 是「從分布中抽樣」，代表同樣的 $S^{\phi}$ 會產生不同的 $S^{\psi}$；緊接著又宣稱 $S^{\psi}_t \equiv S^{\psi}_{t+k}$，兩者矛盾。要讓想像層也恆存，只有兩種方法：(1) 把 LLM 的輸出存起來（快取或資料庫），或 (2) 用 temperature 0 加上支援 seed 的本機模型，而且版本不能換。論文兩個都沒做到「零儲存」。第 2.4 節的「中保真度＝讀快取」其實也承認了要存。

---

## 6. 總結

1. 代理人需要一個能長期活動、又可以控制的世界，傳統網站和完全生成的世界模型各有缺點
2. WWM：世界的結構和規則寫在程式裡（物理層），LLM 只負責補描述和劇情（想像層），而且是先算物理、再寫敘述
3. 四個原則：物理與想像分開、用型別介面限制 LLM 輸出、用雜湊做無限的程序生成、LLM 不可用時退回快取或範本
4. 用 7 個網頁示範說明這個概念可以套到地圖、遊戲、百科、小說等不同類型

> [!WARNING]
> **限制**
> - **沒有任何量化實驗**：「可控」「一致」「無幻覺」都沒有量測。結論說 WWM 是「hallucination-free worlds」（無幻覺的世界），但 WWMPedia、旅遊指南的內容全由 LLM 撰寫，型別介面只能保證格式正確，擋不了內容錯誤（例如 LLM 可以把錯的景點填進格式正確的 JSON）
> - **核心賣點和程式碼不符**：雜湊固定 LLM 輸出、檔案快取，在公開程式碼裡都沒有（第 5.2 節）
> - **「神經–符號」的分工本身不新**：程式生成 + 文字修飾是遊戲界的程序生成老方法；LLM 輸出 JSON schema 也是現在的標準做法。新意主要在命名和歸納
> - WWMPedia 和 Bookshelf 沒有程式碼，WWMPedia 的圖是卡通重繪，無法驗證
> - 圖說錯誤：Figure 5 圖說列了四步、圖只有三格；Figure 8 的「lyrics」應為「relics」；Figure 11 把 LLM 標成「Symbolic Core」；圖上的錯字「Itnerary」「Strctured」「profle」；正文的「Grokpedia」和圖說的「Grokipedia」不一致；v1 的「four core design principles（Section 3）」應為 Section 2（v2 已修正）
> - 相關研究沒有提到最接近的工作，例如 Genie 系列、AI Dungeon 類的 LLM 文字冒險，也沒有討論 LLM 結構化輸出的既有研究

> [!TIP]
> **可以帶走的思路**
> - **事實和數字由程式算，LLM 只負責講**：把會被追究對錯的部分（警戒等級、門檻、座標、資源量）放在確定性的程式裡，LLM 只根據程式算好的結果寫說明。這比讓 LLM 自己推導再事後檢查可靠得多
> - **用型別當契約**：LLM 的輸出先定好 JSON schema，程式只接受合格的欄位和列舉值；效果、動作只能從程式認得的詞彙中選（AI Spire 的效果代碼）
> - **一定要有退路**：LLM 逾時或失敗時，退回快取或範本，系統的核心功能不受影響
> - **想要「每次都一樣」就要存**：不要相信 seed 能讓雲端 LLM 輸出固定。要可重現，就把第一次的輸出快取起來，用輸入的雜湊當鍵
> - **長對話、長文件生成時，帶到下一輪的狀態要有型別、而且小**（Bookshelf 的心得），這和代理人記憶的設計是同一件事

---

## 7. 我的筆記

### 重點筆記
>

### 可以借用的地方
> 做災防 RAG 助理時，可以照 WWM 的分工：雨量、震度、警戒等級、撤離門檻全部由程式從官方資料算好，放進固定的 JSON 欄位（例如 `{ "level": "紅色", "rain_24h_mm": 350, "threshold_mm": 200 }`），LLM 只負責把這些數字寫成民眾看得懂的說明，而且不准改數字；LLM 掛掉時退回範本句「目前 X 地區 24 小時雨量 Y 毫米，已達 Z 警戒」。WWMPedia「檢索與版面由程式固定、內容由 LLM 依證據填寫、附引用」的頁面結構，也可以用在自動產生的災情摘要頁。

### 疑問
> 想像層要恆存，到底該存多少？全部存起來就退回資料庫了，作者說的「無限」和「不用儲存」能不能同時成立？型別介面擋得住格式錯誤，擋得住內容錯誤嗎？WWMPedia 的引用正確率是多少？

### 相關
- 概念：[[World model|世界模型]]、[[Neuro-symbolic AI|神經符號 AI]]、[[Procedural generation|程序生成]]、[[Constrained decoding|受限解碼]]、[[Retrieval-augmented generation|RAG]]、[[Agent memory|代理人記憶]]、[[ReAct]]
- 論文引用的相關工作：Ha & Schmidhuber, *World Models*（NeurIPS 2018）；WebDreamer（Gu et al., 2024，用 LLM 模擬網頁操作的結果）；RAP（Hao et al., EMNLP 2023）；Generative Agents（Park et al., UIST 2023）；Voyager（Wang et al., 2023，Minecraft 中累積可執行技能）；Unbounded（Li et al., 2024，生成式無限遊戲）
