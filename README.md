# YouTube to Article Skill 📺 ➡️ 📝

A professional **Hermes Agent Skill** definition for transforming YouTube videos into high-quality, structured, and readable articles.

---

# YouTube 轉文章技能定義 📺 ➡️ 📝

這是一個專為 **Hermes Agent** 設計的技能定義檔，旨在將 YouTube 影片轉化為高品質、結構化且易於閱讀的深度文章。

## 📖 Overview / 概述

Unlike simple summarization tools, this skill encodes a **production-grade pipeline** for knowledge extraction. It guides the AI agent to avoid common pitfalls (such as subtitle mismatch or loss of detail) and ensures the final output maintains a professional editorial structure.

與簡單的總結工具不同，本技能封裝了一套**工業級的知識提取流水線**。它引導 AI Agent 避開常見的實作坑點（如字幕不匹配或細節丟失），確保最終產出符合專業編輯結構的深度文章。

## 🛠️ The Pipeline / 處理流水線

The skill implements a three-stage process:
本技能實作了三個階段的處理流程：

1.  **Extraction (提取)**: 
    - Use `yt-dlp` to fetch metadata and `.vtt` subtitles.
    - Implement a priority-based language matching system (Traditional Chinese -> Simplified Chinese -> English -> etc.).
    - 使用 `yt-dlp` 獲取元數據與 `.vtt` 字幕。
    - 實作基於優先級的語言匹配系統（繁體中文 -> 簡體中文 -> 英文 -> 等）。

2.  **Cleaning (清洗)**: 
    - Remove WEBVTT headers, timestamps, and HTML tags.
    - Perform consecutive line deduplication to remove repetitive AI-generated captions.
    - 移除 WEBVTT 頭部、時間戳與 HTML 標籤。
    - 執行連續行去重，消除自動生成字幕中常見的重複內容。

3.  **Synthesis (綜合重構)**: 
    - Instead of linear summarizing, the agent is guided to reconstruct content into "Topic Blocks".
    - Preserve specific cases, data, and gold quotes while converting spoken language to professional written Chinese.
    - 非線性總結，而是引導 Agent 將內容重構為「主題塊」。
    - 在將口語轉化為專業書面中文的同時，完整保留具體案例、數據與金句。

## 📂 Repository Structure / 專案結構

- `SKILL.md`: The core definition file. Contains trigger conditions, detailed execution steps, and a comprehensive "Pitfalls" section.
- `references/`: Supporting documentation for advanced implementations:
    - `apple-notes-export.md`: Guide for clean-text export to Apple Notes.
    - `common-failures.md`: Diagnostic workflow for conversion failures.

- `SKILL.md`: 核心定義文件。包含觸發條件、詳細執行步驟以及詳盡的「坑點分析」。
- `references/`: 進階實作的參考文檔：
    - `apple-notes-export.md`: 導出至 Apple Notes 的純文字化指南。
    - `common-failures.md`: 轉換失敗的故障診斷流程。

## 🚀 How to Use / 如何使用

### For Agent Developers / 給開發者
Import this skill into your Hermes Agent's skill library. The agent will automatically follow the encoded workflow when a user provides a YouTube link and requests an article.
將此技能導入您的 Hermes Agent 技能庫。當用戶提供 YouTube 連結並要求轉化為文章時，Agent 將自動遵循定義的工程工作流執行。

### For Users / 給用戶
Once the skill is enabled, simply tell your agent: 
*"Convert this video into a structured article: [YouTube URL]"*
一旦啟用此技能，只需告訴您的 Agent：
*「將這支影片轉化為結構化文章：[YouTube 連結]」*

## 📝 License / 授權
MIT
