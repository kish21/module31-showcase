# Failure Modes & Resilience — Module31

This document covers how Module31 behaves when external dependencies fail, and what mitigations are in place to protect users and the platform's integrity.

---

## Dependency Map

```
Module31
  ├── Supabase (PostgreSQL + Auth + Storage)   — CRITICAL
  ├── Modal.com (GPU workers)                  — HIGH (AI tools only)
  ├── OpenAI GPT-4o API                        — MEDIUM (AI Creative Director)
  ├── Google Gemini API                        — MEDIUM (Vision Agent)
  ├── Sarvam AI TTS                            — MEDIUM (Translator, Indic TTS)
  ├── Hostinger (hosting)                      — CRITICAL
  └── FFmpeg (bundled binary)                  — HIGH (standard tools)
```

---

## Failure Mode 1: Supabase Unavailable

**Impact:** Auth fails → no login/signup. Credit checks fail → all tools blocked for authenticated users. Storage unavailable → cannot write output files.

**Detection:** Supabase client throws connection error on any DB or storage operation.

**Behavior:**
- If auth fails on page load: user sees a "Service temporarily unavailable" message; guest trial still works for tools that don't require credit deduction
- If credit check fails mid-request: API route returns 503 with message "Unable to verify credits — please try again"
- If storage write fails after processing: temp file is cleaned up; user sees error and is advised to retry; **credits are refunded** (since the job did not complete successfully)

**Mitigation:**
- Supabase Pro plan includes 99.9% uptime SLA
- Credit deduction and usage log are written before storage upload — if storage fails, the refund logic in the error handler restores credits
- No user data is permanently lost: FFmpeg output is in temp disk until cleanup runs

**V3 improvement:** Retry queue for storage writes (3 attempts with exponential backoff) before triggering refund.

---

## Failure Mode 2: Modal.com Worker Unavailable or Times Out

**Impact:** AI tools (Translator, Vision, Ad Generator, Motion Artist, Industrial Pipeline) cannot process jobs.

**Detection:** Shell command `python -m modal run <worker>` returns non-zero exit code, or Next.js API route hits its timeout threshold.

**Behavior:**
- API route catches the subprocess error
- **Credits are refunded** to the user's profile (atomic DB UPDATE)
- Error is logged to `usage_logs` with `status: 'error'`
- User receives a specific error message: "AI processing failed — your credits have been refunded. Please try again."
- Standard (FFmpeg) tools are completely unaffected — they have no Modal dependency

**Mitigation:**
- Credit deduction + refund are atomic DB operations — no partial state where credits are deducted without refund on failure
- Each worker has its own failure surface — one worker crashing does not affect others
- Modal workers are stateless — retrying the job creates a fresh container

**V3 improvement:** Async job queue (Redis/BullMQ) means timeout is no longer a risk — the API route returns a job ID immediately, and the worker runs to completion regardless of HTTP connection state.

---

## Failure Mode 3: Modal Cold Start Delay

**Impact:** AI tool response is slow (10–30 second delay before processing begins). Not a failure — but user-perceived as one if not communicated.

**Detection:** Not detectable before it happens; observable as long loading times.

**Behavior:**
- Loading spinner shown on frontend during processing
- Progress message: "Starting AI worker… (this may take up to 30 seconds on first use)"
- If timeout occurs: treated as Failure Mode 2 (refund + error)

**Mitigation:**
- User expectation set in tool UI ("AI processing typically takes 30–120 seconds")
- Modal workers can be "warmed" by keeping one container alive — not implemented in V2 (adds idle cost), planned for high-demand tools in V3

---

## Failure Mode 4: OpenAI API Unavailable or Rate Limited

**Impact:** AI Creative Director (`/api/ai-creative-director`) cannot generate scene graphs. Affects: Ad Generator, Motion Artist tools that depend on GPT-4o for storyboard generation.

**Detection:** OpenAI API returns 429 (rate limit) or 5xx (server error).

