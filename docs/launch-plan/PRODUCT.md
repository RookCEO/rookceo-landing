# Rook — Product Definition

## What Rook Is

Rook is a personal AI business agent. Not a chatbot. Not a suggestion engine. An autonomous worker that:

- Monitors your email and takes action
- Runs scheduled daily/weekly tasks without being asked
- Researches competitors, markets, and opportunities
- Drafts content, reports, and communications
- Sends you daily briefings: "Here's what I did and found today"
- Remembers everything — no re-explaining context

## What Rook Is NOT

- Not ChatGPT Plus
- Not a chat window you type into and read suggestions from
- Not something that needs babysitting
- Not a general-purpose assistant — built for small business operations

---

## Pricing

### Rook CEO — $49/month
The flagship. One plan. Full capability from day one.

- Your own private Rook instance (hosted, no setup required)
- Full persistent memory — Rook remembers everything, always
- Email monitoring (IMAP) — reads inbox, flags urgent items, drafts responses
- Scheduled autonomous tasks — daily briefs, weekly reports, runs without prompting
- Competitor monitoring — weekly report on what competitors are posting/doing
- Web research on demand — Rook investigates anything you need
- Social media content research — trending topics in your niche
- Daily business briefing delivered to your inbox
- Private Discord community (CEO peer group)
- Ready in under 5 minutes, no technical setup

**Positioning:** More capable than general AI assistants. Purpose-built for small business CEOs. $49 signals premium — business owners don't blink at $49/month if the ROI is clear.

**vs. Heyron ($29):** They're a personal AI. We're a business CEO agent. That's worth $20 more per month.

### Custom — $297+/month
- Done-for-you configuration for specific business workflows
- Custom skill development
- White-glove onboarding call
- Monthly async strategy report from Rook
- For established businesses with specific tooling needs

---

## Customer Journey

```
TikTok video → clicks link in bio → getrook.__ landing page →
joins waitlist OR clicks "Get Rook" → email capture →
payment (Stripe) → provisioned Rook instance within 5 min →
welcome email with first instructions → Discord access
```

---

## Feature Roadmap

### Launch (Week 1-2)
- Hosted instance with chat + memory
- Web research
- Daily email briefing
- TikTok Shop listing

### Month 1
- Email monitoring (IMAP)
- Scheduled tasks
- Competitor research report

### Month 2
- Social content calendar generation
- Multi-channel monitoring (TikTok + Instagram + LinkedIn)
- Lead research and outreach drafting

### Month 3+
- CRM integration
- Invoicing/bookkeeping summaries
- Customer communication templates
- Team-mode (multiple Rooks for different departments)

---

## Infrastructure

### Hosted Customer Instances (Starter + Business)
- Provider: DigitalOcean Droplets
- Cost per customer: ~$6/month (1GB RAM, 1 vCPU)
- Margin: ~80% on Starter, ~87% on Business
- Spin-up time: under 5 minutes (automated)
- Each customer gets isolated container (privacy: their data never touches another customer's)

### Customer Data Privacy
- Fully isolated per customer
- No shared model training on customer data
- Data export available on cancel
- GDPR-friendly architecture

### Tech Stack (Internal — Never Mention Publicly)
- Gateway: openclaw (open source AI agent framework)
- LLM inference: local + API
- Memory: persistent, structured context
- Email: IMAP monitoring
- Video pipeline: Orpheus TTS + SadTalker + FFmpeg

---

## TikTok Shop Listing (Digital Service)

**Product name:** "Rook CEO — Your AI Business Agent"
**Category:** Digital services / Software subscriptions
**Price:** $49/month (links to Stripe checkout for recurring)
**Description:**
> Rook is an AI CEO that actually runs your business. Not a chatbot — a worker. It monitors your email, researches your market, tracks your competitors, drafts your content, and sends you daily reports. Your own private Rook, ready in 5 minutes.

---

## Seller Account Setup

**Entity:** DataNova Consulting LLC
**Document type:** Corporate seller
**Required:**
- Business name: DataNova Consulting LLC
- EIN (Employer Identification Number)
- Registered business address
- Last 4 digits of tax ID
- W9 form
- Merchant category code: Software / Digital Services

**Note:** Do NOT use personal SSN — use DataNova Consulting LLC EIN throughout.

**Application URL:** https://seller-us.tiktok.com/account/register
**Approval time:** 1-3 business days
**Follower requirement:** None (confirmed by TikTok FAQ)
