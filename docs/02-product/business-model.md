# Business Model — Module31

## Model Overview

Module31 operates on a **pay-as-you-go credit model**. Users buy credits in bundles and spend them per tool use. There is no subscription. Credits do not expire. Pricing is controlled by administrators in real time — no code deployment required to adjust costs.

---

## Monetization Funnel

```
AWARENESS
  ↓
Guest Trial (5 free uses, watermarked output)
  ↓  [25% target conversion]
Signup → 20 Free Credits
  ↓  [tool usage, value realized]
Credit Depletion
  ↓  [35% target replenishment within 30 days]
Credit Bundle Purchase
  ↓  [repeat]
Retained Paying User
```

### Why This Funnel Works

**Step 1 — Guest trial reduces acquisition friction.** Creators will not sign up for a tool they have never used. The guest trial lets them process a real file and experience the output quality before committing an email address. The watermark is not punitive — it is a brand impression on every guest output.

**Step 2 — Free credits on signup create activation.** 20 credits is enough to try 10–20 standard tools or 1–2 AI tools. This is intentional: the goal is for users to experience the AI tools (higher perceived value) within their free credits, not just the standard tools.

**Step 3 — Credit depletion is a natural repurchase trigger.** Unlike a subscription that auto-renews, credit depletion creates a moment of intent — the user actively decides to continue. This is higher-quality retention signal than passive subscription renewal.

---

## Credit Structure

### Credit Costs Per Tool

| Tool Category | Credits | Rationale |
|---|---|---|
| Standard tools (compress, trim, crop, etc.) | 1–3 | Low server cost; should feel nearly free |
| Audio tools (normalize, extract, waveform) | 1–2 | Minimal processing cost |
| Subtitles (hardcoded burn) | 3 | FFmpeg + font rendering overhead |
| Vision Agent | 5 | Gemini API call cost |
| Ad Generator | 5 | GPT-4o call + Remotion render |
| Motion Artist | 10 | Blender GPU render (60–180s) |
| Video Translator | 25 | Whisper ASR + TTS synthesis + GPU time |
| Industrial Pipeline | 15 | Blender SVD + GPT-4o scene generation |

*All costs are configurable in real time via the admin config panel — no redeploy required.*

### Credit Bundle Pricing (Target)

| Bundle | Credits | Price | Per-Credit Cost |
|---|---|---|---|
| Starter | 100 | $5 | $0.05 |
| Creator | 300 | $12 | $0.04 |
| Pro | 1,000 | $35 | $0.035 |
| Team | 5,000 | $150 | $0.03 |

**Pricing logic:** Bundles incentivize higher upfront commitment (better unit economics) while keeping the entry bundle ($5) low enough that price is not a barrier to first purchase.

---

## Unit Economics

### Standard Tools
| Variable | Value |
|---|---|
| Server CPU cost per job | ~$0.001–$0.005 |
| Credits charged per job | 1–3 credits = $0.035–$0.15 |
| Gross margin per job | ~95–99% |

Standard tools are extremely high-margin. They are the volume layer of the business.

### AI Tools (Modal GPU)
| Tool | GPU Cost (T4) | Credits Charged | Revenue | Gross Margin |
|---|---|---|---|---|
| Video Translator | $0.05–$0.20 | 25 × $0.04 = $1.00 | $1.00 | 80–95% |
| Motion Artist | $0.10–$0.30 | 10 × $0.04 = $0.40 | $0.40 | 25–75% |
| Ad Generator | $0.03–$0.10 | 5 × $0.04 = $0.20 | $0.20 | 50–85% |
| Vision Agent | $0.02–$0.08 | 5 × $0.04 = $0.20 | $0.20 | 60–90% |

AI tools have variable margin depending on file length and complexity. The credit system absorbs this variance — a 10-minute translation costs the same credits as a 2-minute one, so average margin is maintained across the user base.

**Key protection mechanism:** Credits are deducted before Modal worker invocation. A user cannot consume GPU compute without first paying credits. This prevents runaway spend from repeated requests or failed jobs with retry logic.

---

## Admin-Driven Pricing Control

One of the core architectural decisions was to store all pricing in the `system_config` database table, not in code.

**What this enables:**
- Adjust credit cost of any tool in seconds via the admin config panel
- Run A/B pricing experiments without engineering involvement
- Respond to Modal.com cost changes by adjusting AI tool credits immediately
- Offer promotional pricing (reduce guest trial cost, increase free credits) without a deploy

**Example scenario:** Modal increases T4 GPU pricing by 20%. Without touching code, an admin increases `translator_cost` from 25 to 30 credits. The change takes effect on the next tool use.

---

## Revenue Scenarios

### Conservative (Early Traction)
- 500 active users/month
- Average spend: $8/month
- Monthly Revenue: **$4,000**

### Growth (Post-Product Hunt)
- 2,000 active users/month
- Average spend: $12/month
- Monthly Revenue: **$24,000**

### Scale (With Team Tier)
- 5,000 users + 50 team accounts ($150/mo)
- Monthly Revenue: **$60,000–$80,000**

---

## Cost Structure

| Cost Item | Monthly Estimate | Notes |
|---|---|---|
| Hostinger (hosting) | $15–30 | Next.js standalone deployment |
| Supabase (DB + auth + storage) | $25–100 | Scales with storage and API calls |
| Modal.com (GPU workers) | Variable | Pay-per-second T4 GPU; ~$0.001/s |
| OpenAI API (GPT-4o) | Variable | ~$0.005 per creative director call |
| Google Gemini API | Variable | ~$0.002 per vision analysis |
| Sarvam AI TTS | Variable | Per-character pricing |
| Domain | $1/mo amortized | |
| **Total fixed** | **~$50–150/mo** | GPU and AI API costs are variable and covered by credit revenue |

**The key insight:** Fixed costs are very low (~$50–150/mo). Variable costs (GPU, AI APIs) are directly covered by credit revenue — every AI tool call that consumes GPU compute has already been paid for in credits before the call is made.

---

## Competitive Pricing Position

| Platform | Pricing Model | Monthly Cost (avg creator) |
|---|---|---|
| Kapwing | Subscription | $24/mo |
| Adobe Express | Subscription | $10/mo |
| Descript | Subscription | $24/mo |
| CloudConvert | Subscription + usage | $14/mo |
| **Module31** | **Pay-per-use** | **$8–15/mo** (depends on volume) |

Module31 is priced to be cheaper than any single competitor for occasional to moderate users, while being comparable for heavy users — with the added benefit of covering AI tools that competitors charge separately for.
