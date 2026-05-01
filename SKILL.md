---
name: youtube-to-article
description: Transform YouTube videos into high-quality, structured articles via a FastAPI web app (v3.2). Supports multi-language subtitles, concurrent batch processing, 4-language UI, smart URL parsing, multi-tab results, and clean-text export to Apple Notes.
category: productivity
version: 3.2
---

# SKILL: youtube-to-article (v3.2)

將 YouTube 影片透過「字幕提取 -> 數據清洗 -> LLM 重構」的流水線，將口語化的影片內容轉化為結構清晰、適合閱讀的高品質文章。支援多個影片並行處理、多國語言字幕自動匹配、4 語系 UI 切換，以及 Apple Notes 純文字導出。

## 觸發條件

- 用戶提供 YouTube 連結並要求「轉成文章」、「總結成深度博文」或「將影片內容文字化」時。
- 用戶要求啟動或管理本工具服務器時（`start.sh`）。

## 專案結構

```
youtube-article-tool/
├── app/
│   ├── main.py              # FastAPI 後端入口 (uvicorn)
│   └── templates/
│       └── index.html       # Tailwind CSS 前端 (663行, 4語系 i18n)
├── data/
│   └── history.json         # 轉換紀錄 (去重, 最多50條)
├── docs/
│   ├── README.en.md
│   ├── README.zh-Hant.md
│   ├── README.zh-Hans.md
│   └── README.ja.md
├── start.sh                 # 啟動腳本 (port 8080, --reload)
├── PATCH_NOTES.md           # 版本歷史
├── README.md                # Hub-and-Spoke 導航 README
└── requirements.txt
```

## 技術棧

- **後端**: FastAPI + uvicorn (port 8080, --reload 模式)
- **前端**: Tailwind CSS (CDN) + Marked.js + 原生 JS (無框架)
- **字幕**: yt-dlp (`--write-auto-subs --sub-format vtt`)
- **LLM**: Google Gemini (`gemini-flash-latest`, httpx.AsyncClient)
- **執行**: asyncio.gather 並行處理

## 執行步驟

### 第一步：鑑別使用場景

有兩種使用本工具的方式，根據用戶的情境選擇：

#### A. 網頁工具 (主要入口)
用戶想要使用完整的 Web UI（含歷史、多結果分頁、i18n 等）。路徑：`/Users/lunker/youtube-article-tool`
啟動：執行 `start.sh`（內部呼叫 `uvicorn app.main:app --reload --port 8080`）
網址：`http://127.0.0.1:8080`

#### B. 單次調用 (腳本模式)
用戶只需快速轉換一則影片，不開網頁。直接使用 `yt-dlp` + `call_llm` 流程（見步驟二~四），手動輸出結果。

### 第二步：字幕提取 (Extraction)

**並行提取，每則影片使用獨立臨時目錄防止文件覆蓋。**

後端邏輯 (`process_single_video`):
1. `yt-dlp --dump-json --skip-download` 取得元數據（title, thumbnails, id 等）
2. `yt-dlp --write-subs --write-auto-subs --sub-langs "<langs>" --skip-download --sub-format vtt` 下載字幕
3. **不設 `check=True`**：429 速率限制只記錄警告，繼續嘗試使用已下載的字幕
4. 使用 `glob.glob` 遞迴搜索 `.vtt` 文件

**優先語言清單 (關鍵)：**
```
["zh", "zh-TW", "zh-HK", "zh-CN", "zh-Hans", "zh-Hant", "en", "ja", "ko"]
```
*注意：必須包含基礎的 `zh` 標籤，否則部分自動生成字幕無法匹配。*

### 第三步：清洗字幕 (Cleaning)

`clean_vtt()` 函數：
1. 移除 `WEBVTT` 頭、時間戳（`-->`）、HTML 標籤（`<c>`, `</c>` 等）
2. **連續行去重**：相鄰完全相同的行只保留第一行
3. 合併為單一字串傳遞給 LLM

### 第四步：LLM 重構 (Synthesis)

`call_llm(text, meta, api_key, target_lang)` 發送 POST 請求到 Gemini REST API。

