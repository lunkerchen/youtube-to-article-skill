# youtube-to-article-skill

> **Hermes Agent skill** — Convert YouTube videos into structured, readable articles via two complementary pipelines.
>
> **Hermes Agent 技能** — 透過兩種互補的 pipeline 將 YouTube 影片轉化為結構清晰、可讀性高的文章。

[![GitHub](https://img.shields.io/badge/Hermes-Agent_Skill-8B5CF6)](https://hermes-agent.nousresearch.com)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

---

## Overview / 概述

A Hermes agent skill that transforms YouTube video content into well-structured articles. When invoked, the agent asks which pipeline to use:

本 Hermes Agent 技能能將 YouTube 影片內容轉為結構化文章。啓用時，agent 會詢問要用哪條 pipeline：

### Pipeline A: Web UI (Subtitle → Gemini)

| | |
|---|---|
| **Approach / 方式** | FastAPI web app — extracts subtitles via `yt-dlp`, rewrites with Google Gemini |
| **Speed / 速度** | Fast (~2 min) / 快（約 2 分鐘） |
| **Requirement / 需求** | Video must have subtitles (auto or uploaded) / 影片需有字幕 |
| **Features / 特性** | Batch processing, multi-language UI, history, Apple Notes export |
| **Port / 埠** | `http://127.0.0.1:8080` |

### Pipeline B: CLI ASR (Download → ElevenLabs Scribe)

| | |
|---|---|
| **Approach / 方式** | Downloads video audio, transcribes via ElevenLabs Scribe, then produces article |
| **Speed / 速度** | Slower (download + ASR processing) / 較慢（需下載+轉錄） |
| **Requirement / 需求** | No subtitles needed / 不需要字幕 |
| **Features / 特性** | Any language support, higher accuracy, pure CLI |
| **Output / 輸出** | `~/videos/edit/article/<title>.md` |

---

## Usage / 使用方法

### In Hermes Agent

```bash
# Load the skill and run
youtube-to-article
```

The agent will prompt you to choose between Pipeline A (Web UI) and Pipeline B (CLI ASR). You can also answer directly:

Agent 會問要用 Pipeline A（Web UI）還是 Pipeline B（CLI ASR）。也可以直接回答：

- `A` / `Web UI` / `用A`
- `B` / `CLI` / `轉錄` / `跑cli`

---

## Pipeline A Details / 詳細說明

### Project Structure / 專案結構

```
youtube-article-tool/
├── app/
│   ├── main.py                 # FastAPI backend (uvicorn)
│   └── templates/
│       └── index.html          # Tailwind CSS frontend, 4-lang i18n
├── data/
│   └── history.json            # Conversion history (dedup, max 50)
├── docs/
│   ├── README.en.md
│   ├── README.zh-Hant.md
│   ├── README.zh-Hans.md
│   └── README.ja.md
├── scripts/
│   └── export-obsidian.py      # Batch sync to Obsidian vault
├── start.sh                    # Launch script (port 8080)
├── README.md                   # Hub README
├── requirements.txt
└── .gitignore
```

### Tech Stack / 技術棧

| Component / 組件 | Technology / 技術 |
|---|---|
| Backend | FastAPI + uvicorn |
| Frontend | Tailwind CSS (CDN) + Marked.js + vanilla JS |
| Subtitles | yt-dlp |
| LLM | Google Gemini (`gemini-flash-latest`) |
| Async | `asyncio.gather` |

### How It Works / 運作流程

```
YouTube URL → yt-dlp subtitle extraction → text cleaning → Gemini LLM → structured article
```

1. **Subtitle extraction** — Parallel per-video, with language priority fallback and retry logic for 429 rate limits
2. **Text cleaning** — Strip timestamps, HTML tags, dedup consecutive lines
3. **LLM synthesis** — Reorganize spoken content into 3-5 thematic sections with proper context
4. **Display** — Web UI with tabs, Markdown rendering, copy/download, time-grouped history

### Key Technical Details / 關鍵技術細節

**Language priority / 語言優先級：**
```python
["en", "zh", "zh-TW", "zh-HK", "zh-CN", "zh-Hans", "zh-Hant", "ja", "ko"]
```

**LLM endpoint / LLM 端點：**
```
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-flash-latest:generateContent
```

**API Key resolution / 金鑰解析順序：**
1. Frontend form input / 前端表單傳入
2. Environment variable `GEMINI_API_KEY` / 環境變數
3. Error return / 返回錯誤

### Audio Pipeline

| Method | Path | Params | Description |
|--------|------|--------|-------------|
| GET | `/` | — | Serve index.html |
| POST | `/convert` | `urls`, `api_key`?, `target_lang`? | Parallel conversion |
| GET | `/history` | — | History JSON |
| DELETE | `/history` | `url` | Delete record |

### Frontend Features / 前端功能

- **Theme System / 主題系統：** `dark` (default) + `highcontrast` (no light theme)
- **i18n / 多語系：** zh-TW, zh-CN, en, ja — browser language auto-detection
- **Toasts / 通知：** 3 types (info/success/error), progress bar, hover-pause
- **Skeleton loading / 骨架載入：** Shimmer animation during conversion
- **History / 歷史：** Time-grouped (Today/Yesterday/This Week/Earlier)
- **Font size / 字體：** Minimum 12px (`text-xs`), no smaller

---

## Pipeline B Details / 詳細說明

### Workflow / 工作流程

```
YouTube URL → video download → ElevenLabs Scribe ASR → transcript packing → LLM → article
```

### Directory Layout / 目錄結構

```
~/videos/edit/article/
├── downloads/          # Raw video (deleted after processing)
├── transcripts/        # Scribe JSON transcripts (kept)
├── takes_packed.md     # Readable formatted transcript
└── <title>.md          # Final article
```

### Step-by-Step / 逐步流程

| Step / 步驟 | Command / 指令 | Output / 輸出 |
|---|---|---|
| 1. Info / 資訊 | `yt-dlp --print ...` | Video metadata |
| 2. Download / 下載 | `yt-dlp -f best ...` | MP4 file |
| 3. Transcribe / 轉錄 | `transcribe.py --num-speakers N` | Scribe JSON |
| 4. Pack / 打包 | `pack_transcripts.py` | `takes_packed.md` |
| 5. Write / 寫作 | Manual analysis + LLM | `<title>.md` → Obsidian |

### Writing Guidelines / 寫作規範

| Do / 要 | Don't / 不要 |
|---|---|
| Hook opening / 開頭吸引人 | Add information not in video / 添加原文沒有的資訊 |
| Logical sections with H2/H3 / 邏輯分段 | Embellish or exaggerate / 美化或誇大 |
| Preserve data, quotes, stories / 保留數據與金句 | Omit key arguments / 遺漏重要論點 |
| Spoken → written tone / 口語轉書面 | Insert personal commentary / 加入個人評論 |
| Include original video link / 結尾附連結 | — |

### Cleanup / 清理

```bash
# After article is written
rm -f "$VIDEO_PATH"           # Delete raw video (~100-200 MB)
rm -f "${VIDEO_PATH%.*}.wav"  # Delete WAV temp (~40-50 MB)
# Transcript JSON + article are kept
```

---

## Common Pitfalls / 常見問題

### Pipeline A

| Issue / 問題 | Solution / 解法 |
|---|---|
| No subtitles found / 找不到字幕 | Try `--sub-langs all`; check video CC; ensure `en` is first in language list |
| 429 rate limit / 限流 | Verify `--ignore-errors` flag; reduce language count |
| BrokenPipeError | Built-in retry (3x); check yt-dlp + Python versions |
| LLM output too long / 輸出過長 | Auto-triggers Map-Reduce at 80K chars |
| Server won't start / 無法啟動 | Check port 8080: `lsof -i :8080` |

### Pipeline B

| Issue / 問題 | Solution / 解法 |
|---|---|
| yt-dlp fails / 失敗 | Check URL validity, network |
| Scribe fails / 轉錄失敗 | Check ELEVENLABS_API_KEY quota |
| pack_transcripts can't find JSON | Confirm JSON copied to `transcripts/` |
| Obsidian path missing | Skip that step |
| Non-English video / 非英文 | Scribe auto-detects language |

---

## License / 授權

MIT — see [LICENSE](LICENSE).

---

*Part of the [Hermes Agent](https://hermes-agent.nousresearch.com) skill ecosystem.*
