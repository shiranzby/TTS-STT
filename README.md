<p align="center">
  <img src="https://img.shields.io/badge/Shiran-Voice-0891b2?style=for-the-badge&logo=cloudflare&logoColor=white" alt="Shiran Voice">
</p>

<h1 align="center">🎙️ Shiran Voice</h1>
<p align="center"><strong>AI 语音处理平台 — 文字转语音 · 语音转文字 · 批量处理</strong></p>

<p align="center">
  <a href="https://github.com/shiranzby/TTS-STT/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-green" alt="License"></a>
  <a href="https://tts.shy2958779577.workers.dev"><img src="https://img.shields.io/badge/demo-%E5%9C%A8%E7%BA%BF%E4%BD%93%E9%AA%8C-22d3ee" alt="Demo"></a>
  <img src="https://img.shields.io/badge/version-1.0.0-0891b2" alt="Version">
  <img src="https://img.shields.io/badge/platform-Cloudflare_Workers-orange" alt="Platform">
  <img src="https://img.shields.io/github/stars/shiranzby/TTS-STT?style=social" alt="Stars">
</p>

<p align="center">
  <a href="#-快速开始">快速开始</a> ·
  <a href="#-技术架构">技术架构</a> ·
  <a href="#-API-文档">API 文档</a> ·
  <a href="#-设计特色">设计特色</a> ·
  <a href="#-项目结构">项目结构</a>
</p>

---

## 📺 在线体验

> 🌐 **在线地址**：https://tts.shy2958779577.workers.dev

| 文字转语音 | 语音转文字 | 批量处理 |
|:---:|:---:|:---:|
| 21 种中文语音 · 11 种风格 | mp3/wav/m4a 等格式 | 多文件拖拽 · 进度追踪 |
| 语速/音调可调 | 支持自定义 API Token | 逐个下载 / 全部导出 |
| 长文本自动分块并行合成 | 转录结果一键复制 | 🔄 失败重试 |

<p align="center">
  <sub>💡 点击任意按钮切换日/夜模式 · 支持 8 种界面语言</sub>
</p>

---

## 🎯 功能概览

| 功能 | TTS（文字转语音） | STT（语音转文字） |
|------|:---:|:---:|
| **输入方式** | 手动输入 / .txt文件 / .md文件批量 | 音频文件拖拽 / 多文件批量 |
| **语音/模型** | Microsoft Edge TTS (21种中文语音) | SiliconFlow SenseVoiceSmall |
| **参数控制** | 语音选择 + 风格 + 语速 + 音调 | 默认 Token / 自定义 API Token |
| **结果操作** | ▶ 在线播放 · 📥 下载MP3 | 📋 复制 · 📥 下载TXT · 🔍 展开全文 |
| **批量处理** | ✅ 多文件逐个生成 · 📦 全部下载 | ✅ 多文件并行转录 · 📦 全部导出 |
| **断点续传** | ✅ 失败重试 · 已完成跳过 | ✅ 失败重试 · 已完成跳过 |
| **文件限制** | 单文件最大 100MB，内部自动分块 | 单文件最大 100MB |

---

## 🚀 快速开始

### 前置要求

