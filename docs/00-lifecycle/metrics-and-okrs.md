# Metrics & OKRs — Module31

## Northstar Metric

**Processed minutes per active user per week**

This metric captures the core value exchange: users get media processing done. It is a leading indicator of both product stickiness (users returning to process more) and revenue potential (more processing = more credits consumed). It is better than "weekly active users" because it measures depth of engagement, not just logins.

---

## Objective 1 — Activate New Users Effectively

*Users who experience value in their first session will return. The guest trial is the activation mechanism.*

| Key Result | Target | Measurement |
|---|---|---|
| KR1: Guest → Signup conversion rate | ≥ 25% of guests who hit the trial limit sign up | `usage_logs` guest attempts vs new `profiles` created |
| KR2: Time-to-first-processed-file for new signups | < 90 seconds from account creation | Timestamp delta: `profiles.created_at` → first `usage_logs` entry |
| KR3: Tool diversity in first session | ≥ 40% of new users try 2+ tools in first week | Count distinct `tool_name` per user in `usage_logs` within 7 days of signup |

**Why these KRs:** Activation fails silently. A user who signs up but never processes a file is not activated. KR1 validates the guest funnel. KR2 catches onboarding friction. KR3 catches single-tool lock (a user who only compresses videos will churn when they find a dedicated compressor).

---

## Objective 2 — Retain and Grow Credit Consumption

*A healthy credit economy means users are getting recurring value, not just trying the platform once.*

| Key Result | Target | Measurement |
|---|---|---|
| KR1: 30-day credit replenishment rate | ≥ 35% of paying users refill credits within 30 days of depletion | `usage_logs` credit events + admin refill events |
| KR2: Average tools used per user per month | ≥ 4 distinct tools | Distinct `tool_name` count per user per rolling 30-day window |
| KR3: Monthly processed minutes growth | +20% month-over-month | Sum of estimated processing minutes from `usage_logs` |

**Why these KRs:** Replenishment rate is the closest proxy to retention in a credit model (there is no subscription renewal to track). Tool diversity prevents single-tool dependency. Processing volume growth validates that the platform is becoming a workflow tool, not a novelty.

---

## Objective 3 — Keep GPU Economics Healthy

*AI tools are expensive to run. The credit model must cover GPU cost with margin.*

| Key Result | Target | Measurement |
|---|---|---|
| KR1: GPU cost coverage ratio | Credits charged ≥ 2x Modal GPU cost per AI tool run | Per-tool cost in `system_config` vs Modal.com billing per invocation |
| KR2: Failed AI job rate | < 5% of AI tool invocations fail after credit deduction | `usage_logs` entries with error state / total AI tool invocations |
| KR3: Modal cold start impact | < 10% of AI jobs delayed > 30 seconds due to cold start | Modal.com invocation logs — p90 queue latency |

**Why these KRs:** KR1 ensures the business model is solvent. KR2 matters because failed jobs after credit deduction create support tickets and erode trust (the "no double-charge guarantee"). KR3 tracks a known Modal.com tradeoff — cold starts on GPU workers can frustrate users even when the processing itself succeeds.

---

## Per-Tool Success Criteria

Each tool is independently viable. Before investing further in a tool, it must pass:

| Criterion | Threshold |
|---|---|
| Tool used by ≥ 10% of active users in any 30-day window | Indicates genuine demand |
| Error rate < 3% | Indicates production stability |
| Credit cost covers infrastructure cost with ≥ 1.5x margin | Indicates economic viability |

Tools that fail these criteria for 2 consecutive months are candidates for deprecation or reimplementation.

---

## Unit Economics Model

### Standard Tools (FFmpeg-based)
| Variable | Estimate |
|---|---|
| Server CPU time per job | ~2–8 seconds |
| Estimated server cost per job | ~$0.001–$0.005 |
| Credit price per job | 1–3 credits (~$0.05–$0.15 at target bundle price) |
| **Margin** | **~30–50x** |

### AI Tools (Modal GPU)
| Tool | GPU Time | Estimated Modal Cost | Credits Charged | Target Margin |
|---|---|---|---|---|
| Video Translator | 30–120s (T4) | $0.05–$0.20 | 25 credits (~$1.25) | ~6–25x |
| Ad Generator | 15–45s (T4) | $0.03–$0.10 | 5 credits (~$0.25) | ~2.5–8x |
| Motion Artist | 60–180s (T4) | $0.10–$0.30 | 10 credits (~$0.50) | ~1.7–5x |
| Vision Agent | 10–30s (T4) | $0.02–$0.08 | 5 credits (~$0.25) | ~3–12x |

*Credit bundle target price: $5 for 100 credits = $0.05 per credit.*

**Key insight:** The credit model allows admin-driven margin adjustment without code deploys. If Modal pricing increases, the `ad_generator_cost` value in `system_config` can be updated in seconds.

---

## Anti-Metrics (What We Explicitly Don't Optimize For)

| Metric | Why We Avoid It |
|---|---|
| Total registered users | A vanity metric — guest trial creates accounts that never process anything |
| Page views | Platform is tool-first; users come to work, not browse |
| Session duration | Longer sessions could mean users are confused, not engaged |
| Number of tools available | Shipping tools for signal-gathering is fine; shipping them for count is not |
