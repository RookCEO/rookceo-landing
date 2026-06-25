# Rook Launch Checklist

## Week 1: Foundation

### Domain & Presence
- [ ] Register domain (recommended: rookceo.com ~$11/yr or getrook.ai $85/yr)
- [ ] Create TikTok account (@GetRook or @RookCEO — check availability)
- [ ] Apply for TikTok Shop (entity: DataNova Consulting LLC, EIN required)
- [ ] Set up simple landing page with email waitlist capture

### Brand Assets
- [ ] Generate Rook capybara avatar (3 poses) via DALL-E
  - Arms crossed (profile pic)
  - Pointing at screen (teaching)
  - Waving (greeting/intro)
- [ ] Download Orpheus TTS model
  ```bash
  huggingface-cli download canopylabs/orpheus-3b-0.1-ft-GGUF \
    orpheus-3b-0.1-ft-q4_k_m.gguf --local-dir ~/models/orpheus
  ```
- [ ] Start Orpheus TTS server on Mac (port 8083)
- [ ] Test Rook's voice — record sample line
- [ ] Install SadTalker for avatar animation

### First Skills (openclaw — internal)
- [ ] `email-intake` skill (IMAP monitoring)
- [ ] `tiktok-scrape` skill (Playwright profile scraper)
- [ ] `youtube-analyze` skill (YouTube Data API)
- [ ] Get YouTube Data API key (free, Google Cloud Console)

### Payment Infrastructure
- [ ] Create Stripe account (or use existing)
- [ ] Create $49/month recurring price (flagship — Rook CEO)
- [ ] Create $297/month recurring price (custom tier)
- [ ] Set up simple payment link (no full checkout needed yet)
- [ ] Connect Stripe → email notification on new subscriber

---

## Week 2: First Content

### TikTok Shop
- [ ] TikTok Shop approved (applied Week 1)
- [ ] List "Starter — Get Your Own Rook" for $29/month
- [ ] Add product description and demo video (first video = demo)

### First 5 Videos
- [ ] Video 1: "I made an AI the CEO of my business. Day 1."
  - Andrew on camera, introduce Rook concept
  - Show Rook's interface briefly
  - Hook: what Rook did on day 1
- [ ] Video 2: "Rook's first money scheme"
  - Rook identifies a revenue opportunity
  - Andrew reacts: "I'm doing it / I'm not doing it"
- [ ] Video 3: "What Rook knows about your business" (educational)
  - Rook explains 3 things small businesses get wrong
  - Positions Rook as expert
- [ ] Video 4: "Rook reviewed a business. Here's what it found." (free public audit)
  - Pick a commenter or public business TikTok
  - Rook delivers 3 findings
  - CTA: "Want Rook to review yours?"
- [ ] Video 5: "Get your own Rook — here's how"
  - Direct product explanation
  - What you get, how it works, how to sign up

### Video Production Checklist (per video)
- [ ] Script drafted by Rook (`video-script` skill)
- [ ] Andrew records his segments (phone, good lighting)
- [ ] Rook voiceover generated (Orpheus TTS)
- [ ] Rook avatar animated (SadTalker)
- [ ] FFmpeg: combine segments, add captions, add music
- [ ] Review final cut (< 2 min)
- [ ] Upload to TikTok with hashtags

---

## Week 3-4: Scale Content

### Operational
- [ ] `audit-report` skill live (full audit generation)
- [ ] `video-script` skill live (automated script generation)
- [ ] `voiceover` skill live (Orpheus TTS via openclaw)
- [ ] `video-assemble` skill live (SadTalker + FFmpeg pipeline)
- [ ] First customer fulfilled via automated pipeline

### Content
- [ ] Daily videos (7/week)
- [ ] Reply to every comment within 1 hour of posting
- [ ] Run "Rook reviews YOUR business" campaign — review 5 commenters publicly
- [ ] Post Week 2 results: "Here's what week 1 looked like"

---

## Week 5-8: Viral Push

### Product
- [ ] DigitalOcean provisioning automated (order-intake → spin up droplet)
- [ ] Welcome email sequence live (3 emails over first week)
- [ ] Discord community set up
- [ ] Business tier ($49) live with email monitoring feature

### Content — Set Up Viral Moments
- [ ] Experiment: "I'm leaving Rook alone for a week. No instructions."
- [ ] Results video: "Here's what Rook did while I was away"
- [ ] Emotional angle: "Rook said something unexpected"
- [ ] Milestone: "Rook helped X businesses this month"

### Growth Levers
- [ ] Consider $1 trial week (reduces friction)
- [ ] Referral: "Give a friend 1 free month, get 1 free month"
- [ ] Press: submit to AI newsletter/indie hacker communities

---

## TikTok Shop Application Steps

1. Go to: https://seller-us.tiktok.com/account/register
2. Sign up with business email (DataNova Consulting LLC email)
3. Select "Corporation" entity type
4. Enter: DataNova Consulting LLC
5. Enter: EIN (from DataNova LLC documents)
6. Enter: Registered business address
7. Upload: EIN confirmation letter or SS-4 form
8. Tax info: W9 form
9. Merchant category: Software / Digital Services
10. Bank account: DataNova Consulting LLC business account
11. Submit and wait 1-3 business days

**Follower requirement:** None (confirmed — can sell with 0 followers)

---

## DataNova Consulting LLC Requirements for TikTok Shop

Gather before applying:
- [ ] EIN number
- [ ] Registered business address
- [ ] SS-4 confirmation letter OR EIN verification letter from IRS
- [ ] W9 form (can generate fresh one at irs.gov)
- [ ] Business bank account info (routing + account number)
- [ ] Business email address

---

## Domain Options Ranked (see DOMAINS.md for full list)

**Recommended:**
1. **rookceo.com** — $11.28/yr ✅ AVAILABLE — "Rook CEO" perfectly tells the story
2. **getrook.ai** — $85/yr — premium .ai credibility if budget allows

**Budget alternatives (under $10/yr):**
- rookceo.org — $7.48/yr (same name, less authoritative TLD)
- tryrook.xyz — ~$2/yr

---

## Post-Launch Metrics to Track Weekly

| Metric | Tool | Target |
|--------|------|--------|
| Video views | TikTok analytics | 10k+ avg by week 4 |
| Profile visits | TikTok analytics | 500+/day by week 3 |
| Link clicks | Bitly/TikTok | >2% of profile visits |
| Landing page conversions | Simple analytics | >20% email capture |
| Paid conversions | Stripe | Growing week over week |
| Churn | Stripe | <10%/month |
