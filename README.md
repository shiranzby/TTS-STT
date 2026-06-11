<p align="center">
  <img src="https://img.shields.io/badge/Shiran-Voice-0891b2?style=for-the-badge&logo=cloudflare&logoColor=white" alt="Shiran Voice">
</p>

<h1 align="center">🎙️ Shiran Voice</h1>
<p align="center"><strong>AI-powered voice processing platform — TTS + STT, built on Cloudflare Workers</strong></p>

<p align="center">
  <a href="https://github.com/shiranzby/Shiran-Voice/blob/master/LICENSE"><img src="https://img.shields.io/badge/license-MIT-green.svg" alt="License"></a>
  <a href="https://tts.shy2958779577.workers.dev"><img src="https://img.shields.io/badge/demo-online-22d3ee.svg" alt="Demo"></a>
  <img src="https://img.shields.io/badge/version-1.0.0-0891b2.svg" alt="Version">
  <img src="https://img.shields.io/badge/platform-Cloudflare_Workers-orange.svg" alt="Platform">
</p>

---

## 📖 Introduction

Shiran Voice is a lightweight, zero-cost AI voice platform deployed entirely on Cloudflare Workers. It integrates both **Text-to-Speech** (Microsoft Edge TTS, 20+ voices) and **Speech-to-Text** (SiliconFlow SenseVoiceSmall), supporting batch processing with real-time progress tracking.

**Key highlights:**
- ✅ 100% free — runs on Cloudflare Workers free tier
- 📦 Multi-file batch TTS/STT with progress tracking
- 🌐 8-language i18n (EN/ZH/JA/KO/ES/FR/DE/RU)
- 🌓 Dark/light theme toggle
- 📋 STT transcription history with localStorage
- 🔄 Retry on failure
- 🚀 Single-command deploy

---

## 🧰 Tech Stack

| Layer | Technology |
|-------|-----------|
| **Runtime** | Cloudflare Workers (V8 isolates) |
| **Frontend** | Vanilla HTML5 + CSS3 + JavaScript (0 dependencies) |
| **TTS Engine** | Microsoft Edge TTS (SSML + HMAC auth) |
| **STT Engine** | SiliconFlow API (SenseVoiceSmall) |
| **Build** | Node.js inline builder |
| **Deploy** | Wrangler CLI |

---

## 🔄 Workflow

```
┌─────────────────────────────────────────────────────┐
│                    Client (Browser)                  │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐ │
│  │ Text In  │  │ File In  │  │ Batch Multi-File  │ │
│  └────┬─────┘  └────┬─────┘  └────────┬──────────┘ │
│       └──────────────┼────────────────┘             │
│                      ▼                              │
│            POST /v1/audio/speech                    │
│                  (JSON / FormData)                   │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────┐
│              Cloudflare Worker (worker.js)            │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐ │
│  │ Edge TTS    │  │ Text Split  │  │ SiliconFlow  │ │
│  │ (MS Speech) │  │ (≤1500 ch)  │  │ (STT API)    │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬───────┘ │
│         │                │                 │          │
│         ▼                ▼                 ▼          │
│     MP3 Blob      Merged MP3          JSON Text       │
└─────────┼────────────────┼─────────────────┼──────────┘
          │                │                 │
          ▼                ▼                 ▼
      Client Play    Client Download    Client Display
```

**TTS pipeline:**
1. Text/file uploaded → worker reads content
2. `optimizedTextSplit()` splits by sentences (max 1500 chars/chunk)
3. Chunks processed in parallel batches of 3 (200ms delay)
4. All MP3 blobs merged → single response

**STT pipeline:**
1. Audio file uploaded → worker forwards to SiliconFlow
2. Response text cleaned (removes `<|...|>` tags + emoji)
3. JSON response with plain text

---

## 🚀 Quick Deploy

