# Tech Stack — Module31

## Stack at a Glance

| Layer | Technology | Version |
|---|---|---|
| Frontend Framework | Next.js | 15+ |
| UI Library | React | 19 |
| Language | TypeScript | 5 |
| Styling | Tailwind CSS | 4 |
| Animation | Framer Motion | 12 |
| Icons | Lucide React | 0.576 |
| Database + Auth + Storage | Supabase | 2.x |
| Media Processing | FFmpeg (via ffmpeg-static) | 6.x |
| GPU Workers | Modal.com | Python 3.11 |
| AI (Storyboard) | OpenAI GPT-4o | API |
| AI (Vision) | Google Gemini 1.5 Pro | API |
| AI (ASR) | Faster Whisper Large-v3 | Python |
| AI (TTS) | Sarvam AI | API |
| AI (TTS, Indic) | IndicF5 / Fish Speech | Python |
| 2D Rendering | Remotion | 4.x |
| 3D Rendering | Blender | 4.0.2 |
| Deployment | Hostinger | Node.js standalone |

---

## Detailed Stack Decisions

### Frontend

#### Next.js 15 + React 19
**Why:** Full-stack capability — handles both the UI and API routes in one codebase. Server-side rendering improves initial load performance. App Router (introduced in Next.js 13) enables file-system-based routing that maps cleanly to the tool-per-page architecture.

**Alternative considered:** Remix — similar full-stack model, but smaller ecosystem and less FFmpeg/media precedent in the community.

**Key usage:** Each tool is a `page.tsx` file under `/app/<tool-name>/`. API routes live under `/app/api/<tool-name>/route.ts`. Shared layouts handle auth state, navigation, and credit display.

#### TypeScript 5
**Why:** Type safety across the entire codebase — frontend components, API route handlers, Supabase query results, and shared utility functions all share types. Catches interface mismatches between frontend and backend at compile time.

**Alternative considered:** Plain JavaScript — rejected because the surface area of this codebase (30+ tools, multiple AI integrations, complex auth state) would be unmanageable without types.

#### Tailwind CSS 4
**Why:** Utility-first styling allows rapid UI iteration without leaving the component file. Dark mode support is built-in. The `cn()` utility (clsx + tailwind-merge) handles conditional class composition.

**Alternative considered:** CSS Modules — more explicit but slower to iterate. Styled Components — runtime CSS-in-JS has performance overhead.

#### Framer Motion 12
**Why:** Declarative animation for tool card entrance animations, modal transitions, and loading states. The `AnimatePresence` component handles unmount animations cleanly.

**Alternative considered:** CSS transitions — sufficient for simple cases, but Framer Motion's layout animations and gesture support would require significant custom CSS to replicate.

---

### Backend (Next.js API Routes)

#### fluent-ffmpeg + ffmpeg-static
**Why:** `fluent-ffmpeg` provides a Node.js API for FFmpeg that is more readable than raw shell commands. `ffmpeg-static` bundles the FFmpeg binary as an npm package — no system installation required, works identically on Windows (dev) and Linux (production).

**Alternative considered:** Child process with raw FFmpeg shell command — possible, but harder to read, escape, and maintain. Cloud video APIs (Cloudinary, Mux) — external dependency, cost per operation, privacy concern.

**Key pattern:**
```
ffmpeg.setFfmpegPath(require('ffmpeg-static'))
ffmpeg(inputPath)
  .videoCodec('libx264')
  .outputOptions(['-crf 28'])
  .output(outputPath)
  .run()
```

#### uuid
**Why:** Generates unique temp file names (`<uuid>-<timestamp>-input.mp4`) to prevent filename collisions when multiple users process files simultaneously.

---

### Data Layer

#### Supabase (PostgreSQL + Auth + Storage)
**Why:** Replaces three separate services (database, auth provider, object storage) with one. Row-Level Security at the PostgreSQL layer enforces per-user data isolation without application-level access control logic. The `@supabase/ssr` package handles server-side JWT validation in API routes.

**Alternative considered:** Firebase — NoSQL only; no relational queries for credit deduction and usage log aggregation. PlanetScale + Clerk + S3 — best-in-class each, but three vendors, three billing accounts, three SDKs.

**Key patterns:**
- Credit deduction: `UPDATE profiles SET credits = credits - $cost WHERE user_id = $id AND credits >= $cost RETURNING credits` (atomic; fails safely if credits insufficient)
- RLS: `CREATE POLICY "Users read own profile" ON profiles FOR SELECT USING (auth.uid() = user_id)`

---

### GPU Compute

