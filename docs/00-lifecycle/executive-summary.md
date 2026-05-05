# Executive Summary — Module31

## What Is It

Module31 is a privacy-first, credit-based media processing SaaS platform. It provides 30+ professional video and audio tools — from basic compression and trimming to AI-powered translation, cinematic ad generation, and 3D rendering — in a single unified product.

**Live at:** [module31.com](#) | **Built by:** Kishore K V (solo, end-to-end)

---

## The Problem

Content creators and marketing teams face three compounding problems:

1. **Fragmentation** — A typical creator uses 7+ separate tools to complete a single video workflow (compress → subtitle → translate → generate thumbnail). Each requires a separate login, upload, and download cycle.

2. **Privacy risk** — Most tools (Clideo, Kapwing, CloudConvert) store uploaded files permanently or for 24+ hours on third-party infrastructure. Enterprises with client footage, sensitive recordings, or proprietary content cannot safely use these tools.

3. **Subscription waste** — Most tools charge $15–50/month for features used 2–3 times a month. Users pay for idle capacity, not actual usage.

---

## The Solution

Module31 consolidates all common media tools into one platform with three foundational principles:

| Principle | Implementation |
|---|---|
| **Privacy-first** | Files deleted after configurable retention window (default: 2 hours). No permanent cloud storage. |
| **Pay-as-you-go** | Credit bundles — users pay per operation, not per month. Admins can adjust costs in real time without code deploys. |
| **Zero friction for discovery** | Guest trial mode: 5 free uses with watermark before a signup gate. Users experience value before committing. |

---

## How It Works

```
User uploads file → API route validates + deducts credits → 
  [FFmpeg tools]  → Process locally on server → Return URL
  [AI/GPU tools]  → Shell out to Modal GPU worker → Return URL
→ User downloads result → File deleted after retention window
```

- **Standard tools** (compress, trim, crop, subtitles, etc.) are processed by FFmpeg running directly in Next.js API routes — fast, cheap, no external dependency.
- **AI tools** (translator, ad generator, motion artist, vision) are offloaded to serverless GPU workers on Modal.com — scales to zero, no idle cost.
- **All state** (user credits, usage logs, config) lives in Supabase (PostgreSQL) with Row-Level Security.

---

## Key Technical Bets

| Bet | Rationale | Risk Mitigated |
|---|---|---|
| Modal.com for GPU workers | Serverless GPU — no idle cost, auto-scaling, Python ecosystem | Eliminated $500+/mo idle GPU server cost |
| Credits deducted before GPU invocation | Gate spend before committing compute | Prevents runaway cost from repeated requests |
| Admin-driven runtime config | Costs, watermarks, retention in DB — no redeploys | Enables live pricing experiments without engineering cycles |
| Next.js full-stack | One codebase, one deployment, SSR + API routes | Eliminated backend-frontend split team overhead |
| FFmpeg bundled via npm | `ffmpeg-static` package — no system install | Deterministic binary across dev/prod; zero infra dependency |

---

## Product Scope

- **30+ working tools** across video processing, audio, AI translation, motion graphics, and 3D rendering
- **Role-based access**: customer / admin / superadmin with per-role capabilities
- **Guest trial funnel**: unauthenticated users get 5 watermarked uses → signup gate
- **Admin dashboard**: credit management, support tickets, runtime config panel
- **Multi-model AI**: GPT-4o (storyboard), Gemini 1.5 Pro (vision), Faster Whisper (ASR), Sarvam AI (Indic TTS), Blender (3D rendering), Remotion (2D motion)

---

## Outcomes

- Shipped a production-ready SaaS with 30+ functional tools as a solo builder
- Demonstrated end-to-end product ownership: discovery → PRD → architecture → build → deployment
- Architected a system that scales GPU cost to zero when idle, with no manual infra management
- Designed a monetization model that allows live pricing experimentation without code changes

---

## Documentation Index

This repository contains the full product and architecture documentation:

- [Product Lifecycle](product-lifecycle.md) — Full journey from trigger to ship
- [Metrics & OKRs](metrics-and-okrs.md) — How success is defined and measured
- [PRD](../02-product/prd.md) — Full product requirements
- [System Design](../03-architecture/system-design.md) — Architecture deep dive
- [Architecture Decisions](../03-architecture/architecture-decisions.md) — 7 key decisions with tradeoffs
