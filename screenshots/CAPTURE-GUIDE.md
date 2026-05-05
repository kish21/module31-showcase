# Screenshot Capture Guide — Module31

This guide tells you exactly which pages to screenshot, what state the UI should be in, and what to name each file. Screenshots go in the `screenshots/` folder of this repo.

---

## Setup Before Capturing

1. Run the app locally (`npm run dev`) or open the live hosted version
2. Use a browser window at **1440 × 900px** (standard laptop resolution) for consistency
3. Use Chrome DevTools Device Toolbar if needed to set exact dimensions
4. Use a dark-mode browser theme — Module31's dark-mode UI looks best for screenshots
5. Zoom browser to **100%** — no zoom in or out

---

## Screenshot List

### Homepage

| Filename | URL | What to Show |
|---|---|---|
| `01-homepage-grid.png` | `/` | Full tool grid visible — all card rows showing. Make sure the category tabs are visible at the top. Logged-out state (no credit balance in header). |
| `02-homepage-search.png` | `/` | Type "compress" in the search bar — show filtered results with only matching tools visible. |
| `03-homepage-categories.png` | `/` | Click the "AI" category tab — show only AI tools visible in the grid. |
| `04-homepage-loggedin.png` | `/` | Logged-in state — credit balance visible in header nav. |

---

### Standard Tool Pages

| Filename | URL | What to Show |
|---|---|---|
| `05-compress-tool.png` | `/compress` | File selected and ready to process — show the filename displayed, compression settings visible, the "Compress Video" button active. |
| `06-compress-result.png` | `/compress` | After processing — show the output file size, the "Download" button, and the original vs compressed size comparison. |
| `07-subtitles-tool.png` | `/subtitles` | Upload state with a video file selected — subtitle options visible. |
| `08-social-crop.png` | `/crop` | 9:16 crop option selected — show the aspect ratio selector UI. |

---

### AI Tool Pages

| Filename | URL | What to Show |
|---|---|---|
| `09-translator-tool.png` | `/translator` | Language selector visible — target language dropdown open showing multiple options. File selected. |
| `10-translator-result.png` | `/translator` | After processing — translated/dubbed video ready for download. |
| `11-vision-agent.png` | `/vision` | Analysis output shown — the JSON/text analysis result visible below the video. |
| `12-ad-generator.png` | `/ad-generator` | Prompt input filled with a sample prompt — animation style options visible. |
| `13-ad-generator-result.png` | `/ad-generator` | Generated ad video result — download button visible with the output preview. |
| `14-motion-artist.png` | `/motion-artist` | Prompt input + style selector visible — before processing. |

---

### Admin Pages

| Filename | URL | What to Show |
|---|---|---|
| `15-admin-dashboard.png` | `/admin` | User list with credit balances — at least 2–3 rows visible. Credit refill button visible. |
| `16-admin-config.png` | `/admin/config` | System config panel — showing key/value pairs like translator_cost, guest_watermark_enabled, video_retention_hours. |

---

### Auth & Guest Experience

| Filename | URL | What to Show |
|---|---|---|
| `17-guest-trial-banner.png` | any tool page | Guest user state — the watermark notice banner visible at the top of the tool page. |
| `18-signup-gate.png` | any tool (after 5 uses) | The SignupGateModal open — showing the "Sign up to continue" message and trial limit reached. |
| `19-login-modal.png` | any page | Login modal open — showing OAuth options (Google, GitHub, email). |
| `20-credit-display.png` | `/` (logged in) | Close-up of the header — credit balance prominently displayed next to user avatar/menu. |

---

## Capture Tips

**For result screenshots:** Use small test files to get fast processing — a 5-second MP4 is enough to demonstrate the output. You don't need real content; placeholder files work fine.

**For admin screenshots:** If your account is an admin, navigate directly. If not, temporarily set your role in Supabase dashboard.

**For AI tool screenshots:** Use a short file (< 30 seconds) to get the result quickly for the screenshot. The result quality is what matters, not the content.

**Cropping:** Crop out browser chrome (address bar, tabs) unless it adds context. The focus should be on the UI.

**Redacting:** If any screenshots show real user emails or personal data in the admin dashboard, blur or replace them before adding to this repo.

---

## After Capturing

1. Drop all screenshots into the `screenshots/` folder in this repo
2. Name them exactly as listed in the table above
3. The README already references these filenames — the screenshot table will render automatically once the files are present
4. Commit and push to the `module31-showcase` repo

---

## Priority Order (if you don't have time for all 20)

Capture these 8 first — they cover the most important showcase moments:

1. `01-homepage-grid.png` — The tool grid (most impactful first impression)
2. `04-homepage-loggedin.png` — Shows credit system in action
3. `05-compress-tool.png` — Core tool experience
4. `09-translator-tool.png` — AI tool (highest differentiation)
5. `12-ad-generator.png` — Most unique feature
6. `15-admin-dashboard.png` — Shows operational thinking
7. `16-admin-config.png` — Shows admin-driven config (a key architectural decision)
8. `17-guest-trial-banner.png` — Shows the guest funnel
