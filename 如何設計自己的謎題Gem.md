---
tags:
  - 教學指南
  - Gem設計
  - 謎題Gem
  - Stage1
created: '2026-04-18'
status: 可用
---
# 如何設計自己的謎題 Gem

> 給想要設計專屬學科解謎 Gem 的教師。讀完這份之後，你就能依照自己科目的內容，生成屬於自己的謎題 Gem。
>
> 本指南對應「實境解謎遊戲」三階段的 **Stage 1（謎題生成）**。後續 Stage 2（劇本）、Stage 3（網頁）不在此範圍。

---

## 🧭 這份指南要回答的五個問題

1. 什麼叫「出謎題的 Gem」？它在做什麼？
2. 設計前要先想清楚的三件事
3. 一個完整的 Gem 由哪七個段落組成？
4. 兩種常見設計模式：靜默生圖 vs 先驗算後生圖
5. 從零開始：空白範本 + 改造步驟

---

## 一、什麼是「出謎題的 Gem」？

**它不是一個「直接跟 AI 聊天要謎題」的工具**，而是一個**固定規則的出題機器**。

一旦設計好，老師輸入「謎底」（如 `MATH`），Gem 自動根據學科知識產出：
1. **謎題圖**（給學生看的線索，不含答案）
2. **對照表圖**（給學生查的解碼工具）

老師不用每次重寫 prompt，也不用擔心 AI 每次產出的風格或邏輯不一致。

### 範例：運作流程

```
老師輸入：MATH
     ↓
Gem 自動處理：
  M → 查知識庫 → 對應到化學元素「鋁 (Al)」
  A → 查知識庫 → 對應到「氫 (H)」
  T → 查知識庫 → 對應到「鈣 (Ca)」
  H → 查知識庫 → 對應到「氧 (O)」
     ↓
產出兩張圖：
  圖 A：謎題圖（顯示「鋁、氫、鈣、氧」四個中文名，不顯示字母）
  圖 B：對照表圖（A-Z ↔ 原子序 ↔ 元素符號 ↔ 中文名）
     ↓
學生拿到後：
  看圖 A → 查圖 B → 反推回 M、A、T、H
```

---

## 二、設計前要先想清楚的三件事

這三件事決定了整個 Gem 的命運。設計前沒想清楚，後面都是白忙。

### 1. 謎底是什麼形式？

| 謎底類型 | 範例 | 適合情境 |
|----------|------|----------|
| **數字** | `4826` | 鎖箱密碼、數位輸入框（最常見） |
| **英文字母／單字** | `CHILD`、`MATH` | 英文科、跨科主題詞 |
| **中文字／詞語** | `光合作用`、`清明` | 國文、社會、各科關鍵詞 |

💡 **建議：初學者先做數字謎底**。數字對照表簡單、學生輸入不易出錯、AI 生成錯誤率低。

### 2. 要用什麼「學科知識」當翻譯規則？

問自己：**我的課程裡，哪些知識天生就有對應關係？**

```
✅ 適合當翻譯規則（有穩定對應）：
  - 有固定序號的：注音順序、元素週期表、朝代順序、音名
  - 有固定對應的：摩斯密碼、ASCII 碼、色相環位置
  - 有可計算數值的：筆畫數、原子序、年份、角度、二進位

❌ 不適合（歧義大或無唯一解）：
  - 沒有唯一答案的概念（如：民主的意涵、美的定義）
  - 需要主觀判斷才能得出的結果
  - 對照表太大或太複雜，學生無法在活動時間內查完
```

### 3. 學生要做什麼動作來解密？

這決定了**謎題圖該畫什麼**。

| 學生動作 | 謎題圖內容範例 |
|----------|----------------|
| 查表對照（最簡單） | 摩斯符號、元素中文名、色票 |
| 算術計算 | 等差數列、一元一次方程式 |
| 空間推理 | 時鐘指針、座標地圖、週期表位置 |
| 組合辨識 | 六大類食物照片、地標剪影 |

---

## 三、Gem 的標準七段結構

每一個謎題 Gem（以已完成的 18 個為例）都有這七個段落。缺一個就會出問題。

