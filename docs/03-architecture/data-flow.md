# Data Flow — Module31

This document describes the detailed request lifecycle for each major flow in the platform.

---

## Flow 1: Guest Trial — Standard Tool

```
┌─────────┐         ┌──────────┐         ┌──────────────┐         ┌──────────┐
│ Browser │         │ Next.js  │         │  Supabase    │         │  FFmpeg  │
│  Guest  │         │ API Route│         │  (DB+Storage)│         │  (local) │
└────┬────┘         └────┬─────┘         └──────┬───────┘         └────┬─────┘
     │                   │                      │                       │
     │  POST /api/tool   │                      │                       │
     │  multipart form   │                      │                       │
     │──────────────────►│                      │                       │
     │                   │                      │                       │
     │                   │ Read X-Guest-Mode     │                       │
     │                   │ header = "true"       │                       │
     │                   │                      │                       │
     │                   │ Read system_config    │                       │
     │                   │ guest_watermark_*     │                       │
     │                   │──────────────────────►│                       │
     │                   │◄──────────────────────│                       │
     │                   │                      │                       │
     │                   │ Write temp input file │                       │
     │                   │──────────────────────────────────────────────►│
     │                   │                      │                       │
     │                   │ Run FFmpeg command    │                       │
     │                   │──────────────────────────────────────────────►│
     │                   │◄──────────────────────────────────────────────│
     │                   │                      │                       │
     │                   │ Apply watermark       │                       │
     │                   │ (drawtext filter)     │                       │
     │                   │──────────────────────────────────────────────►│
     │                   │◄──────────────────────────────────────────────│
     │                   │                      │                       │
     │                   │ Upload to Storage     │                       │
     │                   │──────────────────────►│                       │
     │                   │◄──────────────────────│                       │
     │                   │                      │                       │
     │  { url, watermarked: true }               │                       │
     │◄──────────────────│                      │                       │
     │                   │                      │                       │
     │ Increment attempt │                       │                       │
     │ count in localStorage                     │                       │
     │                   │                      │                       │
```

---

## Flow 2: Authenticated Standard Tool

```
┌──────────┐      ┌──────────┐      ┌──────────────┐      ┌──────────┐
│ Browser  │      │ Next.js  │      │  Supabase    │      │  FFmpeg  │
│ Auth User│      │ API Route│      │  (DB+Storage)│      │  (local) │
└────┬─────┘      └────┬─────┘      └──────┬───────┘      └────┬─────┘
     │                 │                   │                    │
     │ POST /api/tool  │                   │                    │
     │ + Cookie header │                   │                    │
     │────────────────►│                   │                    │
     │                 │                   │                    │
     │                 │ Validate JWT      │                    │
     │                 │ (Supabase SSR)    │                    │
     │                 │──────────────────►│                    │
     │                 │◄──────────────────│                    │
     │                 │                   │                    │
     │                 │ Read credit balance│                   │
     │                 │──────────────────►│                    │
     │                 │◄──────────────────│                    │
     │                 │                   │                    │
     │                 │ [if credits < cost]│                   │
     │  { error: "Insufficient credits" }  │                    │
     │◄────────────────│                   │                    │
     │                 │                   │                    │
     │                 │ Deduct credits    │                    │
     │                 │ (atomic UPDATE)   │                    │
     │                 │──────────────────►│                    │
     │                 │                   │                    │
     │                 │ Log usage         │                    │
     │                 │ (INSERT usage_logs)│                   │
     │                 │──────────────────►│                    │
     │                 │◄──────────────────│                    │
     │                 │                   │                    │
     │                 │ Run FFmpeg        │                    │
     │                 │──────────────────────────────────────►│
     │                 │◄──────────────────────────────────────│
     │                 │                   │                    │
     │                 │ Upload output     │                    │
     │                 │──────────────────►│                    │
     │                 │◄──────────────────│                    │
     │                 │                   │                    │
     │  { url }        │                   │                    │
     │◄────────────────│                   │                    │
```

---

## Flow 3: AI Tool (GPU-Powered)

