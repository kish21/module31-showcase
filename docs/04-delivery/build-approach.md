# Build Approach — Module31

## Context

Module31 was built solo — one person owning product, design, architecture, frontend, backend, GPU infrastructure, and deployment. The challenge was not capability but prioritization and system design: how do you ship 30+ tools, 5 AI integrations, an admin dashboard, a credit system, and a guest trial funnel without a team?

The answer was a set of deliberate architectural and process decisions that maximized independent delivery and minimized coordination overhead.

---

## Core Principle: Every Tool Is an Island

The most important build decision was the **modular tool architecture**:

- Each tool lives in its own `page.tsx` — completely independent of other tools
- Each tool has its own API route — no shared processing logic between tools
- Adding a new tool requires: one page file + one API route. Nothing else changes.
- Removing or disabling a tool requires: removing the entry from the `TOOLS` array. Nothing else changes.

This meant:
- No tool blocked another tool from shipping
- Tools could be prioritized and shipped in any order
- A bug in the compression tool had zero blast radius on the translation tool
- A new developer (hypothetical) could onboard to one tool without understanding the whole system

---

## Delivery Sequence

### Phase 1: Infrastructure First (Week 1)

Before any tool was built, the platform foundation was established:
- Supabase schema: `profiles`, `usage_logs`, `system_config`
- Auth flow: OAuth login, session management, credit display in header
- Credit system: deduction logic, error handling, usage logging
- `ToolPageWrapper` component: handles guest trial, watermark banner, signup gate for every tool page

**Rationale:** Building infrastructure first meant every subsequent tool could be added in hours, not days. The credit system, auth flow, and guest trial logic only needed to be written once.

### Phase 2: High-Signal Standard Tools (Weeks 2–5)

Tools were shipped in order of expected demand, starting with the highest-volume use cases:
1. Compress (most common creator need)
2. Social Crop (Reels/Shorts/TikTok explosion)
3. Quick Trim (universal workflow step)
4. Format Converter (wide compatibility need)
5. Hardcoded Subtitles (accessibility + algorithm boost for creators)
6. Extract Audio, Normalize, Waveform (audio workflow)
7. Remaining tools (Flip, Speed, Scale, FPS, GIF, Concatenator, Thumbnail, Metadata, Adjust, Mute)

Each tool shipped with a `ComingSoon` placeholder in the grid first — this captured click data before the tool was built, validating priority.

### Phase 3: Admin Operations (Week 4, parallel to Phase 2)

Admin dashboard and config panel were built in parallel with the standard tools:
- User list + credit refill (needed for early testers)
- System config panel (needed to adjust watermark and costs without redeploys)
- Support ticket view (needed for issue tracking)

**Rationale:** Operating a live platform without admin tools is unsustainable. These needed to exist before real users arrived.

### Phase 4: GPU Infrastructure (Week 7–8)

Before the AI tools were built, the Modal.com pipeline was established:
- First Modal worker: a simple test worker that processed a file and returned a URL
- Established the shell-out pattern: `python -m modal run <worker>`
- Established the file-based IPC: worker reads input from Supabase URL, writes output to Supabase
- Established credit-before-GPU pattern and refund logic

**Rationale:** GPU infrastructure has more unknowns than FFmpeg processing. Building and testing the pipeline with a simple worker before adding AI complexity reduced risk.

### Phase 5: AI Tools (Weeks 8–14)

Each AI tool was shipped in order of technical risk:
1. **Video Translator** — highest value, most complex (Whisper + TTS + ffmpeg sync)
2. **Vision Agent** — simpler (Gemini API call + JSON parsing)
3. **AI Ad Generator** — GPT-4o + Remotion (two dependencies, medium complexity)
4. **AI Motion Artist** — GPT-4o + Blender (hardest — Blender headless setup)
5. **Industrial Pipeline** — extends Blender worker (lower marginal complexity)

---

## Key Build Patterns

### ComingSoon Stubs for Demand Signal

Every planned-but-not-yet-built tool was added to the `TOOLS` array with a `comingSoon: true` flag. This rendered a "Coming Soon" badge on the tool card and linked to a stub page.

**Result:** Click analytics on stub pages revealed which tools users actually wanted. Tools with high click-through on `ComingSoon` were prioritized. This prevented building tools that looked good on paper but had no real demand.

### `ToolPageWrapper` — Write Once, Apply Everywhere

Every tool page uses `ToolPageWrapper`, which handles:
- Guest trial attempt tracking (localStorage + API header)
- Signup gate modal (shown when trial limit hit)
- Watermark notice banner (for guest users)
- Credit balance check UI

**Result:** Adding a new tool page requires zero thought about auth, guest handling, or credit display. All of that logic is in the wrapper. A new tool page is just: upload UI + API call + download button.

### System Config — Admin-Driven Operations

All configurable values (credit costs, watermark settings, retention hours) are stored in `system_config` and read at request time. No hardcoded values for anything that might need to change.

**Result:** Adjusting pricing, toggling the watermark, or extending the retention window takes 10 seconds in the admin panel. No engineering cycle, no redeploy.

### Credits Before GPU — Cost Control Gate

Every AI tool route deducts credits before invoking Modal. The refund handler in the error path is the only complexity this adds.

**Result:** Zero incidents of uncontrolled GPU spend. Even if a user found a way to submit rapid requests, they would exhaust their own credits before causing material cost.

---

## Testing Approach

**Standard tools:** Manual testing with representative files (compressed MP4, long-duration files, edge-case formats). Each tool was tested with:
- A normal input file (expected path)
- An oversized file (> 500MB)
- An unsupported format (should reject gracefully)
- Concurrent requests (to verify temp file isolation by UUID)

**AI tools:** Manual testing with varied prompts and media lengths. Specifically tested:
- Short files (< 30 seconds) — should complete before timeout
- Long files (> 10 minutes) — chunked processing validation
- Invalid prompts — GPT-4o should return a usable scene graph even for vague inputs
- GPU cold start — first invocation delay handled by loading state

**Credit system:** Manual verification that:
- Credits deducted before GPU call
- Refund issued on failure
- Zero credits blocks tool access
- Admin credit refill reflected immediately

**No automated test suite exists in V2.** This is a known gap. See [Lessons Learned](lessons-learned.md).

---

## Deployment Process

1. Build: `next build` (generates `.next/standalone`)
2. Upload to Hostinger via SSH/FTP
3. Restart Node.js process
4. Verify: manual smoke test of compress tool + one AI tool

Modal workers are deployed separately:
1. `modal deploy src/modal-workers/<worker>.py` per worker
2. Workers are versioned independently of the Next.js app

No CI/CD pipeline in V2. Deployments are manual and infrequent (on feature completion). V3 target: GitHub Actions pipeline with smoke test on deploy.

---

## Solo Delivery Lessons

| Pattern | Why It Worked |
|---|---|
| Infrastructure before features | Every tool benefited from auth, credits, and guest trial without rebuilding them |
| ComingSoon stubs | Saved 2–3 weeks of building tools nobody would use |
| Modular tool architecture | Any tool could be shipped, debugged, or reverted independently |
| Admin-driven config | Eliminated engineering cycles for operational changes |
| Credits before GPU | Eliminated a whole class of potential cost incidents |
| One tool at a time | Focus beats parallelism for solo builders — tools shipped faster when done sequentially |