```yaml
# 七段結構總覽
1. version + profile        # 身分證
2. knowledge_base           # 翻譯規則
3. system_behavior          # 行為規則（含 primary_directive、workflow、prohibitions）
4. puzzle_image             # 謎題圖生成規則（雙模式）
5. reference_chart_image    # 對照表圖生成規則
6. user_manual              # 教師使用手冊
```

### 段落 1：`version` + `profile` — Gem 的身分證

```yaml
version: 1.0
profile:
  name: 元素週期表密碼（進階版）
  role: 圖像生成引擎，將英文單字謎底轉換為化學元素視覺謎題
  image_model: Nano Banana 2
  description: >
    接收英文單字後，生成兩張風格一致的圖：
    【圖 A．謎題圖】供學生觀察元素中文名
    【圖 B．對照表圖】供學生查「字母↔原子序↔元素符號↔中文名」
```

**要點：**
- `name`：中文命名，格式「主題 + 密碼」
- `role`：一句話講清楚 Gem 是做什麼的（**圖像生成引擎**四個字很重要，讓 Gemini 走繪圖路線）
- `description`：用列點描述輸出什麼，學生做什麼

---

### 段落 2：`knowledge_base` — 翻譯規則（最重要的一段）

這是 Gem 的靈魂。**不能靠 AI 記憶，必須直接寫出完整對應表**。

#### 類型 A：靜態對照表（最常用，佔 14/18）

```yaml
knowledge_base:
  cipher_chart:
    A: ".-"
    B: "-..."
    C: "-.-."
    # ... 寫到 Z
```

適用：摩斯密碼、A1Z26、注音、簡譜、色相環、二進位、食物分類

#### 類型 B：邏輯引擎（規則生成，佔 2/18）

```yaml
knowledge_base:
  logic_engine:
    rule: "對每位數字 D，隨機挑公差 d（|d|≥2），產生 6 個等差數列項，
          再把 D 插入為『入侵者』讓學生找出"
```

適用：等差數列、方程式圖形

#### 類型 C：座標地圖（空間映射，佔 1/18）

```yaml
knowledge_base:
  coordinate_map:
    A: [1, 4]
    B: [-3, 7]
    # ... 25 或 26 組
```

適用：直線攔截

#### 類型 D：多面板整合（跨科，佔 1/18）

```yaml
knowledge_base:
  panel_A: { ... }  # 歷史軸
  panel_B: { ... }  # 地理軸
  panel_C: { ... }  # 公民軸
```

適用：社會解碼盤

**💡 新手建議**：先用類型 A（靜態對照表），複製現有 Gem 改 knowledge_base 即可。

---

### 段落 3：`system_behavior` — 行為規則（四小部分）

```yaml
system_behavior:
  primary_directive: |
    # 核心指令
    每次接到使用者輸入後，遵守以下固定流程。
    禁止憑記憶推算；所有對照必須來自 knowledge_base。
  
  workflow:
    step_1_receive: 接收並驗證輸入
    step_2_verify: 輸出驗算表給教師（可選）
    step_3_generate_puzzle: 生成謎題圖
    step_4_generate_reference: 生成對照表圖
    step_5_silent: 提示確認
  
  prohibitions:
    - 謎題圖嚴禁出現阿拉伯數字
    - 謎題圖嚴禁出現拉丁字母
    - 謎題圖嚴禁出現「答案」、「密碼」等洩題字樣
    - 禁止生成不在 knowledge_base 中的元素
```

**四個禁令的寫法通則：**
1. 禁止圖中直接顯示謎底
2. 禁止出現答案字樣（「answer」、「密碼」、「key」）
3. 禁止超出 knowledge_base 的內容
4. 禁止 AI 憑記憶推算（強制查表）

---

### 段落 4：`puzzle_image` — 謎題圖生成（雙模式設計）

**為什麼要兩個模式？** 給老師彈性：
- `mode_1`：**沉浸式**（古卷軸、奇幻、戲劇感），適合主題遊戲
- `mode_2`：**教學用**（現代、清爽、數據清楚），適合示範或補救教學

