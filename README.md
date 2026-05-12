# youtube-to-article-skill

Hermes Agent skill for converting YouTube videos into structured articles.

Two pipelines available:

## Pipeline A: Web UI (Subtitle + Gemini)
FastAPI web app — extracts subtitles via yt-dlp, rewrites with Google Gemini.
Fast (< 2 min), needs subtitles, supports batch processing.

## Pipeline B: CLI ASR (Download + Scribe)
Downloads video, transcribes via ElevenLabs Scribe, then produces article.
Works without subtitles, supports any language, more accurate but slower.

## Usage in Hermes

Load the skill and run:

```
youtube-to-article
```

You'll be prompted to choose pipeline A or B.

## License

MIT
