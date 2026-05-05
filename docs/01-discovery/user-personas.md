# User Personas — Module31

---

## Persona 1: The Volume Creator

**Name:** Arjun, 27  
**Role:** Full-time YouTube creator (tech reviews), 180K subscribers  
**Location:** Bengaluru, India  
**Device:** MacBook Pro + iPhone for filming

### Background
Arjun uploads 3 videos per week. Each video requires compression (raw footage from camera is too large to store), trimming, subtitle generation, and a thumbnail. He also repurposes content for Instagram Reels (needs 9:16 crop) and occasionally creates a Hindi-dubbed version for a second channel.

### Current Stack
- HandBrake (compress) — free but desktop, slow
- Kapwing (subtitles) — $24/mo
- iMovie (trim) — free but no browser access
- Canva (thumbnail) — $13/mo
- Manual subtitle translation — done himself, 2–3 hours per video

**Monthly spend: ~$37/month + significant time**

### Jobs To Be Done
- Compress 4K footage to upload-ready size in under 5 minutes
- Generate accurate Hindi subtitles without manual typing
- Crop the same video for YouTube and Reels without re-editing
- Create a thumbnail quickly without switching to another app

### Frustrations
- Kapwing stores his videos even after he deletes the project — he's not sure where they go
- HandBrake requires a desktop and takes too long
- Subtitles on Kapwing are good but the export quality sometimes drops
- He wishes he could translate the audio to Hindi automatically instead of re-recording

### How Module31 Serves Arjun
- Compress, Trim, Social Crop, Thumbnail Generator — all in one platform, browser-based
- Video Translator: auto-dubs Hindi from English audio (Faster Whisper + Sarvam AI TTS)
- Hardcoded Subtitles: burns captions directly into the video for Reels
- Credit model: Arjun processes ~15 files/week. At 2 credits per standard tool and 25 for translator, his monthly spend is predictable and lower than his current stack
- Privacy guarantee: files auto-deleted in 2 hours — no lingering footage concerns

### Activation Path
Arjun discovers Module31 via a Reddit post in r/NewTubers. He uses the guest trial to compress a video without signing up. Impressed by the speed, he signs up and gets 20 free credits. He processes a Reels crop and a subtitle job. He buys his first credit bundle when his 20 credits run out.

---

## Persona 2: The Agency Marketing Manager

**Name:** Priya, 34  
**Role:** Social Media Manager at a mid-sized digital agency  
**Location:** Mumbai, India  
**Team:** 4 people handling 6 client accounts

### Background
Priya manages video content for 6 clients, ranging from a FMCG brand to a fintech startup. Her team processes 10–15 client videos per week — reformatting for different platforms, adding branded captions, normalizing audio levels for consistency. Client footage often contains product demos, executive interviews, and pre-release campaign materials.

### Current Stack
- Adobe Express: $10/mo (one team license)
- CloudConvert: $14/mo (for format conversion)
- Rev.com: $1.25/min for captions (expensive for volume)
- WeTransfer (file sharing between team and clients)

**Monthly spend: ~$50–80/month + Rev.com variable cost**

### Jobs To Be Done
- Process client videos without exposing them to third-party servers with unclear retention
- Add captions to multiple videos quickly (not manually, not at $1.25/min)
- Normalize audio across a batch of videos for consistent brand output
- Convert formats for different client delivery requirements (MP4, MOV, WebM)

### Frustrations
- Adobe Express stores everything in Creative Cloud — clients have asked about this
- Rev.com is expensive for the volume the team produces
- CloudConvert stores files for 24 hours — fine for most cases but a concern for embargoed content
- The team needs a tool they can all use without sharing login credentials

### How Module31 Serves Priya
- Hardcoded Subtitles: accurate, fast, no $1.25/min rate
- Audio Normalizer: consistent loudness across all client videos
- Format Converter: MP4/MOV/WebM as needed
- Privacy-first: 2-hour auto-delete is a genuine enterprise-adjacent selling point
- Credit model: the agency buys a shared credit bundle for the team. Usage is logged per operation.
- Admin dashboard (if Priya is the account admin): can see usage logs, manage credits

### Activation Path
Priya finds Module31 via a search for "video tool with file privacy" or "caption tool no cloud storage." The privacy positioning in the homepage copy resonates immediately. She signs up, processes a caption job to test quality, and pitches it to her team as a replacement for Rev.com's basic captioning use case.

---

## Persona 3: The Indie Developer / Technical Founder

**Name:** Vikram, 29  
**Role:** Solo founder building a podcast platform  
**Location:** Hyderabad, India  
**Background:** Software engineer, 5 years at a product company, now building independently

### Background
Vikram is building a podcast platform that automatically generates video clips from podcast audio — a short-form video summary of each episode for social promotion. He needs to: extract audio, normalize it, generate a waveform visualization video, add subtitles, and crop for different platforms. He's currently doing this manually for each episode.

### Current Stack
- ffmpeg CLI (self-managed) — powerful but fragile, environment-specific
- Custom Python script for waveform generation — takes 20 minutes to render
- Manual subtitle generation using Whisper CLI — works but requires local GPU
- No thumbnail automation — creates each manually in Figma

**Monthly spend: $0 in SaaS, but ~6 hours/episode in manual work**

### Jobs To Be Done
- Automate the clip-to-social-video pipeline without maintaining FFmpeg infrastructure
- Generate waveform visualization videos without a local GPU
- Auto-generate subtitles without running Whisper locally
- Eventually: call these operations via API for his platform's backend

### Frustrations
- Local FFmpeg setup breaks across environments (M1 Mac vs Linux CI/CD)
- Whisper requires CUDA — his dev machine doesn't have a GPU
- Each manual step takes time away from actual product development
- Wants to eventually offer this as a feature to his podcast platform's users — needs an API layer

### How Module31 Serves Vikram
- Audio Waveform tool: generates a waveform video from audio in the browser — no GPU needed
- Hardcoded Subtitles: runs Whisper on Modal GPU — accurate, fast, no local dependency
- Extract Audio, Normalize, Social Crop: replaces his manual ffmpeg pipeline
- V3 API (planned): Vikram is explicitly the target user for the developer API — programmatic access to all tools

### Activation Path
Vikram finds Module31 via a Product Hunt listing or an Indie Hackers post. He immediately recognizes the value of cloud-side FFmpeg + Whisper — it solves his infrastructure problem. He uses it for his own workflow first, then tracks whether Module31 ships a developer API (which would let him integrate it into his platform).

---

## Persona Comparison Summary

| Attribute | Arjun (Creator) | Priya (Agency PM) | Vikram (Developer) |
|---|---|---|---|
| **Primary need** | Speed + AI translation | Privacy + captioning | Infrastructure replacement |
| **Volume** | High (15+ files/week) | Medium (10–15 files/week) | Low but automated (per episode) |
| **Price sensitivity** | Medium | Low (business expense) | Low (time > money) |
| **AI tool interest** | High (Translator) | Medium (Subtitles) | High (API access) |
| **Privacy concern** | Low | High | Medium |
| **V3 API interest** | Low | Low | Very High |
| **Activation trigger** | Tool quality | Privacy positioning | Infrastructure convenience |
