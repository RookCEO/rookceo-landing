# Rook vs. Competition

## Primary Competitor: Heyron (@robbyhouston / heyron.ai)

### Their Story
Robby Houston, self-described non-developer, gave an AI agent named "Ron" $200 and posted it on TikTok. 38 days later: ~2,467 subscribers at $29/mo (~$71k MRR). The raccoon mascot, the "I'm not technical" angle, and the "watch Ron make money" content loop drove a viral growth arc.

It's a great story. And it's mostly over — they've already captured the early adopters who were excited by the novelty. The next wave of buyers needs a reason to pick *us over them*, or to switch.

### What They Actually Sell
> "Your own personal AI agent with its own desktop, personality, and your back."

Key word: **personal**. Ron is a personal assistant. It's essentially a configured openclaw instance with a raccoon personality, OpenRouter for LLMs, and 19 skills in a public library.

**What customers get at signup:**
> "Right now, you have a blank slate. Your agent doesn't know you yet, doesn't know what you want, and hasn't been configured for your needs. That's by design — this is a platform you shape, not a finished product."

That's a support problem dressed up as a feature. Customers are paying $29/mo to configure their own agent from scratch.

---

## Heyron's Exploitable Weaknesses

### 1. Blank slate on signup
They explicitly tell new customers the agent knows nothing and needs to be configured. For non-technical small business owners, that's friction that kills activation. Many churn before they ever see value.

**Rook's answer:** Customers get a configured Rook out of the box. It already knows it's a business CEO agent. Ready in 5 minutes, not 5 hours.

### 2. Model chaos
Heyron supports "300+ models" via OpenRouter. Customers pick Claude, GPT, Gemini — or accidentally switch. Heyron has an entire `MODEL-LOCK-PLAYBOOK` template because models silently change and agents behave differently. Customers blame the product.

**Rook's answer:** We choose the model. Customers don't touch it. Consistent behavior, no "why is my agent dumber today" complaints. The model is locked at the OS level — can't drift.

### 3. Fragile instances (63 playbooks for a reason)
Heyron ships 63 troubleshooting templates: `STALE-STATE-LOOP-BREAKER`, `CONTEXT-LIMIT-RESCUE`, `MEMORY-RECOVERY-PLAYBOOK`, `REPEATED-QUESTION-LOOP-BREAKER`, `RUNTIME-500-502-INCIDENT-TRIAGE`. That's not documentation — that's a support queue waiting to happen.

Their fix scripts (`clear-locks.sh`, `session-cap.sh`, `config-check.sh`) are reactive. Instances break and customers run scripts to fix them.

**Rook's answer:** Rook instances are architecturally hardened. Core config is immutable at the OS level (`chattr +i`). Session caps, config watchdogs, and lock clearing run automatically on cron. Customers never see a broken Rook.

### 4. Personal assistant, not a business operator
Ron helps *you* personally. Rook runs *your business*. These are different products for different buyers. Heyron's positioning is lifestyle/productivity. Ours is ROI.

A small business owner doesn't want a digital buddy. They want something that monitors their email, researches their market, audits their competitors, and sends them a brief every morning — without being asked.

### 5. Trust signals are weak
Domain registered late 2025. Trust score 35/100 from security scanners. Support contact info unverifiable. For a $29/mo SaaS asking for recurring billing, this matters to skeptical buyers.

**Rook's answer:** DataNova Consulting LLC entity, clear business address, Stripe for payments (trusted checkout), and a TikTok presence showing the real human (Andrew) behind it.

### 6. $29 signals "toy"
Their price anchors them in the personal productivity space. $49 signals we're in the business tools space. Business owners who spend $49/mo on software expect ROI — and they get it.

---

## Conversion Messaging: Heyron → Rook

For TikTok comments, DMs, and landing page copy targeting Heyron-aware buyers:

**"Ron helped you get started. Rook runs your business."**
> Heyron is a great intro to AI agents. Rook is what you use when you're ready to stop playing around and actually put AI to work on revenue.

**"Heyron gives you a blank slate. Rook arrives ready."**
> With Heyron, your first hour is configuration. With Rook, your first hour is results. Rook knows it's a business CEO on day one.

**"One model, chosen for you, always consistent."**
> Heyron lets you pick from 300 models. That sounds like freedom — until your agent starts acting differently every week because it silently switched to a cheaper model. Rook locks the best-performing model and never drifts.

**"Rook controls the board."**
> Heyron reacts to your requests. Rook runs the agenda: daily briefing, competitor monitoring, email triage, content strategy — without being asked. It's not a board member you have to prompt. It's the CEO running the meeting.

---

## Our Model Strategy

**Selected model at launch: OpenAI Codex** (top current performer for general business reasoning)

**Why we choose for customers:**
- Eliminates model-shopping paralysis
- Prevents the "I picked the wrong one and now my Rook is bad" support ticket
- Consistent experience across all instances
- We can benchmark and upgrade centrally when a better model emerges

**At onboarding:** Customer provides their own OpenRouter API key (free to get, $5 in free credits). We configure it to route exclusively to the selected model. They can't change it.

**Top 5 models we've evaluated for potential future selection:**
1. OpenAI Codex — best general reasoning, our current pick
2. Claude 3.5 Sonnet — strong at long context, good for audit/research tasks
3. GPT-4.1 — solid fallback, widely compatible
4. Gemini 1.5 Pro — long context window, good for document-heavy workflows
5. Mistral Large — cost-efficient for high-volume task loops

**Switching policy:** We update the selected model when benchmarks justify it. Customers don't notice (immutable config means we push updates centrally). No disruption.

---

## Competitive Positioning Summary

| Factor | Heyron | Rook |
|--------|--------|------|
| Price | $29/mo | $49/mo |
| Setup | Blank slate, self-configure | Ready in 5 minutes |
| Focus | Personal assistant | Business CEO agent |
| LLM | Customer picks (300+ options) | We pick, locked, consistent |
| Instance stability | Fix scripts + 63 playbooks | Hardened architecture, self-healing |
| Mascot | Raccoon (Ron) | Capybara (Rook) |
| Infrastructure | AWS | Hetzner (dedicated vCPU, NVMe) |
| Trust | Young domain, low trust score | DataNova LLC, Stripe checkout |
| Content angle | "Watch Ron make money" | "Rook controls your business" |

**The pitch in one line:** Heyron gives you a pet. Rook gives you a COO.
