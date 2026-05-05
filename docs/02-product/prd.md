# Product Requirements Document — Module31

**Version:** 2.0 (post-V2 AI tools launch)  
**Author:** Kishore K V  
**Status:** Living document — updated with each major release

---

## 1. Problem Statement

Content creators and marketing teams use 7+ separate tools to process media, each storing files permanently and charging monthly subscriptions for intermittent use. No unified, privacy-respecting, pay-per-use alternative exists in the mid-market segment.

See [Problem Statement](../01-discovery/problem-statement.md) for full detail.

---

## 2. Goals

| Goal | Success Metric |
|---|---|
| Consolidate media workflow into one platform | ≥ 30 tools covering all common creator operations |
| Deliver credible privacy guarantee | Files auto-deleted within configurable window (default 2 hours) |
| Enable pay-per-use monetization | Credit-based model with admin-controlled per-tool costs |
| Reduce activation friction | Guest trial with 5 free watermarked uses before signup gate |
| Support AI-powered tools at scale | GPU workers that scale to zero — no idle compute cost |

## 3. Non-Goals

| Non-Goal | Rationale |
|---|---|
| Real-time collaborative editing | Adds significant complexity; not the core use case |
| Mobile app (iOS / Android) | Desktop workflow dominates media processing; browser is sufficient |
| Video hosting / CDN | Not a distribution platform — processing only |
| Freemium with feature gating | Feature gating creates resentment; credit model is cleaner |
| Enterprise SSO / procurement | Solo-operated; self-serve only in V1 and V2 |
| Offline processing | Browser-based platform; connectivity assumed |

---

## 4. User Personas

See [User Personas](../01-discovery/user-personas.md) for full profiles. Summary:

- **Arjun** — Volume creator; needs speed, AI translation, and cross-platform crop
- **Priya** — Agency PM; needs privacy guarantee and captioning at scale
- **Vikram** — Indie developer; needs infrastructure replacement and eventual API access

---

## 5. Feature Requirements

### P0 — Must Have (V1 Launch)

#### 5.1 Tool Grid Homepage
- Searchable, filterable grid of all tools
- Category tabs: Video, Audio, AI, Coming Soon
- Each card shows: tool name, description, credit cost, usage count
- Guest trial banner shown for unauthenticated users
- Credit balance shown in header for authenticated users

#### 5.2 Standard Media Tools (FFmpeg-powered)
Each tool follows: upload → validate → process → return download URL

| Tool | Key Requirement |
|---|---|
| Compress | Target size or quality slider; output < 80% of input size |
| Quick Trim | Start/end timestamp inputs; frame-accurate cut |
| Social Crop | 9:16 output; auto-center or manual position |
| Video to GIF | Max 10MB output; configurable frame rate |
| Format Converter | MP4, MOV, AVI, WebM input/output |
| Extract Audio | MP3 or WAV output |
| Audio Normalizer | LUFS target normalization |
| Hardcoded Subtitles | Burn captions into video; font and position options |
| Video Concatenator | Join up to 5 clips; same format required |
| Thumbnail Generator | Extract any frame as JPEG/PNG |
| Speed Master | 0.25x to 4x speed range |
| Resolution Scaler | Target width/height with aspect ratio lock |
| Frame Rate Converter | 24, 25, 30, 60 fps targets |
| Metadata Stripper | Remove all embedded metadata from file |
| Flip / Rotate | Horizontal flip, vertical flip, 90/180/270 rotation |
| Visual Adjustments | Brightness, contrast, saturation, hue sliders |
| Audio Waveform | Waveform visualization video from audio file |
| Audio Remover | Mute video; keep video stream only |

#### 5.3 Authentication
- Supabase OAuth (Google, GitHub, email)
- User profile with role field (customer / admin / superadmin)
- 20 free credits on new account creation
- Credit balance visible in header at all times

#### 5.4 Guest Trial
- 5 free uses before signup gate (configurable via `system_config`)
- Watermark applied to output (text, color configurable)
- Attempt count tracked in localStorage + server-side session
- Signup gate modal on trial limit hit
- Guest trial works for all standard tools

#### 5.5 Credit System
- Each tool deducts a configurable number of credits
- Credit deduction logged to `usage_logs` table
- Users with 0 credits are blocked from processing (error shown)
- Credit cost per tool stored in `system_config` — no redeploy needed

---

### P1 — High Priority (V2 Launch)

#### 5.6 AI Tools (GPU-Powered)