```yaml
puzzle_image:
  mode_1_ancient_scroll:
    aspect_ratio: "16:9"
    prompt_template: |
      Create an ancient alchemy parchment scroll, sepia tones, aged paper
      with tattered edges and tea stains. On the scroll, display ONLY these
      Chinese element names in a horizontal row, in order:
      [ELEMENT_CHINESE_NAMES_IN_ORDER]
      
      - Use large, hand-drawn traditional Chinese calligraphy
      - Each character in its own ornate box
      - STRICT RULES:
        * NO atomic numbers
        * NO Latin letters
        * NO chemical symbols
        * NO words like "answer"
      - Background: faint alchemical sigils
  
  mode_2_modern_card:
    aspect_ratio: "16:9"
    prompt_template: |
      Create a clean modern educational card ...
      ...
```

**Prompt 撰寫五大要素：**

1. **風格關鍵字**（第一句定調）
   - 範例：`ancient parchment scroll`、`chalkboard diagram`、`flat design card`

2. **版面布局**（怎麼排）
   - 範例：`horizontal row`、`3x3 grid`、`vertical stack`、`two panels`

3. **內容佔位符**（用 `[變數名]` 留位）
   - 範例：`[ELEMENT_CHINESE_NAMES_IN_ORDER]`、`[MORSE_SYMBOLS]`

4. **STRICT RULES**（嚴格禁令，英文大寫強調）
   - 範例：`NO answer text`、`NO Chinese characters`、`NO pinyin`

5. **氛圍／情境**（最後補一句）
   - 範例：`mood: mysterious cipher scroll from medieval laboratory`

**💡 竅門**：每個 prompt 控制在 150–250 字內。太短 AI 自由發揮會失控；太長 AI 會忽略某些規則。

---

### 段落 5：`reference_chart_image` — 對照表圖

對照表是**學生解謎的鑰匙**，必須：
1. **完整**（所有 mapping 都要有）
2. **雙向查閱**（老師編碼用、學生解碼用）
3. **列印清晰**（預期印 A4）

```yaml
reference_chart_image:
  aspect_ratio: "3:4"
  prompt_template: |
    Create an ancient reference scroll matching the puzzle image style.
    
    Layout: Four columns, 26 rows. Header: 字母 | 原子序 | 元素符號 | 中文名
    
    Fill the table EXACTLY with this data:
    A | 1  | H  | 氫
    B | 2  | He | 氦
    ...（把所有 26 列全部寫出來）
    
    - Title: "ELEMENTAL CIPHER KEY"
    - All cell values must be perfectly legible at A4 print size
```

**關鍵技巧**：
- **把所有對照資料直接寫進 prompt**（不要相信 AI 會記住）
- `aspect_ratio` 用 `3:4`（直式表格適合印 A4）
- 強調「EXACTLY」、「legible at A4」防止 AI 自作主張刪減

---

### 段落 6：`user_manual` — 教師使用手冊

```yaml
user_manual: |
  ▍使用流程
  1. 輸入英文單字謎底（建議 4-6 字母，如 MATH、CODE）
  2. Gem 輸出驗算表 → 核對無誤後回覆「1」或「2」選視覺模式
  3. Gem 生成兩張圖：謎題圖、對照表圖
  4. 學生解謎流程：看中文名 → 查對照表 → 反推字母

  ▍推薦謎底單字
  - 數學相關：MATH, ADD, SUM
  - 自然相關：CELL, ATOM, GENE, SALT

  ▍注意事項
  ⚠️ AI 對 20 號以後的元素記憶不穩，務必核對驗算表
  ⚠️ 若圖片與驗算表不符，請回覆「請重新生成」
```

**三段式結構**：
1. 使用流程（1-2-3-4 步驟）
2. 推薦謎底（給老師靈感）
3. 注意事項（⚠️ 風險提醒）

---

## 四、兩種設計模式：怎麼選？

### 模式 A：靜默直接生圖（Silent Protocol）

```
老師輸入 → AI 直接生成圖 → 完成
```

**特徵：**
- `primary_directive: "DO NOT RESPOND WITH TEXT. TRIGGER image tool IMMEDIATELY."`
- 無驗算表步驟
- 快速、流暢、教師不用確認

**適用場景：**
- knowledge_base 是固定、不易出錯的對照（A-Z 摩斯、A1Z26 字母序號）
- 謎底長度短（2-4 字），AI 出錯機率低
- 示範或批量產題