### Prerequisites
- [Node.js](https://nodejs.org/) ≥ 18
- [Cloudflare account](https://dash.cloudflare.com/)
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/) (`npm install -g wrangler`)

### 1. Clone & Install
```bash
git clone https://github.com/shiranzby/Shiran-Voice.git
cd Shiran-Voice
npm install
```

### 2. Configure
```toml
# wrangler.toml
name = "tts"
main = "index.js"
compatibility_date = "2024-01-15"
compatibility_flags = ["nodejs_compat"]
```

Set STT API token (optional):
```bash
npx wrangler secret put SILICONFLOW_TOKEN
```

### 3. Build & Deploy
```bash
npm run build    # Builds index.js from src/
npm run deploy   # Deploys to Cloudflare Workers
```

Your site will be live at `https://tts.<your-subdomain>.workers.dev`.

> **Note:** The Edge TTS integration uses a built-in HMAC key for Microsoft Speech Services. No additional API key is needed for TTS.

---

## 📁 File Structure

```
Shiran-Voice/
├── src/
│   ├── template.html     # Full frontend (HTML + CSS + JS, ~3200 lines)
│   ├── worker.js          # Cloudflare Worker backend (~625 lines)
│   └── worker_base.js     # Legacy worker version
├── build.js               # Inline builder: embeds HTML into worker
├── index.js               # Generated single-file worker (build output)
├── package.json           # Project config (v1.0.0)
├── wrangler.toml          # Wrangler deploy config
├── LICENSE                # MIT
├── .gitignore
└── README.md
```

**Build system:** `build.js` reads `src/template.html` + `src/worker.js`, embeds the HTML as a JS string replacing the `/* __HTML_PAGE__ */` placeholder, and outputs a single `index.js` ready for Cloudflare Workers deployment.

---

## 🎯 Features

### TTS (Text-to-Speech)
| Feature | Detail |
|---------|--------|
| Voices | 21 Chinese voices (Xiaoxiao, Yunxi, etc.) |
| Styles | 11 styles (general, assistant, chat, newscast, etc.) |
| Controls | Speed / Pitch sliders with live preview |
| Input | Manual text or .txt/.md file upload |
| Output | Inline audio player + MP3 download |
| Batch | Multiple files → per-file play/download → bulk ZIP download |
| Long text | Auto sentence-level splitting, parallel chunk processing |

### STT (Speech-to-Text)
| Feature | Detail |
|---------|--------|
| Formats | mp3, wav, m4a, flac, aac, ogg, webm, amr, 3gp |
| Limit | 100 MB per file |
| Token | Default + custom SiliconFlow API token |
| Result | View / Copy / Edit / Convert to TTS |
| Batch | Multiple files → per-file expand/copy/download |
| History | Auto-saved to localStorage, 50 entries |

### UX
| Feature | Detail |
|---------|--------|
| i18n | 8 languages, auto-detect browser locale |
| Theme | Dark/light toggle with smooth animation |
| Toast | Slide-in notifications (success/error) |
| History | TTS generation history, 50 entries, restore |

---

## 🔧 API Endpoints

### `GET /`
Returns the HTML frontend page.

### `POST /v1/audio/speech`
Generate speech from text.

**JSON mode:**
```json
{
  "input": "Hello, world!",
  "voice": "zh-CN-XiaoxiaoNeural",
  "speed": "1.0",
  "pitch": "0",
  "style": "general"
}
```
Returns: `audio/mpeg` blob.

**Multipart mode:**
```
file: .txt/.md file
voice, speed, pitch, style: optional parameters
```
Returns: `audio/mpeg` blob.

### `POST /v1/audio/transcriptions`
Transcribe audio to text.

```
file: audio file (mp3/wav/m4a/flac/aac/ogg/webm/amr/3gp)
token: (optional) custom SiliconFlow API token
```
Returns: `{ "text": "transcription result" }`.

---

## 📝 License

MIT © Shiran

---

## 🙏 Credits

- **TTS Engine** — Microsoft Edge TTS (Cognitive Services)
- **STT Engine** — [SiliconFlow](https://siliconflow.cn) (SenseVoiceSmall)

---

<p align="center">
  <sub>Made with ❤️ by Shiran · <a href="https://tts.shy2958779577.workers.dev">Live Demo</a></sub>
</p>
