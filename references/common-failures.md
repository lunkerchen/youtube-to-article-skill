# YouTube-to-Article 常見故障診斷

當用戶報告「所有影片轉換均失敗」或單個影片失敗時，請遵循以下診斷路徑：

## 1. 基礎工具鏈驗證
- **yt-dlp 狀態**：確認 `yt-dlp` 已安裝且版本最新。
- **連通性測試**：運行 `yt-dlp --dump-json <URL>` 驗證是否能獲取元數據。若報 403/429，可能是 YouTube 封鎖了 IP 或需要 Cookie。
- **伺服器運行**：確認 `http://127.0.0.1:8080` 是否回應。如無回應，執行 `start.sh` 重啟。

## 2. 字幕提取階段 (Extraction)
- **字幕存在性**：檢查 `temp/` 目錄下是否有 `.vtt` 文件。
- **語言匹配**：確認 `common_langs` 列表中包含 `zh`。許多影片僅標記為 `zh` 而非 `zh-TW`，缺失此標籤會導致「找不到可用字幕」。
- **429 限流**：若錯誤訊息包含 `429` 或 `Too Many Requests`，等待 1-2 分鐘後重試。

## 3. LLM 處理階段 (Synthesis)
- **API Key 驗證**：
  - 前端表單傳入優先。檢查瀏覽器的 API Key 輸入框是否為空。
  - 若無前端輸入，檢查環境變數 `GEMINI_API_KEY` 是否正確設定。
  - 注意：`main.py` 中的 `LLM_API_KEY=***` 殘留佔位符已被移除（2026-04-30 已修復），不應再出現此問題。
- **響應分析**：檢查 LLM 返回的 JSON。
  - `401/403` -> API Key 無效或權限不足
  - `429` -> 觸發速率限制 (Quota exceeded)
  - `500` -> 模型後端故障
- **Token 超限**：若影片長度 >1 小時，字幕可能超過 Gemini Flash 的有效容量。考慮實作 Map-Reduce 分段處理。

## 4. 前端問題
- **UI 語系切換失效**：檢查瀏覽器 Console 是否有 JS 錯誤。常見原因：
  - `changeUILanguage()` 中使用了 `querySelectorAll` 索引，但 DOM 結構已變更
  - `localStorage` 中儲存了無效的語言值
- **頁面顯示空白**：檢查 `app/templates/index.html` 是否存在（14-36KB）。重新整理頁面 (`Cmd+Shift+R`) 跳過快取。

## 5. 導出階段 (Export)
- **Apple Notes 權限**：確認 Terminal/Agent 擁有「自動化」Notes 的權限。
- **文本長度**：若文章極長，`osascript` 直接傳參會失敗。必須使用「寫入暫存檔 -> AppleScript 讀取文件」的模式。
- **亂碼問題**：AppleScript 必須使用 `read fRef as «class utf8»` 而非 `as string`，否則 MacRoman 編碼會導致亂碼。

## 6. 歷史紀錄問題
- **history.json 不存在**：首次使用時自動建立 `data/history.json`。若手動刪除，重啟服務即可重建。
- **前端 `data.forEach is not a function`**：通常是 `history.json` 為空陣列而非物件陣列，或格式損壞。檢查 JSON 有效性。
