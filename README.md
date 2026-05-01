# YouTube to Article Skill 📺 ➡️ 📝

A professional **Hermes Agent Skill** definition for transforming YouTube videos into high-quality, structured articles via a v3.2 FastAPI web app.

## Language Versions

- [English](./docs/en.md)
- [繁體中文](./docs/zh-TW.md)
- [日本語](./docs/jp.md)

---

## 🛠️ What's Inside

| File | Purpose |
|------|---------|
| `SKILL.md` | Core Hermes Agent skill definition with triggers, pipeline, pitfalls, and verification |
| `references/common-failures.md` | Troubleshooting guide for conversion failures |
| `references/apple-notes-export.md` | osascript pattern for clean Apple Notes export |

## Key Features

- Parallel conversion via `asyncio.gather` with isolated temp directories
- Smart subtitle matching with 9-language priority chain
- 4-language UI (ZH-TW, ZH-CN, EN, JA) with localStorage persistence
- Map-Reduce chunking for videos over 1 hour
- Apple Notes & Obsidian export workflows

## 🚀 How to Use

1. **Clone** this repo
2. **Import** `SKILL.md` into your Hermes Agent skill library
3. **Trigger** your agent: *"Convert this video into an article: [URL]"*

## 📝 License

MIT
