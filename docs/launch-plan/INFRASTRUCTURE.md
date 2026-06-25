# Rook — Infrastructure & Instance Architecture

## Customer Instance Hosting

### Provider: Hetzner Cloud
- **Plan:** CX32 — 4 vCPU (dedicated), 8GB RAM, 80GB NVMe, 20TB bandwidth
- **Cost:** ~$6.25/mo per customer instance
- **Margin:** ~87% on $49/mo subscription
- **Why not Contabo:** Aggressive CPU oversubscription, noisy neighbor risk. We sell on performance — variable CPU kills that story.
- **Why not DigitalOcean:** Hetzner gives 4 dedicated vCPU + 8GB for less than DO's 1GB/1vCPU.
- **Why not Hostinger:** $8.99 is intro pricing; month-to-month provisioning hits the full rate (~$14.99).

### Comparison vs Heyron
Heyron runs on AWS per-customer. AWS t3 instances at comparable specs run $15-20/mo. Their margin is lower and their LLM latency is higher (cloud API round-trips via OpenRouter). Our edge: Hetzner dedicated vCPU + local LLM inference.

---

## Instance File Layout

Each customer instance uses a split filesystem: **immutable identity layer** vs **mutable data layer**.

```
/opt/rook/
  config/          ← chattr +i (IMMUTABLE — agent cannot write here)
    SOUL.md        ← Rook's personality, values, operating principles
    AGENTS.md      ← Operating rules, constraints, escalation behavior
    skills/        ← Installed skill definitions
    openclaw.json  ← Gateway config (model endpoint, channels, etc.)
    config.bak/    ← Known-good config snapshots for watchdog restore
  data/            ← Writable (agent CAN write here)
    sessions/      ← Conversation history (.jsonl)
    memory/        ← Persistent memory (mem0 or qmd)
    tasks/         ← Active task state
    workspace/     ← Agent working directory
```

### Why chattr +i
`chattr +i` sets the immutable flag at the filesystem level. Even root cannot modify the file without explicitly removing the flag first. This means:
- Agent cannot overwrite its own SOUL.md no matter how confused it gets
- Agent cannot remove skills from its config
- Agent cannot introduce bad keys into openclaw.json that crash the gateway
- No amount of "improve yourself" prompting can nerf the instance

```bash
# Apply on provisioning:
chattr +i /opt/rook/config/SOUL.md
chattr +i /opt/rook/config/AGENTS.md
chattr +i /opt/rook/config/openclaw.json
chattr -R +i /opt/rook/config/skills/

# To update (admin only):
chattr -i /opt/rook/config/SOUL.md
# edit...
chattr +i /opt/rook/config/SOUL.md
```

---

## Instance Health: Cron Jobs

Three cron jobs run on every customer instance. Deploy to `/etc/cron.d/rook-health`.

### 1. Session Cap (every 15 min)
Prevents context bloat that causes incoherent agent behavior.
```bash
# Keep last 50 messages per session file
find /opt/rook/data/sessions -name "*.jsonl" -size +500k | while read f; do
  lines=$(wc -l < "$f")
  if [ "$lines" -gt 50 ]; then
    tail -50 "$f" > "$f.tmp" && mv "$f.tmp" "$f"
    echo "[$(date)] Capped $f: was $lines lines"
  fi
done
```

### 2. Config Watchdog (every 5 min)
Detects config drift and restores from backup.
```bash
GOOD_HASH=$(cat /opt/rook/config/config.bak/openclaw.json.sha256)
CURR_HASH=$(sha256sum /opt/rook/config/openclaw.json | cut -d' ' -f1)
if [ "$GOOD_HASH" != "$CURR_HASH" ]; then
  echo "[$(date)] Config drift detected — restoring"
  chattr -i /opt/rook/config/openclaw.json
  cp /opt/rook/config/config.bak/openclaw.json /opt/rook/config/openclaw.json
  chattr +i /opt/rook/config/openclaw.json
fi
```
*(Note: with chattr +i in place this should never fire — it's a belt-and-suspenders check.)*

### 3. Lock Clearing (every 30 min)
Unsticks frozen sessions.
```bash
find /root/.openclaw -name "*.lock" -delete 2>/dev/null
```

---

## LLM Strategy: Why OpenRouter (and When Not To)

### How Heyron Uses OpenRouter
Heyron routes all LLM calls through OpenRouter's API (`https://openrouter.ai/api/v1`), which is OpenAI-compatible. In openclaw, this means setting `openclaw.json` to point at OpenRouter instead of a local server. Benefits for Heyron:
- Customer VPS needs no GPU — all inference is remote
- Model selection UI (Claude, GPT-4, Gemini) is a selling point
- One API key per instance, centrally manageable

**The cost:** Every agent action is a cloud API round-trip. Latency is 500ms-3s depending on model load. And either the customer pays for tokens, or Heyron eats margin.

### Rook's LLM Strategy

Customer instances on Hetzner don't have access to the home network (192.168.1.3), so they can't use the MacBook's local inference directly. Three options:

| Option | Latency | Cost | Notes |
|--------|---------|------|-------|
| **OpenRouter** (customer-provided key) | 500ms-3s | Customer pays | Flexible model choice, zero infra |
| **OpenRouter** (Rook-provided key) | 500ms-3s | ~$5-15/mo/customer | We eat token costs, better UX |
| **Hosted proxy** (VPS we run) | 200-500ms | Fixed server cost | Consistent, no per-token billing |

**Recommended approach (MVP):** Each customer provides their own OpenRouter API key during onboarding. Simple, zero token cost to us, model-agnostic. Include in welcome email: "Get your free OpenRouter key at openrouter.ai — $5 in free credits included."

**Later:** If customer LTV justifies it, absorb the token costs and build a shared inference layer.

### openclaw.json LLM Config
```json
{
  "model": {
    "provider": "openrouter",
    "base_url": "https://openrouter.ai/api/v1",
    "api_key": "${OPENROUTER_API_KEY}",
    "model_id": "anthropic/claude-3-haiku"
  }
}
```
Point at local llama-server instead for dev/internal instances (the Pi and MacBook setup).

---

## Provisioning Flow (order-intake → live instance)

```
Stripe webhook: payment confirmed
  → Create Hetzner CX32 droplet via API
  → Wait for IP (~30-60s)
  → SSH: apply Rook customer config
      - Copy /opt/rook/config/ from template
      - Set customer name, API keys in environment
      - Apply chattr +i to config layer
      - Start 3 cron jobs
      - Start openclaw gateway
  → Send welcome email with:
      - Their Rook URL
      - OpenRouter signup link + instructions
      - Discord community invite
  → Total time: < 5 minutes
```

---

## What Heyron Does vs What We Do

| Problem | Heyron's Approach | Rook's Approach |
|---------|-------------------|-----------------|
| Agent nerfing itself | Reactive fix scripts (clear-locks, config-check) + 63 playbook templates | Proactive: chattr +i makes core config unwritable at OS level |
| Context bloat | session-cap.sh cron (50 msg limit) | Same, stolen directly — it works |
| Config drift | AGENTS-DRIFT-PREVENTION playbook | Config watchdog cron + chattr +i |
| Model switching | MODEL-LOCK-PLAYBOOK | Lock model in openclaw.json, immutable |
| Instance recovery | Manual support playbooks | Self-healing cron + watchdog |

Their fix scripts are a **support burden**. Ours should self-heal.
