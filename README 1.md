# 🤖 Automated Multi-Channel Content System | n8n + AI + APIs

An automated content publishing system built on self-hosted n8n that reads RSS feeds, generates platform-specific posts using AI, and publishes to 5 social media platforms daily — without manual intervention.

## 📌 What This Does

- Pulls fresh articles from **16 RSS sources** (TechCrunch, Wired, HubSpot, VentureBeat, etc.)
- Filters out duplicates and articles older than 7 days
- Generates **platform-specific copy** via Claude (Anthropic) with custom prompts per channel
- Fetches a relevant image from Pexels API
- Publishes to **Facebook, Instagram, Threads, Telegram, and LinkedIn** — fully automated

## 🏗️ Architecture

```
Schedule Trigger (daily 09:06)
    → Code JS (RSS fetch)
    → RSS Read (16 sources)
    → Filter (keywords)
    → Code JS2 (dedup + date filter 7 days)
    → Sort → Limit
    → Google Sheets (check published history)
    → Append to Sheet
    → Basic LLM Chain (Claude — generates FB/IG/Threads/TG copy in Ukrainian)
    → Code JS1 (JSON cleanup)
    → Basic LLM Chain Pexels (generates image keyword)
    → HTTP Request Pexels (fetch image)
    → Switch (routes by platform)
        ├── Facebook Graph API
        ├── Instagram Graph API
        ├── Threads API
        └── Telegram Bot API
```

## 🧠 AI Prompts Logic

Each platform gets a tailored prompt with strict formatting rules:

| Platform | Language | Length | Style |
|----------|----------|--------|-------|
| Facebook | Ukrainian | 400–800 chars | Hook + 2-3 insights + question |
| Instagram | Ukrainian | 650–800 chars | Summary + CTA + 10-15 hashtags |
| Threads | Ukrainian | 200–400 chars | Provocative fact + practitioner opinion |
| LinkedIn | English | 800–1200 chars | Hook 140 chars + 3-5 deep insights |

**Formatting rules enforced in every prompt:**
- Single `\n` between paragraphs (no empty lines)
- Max 1 emoji (hook only)
- Arrows `→` instead of bullet points
- No em dash `—`
- No brand names
- No URLs

## 🔧 Tech Stack

| Layer | Tool |
|-------|------|
| Automation engine | n8n (self-hosted, Docker) |
| Server | Hetzner VPS |
| DNS / CDN | Cloudflare |
| AI model | Anthropic Claude + Groq |
| Images | Pexels API |
| Tracking | Google Sheets |
| Publishing | Facebook Graph API, Instagram Graph API, Threads API, Telegram Bot API, LinkedIn API |

## 📁 Files

```
├── Posting_Automation_CLEAN.json   # n8n workflow export (credentials removed)
└── README.md
```

## 🚀 How to Import

1. In n8n: **Workflows → Import from file**
2. Upload `Posting_Automation_CLEAN.json`
3. Replace all `YOUR_*_HERE` placeholders with your credentials:
   - `YOUR_FACEBOOK_PAGE_TOKEN_HERE` — Facebook Page Access Token
   - `YOUR_FACEBOOK_ACCESS_TOKEN_HERE` — Facebook User Access Token
   - `YOUR_THREADS_ACCESS_TOKEN_HERE` — Threads Access Token
   - `YOUR_GOOGLE_SHEETS_ID_HERE` — Google Sheets document ID
   - Connect credentials: Anthropic API, Groq API, Telegram Bot, LinkedIn OAuth2
4. Update the Google Sheets node with your sheet name
5. Set your schedule trigger time

## 💡 Key Technical Highlights

**Duplicate detection (JavaScript node):**
```javascript
const sheetTitles = $('Get row(s) in sheet').all()
  .map(item => item.json.Title ? item.json.Title.substring(0, 30).toLowerCase().trim() : '');
const sevenDaysAgo = Date.now() - (7 * 24 * 60 * 60 * 1000);
return $input.all().filter(item => {
  const title = item.json.title ? item.json.title.substring(0, 30).toLowerCase().trim() : '';
  const isDuplicate = sheetTitles.some(t => t && title && t === title);
  const pubDate = item.json.pubDate || item.json.isoDate || null;
  const isRecent = pubDate ? new Date(pubDate).getTime() > sevenDaysAgo : true;
  return !isDuplicate && isRecent;
});
```

**Random image selection from Pexels (Edit Fields node):**
```javascript
{{ $input.item.json.photos[Math.floor(Math.random() * Math.min($input.item.json.photos.length, 10))].src.large }}
```

## 👤 Author

**Eugene Bondarenko** — Marketing Automation Architect  
Specializing in n8n infrastructure, CRM sync, and AI-driven workflows.

- 🔗 [Upwork Profile](https://www.upwork.com/freelancers/~0108dfb61b408b16b2)
- 💼 [LinkedIn](https://linkedin.com/in/eugene-bondarenko-887351370)
