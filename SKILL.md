---
name: voice-message
version: 1.0.0
description: Send voice messages across chat channels (Telegram, Discord, Feishu/Lark, Signal, WhatsApp, and others) using edge-tts for text-to-speech and ffmpeg for audio conversion. 发送语音消息到各聊天渠道。IMPORTANT - Feishu/Lark does NOT support asVoice=true via the message tool; you MUST use this skill to send voice messages on Feishu, otherwise it will send a file attachment instead of a voice bubble.
---

# Voice Message

Send text as voice messages to any chat channel.

## Prerequisites

- `edge-tts` — Microsoft Edge TTS (pip install edge-tts)
- `ffmpeg` / `ffprobe` — audio conversion and duration detection

## Default Voices

- Chinese: `zh-CN-XiaoxiaoNeural`
- English: `en-US-JennyNeural`
- Other languages: see [references/voices.md](references/voices.md)

## Step 1: Generate Voice File

Use `scripts/gen_voice.sh` to convert text to an ogg/opus file:

```bash
scripts/gen_voice.sh "你好" /tmp/voice.ogg
scripts/gen_voice.sh "Hello" /tmp/voice.ogg en-US-JennyNeural
```

Arguments: `<text> <output.ogg> [voice]`
- If voice is omitted, defaults to `zh-CN-XiaoxiaoNeural`.

## Step 2: Send by Channel

### Generic (Telegram, Signal, WhatsApp, etc.)

Use the message tool directly:

```
action=send, asVoice=true, filePath=/tmp/voice.ogg
```

This works for most channels. Telegram confirmed working.

### Feishu/Lark

Feishu does NOT support `asVoice=true` via the message tool. You must use the native API (two-step: upload + send).

Use `scripts/send_feishu_voice.sh`:

```bash
scripts/send_feishu_voice.sh /tmp/voice.ogg <receive_id> <tenant_access_token> [receive_id_type]
```

- `receive_id_type`: `open_id` (default), `chat_id`, `user_id`, `union_id`, `email`

#### How to get tenant_access_token

⚠️ **Multi-account support:** OpenClaw may have multiple Feishu accounts configured in `~/.openclaw/openclaw.json` → `channels.feishu.accounts`. Each agent/bot uses its own account. You MUST use the credentials matching the current agent.

**Steps to get the correct credentials:**

1. Check the current agent name from the runtime info (e.g. `agent=bug`)
2. Read `~/.openclaw/openclaw.json` → `channels.feishu.accounts.<agent_name>` for `appId` and `appSecret`
3. If the agent name is not found in accounts, fall back to the top-level `channels.feishu.appId` / `channels.feishu.appSecret`

Example (agent=bug):
```bash
# Extract credentials for the current agent from openclaw.json
AGENT_NAME="bug"  # from runtime info
CREDS=$(python3 -c "
import json
with open('$HOME/.openclaw/openclaw.json') as f:
    c = json.load(f)
feishu = c.get('channels',{}).get('feishu',{})
accts = feishu.get('accounts',{})
acct = accts.get('$AGENT_NAME', {})
app_id = acct.get('appId') or feishu.get('appId','')
app_secret = acct.get('appSecret') or feishu.get('appSecret','')
print(f'{app_id} {app_secret}')
")
APP_ID=$(echo $CREDS | cut -d' ' -f1)
APP_SECRET=$(echo $CREDS | cut -d' ' -f2)

curl -s -X POST "https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal" \
  -H "Content-Type: application/json" \
  -d "{\"app_id\":\"$APP_ID\",\"app_secret\":\"$APP_SECRET\"}"
```

❌ **DO NOT** use the top-level `appId`/`appSecret` directly when `accounts` exists — using the wrong app's credentials causes `99992361 open_id cross app` errors.

#### Critical parameters (voice bubble vs file attachment)

The difference between a voice bubble and a file attachment depends on these exact parameters:

**Upload** (`POST /open-apis/im/v1/files`):
- `file_type=opus` ← MUST be `opus`, not `stream` or `file`
- `file_name=voice.ogg`
- `duration=<milliseconds>` ← integer, e.g. `6126`
- `file=@/tmp/voice.ogg`

**Send** (`POST /open-apis/im/v1/messages`):
- `msg_type=audio` ← MUST be `audio`, not `file`
- `content="{\"file_key\":\"<file_key>\"}"`

⚠️ If you use `file_type=stream` + `msg_type=file`, it becomes a file attachment (not a voice bubble).

#### Steps performed by the script:
1. `ffprobe` to get duration in milliseconds
2. Upload to `POST /open-apis/im/v1/files` with `file_type=opus`, `duration=<ms>`
3. Send `msg_type=audio` with the returned `file_key`

### Discord

Discord voice messages require a waveform and special flags.

1. Generate ogg with `scripts/gen_voice.sh`
2. Generate waveform: `python3 scripts/gen_waveform.py /tmp/voice.ogg`
   - Outputs JSON: `{"duration_secs": 4.2, "waveform": "base64..."}`
3. Call Discord API:

```bash
curl -X POST "https://discord.com/api/v10/channels/${CHANNEL_ID}/messages" \
  -H "Authorization: Bot ${BOT_TOKEN}" \
  -F "payload_json={\"flags\":8192,\"attachments\":[{\"id\":0,\"filename\":\"voice-message.ogg\",\"duration_secs\":${DURATION},\"waveform\":\"${WAVEFORM}\"}]}" \
  -F "files[0]=@/tmp/voice.ogg;type=audio/ogg;filename=voice-message.ogg"
```

- `flags: 8192` = IS_VOICE_MESSAGE (1 << 13)
- Missing waveform/duration causes error 50161

### Fallback

If `asVoice=true` does not produce a voice bubble on a channel:
1. Try sending via the platform's native API
2. If native API unavailable, send as audio file attachment