**Prompt 架構：**
1. 角色設定：頂尖科技記者與知識策展人
2. 輸入：影片元數據 + 清洗後字幕
3. 任務：改寫為高品質的 `{target_lang}` 文章
4. 寫作準則：
   - 邏輯重構：提取 3-5 個核心主題，以主題塊組織
   - 保留精髓：具體案例、數據、金句、故事
   - 補全上下文：對影片預設已知但讀者陌生的背景補充解釋
   - 語調轉換：口語轉專業書面，但保留原作者觀點色彩
   - 格式：Markdown，清晰的 H1/H2/H3 結構
   - **禁令**：絕對禁止 LaTeX 符號 -> 統一使用 '->'

**技術細節：**
- 端點：`https://generativelanguage.googleapis.com/v1beta/models/gemini-flash-latest:generateContent`
- API Key 優先級：前端表單傳入 → 環境變數 `GEMINI_API_KEY` → 返回錯誤
- Timeout: 120 秒
- 非同步調用 (`httpx.AsyncClient`)

### 第五步：前端顯示與操作 (Web UI)

轉換完成後，結果在 Web UI 中展示：
1. **多結果分頁**：批量轉換時以 Tab 切換各影片結果
2. **Markdown 渲染**：使用 `marked.js` 將 LLM 返回的 Markdown 渲染為 HTML
3. **下載 MD**：`Blob` + `URL.createObjectURL` + 隱藏 `<a>` 點擊觸發
4. **複製內容**：Clipboard API + 成功 Toast 通知
5. **歷史紀錄**：轉換完成自動存入 `history.json`，前端展示縮圖+標題+日期

## 前段架構 (Frontend)

### 1. 視覺設計系統
- 深色主題 (`bg-[#0f172a]`)，藍色漸層主色 (`#3b82f6` / `#60a5fa`)
- 自訂 prose 類別：漸層 H1、藍色左邊框 H2、圓角 3xl 卡片
- 玻璃態效果 (`backdrop-blur-sm`)，平滑過渡動畫
- 響應式設計：行動裝置適配

### 2. i18n 多語系

**支援語言：** zh-TW（繁體中文）, zh-CN（簡體中文）, en（English）, ja（日本語）

**實作方式：**
- `const i18n = {...}` 字典，每語系包含約 22 個鍵值 (title, subtitle, apiLabel, convertBtn, errorTitle, loadingSteps 等)
- `changeUILanguage()` 函數透過 `document.getElementById` 或精確選擇器更新 DOM
- `localStorage.setItem('ui_lang', lang)` 持久化使用者偏好
- `window.onload` 時自動讀取 `localStorage` 並恢復語言設定

**坑點：**
- 絕對禁止依賴 `querySelectorAll('label')` 的索引順序（如 `labels[2]`），佈局微調會導致索引錯位
- 必須使用 `id` 定位或比對 `innerText` 內容來匹配元素
- `changeUILanguage()` 中的未捕獲錯誤會中斷 `window.onload` 中後續的 `loadHistory()` 執行

### 3. 關鍵交互
- **智慧 URL 解析**：支援逗號、空格、換行分隔，正則驗證 YouTube 連結格式
- **Ctrl+Enter 快捷鍵**：文字輸入區按下快捷鍵直接觸發轉換
- **清除確認**：清除按鈕附帶確認對話框，防止誤操作
- **Toast 通知**：非侵入式提示（複製成功、刪除失敗），取代 `alert()` 阻塞式彈窗
- **載入進度條**：4 步驟動態文字 + 百分比進度條
- **錯誤顯示區域**：內聯顯示錯誤資訊，支援「部分失敗」的清單展示

### 4. 縮圖處理 (`get_best_thumbnail`)
優先級：`meta.thumbnails[-1].url` (yt-dlp 提供最高解析度) → `hqdefault.jpg` (YouTube 預設，比 maxresdefault 穩定)

### 5. history.json 管理
- `load_history()` / `save_history(item)` / `delete_history_item(url)`
- 去重邏輯：以 `url` 欄位為鍵，新紀錄插入頭部
- 上限 50 條
- 前端透過 `GET /history` 與 `DELETE /history?url=...` API 操作