| Tool | Stack | Key Requirement |
|---|---|---|
| Video Translator | Faster Whisper + Sarvam AI TTS | 30+ language pairs; dubbed audio synced to original timing |
| Vision Agent | Gemini 1.5 Pro | Frame-by-frame analysis; JSON output + human-readable summary |
| AI Ad Generator | GPT-4o + Remotion | Text prompt → motion typography video; 4 animation styles |
| AI Motion Artist | GPT-4o + Blender | Text prompt → cinematic 3D motion video on NVIDIA T4 |
| Industrial Pipeline | GPT-4o + Blender SVD | Product photo input → commercial-quality video output |

- All AI tools: credits deducted **before** Modal worker invocation
- Progress indicator shown during GPU processing (jobs can take 30–180 seconds)
- On worker failure: credits refunded; error message shown
- Output file URL returned and available for download for the retention window

#### 5.7 Admin Dashboard
- User list with credit balance display
- Manual credit refill for individual users
- Support ticket view and response
- Accessible only to `admin` and `superadmin` roles

#### 5.8 System Config Panel
- Key/value pairs editable via UI (no DB console needed)
- Required config keys:
  - `guest_trial_limit` — number of free uses
  - `guest_watermark_enabled` — boolean
  - `guest_watermark_text` — string
  - `guest_watermark_color` — color code
  - `video_retention_hours` — cleanup interval
  - Per-tool credit costs (e.g., `translator_cost`, `ad_generator_cost`)
- Changes take effect immediately — no server restart required

---

### P2 — Nice to Have (V3 Planned)

#### 5.9 Developer API
- Authenticated REST endpoints for all tools
- API key management per user
- Webhook callbacks for async AI jobs
- Rate limiting and per-key credit assignment

#### 5.10 Team Workspaces
- Shared credit pool for a team
- Shared output library (recent processed files)
- Admin user for workspace management

#### 5.11 Job Queue
- Redis/BullMQ queue for AI tool requests
- Prevents timeout on slow Modal cold starts
- Webhook or polling endpoint for job status

---

## 6. User Flows

### 6.1 Guest Trial Flow
```
Land on homepage
  → Click any tool card
  → Tool page loads (no signup required)
  → Upload file → Process → Download (with watermark)
  → Attempt count incremented (localStorage + server)
  → After 5 uses: SignupGateModal shown
  → User signs up → 20 free credits → watermark removed
```

### 6.2 Authenticated Standard Tool Flow
```
User selects tool from grid
  → Tool page loads
  → System checks credit balance
  → If 0 credits: show "Buy credits" CTA
  → User uploads file
  → API validates file (type, size)
  → Credits deducted from profile
  → Usage logged to usage_logs
  → FFmpeg processes file
  → Output URL returned
  → User downloads file
  → File scheduled for deletion after retention window
```

### 6.3 AI Tool Flow (GPU)
```
User selects AI tool
  → System checks credit balance
  → Credits deducted (before GPU invocation)
  → API writes input to temp storage
  → Shells out: python -m modal run <worker>
  → Progress indicator shown (polling or timeout)
  → Modal worker processes on GPU
  → Output URL returned
  → User downloads result
```

---

## 7. Technical Requirements

| Requirement | Specification |
|---|---|
| Max upload size | 1GB (configurable via Next.js body limit) |
| Supported video formats | MP4, MOV, AVI, WebM, MKV |
| Supported audio formats | MP3, WAV, AAC, M4A |
| Processing timeout | 5 min for standard tools; 10 min for AI tools |
| Output retention | Configurable (default: 2 hours) |
| Concurrent users | Stateless API — limited by server capacity, not design |
| Auth session | Supabase JWT; server-side validation on all API routes |
| Credit deduction | Atomic DB operation; no partial deductions |

---

## 8. Out of Scope (Explicitly)

- Video player or preview within the platform
- Comments, annotations, or collaboration on processed files
- Batch processing of multiple files simultaneously (V3+)
- Integration with YouTube / TikTok / Instagram APIs for direct publish
- AI-generated voiceover (separate from translation)

---

## 9. Open Questions (Resolved)

| Question | Resolution |
|---|---|
| Should credits be per-use or per-minute of video? | Per-use — simpler to communicate; per-minute creates anxiety for long files |
| Should guest trial be tracked server-side or client-side? | Both — localStorage for speed; server session for anti-abuse |
| Should we gate AI tools behind a higher credit tier? | No — all tools available on the same credit system; cost reflects GPU expense |
| Should output files be stored in Supabase Storage or returned inline? | Supabase Storage URL with time-limited access; better for large files |
| Should Modal workers be synchronous or async? | Synchronous in V2 (simpler); async with job queue planned for V3 |
