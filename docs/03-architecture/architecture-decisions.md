# Architecture Decision Records — Module31

Architecture Decision Records (ADRs) document significant architectural choices: what was decided, what alternatives were considered, and why the chosen approach won. These are written to be useful to future contributors and to demonstrate the reasoning behind the system's shape.

---

## ADR-001: Modal.com for GPU Workers

**Status:** Accepted  
**Date:** V2 design phase

### Context
Five tools (Video Translator, Vision Agent, Ad Generator, Motion Artist, Industrial Pipeline) require GPU compute — specifically for Whisper ASR, Blender rendering, and Remotion rendering. A hosting decision was needed for these workloads.

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| **AWS Lambda** | Familiar, good DX | No GPU support; 15-min timeout; cold start for large packages |
| **AWS EC2 GPU instance** | Full control, persistent | $200–500/mo idle cost even with no users; requires DevOps |
| **Replicate** | Easy ML model hosting | Limited to pre-packaged models; can't run custom Blender/Remotion |
| **RunPod** | Cheap GPU, flexible | Requires persistent instance management; no serverless option |
| **Modal.com** | Serverless GPU, Python-native, scales to zero | Cold starts on first invocation (~10–30s); newer platform |

### Decision
**Modal.com.** Serverless GPU workers that scale to zero eliminate idle cost entirely. Python-native environment allows running arbitrary code (Blender, Remotion, Whisper, custom scripts) without being constrained to pre-packaged models. The cold start penalty is acceptable for non-realtime workloads.

### Consequences
- GPU cost is $0 when no jobs are running — critical for a solo-operated platform
- Cold start latency (10–30s) is visible to users; mitigated with loading states and V3 job queue
- Worker code is Python — requires context switching from the Node.js main codebase
- Modal's container system allows pinning specific CUDA versions, library versions, and Blender builds

---

## ADR-002: Next.js Full-Stack (No Separate Backend)

**Status:** Accepted  
**Date:** V1 design phase

### Context
The platform needs a frontend UI and a backend API. The decision was whether to use a separate backend service (Node.js/Express, FastAPI, etc.) or a full-stack framework.

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| **Next.js full-stack** | Single codebase, single deploy, SSR + API routes, TypeScript throughout | API routes are not independently scalable |
| **Next.js + separate Express API** | Independent scaling | Two codebases, two deployments, CORS complexity |
| **Next.js + FastAPI (Python)** | Python for ML-adjacent logic | Two languages, two deployments, type mismatch |
| **Remix** | Good full-stack DX | Smaller ecosystem; less FFmpeg/media processing precedent |

### Decision
**Next.js full-stack.** For a solo-built platform where the API routes do not need independent scaling from the frontend, a single codebase is strictly better. All API routes are stateless — they process a request and return. If scaling is needed, the Next.js standalone build can be scaled horizontally behind a load balancer.

### Consequences
- One deployment, one environment config, one TypeScript project
- API routes share type definitions with frontend components — no API contract drift
- FFmpeg processing in API routes means file I/O happens on the same server as the UI — acceptable for current scale
- V3 API layer would require extracting API routes to a standalone service

---

## ADR-003: Credit Model, Not Subscription

**Status:** Accepted  
**Date:** V1 design phase

### Context
A monetization model was needed. The two primary options for self-serve SaaS are subscription (monthly fee for access) and consumption-based (pay per use).

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| **Monthly subscription** | Predictable revenue; simple | Misaligns with intermittent usage; high churn from occasional users |
| **Per-minute pricing** | Aligns with actual video processing time | Creates anxiety for long files; hard to communicate |
| **Freemium (feature gate)** | Easy top-of-funnel | Frustrates users; feels manipulative; increases support tickets |
| **Credit bundles (chosen)** | Aligns with usage; no waste; admin-adjustable | More complex to communicate than flat subscription |

### Decision
**Credit bundles.** Users buy credits upfront and spend them per tool use. Credits do not expire. Each tool has a configurable credit cost stored in the database — no code deploy needed to adjust pricing.

### Consequences
- Revenue is lumpy (credit purchases) rather than recurring (subscription) — requires cash-in-advance purchases to fund operations
- Credit cost per tool can be adjusted in real time by admin — enables live A/B pricing experiments
- Users with 0 credits are blocked from processing — prevents debt; requires clear UX for credit purchase CTA
- Free credits on signup (20) reduces friction for first-time users without permanent free tier commitment

---

## ADR-004: FFmpeg Bundled via npm Package

**Status:** Accepted  
**Date:** V1 design phase

### Context
All standard media tools require FFmpeg. The decision was how to make FFmpeg available in the Next.js server environment.

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| **System FFmpeg (apt install)** | Full-featured, latest version | Requires Dockerfile; environment-specific; breaks on Hostinger |
| **ffmpeg-static npm package** | Zero system dependency; deterministic version | Older FFmpeg version; no GPU acceleration |
| **CloudConvert API** | No binary management | External dependency; per-conversion cost; privacy concern |
| **Cloudflare Workers (WASM FFmpeg)** | Serverless, no binary | WASM FFmpeg is incomplete; size limits |

### Decision
**`ffmpeg-static` npm package** with `ffprobe-static` companion. The package bundles the FFmpeg binary as a Node.js package dependency — it resolves to the correct path at runtime regardless of environment. No Dockerfile, no system configuration, no external API call.