## API 端點

| Method | Path | 參數 | 說明 |
|--------|------|------|------|
| GET | `/` | — | 返回 index.html |
| POST | `/convert` | `urls` (Form), `api_key` (Form, optional), `target_lang` (Form, optional, default "中文") | 並行轉換，返回 `{results: [{url, article?, error?}]}` |
| GET | `/history` | — | 返回歷史紀錄 JSON 陣列 |
| DELETE | `/history` | `url` (query string) | 刪除指定紀錄 |

## 智能斷句與分段 (Context Window Management)

**情境：** 超長影片（1 小時以上的訪談/演講/課程）的字幕可能超過 LLM 的 token 上限（Gemini Flash 約 128K tokens），或導致回應品質下降（細節丟失、邏輯跳躍）。

### 實作方案：Map-Reduce (分段摘要再總結)

若字幕清洗後的字串長度 > 80,000 字符，應實作此流程：

#### Map 階段 (分段摘要)
1. 將清洗後字幕按 **4,000 字符** 為單位分割為若干區塊（Chunk），相鄰區塊保留 **500 字符重疊 (Overlap)** 以確保上下文連貫
2. 每個 Chunk 獨立發送 LLM 調用，Prompt 要求：
   - 提取該段落的核心論點、重要數據與金句
   - 保留該段落中提到的專有名詞、人名與時間線索
   - 標註該段落在原始影片中的主題歸屬（如「關於模型架構的討論」「關於商業模式的提問」）
3. 每個 Chunk 的可選「摘要長度」約為輸入的 20-30%

#### Reduce 階段 (跨區塊合併)
1. 將所有 Chunk 摘要按影片時間順序拼接
2. 發送第二次 LLM 調用，Prompt 要求：
   - 合併重複內容（跨 Chunk 的相同主題）
   - 將時間線結構轉化為**主題塊結構**（3-5 個核心主題）
   - 確保邏輯連貫、敘述流暢，看不出分段痕跡
3. 最終輸出完整的結構化文章

#### 替代方案：滑動窗口 (Sliding Window)
適用於 LLM 支援極長上下文（如 Gemini 1.5 Pro 的 2M tokens）時：
1. 將整個字幕送入 LLM，定義「**窗口大小 + 重疊步長**」結構
2. 在同一調用中使用 `parts` 陣列分段傳遞，利用模型的長上下文處理能力
3. 更簡單但不夠精確，較短影片適合

#### 驗證標準
- [ ] 長影片產出的文章是否有明顯斷裂感？
- [ ] 跨 Chunk 的主題是否有重複？
- [ ] 金句和關鍵數據是否完整保留（可與原始字幕交叉比對）？
- [ ] 最終文章長度是否不超過 8,000 字？

---

## 附錄 A：導出至 Apple Notes (純文字化)

*非主流程，僅在用戶明確要求將文章寫入 Apple Notes 時使用。*

Apple Notes 不支援 Markdown。導出目標是**有視覺層級的純文字**。

**轉換規則 (禁用 Markdown 符號)：**
- `# H1` → `【 標題 】` + 下方 `=` 下劃線
- `## H2` → `■ 標題` + 下方 `-` 下劃線
- `### H3` → `▶ 標題`
- `> Blockquote` → `| 引言內容`
- `- 列表` → `  • 項目`
- **禁止**殘留 `**`, `_`, `[text](url)` 等符號

**osascript 導出模式 (檔案中介，避免轉義地獄)：**
1. 將純文字寫入暫存 `.txt` 檔
2. AppleScript 透過 `open for access file bodyPath` 讀取
3. 使用 `read fRef as «class utf8»` 強制以 UTF-8 讀取（避免 MacRoman 亂碼）
4. 標題用變數傳遞，長度限制 200 字符
5. 30 秒 timeout
6. `finally` 塊確保暫存檔被清理

## 附錄 B：導出至 Obsidian

僅在用戶要求將文章存到 Obsidian 知識庫時使用。

