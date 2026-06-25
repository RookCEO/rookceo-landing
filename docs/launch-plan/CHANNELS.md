# Rook — Content Distribution Channels

## Strategy
TikTok is the flywheel. Everything else is repurposing. One viral TikTok > 50 Substack posts.
Different platforms reach different buyer profiles — all funnel to rookceo.com waitlist → Stripe.

| Platform | Audience | Content type | Posting method |
|----------|----------|-------------|----------------|
| TikTok (@rookceo) | Young entrepreneurs, side hustlers | Short video, 15-60s | `tiktok-uploader` (Playwright, session ID ✅) |
| YouTube (@rookceo) | Business owners wanting depth | Long-form, "week in review", tutorials | YouTube Data API v3 (OAuth — see below) |
| Substack | Serious operators, newsletters | Written deep-dives, Rook's weekly report | Playwright automation (no public API) |
| Facebook | Older small business owners, groups | Repurposed video + written posts | Facebook Graph API (see below) |

---

## TikTok ✅
Session ID already configured. `tiktok-uploader` skill installed on rook.
Pipeline: `video-script` → `voiceover` (Orpheus :8083) → `video-assemble` (SadTalker + FFmpeg) → upload.

---

## YouTube — OAuth Setup

YouTube uploads require OAuth 2.0 (not just the Data API key). One-time setup:

### Step 1: Google Cloud Console ✅ (already done)
- OAuth app name: **rook**
- `YOUTUBE_CLIENT_ID` and `YOUTUBE_CLIENT_SECRET` already in `~/.openclaw/.env` on rook

### Step 2: Enable the right scopes
Make sure YouTube Data API v3 is enabled and OAuth consent screen has scope:
- `https://www.googleapis.com/auth/youtube.upload`
- `https://www.googleapis.com/auth/youtube`

### Step 3: First-time auth (one-time, run on Mac — needs browser)
```bash
# Install google auth library if needed
pip install google-auth-oauthlib google-api-python-client

# Run the auth flow - opens browser, log in as the @rookceo Google account
python3 -c "
from google_auth_oauthlib.flow import InstalledAppFlow
import os, json
client_config = {
    'installed': {
        'client_id': os.environ['YOUTUBE_CLIENT_ID'],
        'client_secret': os.environ['YOUTUBE_CLIENT_SECRET'],
        'redirect_uris': ['urn:ietf:wg:oauth:2.0:oob'],
        'auth_uri': 'https://accounts.google.com/o/oauth2/auth',
        'token_uri': 'https://oauth2.googleapis.com/token'
    }
}
flow = InstalledAppFlow.from_client_config(
    client_config,
    ['https://www.googleapis.com/auth/youtube.upload']
)
creds = flow.run_local_server(port=0)
with open('youtube_token.json', 'w') as f:
    f.write(creds.to_json())
print('Done — token saved to youtube_token.json')
"
```
Save `youtube_token.json` to `~/.openclaw/youtube_token.json` on rook.
The refresh token inside is long-lived — no need to re-auth unless revoked.

### Step 4: Add to .env on rook
```
YOUTUBE_TOKEN_PATH=/home/claw/.openclaw/youtube_token.json
```
(YOUTUBE_CLIENT_ID and YOUTUBE_CLIENT_SECRET already set ✅)

---

## Facebook — Graph API Token Setup

For posting to a Facebook Page (create a RookCEO page if not done):

### Step 1: Create Facebook Developer App
1. Go to developers.facebook.com → **My Apps → Create App**
2. Type: **Business**
3. Name: `RookCEO`
4. Add products: **Facebook Login** + **Pages API**

### Step 2: Get a Page Access Token
1. Graph API Explorer: developers.facebook.com/tools/explorer
2. Select your app → select your RookCEO Page from the dropdown
3. Add permissions: `pages_manage_posts`, `pages_read_engagement`, `publish_to_groups` (if posting to groups)
4. Click **Generate Access Token** → log in as your Facebook account
5. This gives you a **short-lived token (1 hour)**

### Step 3: Exchange for Long-Lived Token (60 days)
```bash
# Replace values with your actual app ID, secret, and short-lived token
curl "https://graph.facebook.com/oauth/access_token\
?grant_type=fb_exchange_token\
&client_id=YOUR_APP_ID\
&client_secret=YOUR_APP_SECRET\
&fb_exchange_token=SHORT_LIVED_TOKEN"
```
Returns a token valid for ~60 days.

### Step 4: Get a Never-Expiring Page Token
```bash
# Get your user ID first
curl "https://graph.facebook.com/me?access_token=LONG_LIVED_USER_TOKEN"

# Then get the page's permanent token
curl "https://graph.facebook.com/YOUR_USER_ID/accounts?access_token=LONG_LIVED_USER_TOKEN"
```
The `access_token` field in the page object is a **permanent Page Access Token** — it doesn't expire unless permissions are revoked.

### Step 5: Add to .env on rook
```
FACEBOOK_PAGES_ID=your_rookceo_page_id
FACEBOOK_PAGES_ACCESS_TOKEN=your_permanent_page_access_token
```

---

## Content Repurposing Flow

```
1. TikTok video produced (Hook/Body/CTA, 30-60s)
   ↓
2. Upload to TikTok via tiktok-uploader
3. Upload same video to YouTube Shorts (same file, no re-edit needed)
4. Extract script → Facebook post (text + video link)
5. Combine 4 TikToks → Substack "Rook's week" digest (written)
```

One piece of content, four channels. Rook runs the posting loop autonomously once the pipeline skills are built.

---

## Content Pillars (for all channels)

1. **"Rook controls the board"** — decisions Rook made, what happened
2. **"Free public audit"** — Rook reviews a commenter's business live
3. **"Rook's money scheme"** — Rook identifies a revenue opportunity
4. **"What Rook knows about your business"** — educational, positions as expert
5. **"Week in review"** — what Rook did autonomously this week
6. **Competitor contrast** — "Heyron gives you a pet. Rook gives you a COO."
