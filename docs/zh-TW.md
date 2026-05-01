# YouTube 轉文章技能定義 📺 ➡️ 📝

專為 **Hermes Agent** 設計的 v3.2 技能定義檔，用於將 YouTube 影片轉化為結構化的深度文章。

## 📖 概述

本技能封裝了一套**工業級的知識提取流水線**，引導 AI Agent 避開常見的實作坑點，確保最終產出符合專業編輯結構的高品質文章。與簡單的總結工具不同，它透過「字幕提取 → 數據清洗 → LLM 重構」三階段流程，將口語化的影片內容轉化為結構清晰、易於閱讀的書面文章。

## 🛠️ 流水線概覽

### 1. Extraction（提取）

使用 `yt-dlp` 獲取 YouTube 影片的元數據與 VTT 格式字幕，並實作優先級語言匹配系統：

**優先語言清單：**
```
["zh", "zh-TW", "zh-HK", "zh-CN", "zh-Hans", "zh-Hant", "en", "ja", "ko"]
```
*注意：必須包含基礎的 `zh` 標籤，否則部分自動生成字幕無法匹配。*

**關鍵實作細節：**
- 先執行 `yt-dlp --dump-json --skip-download` 取得元數據（title, thumbnails, id）
- 再執行 `yt-dlp --write-subs --write-auto-subs --sub-langs "<langs>" --skip-download --sub-format vtt` 下載字幕
- **不設 `check=True`**：429 速率限制只記錄警告，繼續嘗試使用已下載的字幕
- 使用 `glob.glob` 遞迴搜索 `.vtt` 文件
- 每則影片使用獨立 `os.urandom(4).hex()` 臨時目錄，防止並行處理時文件互相覆蓋

### 2. Cleaning（清洗）

`clean_vtt()` 函數負責將原始 VTT 字幕轉化為乾淨的文字：

1. 移除 `WEBVTT` 頭、時間戳（`-->`）、HTML 標籤（`<c>`、`</c>` 等）
2. **連續行去重**：相鄰完全相同的行只保留第一行
3. 合併為單一字串傳遞給 LLM

### 3. Synthesis（重構）

透過 Google Gemini Flash LLM 將清洗後的字幕重新組織為主題塊結構：

**Prompt 架構要點：**
- 角色設定：頂尖科技記者與知識策展人
- 邏輯重構：提取 3-5 個核心主題，以主題塊組織
- 保留精髓：具體案例、數據、金句、故事
- 補全上下文：對影片預設已知但讀者陌生的背景補充解釋
- 語調轉換：口語轉專業書面，但保留原作者觀點色彩
- 格式：Markdown，清晰的 H1/H2/H3 結構
- **禁令**：絕對禁止 LaTeX 符號 → 統一使用 `->`

**技術細節：**
- 端點：`https://generativelanguage.googleapis.com/v1beta/models/gemini-flash-latest:generateContent`
- API Key 優先級：前端表單傳入 → 環境變數 `GEMINI_API_KEY` → 返回錯誤
- Timeout：120 秒
- 非同步調用（`httpx.AsyncClient`）

## 🚀 v3.2 增強功能

- **智慧 URL 解析**：支援逗號、空格、換行分隔，正則驗證 YouTube 連結格式
- **多結果分頁標籤**：批量轉換時以 Tab 切換各影片結果
- **4 語系 UI**：繁體中文、简体中文、English、日本語，`localStorage` 持久化使用者偏好
- **Ctrl+Enter 快捷鍵**：文字輸入區按下快捷鍵直接觸發轉換
- **Map-Reduce 分段處理**：>1 小時長影片自動觸發分段摘要再合併
- **Toast 通知**：非侵入式提示取代 `alert()` 阻塞式彈窗
- **載入進度條**：4 步驟動態文字 + 百分比進度條

### Map-Reduce 分段處理詳情

當字幕清洗後的字串長度 > 80,000 字符時自動觸發：

**Map 階段（分段摘要）：**
- 按 4,000 字符為單位分割，相鄰區塊保留 500 字符重疊以確保上下文連貫
- 每個 Chunk 獨立發送 LLM 調用，提取核心論點、數據與金句
- 每個 Chunk 摘要長度約為輸入的 20-30%

**Reduce 階段（跨區塊合併）：**
- 將所有 Chunk 摘要按時間順序拼接
- 發送第二次 LLM 調用，合併重複內容，轉化為 3-5 個核心主題
- 確保最終文章邏輯連貫、無分段痕跡，不超過 8,000 字

## 📂 內容結構

```
youtube-to-article-skill/
├── SKILL.md                        # 核心定義（觸發條件、執行步驟、前端架構、i18n、API 端點、坑點、驗證標準）
├── docs/
│   ├── zh-TW.md                    # 本文件：繁體中文技能說明
│   ├── zh-CN.md                    # 簡體中文技能說明
│   ├── en.md                       # English skill description
│   └── ja.md                       # 日本語スキル説明
└── references/
    ├── common-failures.md          # 故障診斷指南：處理「轉換失敗」的標準診斷流程
    └── apple-notes-export.md       # Apple Notes 導出指南：osascript 模式的完整實作
```

### SKILL.md 核心內容

`SKILL.md` 包含完整的技能定義，涵蓋：

- **觸發條件**：用戶提供 YouTube 連結並要求「轉成文章」、「總結成深度博文」或「將影片內容文字化」時
- **執行步驟**：從鑑別使用場景到前端顯示的完整工作流
- **前端架構**：深色主題視覺設計系統、i18n 多語系實作、關鍵交互細節
- **i18n 坑點**：禁止依賴 `querySelectorAll` 索引順序，必須使用 `id` 定位
- **API 端點表**：`GET /`、`POST /convert`、`GET /history`、`DELETE /history`
- **坑點分析**：字幕提取陷阱、後端安全注意事項、前端 DOM 操作陷阱、瀏覽器快取問題
- **驗證標準**：8 項關鍵驗證項目

## 🚀 如何使用

將此技能導入您的 Hermes Agent 技能庫。當用戶提供 YouTube 連結並要求轉化為文章時，Agent 將自動遵循定義的工程工作流執行。

### 使用模式

**A. 網頁工具（主要入口）：**
- 路徑：`/Users/lunker/youtube-article-tool`
- 啟動：執行 `start.sh`（內部呼叫 `uvicorn app.main:app --reload --port 8080`）
- 網址：`http://127.0.0.1:8080`

**B. 單次調用（腳本模式）：**
- 快速轉換一則影片，不開網頁
- 直接使用 `yt-dlp` + `call_llm` 流程，手動輸出結果

## 📝 授權

MIT
