# YouTube to Article Skill 📺 ➡️ 📝

A professional **Hermes Agent Skill** definition for transforming YouTube videos into high-quality, structured, and readable articles.

## 📖 Overview
Unlike simple summarization tools, this skill encodes a **production-grade pipeline** for knowledge extraction. It guides the AI agent to avoid common pitfalls (such as subtitle mismatch or loss of detail) and ensures the final output maintains a professional editorial structure.

## 🛠️ The Pipeline
The skill implements a three-stage process:
1. **Extraction**: Use `yt-dlp` to fetch metadata and `.vtt` subtitles.
2. **Cleaning**: Remove WEBVTT headers, timestamps, and HTML tags; perform consecutive line deduplication.
3. **Synthesis**: Reconstruct content into "Topic Blocks" using professional written Chinese.

## 📂 Contents
- `SKILL.md`: Core definition, trigger conditions, and pitfalls.
- `references/`: Detailed guides for Apple Notes export and failure diagnosis.

## 🚀 How to Use
Import this skill into your Hermes Agent's skill library. The agent will automatically follow the encoded workflow when a user provides a YouTube link and requests an article.

## 📝 License
MIT
