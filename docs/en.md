# YouTube to Article Skill

A professional Hermes Agent Skill definition for the v3.2 YouTube-to-Article tool — a FastAPI web app that transforms YouTube videos into structured articles.

## Pipeline Overview
1. Extraction: yt-dlp metadata + VTT subtitle download with 9-language priority chain
2. Cleaning: Remove WEBVTT headers, timestamps, HTML tags, consecutive-line dedup
3. Synthesis: Google Gemini Flash restructuring into thematic blocks (H1/H2/H3), preserving data and quotes

## v3.2 Enhancements
- Smart URL parsing for messy paste input
- Multi-result tabs for batch conversion
- 4-language UI (ZH-TW, ZH-CN, EN, JA) with localStorage persistence
- Ctrl+Enter keyboard shortcut
- Map-Reduce chunking for >1hr videos
- Toast notifications instead of alert()

## Repository Contents
- SKILL.md: Core definition with trigger conditions, pipeline steps, frontend architecture, i18n details, API endpoints, pitfalls, and verification standards
- references/common-failures.md: Troubleshooting guide
- references/apple-notes-export.md: osascript pattern for Apple Notes

## How to Use
Import SKILL.md into your Hermes Agent library. When a user provides a YouTube URL and requests an article, the agent follows the encoded workflow automatically.

## License
MIT