#### Modal.com
**Why:** Serverless GPU — containers spin up on demand and scale to zero. Python-native environment allows running arbitrary code: Blender, Remotion (Node.js in container), Whisper, custom scripts. No idle cost — critical for a solo-operated platform where GPU would otherwise sit idle most of the day.

**Alternative considered:** AWS Lambda — no GPU support. EC2 GPU instance — $200–500/month idle cost. Replicate — limited to pre-packaged models; cannot run custom Blender builds.

**Key pattern:**
```python
@modal.function(gpu="T4", timeout=600)
def process_video(input_url: str, config: dict) -> str:
    # Download, process, upload, return output URL
    ...
```

---

### AI Models

#### OpenAI GPT-4o — AI Creative Director
**Why:** Best instruction-following for structured JSON output (scene graphs). Used to translate user text prompts into structured rendering instructions (motion style, typography, timing, color palette).

**Alternative considered:** Claude 3.5 Sonnet — excellent reasoning, but GPT-4o's JSON mode (`response_format: { type: "json_object" }`) is more reliable for structured output at the time of implementation.

**Usage:** Called once per AI tool job (Ad Generator, Motion Artist) before GPU invocation. Adds ~1–2 second overhead.

#### Google Gemini 1.5 Pro — Vision Analysis
**Why:** Long context window (1M tokens) handles long video frame sequences. Native multimodal capability — video frames are passed directly without format conversion.

**Alternative considered:** GPT-4o Vision — comparable quality but higher per-call cost for the vision use case. Claude 3 Opus — strong vision but slower.

#### Faster Whisper Large-v3 — Speech-to-Text
**Why:** Fastest open-source ASR model at the Large-v3 quality level. Runs on Modal GPU (T4) — no per-character API cost. Handles 30+ languages with strong accuracy.

**Alternative considered:** OpenAI Whisper API — per-minute pricing; less cost-effective for long-form video. AssemblyAI — good API but external dependency and per-minute cost.

#### Sarvam AI — Indic Language TTS
**Why:** Best-in-class TTS for Indian languages (Hindi, Tamil, Telugu, Kannada, etc.). Specifically chosen for the Indic dubbing use case — no other cloud TTS service covers this language set with comparable quality.

**Alternative considered:** Google Cloud TTS — covers some Indic languages but quality is noticeably lower for regional languages. ElevenLabs — English-first; limited Indic support.

---

### Rendering Engines

#### Remotion 4
**Why:** React-based programmatic video rendering — the same React knowledge used for the frontend UI applies to animation definitions. Kinetic typography and motion graphics are defined as React components with timeline-based animation.

**Alternative considered:** After Effects + Bodymovin — professional quality but requires desktop software and manual export. Canvas API — requires significant custom code for each animation type.

**Usage:** 2D ad generation, kinetic motion templates. Runs in a Modal container (Node.js + Remotion CLI).

#### Blender 4.0.2
**Why:** Industry-standard 3D rendering. Headless mode (`blender --background --python script.py`) allows GPU-accelerated rendering without a display. CYCLES render engine on NVIDIA T4 produces cinematic-quality output.

**Alternative considered:** Three.js — web-based 3D; limited rendering quality vs Blender. Unreal Engine — overkill; no Python scripting for programmatic scene generation.

**Usage:** 3D cinematic motion videos, Industrial Pipeline (product photo → commercial). Runs in a Modal container with CUDA + Blender pre-installed.

---

### Deployment

#### Hostinger (Next.js Standalone)
**Why:** Cost-effective VPS hosting for Next.js standalone build. Standalone output bundles only required dependencies — smaller than a full `node_modules` deploy.

**Alternative considered:** Vercel — simpler deployment but higher cost at scale; serverless functions have 10MB limit (incompatible with ffmpeg-static binary). Railway — good DX but more expensive than Hostinger for persistent servers.

**Build config:** `output: 'standalone'` in `next.config.ts`. Body size limit set to 1GB for large video uploads.

---

## What's Not In the Stack (And Why)

| Technology | Why Not Used |
|---|---|
| Redis / job queue | Not yet needed in V2 (synchronous Modal); planned for V3 |
| Docker | Not required — ffmpeg-static eliminates binary dependency; Modal handles GPU containers |
| GraphQL | REST via Next.js API routes is sufficient; GraphQL overhead not justified |
| Prisma / ORM | Supabase client with raw SQL for credit operations; ORM adds abstraction without value here |
| React Native | Platform is desktop-browser-first; mobile app is out of scope |
| WebSockets | No real-time features in V2; would be needed for V3 job status streaming |