### Consequences
- FFmpeg version is pinned to what `ffmpeg-static` ships — no access to bleeding-edge codecs
- No GPU-accelerated encoding (no NVENC) — all encoding is CPU-bound
- Works identically in development (Windows/Mac) and production (Linux) — zero environment divergence
- Upgrading FFmpeg means upgrading the npm package — no infrastructure change

---

## ADR-005: Supabase for Database, Auth, and Storage

**Status:** Accepted  
**Date:** V1 design phase

### Context
The platform needs: a relational database (user profiles, credits, usage logs), an authentication system (OAuth + JWT), and file storage (temp uploads, output files). These could be separate services or combined.

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| **Supabase** | PostgreSQL + Auth + Storage + RLS in one | Vendor lock-in; limited advanced PostgreSQL features in hosted version |
| **Firebase** | Easy Auth + Storage | NoSQL only; no relational queries; weaker RLS |
| **PlanetScale + Auth0 + S3** | Best-in-class each | Three services; three billing accounts; three SDKs |
| **Railway (PostgreSQL) + Clerk + S3** | Good DX | Multiple vendors; more integration surface |

### Decision
**Supabase.** Single vendor for all three needs. Row-Level Security at the database level means per-user data isolation is enforced at the infrastructure layer, not in application code — a significant security advantage. The Supabase client SDK works in both server-side (API routes) and client-side (React) contexts.

### Consequences
- All data operations go through Supabase — single point of failure, but Supabase's uptime is strong (99.9% SLA on Pro plan)
- RLS policies handle auth enforcement at DB level — reduces risk of application-layer auth bypass
- Storage URLs are time-limited by default — aligns with privacy-first file retention model
- Migrating off Supabase would require replacing auth, DB, and storage simultaneously — vendor lock-in is real

---

## ADR-006: Dual Rendering Engines (Remotion + Blender)

**Status:** Accepted  
**Date:** V2 design phase

### Context
AI video generation tools need a rendering engine. Two fundamentally different rendering approaches were available: web-native (React-based) and 3D/cinematic (Blender-based).

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| **Remotion only** | Web-native, React ecosystem, fast render | No 3D; limited to 2D kinetic typography |
| **Blender only** | Full 3D, cinematic quality, industry standard | Slow render; complex setup; overkill for simple ads |
| **FFmpeg filter-based** | No external dependency | Limited creative output; no motion graphics |
| **Remotion + Blender (chosen)** | Best output for each use case | Two separate stacks to maintain |

### Decision
**Both.** Remotion for 2D kinetic typography ads (fast, web-native, React-driven) and Blender for 3D cinematic motion (premium, GPU-accelerated). Each engine is used for the appropriate tool:
- Ad Generator, Motion Artist (2D style) → Remotion
- Motion Artist (3D style), Industrial Pipeline → Blender

### Consequences
- Two Python worker stacks to maintain (Remotion requires Node.js in the Modal container; Blender requires its own Python environment)
- Users get materially different output quality options — a genuine product differentiator
- Blender rendering is slower (60–180s) than Remotion (15–45s) — reflected in different credit costs
- Blender headless on Linux requires specific CUDA + Python binding setup — documented in worker files

---

## ADR-007: Deduct Credits Before GPU Invocation

**Status:** Accepted  
**Date:** V2 design phase

### Context
AI tools incur real GPU cost (Modal.com billing). The question was: when should credits be deducted — before or after the job runs?

### Options Considered

| Option | Pros | Cons |
|---|---|---|
| **Deduct after success** | User only pays for successful jobs | Abuse vector: user can queue unlimited jobs; no cost control |
| **Deduct after completion (success or fail)** | Simple; fair | Still no protection against concurrent request flooding |
| **Deduct before invocation (chosen)** | Prevents runaway spend; forces user to have credits before using GPU | User pays even if job fails — requires refund logic for errors |
| **Deduct after + rate limiting** | Fair + abuse protection | Complex to implement; still doesn't cap a single expensive job |

### Decision
**Deduct credits before GPU invocation.** Credits are deducted from the user's profile (atomic DB UPDATE) before the Modal worker is called. If the worker fails, the API route catches the error and issues a credit refund (another DB UPDATE). This is the "pre-authorization" model used by payment systems.

### Consequences
- Zero risk of runaway GPU spend — a user with 0 credits cannot trigger a Modal job
- Failed jobs require explicit refund logic — implemented in all GPU route error handlers
- Users sometimes perceive this as "unfair" (credits deducted even if job fails) — mitigated by fast refund and clear error messaging
- Credit deduction + usage log write are done in the same DB transaction — atomic; no partial state

---

## Decision Summary

| ADR | Decision | Key Tradeoff Accepted |
|---|---|---|
| ADR-001 | Modal.com for GPU | Cold start latency for zero idle cost |
| ADR-002 | Next.js full-stack | No independent API scaling for simplicity |
| ADR-003 | Credit model | Lumpy revenue for usage alignment |
| ADR-004 | ffmpeg-static | Older FFmpeg for zero infra dependency |
| ADR-005 | Supabase | Vendor lock-in for unified data layer |
| ADR-006 | Remotion + Blender | Two stacks for output quality breadth |
| ADR-007 | Deduct before GPU | Refund complexity for cost control |
