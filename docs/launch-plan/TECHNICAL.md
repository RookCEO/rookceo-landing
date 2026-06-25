# Rook — Technical Architecture

## Overview

Rook's infrastructure runs across two machines:
- **Raspberry Pi ("rook")** — openclaw gateway, skill execution, customer interaction layer
- **MacBook ("mac")** — LLM inference (vllm-mlx), voice generation (Orpheus TTS), video assembly

Customers never see this. They interact with a hosted instance via web or API.

---

## Skills to Build (Priority Order)

### 1. `email-intake` (Week 1)
Monitor IMAP inbox for orders, inquiries, and keywords.
- Credentials in `~/.openclaw/.env`
- Poll every 5 minutes
- Route: TikTok Shop order emails → order handler
- Route: General inquiries → draft response + notify Andrew

### 2. `tiktok-scraper` (INSTALLED ✅)
Custom skill using yt-dlp (free, no API key). Installed at `~/.openclaw/workspace/skills/tiktok-scraper/`.
- `yt-dlp --dump-json --flat-playlist --playlist-end 20 "https://www.tiktok.com/@HANDLE"`
- Returns per-video: view_count, like_count, comment_count, repost_count, duration, description, upload_date
- Parses to: avg views, avg likes, engagement rate, posting frequency, top video
- Zero cost, runs on Pi, no API key needed
- Limitation: follower count not available via yt-dlp (note in audit or leave blank)

### 3. `youtube-analyze` (Week 1)
Given a YouTube channel URL, return metrics via YouTube Data API v3.
- Subscriber count, total views, video count
- Recent 10 videos: title, views, likes, duration, date
- Upload frequency, estimated weekly views
- API key: free tier, 10,000 quota/day (sufficient for 100s of audits)

### 4. `youtube-transcript` (Week 1)
Given a YouTube video URL, return transcript + LLM summary.
- Fetch transcript via youtube-transcript-api
- Summarize via Codex/OpenAI: key ideas, business opportunities, actionable takeaways
- Use: Rook researches money-making ideas from YouTube

### 5. `audit-report` (Week 2)
Given channel analysis data, generate structured business audit.
- Input: output from `tiktok-scrape` + `youtube-analyze`
- LLM (Codex via OpenAI API) generates report:
  - What's working
  - What's not working
  - 5 specific growth recommendations
  - 3 content ideas tailored to their niche
  - Competitive positioning suggestions
- Output: markdown → converted to PDF for delivery

### 6. `video-script` (Week 2)
Given a topic or audit result, generate a TikTok video script.
- Format: Hook / Body / CTA
- Includes: Rook's speaking lines in `<rook>` tags, Andrew's ad-lib prompts in `<andrew>` tags
- Emotional tone: calm authority, dry wit

### 7. `voiceover` (Week 2)
Given a script, generate Rook's audio via Orpheus TTS.
- Send to Orpheus TTS running on Mac (port 8083)
- Voice: "tara" or "leo"
- Emotional tags: `<chuckle>`, `<sigh>`, `<gasp>` where appropriate
- Output: MP3 file

### 8. `video-assemble` (Week 3)
Assemble final TikTok video from components.
- Input: Rook avatar image + voiceover MP3
- SadTalker: animate Rook's mouth to match audio
- FFmpeg: add captions, branding, B-roll overlays
- Output: MP4 ready for upload (1080x1920, 60fps, <50MB)

### 9. `tiktok-upload` (Week 3)
Upload video to TikTok via Playwright automation.
- Log into TikTok (session cookie management)
- Upload MP4
- Set caption + hashtags from script metadata
- Schedule post time if provided

### 10. `order-intake` (Week 2)
Process incoming TikTok Shop or Stripe orders.
- Parse order email (customer name, email, tier purchased)
- Trigger: provision new Rook instance (DigitalOcean API)
- Send welcome email with credentials
- Add to Discord community

---

## Voice: Orpheus TTS Setup

### Download
```bash
huggingface-cli download canopylabs/orpheus-3b-0.1-ft-GGUF \
  orpheus-3b-0.1-ft-q4_k_m.gguf \
  --local-dir ~/models/orpheus
```

### Run via llama-server (Mac)
```bash
/opt/homebrew/bin/llama-server \
  -m ~/models/orpheus/orpheus-3b-0.1-ft-q4_k_m.gguf \
  --port 8083 \
  --host 0.0.0.0
```