**Behavior:**
- API route returns error to frontend: "AI scene generation is temporarily unavailable — please try again in a few minutes"
- Credits are NOT deducted if the error occurs before GPU invocation (credit deduction happens after scene graph is successfully generated)
- If rate limited: UI shows specific message "Service is busy — please wait a moment and retry"

**Mitigation:**
- Exponential backoff retry (3 attempts) on 429 rate limit responses
- Scene graph generation is the first step — failure here is cheap (no GPU cost incurred)

---

## Failure Mode 5: Gemini API Unavailable

**Impact:** Vision Agent tool cannot analyze video frames.

**Detection:** Gemini API client throws connection error or returns 5xx.

**Behavior:**
- Credits are refunded (same refund flow as Modal failure)
- User receives: "Vision analysis is temporarily unavailable — your credits have been refunded"

**Mitigation:**
- Vision Agent credits are deducted before Gemini call — refund is triggered in error handler
- Gemini API has independent SLA from Modal; isolated failure surface

---

## Failure Mode 6: FFmpeg Binary Failure

**Impact:** All standard (non-AI) tools fail — the ffmpeg-static binary crashes or produces corrupt output.

**Detection:** `fluent-ffmpeg` callback returns error code or stderr output indicates failure.

**Behavior:**
- API route catches FFmpeg error
- Credits are refunded (same refund flow)
- Temp files are cleaned up
- Error is logged with FFmpeg stderr output for debugging
- User receives: "Processing failed — your credits have been refunded. If this persists, contact support."

**Mitigation:**
- `ffmpeg-static` ships a pinned, tested binary — not subject to OS-level updates
- FFmpeg errors are almost always input-related (corrupt file, unsupported codec) — the user's input is the failure point, not the binary
- All API routes validate file type and size before FFmpeg is invoked — filters out most invalid inputs

---

## Failure Mode 7: Hostinger Server Unavailable

**Impact:** Entire platform unavailable.

**Detection:** External uptime monitoring (planned — not yet implemented in V2).

**Behavior:**
- Platform is down — no degraded mode possible
- No data loss: Supabase and Modal are independent; data is safe

**Mitigation:**
- Hostinger offers 99.9% uptime on business plans
- Next.js standalone build can be migrated to another host (Vercel, Railway) within hours if needed
- V3: Add uptime monitoring (Uptime Robot or Better Uptime) with alerting

---

## No Double-Charge Guarantee

The most important user-facing resilience property: **users are never charged credits for a job that did not complete successfully.**

This is enforced by:
1. Credits deducted atomically before GPU/processing invocation
2. Every API route error handler issues a credit refund as the first recovery action
3. Usage log records `status: 'error'` so support can verify refund was issued
4. Refund is a server-side DB operation — cannot be blocked by network issues on the client side

If a user reports a credit deduction without a result, support can verify via `usage_logs` and manually refund via the admin dashboard.

---

## Guest Mode Degradation

If Supabase is unavailable, guest users lose credit tracking but can still use tools — since guest mode does not require DB writes for basic tool use. However, watermark application requires a config read from `system_config` — if that fails, watermark defaults to enabled with fallback text.

| Supabase State | Authenticated User | Guest User |
|---|---|---|
| Available | Full functionality | Full guest trial |
| DB unavailable | Blocked (cannot verify credits) | Standard tools work; watermark uses fallback default |
| Auth unavailable | Cannot log in | Standard tools work |
| Storage unavailable | Cannot get output URLs | Cannot get output URLs |

---

## Summary of Refund Triggers

| Trigger | Refund Issued? | Mechanism |
|---|---|---|
| Modal worker error | Yes | Error handler in GPU API route |
| Modal timeout | Yes | Timeout catch in GPU API route |
| OpenAI API error | Yes (if after deduction) | Error handler in AI Director route |
| Gemini API error | Yes | Error handler in Vision route |
| Supabase storage failure post-processing | Yes | Error handler after FFmpeg success |
| FFmpeg error | Yes | Error handler in FFmpeg routes |
| User cancels during processing | No | Job already running; cannot be recalled |
