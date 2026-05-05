# Lessons Learned — Module31

An honest retrospective. What worked well, what was unexpectedly hard, and what I would do differently if building this again.

---

## What Worked Well

### 1. Modular Tool Architecture
The decision to make each tool a fully independent page + API route was the single best architectural decision. It meant:
- No tool blocked another from shipping
- Bugs were isolated — a broken compression tool had zero impact on the translation tool
- Onboarding to a single tool required understanding only that tool, not the whole system

If I had built a shared processing pipeline (one API route that branched by tool type), the codebase would have been significantly harder to extend and debug.

### 2. Supabase RLS for Auth
Implementing Row-Level Security at the database layer meant I never had to worry about application-level auth bypass. Even if a bug in my API code failed to check the user's JWT, the database would still refuse to return or modify another user's data. This is a meaningful security property for a platform handling user files.

### 3. ToolPageWrapper Pattern
Writing the guest trial, signup gate, and watermark logic once in a wrapper component and applying it to every tool page saved significant time and ensured consistent behavior. When I needed to change the guest trial limit, one change propagated everywhere.

### 4. Admin-Driven Config from Day One
Building the `system_config` table and admin config panel early meant I never had to redeploy to make operational changes. Adjusting credit costs for AI tools as GPU pricing became clearer took seconds, not engineering cycles. If I had hardcoded costs, I would have burned redeploys on every pricing adjustment.

### 5. ComingSoon Stubs for Signal
Publishing stub pages for planned tools before building them gave me real click data on user interest. Two tools I planned to build (video background remover, AI upscaler) had low click-through on their stubs — I deprioritized them and spent that time on tools with higher signal. This saved at least 2 weeks of building things nobody would use.

### 6. Credits Before GPU Invocation
This decision was uncomfortable at first (users pay even for failed jobs — requires refund logic) but proved to be the right call. Zero incidents of uncontrolled GPU spend. The refund path is simple and reliable. The alternative — deducting after success — would have required more complex abuse prevention logic.

---

## What Was Unexpectedly Hard

### 1. Blender Headless on Linux
Getting Blender 4.0 to run headlessly on Modal's Linux containers was significantly harder than expected. The challenges:
- Specific CUDA version requirements for GPU-accelerated rendering
- Python-Blender binding (`bpy`) requires exact version matching
- Font rendering in headless mode requires explicit font path configuration
- The CYCLES render engine needed explicit GPU device selection

**Time cost:** ~4 days of debugging. Would have been 1 day with better documentation of the exact environment stack.

**Fix:** Pinned the exact CUDA version, Blender binary, and Python version in the Modal container spec. Documented the exact working configuration in the worker file comments.

### 2. Whisper Memory on Long Files
Faster Whisper Large-v3 on an NVIDIA T4 (16GB VRAM) runs out of memory for audio files longer than ~10 minutes when using the full model with beam search.

**Time cost:** ~2 days to diagnose and implement chunked audio processing.

**Fix:** Split audio into 5-minute chunks, process each chunk independently, merge transcripts. Added duration check before processing — long files trigger chunked mode automatically.

### 3. Guest Trial Anti-Abuse
The initial implementation tracked guest attempts in localStorage only. Within the first few days of testing, it was clear that a private browsing window reset the counter. This is a meaningful abuse vector: a user could get unlimited free uses by repeatedly opening private windows.

**Fix:** Dual tracking — localStorage (for speed and offline resilience) + server-side session header (`X-Guest-Attempt-Count`). The server validates the attempt count and rejects requests that exceed the limit regardless of localStorage state.

### 4. Synchronous Modal Timeout
Running Modal workers synchronously (waiting for the Python subprocess to complete before returning an HTTP response) caused timeout issues for long Blender renders. Next.js API routes have a default timeout, and a 3-minute Blender render would exceed it.

**Fix:** Increased the API route timeout configuration. Added user-facing messaging ("This may take 2–3 minutes for 3D renders"). The root fix (async job queue) is planned for V3.

**Lesson:** Synchronous GPU jobs are a design smell for anything over 30 seconds. An async job queue should have been in V2, not V3.

### 5. File Type Validation
Early versions of the API routes trusted the file extension declared by the client. A user could rename a `.txt` file to `.mp4` and submit it — FFmpeg would fail with a cryptic error.

**Fix:** Added MIME type validation using the actual file buffer magic bytes, not the client-declared type. Files that don't match expected MIME types for their extension are rejected with a clear error before FFmpeg is invoked.

---

## What I'd Do Differently

### 1. Add a Job Queue from Day One (V1)
The biggest regret. A Redis/BullMQ job queue for AI tools would have:
- Eliminated timeout issues entirely (API returns job ID immediately)
- Enabled webhook callbacks for external integrations
- Made the system more observable (queue depth, job status, retry counts)
- Required only one architectural component that would have been useful from V1

The cost of adding it to V3 is much higher than building it in V1 — existing frontend and API code needs to change to accommodate async job status polling.

### 2. Automated Tests for the Credit System
The credit system (deduction, refund, usage logging) is the most financially sensitive part of the platform. It was tested manually. An automated test suite for the credit flow — especially the refund path on AI tool failure — would have caught two bugs during development that I found manually.

**Lesson:** Any code path that involves money (even credits) deserves automated tests. Manual testing is not sufficient.

### 3. Earlier Uptime Monitoring
The platform went live without external uptime monitoring. If Hostinger had an outage, I would have found out from a user message, not an alert. Setting up Uptime Robot (free tier) from day one would have been 10 minutes of work.

### 4. Structured Logging from Day One
Error handling logs to `console.error` in V2. There is no structured logging pipeline (no Sentry, no Logtail, no Datadog). Debugging production issues requires checking server logs manually.

**Lesson:** Structured logging is cheap to add at the start and expensive to retrofit. Even a free Sentry tier would have caught silent errors I didn't know about.

### 5. Schema Migrations Over Manual DB Changes
Supabase schema changes in V2 were made via the Supabase UI (SQL editor). There is no migration history. If the schema needs to be reproduced (new environment, disaster recovery), it requires manually reconstructing the SQL from memory.

**Lesson:** Even a simple migration file (`001_initial_schema.sql`, `002_add_support_tickets.sql`) would have made the schema reproducible and auditable.

---

## The One Thing I'm Proudest Of

The **credit-before-GPU decision** and its implementation. It was a product decision (user experience: refund on failure), an architecture decision (deduct before invocation), and an operations decision (no runaway GPU spend) — all in one pattern. Getting this right from V1 meant zero cost incidents and zero user complaints about double-charging throughout development and launch.

It's a small thing that shows the intersection of product thinking and system design that I value most: a decision that is simultaneously good for the user, good for the business, and simple to implement.
