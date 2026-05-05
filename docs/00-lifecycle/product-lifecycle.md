# Product Lifecycle — Module31

> From a personal frustration to a production-ready SaaS with 30+ tools, AI integrations, and a live business model.

---

## Phase 1 — Trigger

**What sparked it:** The recurring friction of using 5–7 different browser tools to process a single video. Upload to Clideo to compress. Go to Kapwing to add subtitles. Use a separate tool to extract audio. Each tool has its own account, its own file storage, and its own subscription fee. Files sitting on third-party servers with no clear deletion policy.

**The observation:** Every one of these tools is doing the same thing — wrapping FFmpeg in a web UI. None of them own a moat in the processing itself. The moat is in breadth of tools, user experience, and trust. A platform that combines all of them under one roof — with a credible privacy guarantee — could win on all three.

**The hypothesis:** *If a creator could process any media file — compress, trim, translate, generate an ad — in one place, without a subscription, and without worrying about where their files go, they would prefer it over any single-purpose tool.*

---

## Phase 2 — Discovery

**Who has this problem?**

Three distinct groups emerged during discovery:

1. **Content creators** (YouTubers, TikTokers, Instagram creators) — high volume, frequent tool-switching, sensitive to cost but not to convenience.
2. **Marketing managers at small agencies** — occasional use, high privacy concern (client footage), willing to pay for reliability.
3. **Indie developers and startups** — need bulk processing or programmatic access, not available in any consumer tool.

**How much do they pay today?**

A creator running a standard stack might pay:
- Kapwing Pro: $24/mo
- Adobe Express: $10/mo
- Descript: $24/mo
- CloudConvert: $10/mo (usage-based but still subscription)

**Total: $50–70/month** for tools used intermittently.

**What was validated:**
- Privacy concern is real and underserved — no major player leads with "your files are deleted in 2 hours"
- The subscription model creates resentment — users feel they're paying for tools they barely use
- Guest trial (try before signup) was identified as critical — creators don't want to create accounts to test a tool they've never heard of

**What was ruled out:**
- B2B enterprise with procurement cycles — too slow for a solo builder
- Mobile-first app — desktop workflow is dominant for media processing
- Freemium with feature gating (e.g. "upgrade to unlock HD") — creates artificial friction and trust issues

---

## Phase 3 — Ideation

**What was considered:**

| Option | Why Considered | Why Rejected |
|---|---|---|
| Single-tool product (just a compressor) | Simpler to ship | No moat; instantly replaceable |
| API-first platform (no UI) | Developer-friendly | Excludes non-technical creators; harder to monetize |
| Subscription model | Predictable revenue | Misaligns with intermittent usage; increases churn risk |
| Open source + hosted version | Community traction | Gives away the moat; hard to monetize |
| White-label for agencies | B2B revenue | Sales cycle too long; not solo-builder-viable |

**What won and why:**

A **multi-tool, credit-based, privacy-first web platform** with a guest trial funnel.

- Multi-tool = network effects within the platform (users stay instead of leaving for a specialist tool)
- Credit-based = aligns cost with usage; removes subscription resentment; admin-adjustable without redeploys
- Privacy-first = a credible, differentiated positioning that no major competitor owns
- Guest trial = lowers activation barrier; users experience value before committing

**The key insight that unlocked the architecture:** If every tool is an isolated API route that does one thing well, the platform is infinitely extensible. New tools are additive, not disruptive. A `ComingSoon` component lets you gauge interest in a tool before building it.

---

## Phase 4 — Design

### Product Design Decisions

**Guest Trial Flow**
- 5 free uses with watermark → signup gate
- Watermark text, color, and trial limit are all runtime-configurable (stored in `system_config` DB table)
- Rationale: creators need to see output quality before trusting a new tool. Watermark protects against abuse while allowing genuine evaluation.

**Credit Model Design**
- Users start with 20 free credits on signup (enough to try most tools once)
- Each tool has a configurable credit cost (1 credit for basic tools, 25 for AI translation)
- Credits purchased in bundles (not subscription)
- Admin can adjust per-tool cost in real time via the config panel — no redeploy required
- Rationale: credits abstract away the GPU cost complexity from the user while giving the operator full pricing control