- **目標路徑**：`/Users/lunker/Library/Mobile Documents/iCloud~md~obsidian/Documents/Laban/YouTube-Articles`
- **數據來源**：
  1. **本地 MD 檔**：直接從輸出目錄複製
  2. **history.json**：若本地無 MD 檔，讀取 `data/history.json` 最新記錄的 `title` 與 `article`
- **格式要求**：保持完整 Markdown（H1-H3, 列表, 粗體等）
- **覆蓋邏輯**：同名檔案直接覆蓋

## 坑點與注意事項 (Pitfalls)

### 字幕提取
- **語言標籤陷阱**：許多影片僅標記為 `zh` 而非 `zh-TW`，清單中缺失 `zh` 會導致找不到字幕。已修復（清單已包含 `zh`）。
- **429 速率限制**：yt-dlp 下載字幕時可能被 YouTube 限流。程式碼不設 `check=True`，僅記錄警告並嘗試使用已下載的字幕。
- **臨時目錄隔離**：每則影片使用獨立 `os.urandom(4).hex()` 子目錄，防止並行處理時文件互相覆蓋。

### 後端
- **API Key 安全**：`main.py` 第 30 行的 `LLM_API_KEY=***` 是已被刪除的殘留佔位符（2026-04-30 已修復）。實際使用 `get_api_key()` 從環境變數 `GEMINI_API_KEY` 讀取，或透過程式使用者於前端表單輸入。
- **f-string 限制**：Python f-string 的 `{}` 中禁止使用反斜槓。處理轉義字符需在外部完成。
- **路徑歧義**：啟動腳本 `start.sh` 已 `cd` 到專案目錄，使用了 `BASE_DIR` 絕對路徑。直接執行 `uvicorn app.main:app` 時需確保 CWD 是專案根目錄，否則 Jinja2 模板路徑會失敗。

### 前端
- **DOM 操作陷阱**：`changeUILanguage()` 中禁止使用 `querySelectorAll` 的索引順序。所有 DOM 操作必須有 `if (element)` 檢查，否則未捕獲錯誤會阻塞後續 JS 執行。
- **初始化阻塞**：`window.onload` 中的任一函數崩潰（如 `changeUILanguage` 選取不到元素）會導致 `loadHistory()` 等後續初始化失敗。每個函數必須有 `try...catch` 或 `if` 守衛。
- **瀏覽器快取**：前端 HTML/JS 修改後，瀏覽器快取常導致更新不生效。建議使用無痕模式或 `Cmd+Shift+R` 強制刷新。
- **縮圖 403**：若直接在前端 `<img>` 中使用 YouTube 縮圖 URL，需添加 `referrerpolicy="no-referrer"`。本工具在後端透過 `get_best_thumbnail()` 處理，無前端 403 問題。
- **Toast 與錯誤顯示區的職責分離**：成功操作（複製、下載）使用 Toast 通知；轉換失敗/部分失敗使用 `#errorDisplay` 區塊，提供詳細失敗清單。

### 長影片處理
- **Token 超限**：1 小時以上影片可能超過 Gemini Flash 的有效處理能力。應使用「智能斷句與分段」章節描述的 Map-Reduce 方案。
- **反饋體驗**：處理長影片時使用進度條和步驟文字（4 階段），避免使用者認為服務已卡死。

## 驗證標準
- [ ] 是否支援多連結並行轉換且互不干擾？
- [ ] 是否能正確識別標記為 `zh` 的字幕？
- [ ] 4 語系 UI 切換是否完整更新所有文字元素？
- [ ] API Key 是否可透過前端表單傳入且覆蓋環境變數？
- [ ] 長影片 (>1小時) 是否自動觸發分段處理？
- [ ] 歷史紀錄是否正確去重、刪除、持久化？
- [ ] 錯誤提示是否在頁面內顯示而非彈窗？
- [ ] 多結果分頁在批量轉換時是否正常切換？

## 參考資源
- [故障診斷指南](references/common-failures.md)：處理「轉換失敗」的標準診斷流程。
- [Apple Notes 導出](references/apple-notes-export.md)：完整 osascript 模式代碼。
