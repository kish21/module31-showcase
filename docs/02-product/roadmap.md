# Product Roadmap — Module31

## Roadmap Philosophy

Each version is a complete, shippable product — not a partial implementation. V1 is valuable without V2. V2 extends V1 without breaking it. The modular tool architecture (each tool = isolated API route) means features are additive. Nothing shipped in V1 needs to be undone for V2.

Priority is driven by:
1. **Signal** — which `ComingSoon` tools are clicked most
2. **Margin** — which tools have the best revenue/cost ratio
3. **Differentiation** — which tools no competitor offers
4. **Complexity** — simpler tools ship first to establish trust and platform stability

---

## V1 — Foundation (Shipped)

**Theme:** *Prove the platform. Cover all common creator workflows.*

### Goals
- Ship 18 standard media tools covering the full creator workflow
- Establish authentication, credit system, and guest trial funnel
- Deploy to production with reliable uptime

### What Shipped

| Category | Tools |
|---|---|
| Video Processing | Compress, Quick Trim, Social Crop, Video to GIF, Format Converter |
| Audio | Extract Audio, Audio Normalizer, Audio Remover, Audio Waveform |
| Composition | Video Concatenator, Thumbnail Generator |
| Enhancement | Visual Adjustments, Flip/Rotate, Speed Master, Resolution Scaler, Frame Rate Converter |
| Privacy | Metadata Stripper |
| Captions | Hardcoded Subtitles |

### Infrastructure Shipped
- Next.js full-stack with Supabase auth
- Credit system with per-tool deduction
- Guest trial with watermark (configurable)
- Admin dashboard (user management, credit refill)
- System config panel (all pricing and policies configurable)
- File retention cleanup (auto-delete after configurable window)

### V1 Success Criteria
- All 18 tools process files without errors in ≥ 97% of jobs
- Guest trial conversion ≥ 20% (guests who hit limit → sign up)
- Admin can adjust any credit cost without a code deploy

---

## V2 — AI Layer (Shipped)

**Theme:** *Add the AI capabilities that no single-tool competitor offers.*

### Goals
- Launch 5 AI-powered tools on top of the V1 platform
- Establish GPU worker infrastructure (Modal.com) that scales to zero
- Validate that the credit model can absorb variable GPU costs with positive margin

### What Shipped

| Tool | AI Stack | Value |
|---|---|---|
| Video Translator | Faster Whisper Large-v3 + Sarvam AI TTS | Auto-translate and dub video in 30+ languages including Indic |
| Vision Agent | Gemini 1.5 Pro | AI analysis of video frames — insights, content description, flagging |
| AI Ad Generator | GPT-4o + Remotion | Text prompt → motion typography video ad in 4 styles |
| AI Motion Artist | GPT-4o + Blender 4.0 | Text prompt → cinematic 3D motion video on NVIDIA T4 |
| Industrial Pipeline | GPT-4o + Blender SVD | Product photo → commercial-quality video |

### Infrastructure Shipped
- Modal.com GPU worker pipeline (Python workers on NVIDIA T4)
- AI Creative Director endpoint (GPT-4o for scene graph generation)
- Credit deduction before GPU invocation (cost control gate)
- Support tickets system (users can report AI tool issues)
- Dual rendering engines: Remotion (2D kinetic) + Blender (3D cinematic)

### V2 Success Criteria
- AI tool error rate < 5%
- GPU cost coverage ratio ≥ 2x (credits charged / Modal cost)
- At least 1 AI tool used by ≥ 30% of active users

### Key V2 Learnings
- Blender headless on Linux requires specific CUDA + Python env setup — documented in worker config
- Whisper Large-v3 memory usage on T4 requires chunked audio for files > 10 minutes
- Synchronous Modal invocation times out for long renders — identified job queue as V3 priority
- Gemini 1.5 Pro vision analysis is fast and accurate but returns unstructured JSON — added parsing layer

---

## V3 — Scale & Ecosystem (Planned)

**Theme:** *Turn the platform into infrastructure. Open it up to developers.*

### Goals
- Eliminate synchronous timeout issues for long AI jobs
- Enable team-based usage (agencies, small companies)
- Open developer API for programmatic access

### Planned Features

#### Job Queue (P0)
- Redis + BullMQ queue for all AI tool requests
- Async job processing — API returns job ID immediately
- Client polls `/api/jobs/<id>` for status
- Webhook callback when job completes
- **Why:** Current synchronous Modal invocation times out on slow cold starts. Job queue fixes this without changing the user experience.

#### Webhook Callbacks
- Users register a webhook URL per API key
- Module31 POSTs result URL to webhook on job completion
- Enables integration with external workflows (Zapier, n8n, custom backends)

#### Developer API
- Authenticated REST endpoints for all 30+ tools
- API key generation and management in user dashboard
- Per-key credit assignment (team admin controls spend per key)
- OpenAPI spec published for easy integration
- **Target user:** Vikram (Persona 3) — indie developers building products that need media processing

#### Team Workspaces
- Shared credit pool for a team (up to 10 members)
- Workspace admin can set per-member credit limits
- Shared output library: recent processed files visible to all workspace members
- Usage breakdown by team member in admin view

#### Batch Processing
- Upload multiple files and apply the same tool to all
- Queue-based processing; results available as a ZIP download
- Credit cost: sum of individual tool costs per file

### V3 Success Criteria
- Developer API used by ≥ 5 external integrations in first 3 months
- Team workspace plan generates ≥ 20% of total revenue
- AI job timeout rate (> 30s) < 3% after job queue implementation

---

## What Was Explicitly Deferred

| Feature | Why Deferred |
|---|---|
| Mobile app | Desktop workflow dominates; browser covers the use case |
| Real-time collaboration | Significant complexity; not in core use case |
| Video hosting/CDN | Out of scope — processing platform, not distribution |
| Direct platform publishing (YouTube, TikTok) | OAuth integration complexity; not core value |
| AI voiceover (non-translation) | Separate market; would dilute positioning |
| Generative video (text-to-video from scratch) | Runway ML territory; extremely expensive per job |

---

## Roadmap Timeline (Indicative)

```
Week 1–3   ████████████  V1 — Standard tools, auth, credits, guest trial
Week 4–7   ████████████  V1 — Admin dashboard, config panel, stability
Week 8–11  ████████████  V2 — Modal GPU workers, AI tools (Translator, Vision)
Week 12–14 ████████████  V2 — Ad Generator, Motion Artist, Industrial Pipeline
Week 15    ████          V2 Polish — error handling, retention, UI refinements
─────────────────────────── SHIPPED ───────────────────────────────────────────
TBD        ············  V3 — Job queue, webhooks, developer API
TBD        ············  V3 — Team workspaces, batch processing
```