**Tool-First UX**
- Homepage is a searchable, filterable grid of tools — not a landing page
- Each tool is a standalone page with its own upload → process → download flow
- No cross-tool state; each session is independent
- Rationale: users arrive with intent ("I need to compress this video"). They should reach the tool in one click.

**Admin-Driven Operations**
- Costs, watermark settings, file retention window all stored in `system_config` table
- Admin dashboard for credit management and support tickets
- Superadmin layer for platform-level control
- Rationale: a solo-operated platform cannot afford engineering cycles for every pricing or policy change. Operations must be self-serve.

### Architecture Design Decisions

See [Architecture Decisions](../03-architecture/architecture-decisions.md) for the full 7 ADRs. The headline decisions made during design:

1. **Next.js full-stack** — one codebase, one deployment, no backend/frontend split
2. **Modal.com for GPU** — serverless Python workers; scales to zero; no idle cost
3. **Supabase** — PostgreSQL + auth + storage in one; Row-Level Security for per-user isolation
4. **FFmpeg bundled** — `ffmpeg-static` npm package; consistent binary; no system dependency
5. **Credits before GPU** — deduct credits before invoking Modal workers to prevent runaway spend

---

## Phase 5 — Build

**Methodology: Modular, tool-by-tool delivery**

Each tool follows the same pattern:
1. Add tool config to the `TOOLS` array in `page.tsx` (name, description, category, credit cost)
2. Create the tool page at `/app/<tool-name>/page.tsx` (upload → process → download flow)
3. Create the API route at `/app/api/<tool-name>/route.ts` (validate → process → return URL)
4. For GPU tools: write the Modal Python worker in `/modal-workers/<tool-name>.py`

This pattern meant each tool was independently shippable. No tool blocked another. Priority was determined by signal: tools with more clicks on the `ComingSoon` state were built first.

**Sequencing rationale:**
- **V1 first:** Standard FFmpeg tools — highest demand, zero GPU cost, builds platform trust
- **V2 next:** AI tools — higher value, higher cost, requires credit model to be proven first
- **Parallel:** Admin dashboard and config panel built alongside V1 (needed from day one for operations)

**Key build decisions:**
- `ToolPageWrapper` component handles guest trial tracking, signup gate, and watermark banner for every tool page — write once, applied everywhere
- `ffmpeg-static` package resolves cross-platform binary issues without Dockerfile complexity
- `ComingSoon` component deployed for planned tools immediately — captures click intent before build investment

---

## Phase 6 — Ship & Iterate

**Deployment:**
- Hosted on Hostinger (Next.js standalone build)
- Supabase handles database, auth, and file storage
- Modal workers deployed independently via `modal deploy`
- No CI/CD pipeline yet — manual deploy on feature completion

**What was learned post-ship:**

| Observation | Action Taken |
|---|---|
| Blender headless rendering on Linux requires specific environment setup | Documented exact Modal container config in worker files |
| Whisper Large-v3 requires significant VRAM — T4 is borderline | Added chunked audio processing for files > 10 min |
| Guest users sometimes try to bypass trial limit via private browsing | Moved attempt count to server-side session + localStorage dual-track |
| Admin pricing changes needed a UI — raw DB edits were error-prone | Built `/admin/config` panel with form validation |

**Next iteration (V3 planned):**
- Webhook callbacks for long-running AI jobs (currently synchronous — times out on slow Modal cold starts)
- Team workspaces (shared credit pool, shared output library)
- Developer API (authenticated endpoints for programmatic access)
- Job queue (Redis/BullMQ) to handle concurrent AI requests gracefully

---

## Timeline Summary

| Phase | Duration | Key Output |
|---|---|---|
| Trigger + Discovery | 2 weeks | Problem statement, user interviews, competitive analysis |
| Ideation + Design | 1 week | PRD v1, architecture design, tool priority list |
| V1 Build (FFmpeg tools) | 3 weeks | 18 standard tools, auth, credits, guest trial |
| V2 Build (AI tools) | 4 weeks | 5 AI tools, Modal workers, admin dashboard, config panel |
| Polish + Deploy | 1 week | Production deployment, error handling, retention cleanup |
| **Total** | **~11 weeks** | **30+ tools, production-deployed SaaS** |