**代表範例：** 摩斯密碼、等差數列、代數偏移鎖、二元一次方程式圖形

### 模式 B：先驗算後生圖（Verify-First）

```
老師輸入 → AI 輸出驗算表 → 老師核對 → 輸入「1」或「2」選模式 → AI 生圖
```

**特徵：**
- `workflow` 有 `step_2_verify`
- 驗算表格式清楚（用 `━━━` 包圍）
- 教師必須確認才進行下一步

**適用場景：**
- AI 易犯錯的知識（漢字筆畫、20+ 元素、中文音節）
- 精確度要求高（時鐘角度、座標）
- 正式使用（研習、比賽）

**代表範例：** 元素週期表、時鐘密碼、筆畫密碼、注音序號、音節數密碼

### 決策樹

```
Q1: AI 對這個 knowledge_base 的內容會不會記錯？
    ├─ 會 → 用模式 B（verify-first）
    └─ 不會
        │
    Q2: 謎底長度會不會超過 4 字？
        ├─ 會 → 用模式 B（長謎底錯誤累積機率高）
        └─ 不會 → 用模式 A（silent）
```

---

## 五、從零開始：空白範本

直接複製下方，修改標示 `【填這裡】` 的部分即可。

```yaml
# ==========================================
# 機器人名稱：【填：你的 Gem 名稱】
# 版本：v1.0
# ==========================================

version: 1.0
profile:
  name: 【填：中文名，格式「XX 密碼」】
  role: 圖像生成引擎，將【填：謎底類型】轉換為【填：學科】視覺謎題
  image_model: Nano Banana 2
  description: >
    接收【填：輸入格式】後，生成兩張圖：
    【圖 A．謎題圖】供學生觀察【填：謎題內容】
    【圖 B．對照表圖】供學生查【填：對照關係】

# ======================================================
# 核心資料庫
# ======================================================
knowledge_base:
  cipher_chart:
    # 【填：完整的對應表】
    # 範例（A1Z26）：
    # A: 1
    # B: 2
    # ...
    # Z: 26

# ======================================================
# 系統行為規則
# ======================================================
system_behavior:
  primary_directive: |
    每次接到使用者輸入後，遵守以下固定流程。
    禁止憑記憶推算，所有對照必須來自 knowledge_base.cipher_chart。

  workflow:
    step_1_receive: |
      接收使用者輸入，驗證格式。
      【填：例如「接收 2-8 字英文單字，自動轉大寫」】

    step_2_verify: |  # 如果用模式 A（靜默），刪除此段
      以「純文字」輸出【驗算表】供教師核對：
      ━━━━━━━━━━━━━━━━━━━━━━━
      【驗算表】謎底：【輸入值】
      【填：每個字元的對應列法】

      請教師確認對照無誤。
      請選擇視覺模式：
        1 = 【填：模式 1 名稱】
        2 = 【填：模式 2 名稱】
      輸入「1」或「2」生成圖片。
      ━━━━━━━━━━━━━━━━━━━━━━━

    step_3_generate_puzzle: |
      依選擇的模式生成【圖 A．謎題圖】。

    step_4_generate_reference: |
      緊接著生成【圖 B．對照表圖】，風格必須與【圖 A】一致。

    step_5_silent_after_images: |
      兩張圖完成後，僅輸出：「謎題圖與對照表圖已生成，請核對。」

  prohibitions:
    - 謎題圖中嚴禁出現【填：禁止的內容，如「阿拉伯數字」「拉丁字母」】
    - 謎題圖中嚴禁出現「答案」、「密碼」、「key」等洩題字樣
    - 對照表圖必須完整涵蓋所有 mapping
    - 禁止生成不在 knowledge_base 中的內容

# ======================================================
# 圖 A．謎題圖 — 兩種視覺模式
# ======================================================
puzzle_image:
  mode_1_【填：模式 1 英文識別碼，如 ancient_scroll】:
    aspect_ratio: "16:9"
    prompt_template: |
      Create 【填：風格描述，英文】.
      Display 【填：要顯示的內容，用 [VARIABLE] 佔位】 in 【填：排版】.
      
      - 【填：細節規則 1】
      - 【填：細節規則 2】
      - STRICT RULES:
        * NO 【填：禁止出現的元素 1】
        * NO 【填：禁止出現的元素 2】
        * NO answer text
      
      Mood: 【填：氛圍描述】

  mode_2_【填：模式 2 英文識別碼】:
    aspect_ratio: "16:9"
    prompt_template: |
      【填：第二種風格，與 mode_1 對比】

# ======================================================
# 圖 B．對照表圖
# ======================================================
reference_chart_image:
  aspect_ratio: "3:4"
  prompt_template: |
    Create a reference chart matching the puzzle style.
    
    Layout: 【填：欄數】 columns, 【填：列數】 rows.
    Header: 【填：表頭，如 "字母 | 數字"】
    
    Fill the table EXACTLY with this data:
    【填：完整寫出所有對應資料，不要省略】
    
    - Title: 【填：對照表標題】
    - All cell values must be perfectly legible at A4 print size

# ======================================================
# 使用手冊
# ======================================================
user_manual: |
  ▍使用流程
  1. 【填：輸入什麼】
  2. 【填：Gem 會回什麼】
  3. 【填：確認後發生什麼】
  4. 學生解謎流程：【填：學生怎麼解】

  ▍推薦謎底
  - 【填：範例 1】
  - 【填：範例 2】

  ▍注意事項
  ⚠️ 【填：風險提醒 1】
  ⚠️ 【填：風險提醒 2】
```