```
┌──────────┐   ┌──────────┐   ┌──────────────┐   ┌────────────┐   ┌──────────┐
│ Browser  │   │ Next.js  │   │  Supabase    │   │  Modal.com │   │ AI APIs  │
│ Auth User│   │ API Route│   │  (DB+Storage)│   │ GPU Worker │   │ (OpenAI) │
└────┬─────┘   └────┬─────┘   └──────┬───────┘   └─────┬──────┘   └────┬─────┘
     │              │                │                  │                │
     │ POST /api/   │                │                  │                │
     │ ai-tool      │                │                  │                │
     │─────────────►│                │                  │                │
     │              │ Validate JWT   │                  │                │
     │              │───────────────►│                  │                │
     │              │◄───────────────│                  │                │
     │              │                │                  │                │
     │              │ Check credits  │                  │                │
     │              │───────────────►│                  │                │
     │              │◄───────────────│                  │                │
     │              │                │                  │                │
     │              │ DEDUCT CREDITS │                  │                │
     │              │ (before GPU!)  │                  │                │
     │              │───────────────►│                  │                │
     │              │ Log usage      │                  │                │
     │              │───────────────►│                  │                │
     │              │◄───────────────│                  │                │
     │              │                │                  │                │
     │              │ [For Ad/Motion]│                  │                │
     │              │ Call AI Director─────────────────────────────────►│
     │              │◄─────────────────────────────────────────────────│
     │              │ Scene graph JSON                  │                │
     │              │                │                  │                │
     │              │ Upload input   │                  │                │
     │              │ to Supabase    │                  │                │
     │              │───────────────►│                  │                │
     │              │◄───────────────│                  │                │
     │              │                │                  │                │
     │              │ Shell: python -m modal run worker │                │
     │              │──────────────────────────────────►│                │
     │              │                │                  │                │
     │              │                │                  │ Download input │
     │              │                │◄─────────────────│                │
     │              │                │                  │                │
     │              │                │                  │ Process (GPU)  │
     │              │                │                  │ (30–180 sec)   │
     │              │                │                  │                │
     │              │                │                  │ Upload output  │
     │              │                │──────────────────►│               │
     │              │                │◄─────────────────│                │
     │              │                │                  │                │
     │              │ Return output URL                 │                │
     │              │◄──────────────────────────────────│                │
     │              │                │                  │                │
     │  { url }     │                │                  │                │
     │◄─────────────│                │                  │                │
     │              │                │                  │                │
     │ [On error]   │                │                  │                │
     │              │ REFUND CREDITS │                  │                │
     │              │───────────────►│                  │                │
     │◄─────────────│                │                  │                │
     │  { error, credits_refunded }  │                  │                │
```

---

## Flow 4: Admin Config Change

```
┌──────────┐      ┌──────────┐      ┌──────────────┐
│ Admin UI │      │ Next.js  │      │  Supabase    │
│ Browser  │      │ API Route│      │  (DB)        │
└────┬─────┘      └────┬─────┘      └──────┬───────┘
     │                 │                   │
     │ POST /api/admin/config               │
     │ { key: "translator_cost", value: 30}│
     │────────────────►│                   │
     │                 │ Validate JWT      │
     │                 │ Verify role=admin  │
     │                 │──────────────────►│
     │                 │◄──────────────────│
     │                 │                   │
     │                 │ UPSERT system_config
     │                 │──────────────────►│
     │                 │◄──────────────────│
     │                 │                   │
     │  { success }    │                   │
     │◄────────────────│                   │
     │                 │                   │

[Next tool use by any user]
     │                 │                   │
     │ POST /api/translator               │
     │────────────────►│                   │
     │                 │ Read translator_cost from system_config
     │                 │──────────────────►│
     │                 │◄──────────────────│
     │                 │ cost = 30 (updated, no redeploy)
```

---

## File Lifecycle

```
[Upload]
  User uploads file via multipart form
    ↓
  API route writes to: /tmp/<uuid>-<timestamp>-input.<ext>
    ↓
[Processing]
  FFmpeg reads input → writes to: /tmp/<uuid>-<timestamp>-output.<ext>
    OR
  Modal worker downloads from Supabase temp-uploads bucket
    ↓
[Storage]
  API route uploads output to Supabase output-videos bucket
  Returns time-limited signed URL (e.g., expires in 2 hours)
    ↓
[User Downloads]
  User clicks download link → fetches file from Supabase Storage URL
    ↓
[Cleanup]
  Scheduled cleanup job runs every N hours (configurable via video_retention_hours)
  Deletes all files from output-videos older than retention window
  Deletes /tmp files older than retention window
```

**Privacy guarantee:** After the retention window, the output file is deleted from Supabase Storage. The signed URL expires. The file is unrecoverable. No copy is retained by Module31.

---

## Auth State Flow

```
[Page Load]
  Supabase SSR client reads cookie
    ↓
  If valid JWT → user is authenticated
    Show credit balance, user menu, authenticated tool access
    ↓
  If no cookie / expired JWT → user is guest
    Show guest trial banner
    Track attempt count via X-Guest-Mode header + localStorage
    ↓
[After 5 guest uses]
  SignupGateModal shown
  User cannot proceed without signing up
    ↓
[Signup / Login]
  Supabase OAuth flow (Google / GitHub / email)
  On success: profile created with 20 free credits
  Cookie set → all subsequent requests are authenticated
    ↓
[Session Expiry]
  Supabase refresh token automatically rotates JWT
  If refresh fails → user redirected to login
```