- [Node.js](https://nodejs.org/) ≥ 18
- [Cloudflare 账号](https://dash.cloudflare.com/)
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/)：`npm install -g wrangler`

### 一键部署

```bash
# 1. 克隆仓库
git clone https://github.com/shiranzby/TTS-STT.git
cd TTS-STT

# 2. 安装依赖
npm install

# 3. 构建（将前端 HTML 嵌入 Worker 代码）
npm run build

# 4. 部署到 Cloudflare Workers
npx wrangler login        # 登录 Cloudflare 账号
npx wrangler deploy       # 部署
```

部署成功后终端会输出你的 Workers 域名：
```
https://tts.<你的子域名>.workers.dev
```

### 配置 STT API Token（可选）

语音转录功能依赖硅基流动 API，建议配置自己的 Token：

```bash
npx wrangler secret put SILICONFLOW_TOKEN
```

不配置也可使用内置默认 Token（有频率限制）。

---

## 🧱 技术架构

```
┌─────────────────────────────────────────────────────┐
│                   浏览器 (前端)                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐   │
│  │ 文本输入  │  │ 文件上传  │  │ 批量多文件处理    │   │
│  └────┬─────┘  └────┬─────┘  └───────┬──────────┘   │
│       └──────────────┼────────────────┘              │
│                      ▼                               │
│            POST /v1/audio/speech                     │
│            POST /v1/audio/transcriptions              │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────┐
│              Cloudflare Worker (边缘计算)              │
│                                                       │
│  ┌─────────────┐    ┌──────────────┐    ┌─────────┐ │
│  │ Edge TTS    │    │ 文本自动分块  │    │ 硅基流动 │ │
│  │ (微软语音)  │───→│ (≤1500字符)  │───→│ STT API │ │
│  └──────┬──────┘    └──────┬───────┘    └────┬────┘ │
│         │                  │                  │      │
│         ▼                  ▼                  ▼      │
│     MP3 音频        合并 MP3 流          JSON 文本    │
└─────────┼──────────────────┼──────────────────┼──────┘
          │                  │                  │
          ▼                  ▼                  ▼
      ▶ 在线播放         📥 批量下载        📋 复制文本
```

### TTS 工作流程

1. 接收文本 / 上传文件
2. `optimizedTextSplit()` 按句子智能分割（每块 ≤ 1500 字符）
3. 每批 3 块并行请求 Edge TTS（块间间隔 200ms，批次间隔 800ms）
4. 合并所有 MP3 分片为单一音频流返回

### STT 工作流程

1. 接收音频文件（验证格式和大小）
2. 转发至硅基流动 SenseVoiceSmall 模型
3. 清洗响应文本（去除 `<|...|>` 特殊标记和 emoji）
4. 返回纯文本结果

---

## 📡 API 文档

### `GET /`

返回完整的前端页面（HTML）。浏览器打开即可使用全部功能。

### `POST /v1/audio/speech` — 文字转语音

**JSON 模式（适用于短文本调用）：**

```bash
curl -X POST https://tts.<你的域名>.workers.dev/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{
    "input": "你好，欢迎使用 Shiran Voice。",
    "voice": "zh-CN-XiaoxiaoNeural",
    "speed": "1.0",
    "pitch": "0",
    "style": "general"
  }' \
  --output speech.mp3
```

**Multipart 模式（适用于文件上传）：**

```bash
curl -X POST https://tts.<你的域名>.workers.dev/v1/audio/speech \
  -F "file=@text.txt" \
  -F "voice=zh-CN-XiaoxiaoNeural" \
  -F "speed=1.0" \
  -F "pitch=0" \
  -F "style=general" \
  --output speech.mp3
```

**请求参数：**

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `input` / `file` | string / file | — | 要合成的文本或文本文件 |
| `voice` | string | `zh-CN-XiaoxiaoNeural` | 21 种中文语音可选 |
| `speed` | float | `1.0` | 语速 (0.5~2.0) |
| `pitch` | integer | `0` | 音调 (-50~+50) |
| `style` | string | `general` | 11 种语音风格可选 |

**响应：** `audio/mpeg` — MP3 音频流。

### `POST /v1/audio/transcriptions` — 语音转文字

```bash
curl -X POST https://tts.<你的域名>.workers.dev/v1/audio/transcriptions \
  -F "file=@audio.mp3" \
  -F "token=sk-your-token"
```

**请求参数：**

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `file` | file | 是 | 音频文件（支持 mp3/wav/m4a/flac/aac/ogg/webm/amr/3gp） |
| `token` | string | 否 | 自定义硅基流动 API Token |

**响应：**
```json
{
  "text": "转录结果文本（已去除特殊标记）"
}
```

---

## ✨ 设计特色

### 🎨 界面设计
- **纯原生实现** — 零框架依赖，HTML5 + CSS3 + 原生 JavaScript
- **暗色/亮色主题** — 一键切换，自动保存偏好
- **响应式布局** — 桌面端 / 移动端自适应
- **8 国语言** — 中文 · English · 日本語 · 한국어 · Español · Français · Deutsch · Русский

### 🚀 批量处理
- 支持多文件拖拽和追加
- 每文件独立进度状态（待处理 → 处理中 → ✅ 完成 / ❌ 失败）
- 已完成文件可跳过，失败文件可单独重试
- 全部完成后一键导出（TTS 批量下载 / STT 全部复制）

### 🔧 稳定性
- 长文本自动分块并行合成，突破单次请求限制
- 内置重试机制（429 限频 / 5xx 服务端错误自动重试）
- 单文件 100MB 上限，日主题切换流畅
- 生成历史 / 转录历史自动保存至 localStorage

---

## 📁 项目结构

```
TTS-STT/
├── src/
│   ├── template.html        # 完整前端 (HTML + CSS + JS)
│   ├── worker.js             # Cloudflare Worker 后端逻辑
│   └── worker_base.js        # 旧版 Worker（参考）
├── build.js                  # 构建脚本（内联 HTML → Worker）
├── index.js                  # 构建产物（单文件 Worker）
├── package.json              # 项目配置
├── wrangler.toml             # Wrangler 部署配置
├── LICENSE                   # MIT 许可证
├── .gitignore
└── README.md
```

**构建原理：** `build.js` 读取 `src/template.html` 和 `src/worker.js`，将 HTML 内联到 Worker 代码的 `/* __HTML_PAGE__ */` 占位符处，输出单文件 `index.js` 用于 Cloudflare 部署。

---

## 🔧 本地开发

```bash
# 本地预览
npm run build
npx wrangler dev

# 部署到生产环境
npm run deploy

# 设置环境变量（STT API Token）
npx wrangler secret put SILICONFLOW_TOKEN

# 多环境部署
npm run build
npx wrangler deploy --env=production
npx wrangler deploy --env=staging
```

---

## 📝 License

MIT © Shiran

---

<p align="center">
  <sub>Made with ❤️ by Shiran</sub>
</p>