---

## 六、改造範例：從現有 Gem 改成自己的

**最快的上手方法**：拿一個最接近的 Gem 複製過來，只改 `knowledge_base` 即可。

| 你要做的科目 | 建議改造對象 | 只需修改 |
|-------------|-------------|----------|
| 化學元素英文名（自然） | A1Z26（字母序號） | knowledge_base 改為 A=Hydrogen, B=Helium... |
| 化學符號（自然） | 摩斯密碼 | knowledge_base 改為 A="H", B="He"... |
| 注音符號（國文） | 摩斯密碼 | knowledge_base 改為 1="ㄅ", 2="ㄆ"... |
| 三字經筆畫（國文） | 筆畫密碼 | knowledge_base 的 stroke_character_pool 填入三字經字庫 |
| 台灣 22 縣市（社會） | 世界首都 | knowledge_base 改為縣市名＋地標 |
| 太陽系行星（自然） | 朝代順序 | knowledge_base 改為 1=水星、2=金星... |

**改造五步驟：**

1. **複製 YAML**：從現有 18 個 Gem 挑一個最像的，整段複製
2. **換 name 與 description**：對應到你的學科
3. **換 knowledge_base**：這是最核心的部分，直接寫出完整對應表
4. **調整 prompt_template**：把「元素符號」替換成你的 domain（如「行星圖示」）
5. **測試**：輸入 2-3 個謎底，檢查輸出是否符合預期；不對就修 prompt

---

## 七、常見陷阱與解法

| 症狀 | 原因 | 解法 |
|------|------|------|
| 謎題圖上出現了答案字母 | Prohibitions 不夠具體 | 在 STRICT RULES 加上「NO Latin letters anywhere」 |
| AI 每次生成的風格不一樣 | Prompt 太抽象 | 用具體關鍵字：`sepia tones`, `hand-drawn calligraphy` |
| 對照表內容缺漏或順序錯 | 沒把完整資料寫進 prompt | 直接把所有對應 **逐行寫入** prompt_template |
| AI 記錯對照（特別是 20+ 元素、中文筆畫） | 相信了 AI 記憶 | 改用**模式 B 驗證流程**，先輸出驗算表 |
| 中文字在圖中變形或錯字 | AI 繪圖對中文支援弱 | 指定「traditional calligraphy」、「楷書」；或用符號／英文替代 |
| 謎底長度與對照表不符 | 沒設定輸入驗證 | 在 step_1_receive 加上「若超過 N 字則提醒」 |
| 對照表太密印不清楚 | 用 16:9 橫向 | 改為 `aspect_ratio: "3:4"` 直式，並分欄 |
| 學生解不出來 | 難度估計失誤 | 搭配「對照表」發下當作道具；或提供提示卡 |
| 圖片上有多餘裝飾干擾判讀 | Prompt 太發散 | 加上「minimal decoration, clean layout, no clutter」 |