### Usage
Orpheus uses a special prompt format for TTS. Feed it text with emotion tags:
```
<|audio|>
tara: Three things are holding your business back. <sigh> Let's fix them.
```

Outputs: raw audio tokens → decode to WAV/MP3

### LaunchAgent
Add a new LaunchAgent for Orpheus on Mac (similar to vllm-mlx plist):
- Label: `com.andrew.orpheus-tts`
- Port: 8083
- Model: `~/models/orpheus/orpheus-3b-0.1-ft-q4_k_m.gguf`

---

## Video: SadTalker Setup

SadTalker is an open-source talking-head generation model.

### Install ✅ (installed at ~/apps/SadTalker)

Uses a dedicated conda env `rookceo` (Python 3.8 — required by SadTalker's numpy/torch deps).

```bash
git clone https://github.com/OpenTalker/SadTalker.git ~/apps/SadTalker
conda create -n rookceo python=3.8 -y

# Order matters — pre-install these before requirements.txt or it fails
conda run -n rookceo pip install torch torchvision torchaudio
conda run -n rookceo conda install -c conda-forge lmdb -y
conda run -n rookceo pip install cython

# Install everything else (needs the compiler flag for Apple Silicon)
conda run -n rookceo env CFLAGS="-Wno-error=implicit-function-declaration" \
  pip install -r ~/apps/SadTalker/requirements.txt

# Download model checkpoints (~1.3GB)
cd ~/apps/SadTalker && bash scripts/download_models.sh
```

### Usage
```bash
conda run -n rookceo python ~/apps/SadTalker/inference.py \
  --driven_audio /path/to/rook-voiceover.mp3 \
  --source_image /path/to/rook-avatar.png \
  --result_dir /path/to/output/ \
  --enhancer gfpgan \
  --device mps
```

Output: MP4 of Rook avatar with synced mouth movement.

### Mac GPU acceleration
Uses MPS (Metal Performance Shaders) via `--device mps` flag on Apple Silicon.

---

## Customer Hosting: DigitalOcean

### Per-customer instance
- Droplet: $6/month (1GB RAM, 1 vCPU, 25GB SSD)
- Docker image: prebuilt openclaw container
- Automated provisioning: DigitalOcean API

### Provisioning flow (triggered by `order-intake`)
1. Call DigitalOcean API: create droplet from Rook snapshot
2. Wait for IP (~60 sec)
3. SSH: configure customer's name/persona, set API keys
4. Send customer their login URL
5. Total time: < 5 minutes

### Margin
| Tier | Revenue | Infra cost | Margin |
|------|---------|------------|--------|
| Starter $29 | $29/mo | $6/mo | 79% |
| Business $49 | $49/mo | $8/mo | 84% |
| First 10 customers | $290-490/mo | $60-80/mo | 75-84% |

---

## ClawHub Skills Installed on Rook

| Skill | Status | Purpose |
|-------|--------|---------|
| `tiktok-scraper` | ✅ installed (custom) | yt-dlp profile + video metrics, free |
| `tiktok-growth` | ✅ installed | Content strategy + video script generation |
| `tiktok-uploader` | ✅ installed (needs setup) | Upload/schedule videos via browser automation |
| `tiktok-shop-publish` | ✅ installed | TikTok Shop product + order management |
| `douyin-tiktok-trends` | ✅ installed | Trending topics, hashtags, sounds |

Install command: `openclaw skills install <name>`
All free, no paid API keys required.

---

## MCP Servers (Internal Use)

| MCP | Purpose |
|-----|---------|
| Playwright (already installed) | TikTok scraping, browser automation |
| YouTube Data API | Channel analysis |
| DigitalOcean API | Customer instance provisioning |
| Stripe API | Payment processing |
| Resend/Postmark | Transactional email |

---

## Environment Variables (never commit)

All secrets in `~/.openclaw/.env`:
```
# Email
IMAP_HOST=...
IMAP_USER=...
IMAP_PASS=...

# YouTube
YOUTUBE_API_KEY=...

# DigitalOcean
DO_API_TOKEN=...

# Stripe
STRIPE_SECRET_KEY=...
STRIPE_WEBHOOK_SECRET=...

# Orpheus TTS (running on Mac)
ORPHEUS_URL=http://192.168.1.3:8083

# OpenAI (for Codex/audit generation)
OPENAI_API_KEY=...
```
