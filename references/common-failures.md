# YouTube-to-Article 常見故障診斷

當用戶報告「所有影片轉換均失敗」或單個影片失敗時，請遵循以下診斷路徑：

## 1. 基礎工具鏈驗證
- **yt-dlp 狀態**：確認 `yt-dlp` 已安裝且版本最新。
- **連通性測試**：運行 `yt-dlp --dump-json <URL>` 驗證是否能獲取元數據。若報 403/429，可能是 YouTube 封鎖了 IP 或需要 Cookie。

## 2. 字幕提取階段 (Extraction)
- **字幕存在性**：檢查 `temp/` 目錄下是否有 `.vtt` 文件。
- **語言匹配**：確認 `common_langs` 列表中包含 `zh`。許多影片僅標記為 `zh` 而非 `zh-TW`，缺失此標籤會導致「找不到可用字幕」。

## 3. LLM 處理階段 (Synthesis)
- **API Key 驗證**：檢查 `LLM_API_KEY` 是否被佔位符（如 `"***"`）覆蓋。
- **響應分析**：檢查 LLM 返回的 JSON。
    - `401/403` $\rightarrow$ API Key 無效或權限不足。
    - `429` $\rightarrow$ 觸發速率限制 (Quota exceeded)。
    - `500` $\rightarrow$ 模型後端故障。

## 4. 導出階段 (Export)
- **Apple Notes 權限**：確認 Terminal/Agent 擁有「自動化」Notes 的權限。
- **文本長度**：若文章極長，`osascript` 直接傳參會失敗。必須使用「寫入暫存檔 $\rightarrow$ AppleScript 讀取文件」的模式。