---

## 八、進階技巧（做熟了再玩）

### 1. **雙向跨科串連**
設計時讓謎底成為**另一個 Gem 的輸入**：
- 自然 Gem 輸出英文單字 `SALT` → 英文 Gem 接手做音節密碼

### 2. **難度調整旋鈕**
同一套規則，藉由以下方式調整難度：
- 謎底長度（2 位 vs 5 位）
- 對照表公開程度（全給 vs 半給 vs 不給）
- 干擾項（圖中加入不相關符號）

### 3. **自訂風格模式**
在 Prompt 中加入「{STYLE}」變數，讓教師輸入風格：
```
老師輸入：MATH 浮世繪
Gem 生成浮世繪風格的謎題圖
```

### 4. **多面板整合（跨科）**
社會解碼盤的設計：把三個科目的 knowledge_base 疊成一個解碼盤，每一位謎底數字由一個科目貢獻。

---

## 九、產出檢查清單

設計完 Gem 後，用以下清單自查：

- [ ] `name` 與 `role` 讓人一看就懂做什麼
- [ ] `knowledge_base` **完整寫出所有對應**，不靠 AI 記憶
- [ ] `primary_directive` 明確指示行為（靜默 or 驗算）
- [ ] `prohibitions` 至少 4 條具體禁令
- [ ] `puzzle_image` 有**兩種視覺模式**
- [ ] 每個 `prompt_template` **150–250 字**、含「STRICT RULES」英文清單
- [ ] `reference_chart_image` **逐行寫入所有對應**
- [ ] `user_manual` 有流程、推薦謎底、風險提醒三段
- [ ] 測試 2-3 個不同謎底，輸出都符合預期
- [ ] **謎題圖中不會出現謎底或答案字樣**

全部打勾 = 可以交給其他老師用了 ✅

---

## 十、哪裡找 18 個現成範例？

本專案的 `code/` 資料夾已經有 18 個 paste-ready Gem YAML：

| 領域 | 範例名稱（檔名） |
|------|------------------|
| 自然 | `Gem範例_元素週期表密碼.md` |
| 社會 | `Gem範例_朝代順序密碼.md`、`Gem範例_世界首都密碼.md`、`Gem範例_社會解碼盤密碼.md` |
| 音樂 | `Gem範例_音樂簡譜密碼.md` |
| 美術 | `Gem範例_色相環密碼.md` |
| 科技 | `Gem範例_二進位密碼.md` |
| 健教 | `Gem範例_六大類食物密碼.md` |
| 數學 | `Gem範例_摩斯密碼.md`、`Gem範例_等差數列密碼.md`、`Gem範例_二元一次方程式圖形密碼.md`、`Gem範例_代數偏移鎖密碼.md`、`Gem範例_時鐘密碼.md`、`Gem範例_直線攔截密碼.md` |
| 國文 | `Gem範例_筆畫密碼.md`、`Gem範例_注音序號密碼.md` |
| 英文 | `Gem範例_字母序號密碼.md`、`Gem範例_音節數密碼.md` |

---

## 總結：一頁版

```
1. 先想清楚：謎底是什麼 / 用什麼學科知識當翻譯 / 學生做什麼動作
2. Gem 有七段：version, profile, knowledge_base, system_behavior,
               puzzle_image, reference_chart_image, user_manual
3. knowledge_base 是靈魂：把完整對應表寫出來，不靠 AI 記憶
4. 兩種設計模式：靜默生圖（簡單快） vs 先驗算（精確可靠）
5. puzzle_image 要雙模式（沉浸式 + 教學用）
6. prompt 寫法：風格 + 版面 + 佔位符 + STRICT RULES + 氛圍
7. 對照表把所有資料逐行寫入 prompt
8. 從現有 18 個範例改造最快
9. 測試 2-3 個謎底、跑一次自查清單
10. 交給同事用，持續迭代
```

---

*本指南由 0418 康軒解謎研習專案整理，隨著更多 Gem 實作將持續增補。*
*指南版本：v1.0（2026-04-18）*
