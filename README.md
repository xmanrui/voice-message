# Voice Message

An [OpenClaw](https://github.com/openclaw/openclaw) skill for sending voice messages across chat channels using text-to-speech.

Supports Telegram, Discord, Feishu/Lark, Signal, WhatsApp, and more.

## Features

- Text-to-speech via [edge-tts](https://github.com/rany2/edge-tts) (Microsoft Edge TTS, no API key required)
- Audio conversion via ffmpeg (ogg/opus format)
- Per-channel sending logic:
  - **Generic** (Telegram, Signal, WhatsApp): `message` tool with `asVoice=true`
  - **Feishu/Lark**: Native API upload + audio message (voice bubble, not file attachment)
  - **Discord**: Native API with waveform + IS_VOICE_MESSAGE flag

## Prerequisites

- [edge-tts](https://github.com/rany2/edge-tts): `pip install edge-tts`
- [ffmpeg](https://ffmpeg.org/) / ffprobe

## Quick Start

### 1. Generate voice file

```bash
# Chinese (default voice: zh-CN-XiaoxiaoNeural)
scripts/gen_voice.sh "你好世界" /tmp/voice.ogg

# English
scripts/gen_voice.sh "Hello world" /tmp/voice.ogg en-US-JennyNeural
```

### 2. Send

**Telegram / Signal / WhatsApp:**
```
action=send, asVoice=true, filePath=/tmp/voice.ogg
```

**Feishu/Lark:**
```bash
scripts/send_feishu_voice.sh /tmp/voice.ogg <receive_id> <tenant_access_token>
```

> ⚠️ Feishu does NOT support `asVoice=true` via the message tool. You must use the native API (upload + send) to get a voice bubble instead of a file attachment.

**Discord:**
```bash
# Generate waveform (required by Discord)
python3 scripts/gen_waveform.py /tmp/voice.ogg
# Then call Discord API with flags=8192 (IS_VOICE_MESSAGE)
```

## Default Voices

| Language | Voice | Type |
|----------|-------|------|
| Chinese (Mandarin) | zh-CN-XiaoxiaoNeural | Female, warm |
| English (US) | en-US-JennyNeural | Female, natural |
| English (UK) | en-GB-SoniaNeural | Female |
| Japanese | ja-JP-NanamiNeural | Female |
| Korean | ko-KR-SunHiNeural | Female |

Full list: `edge-tts --list-voices` or see [references/voices.md](references/voices.md)

## Install

```bash
npx clawhub install voice-message
```

Or manually copy to `~/.openclaw/skills/voice-message/`.

## License

MIT

---

# 语音消息

一个 [OpenClaw](https://github.com/openclaw/openclaw) 技能，用于通过文字转语音在各聊天渠道发送语音消息。

支持 Telegram、Discord、飞书/Lark、Signal、WhatsApp 等。

## 功能特点

- 通过 [edge-tts](https://github.com/rany2/edge-tts) 实现文字转语音（微软 Edge TTS，无需 API Key）
- 通过 ffmpeg 进行音频转换（ogg/opus 格式）
- 各渠道独立发送逻辑：
  - **通用**（Telegram、Signal、WhatsApp）：使用 `message` 工具的 `asVoice=true`
  - **飞书/Lark**：原生 API 上传 + 音频消息（语音气泡，而非文件附件）
  - **Discord**：原生 API，带 waveform + IS_VOICE_MESSAGE 标志

## 前置依赖

- [edge-tts](https://github.com/rany2/edge-tts)：`pip install edge-tts`
- [ffmpeg](https://ffmpeg.org/) / ffprobe

## 快速开始

### 1. 生成语音文件

```bash
# 中文（默认声音：zh-CN-XiaoxiaoNeural 小晓）
scripts/gen_voice.sh "你好世界" /tmp/voice.ogg

# 英文
scripts/gen_voice.sh "Hello world" /tmp/voice.ogg en-US-JennyNeural
```

### 2. 发送

**Telegram / Signal / WhatsApp：**
```
action=send, asVoice=true, filePath=/tmp/voice.ogg
```

**飞书/Lark：**
```bash
scripts/send_feishu_voice.sh /tmp/voice.ogg <receive_id> <tenant_access_token>
```

> ⚠️ 飞书不支持 message 工具的 `asVoice=true`，必须使用原生 API（上传 + 发送）才能发出语音气泡，否则会变成文件附件。

**Discord：**
```bash
# 生成 waveform（Discord 必需）
python3 scripts/gen_waveform.py /tmp/voice.ogg
# 然后调用 Discord API，flags=8192（IS_VOICE_MESSAGE）
```

## 默认声音

| 语言 | 声音 | 类型 |
|------|------|------|
| 中文（普通话） | zh-CN-XiaoxiaoNeural | 女声，温暖 |
| 英文（美式） | en-US-JennyNeural | 女声，自然 |
| 英文（英式） | en-GB-SoniaNeural | 女声 |
| 日语 | ja-JP-NanamiNeural | 女声 |
| 韩语 | ko-KR-SunHiNeural | 女声 |

完整列表：`edge-tts --list-voices` 或查看 [references/voices.md](references/voices.md)

## 安装

```bash
npx clawhub install voice-message
```

或手动复制到 `~/.openclaw/skills/voice-message/`。

## 许可证

MIT
