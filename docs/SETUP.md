# 🔧 Setup Guide

This guide walks through getting **AI News Curator V3** running end-to-end in n8n.

## 1. Prerequisites

- An n8n instance (self-hosted, Docker, or n8n Cloud) — **v1.4x+** recommended for `@n8n/n8n-nodes-langchain` support
- A Google AI Studio account for Gemini API access
- Two Telegram bots (one publishes news, one listens for the LinkedIn repurpose trigger)
- A LinkedIn Developer App with OAuth2 access to the Community Management / Share API
- (Optional) A Discord webhook URL if you want cross-posting

---

## 2. Import the workflow

1. In n8n, go to **Workflows → Import from File**.
2. Select `workflows/ai-news-curator-v3.json`.
3. The workflow will appear with all 64 nodes, unconnected to any credentials yet.

---

## 3. Set up credentials

### Google Gemini (PaLM API)
The workflow calls Gemini from **6 separate nodes** (AI gate, research curator, content enrichment, research enrichment, Telegram writer, LinkedIn writer, repurpose engine). You can:
- Use a **single** Gemini credential for all of them, or
- Spread them across multiple API keys/accounts (as the original build does) to stay comfortably under per-key rate limits during high-frequency runs.

In n8n: **Credentials → New → Google Gemini(PaLM) Api** → paste your API key from [Google AI Studio](https://aistudio.google.com/app/apikey).

Then open each `Gemini Model (...)` node in the workflow and select your credential from the dropdown.

### Telegram
You need **two bots**:

| Bot | Purpose |
|---|---|
| **Publisher bot** | Posts the news card (photo + caption) to your channel |
| **Listener bot** | Watches the channel for the just-published message so it can scrape it and repurpose it for LinkedIn |

Create both via [@BotFather](https://t.me/BotFather), grab each token, and add them as **Telegram API** credentials in n8n. Assign them to:
- `V3: Telegram Send Photo + Caption` / `V3: Telegram Send Text Fallback` → publisher bot
- `V3: LinkedIn Bot Listener` / `V3: Download Telegram Photo` → listener bot

Add your bot(s) as **admin** of the target channel so they can post and read messages.

### LinkedIn
1. Create an app at [LinkedIn Developers](https://www.linkedin.com/developers/apps).
2. Request the `w_member_social` scope (and `r_liteprofile`/`profile` depending on your API version) so the app can post on your behalf.
3. In n8n: **Credentials → New → LinkedIn OAuth2 API**, complete the OAuth flow.
4. Assign this credential to `V3: LinkedIn Post with Image` and `V3: LinkedIn Post Text Only`.

### Discord (optional)
Create a webhook in your target channel's **Integrations** settings and copy the URL — no OAuth needed.

---

## 4. Replace placeholders

Search the imported workflow for these placeholder strings and swap in your own values:

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_TELEGRAM_CHANNEL` | Telegram send nodes | `@your_channel_handle` or numeric chat ID |
| `YOUR_LINKEDIN_PERSON_URN` | `V3: LinkedIn Post with Image` / `V3: LinkedIn Post Text Only` | Your LinkedIn person URN, e.g. `AbCdEfGhIj` (find it via the LinkedIn API `/me` endpoint or your OAuth token response) |
| `YOUR_DISCORD_WEBHOOK_URL` | `V3: Discord Webhook (Optional)` | Your Discord channel webhook URL |

> 💡 Tip: use n8n's **Find/Replace across nodes** (or edit the JSON directly before importing) if you're setting this up for multiple channels/brands.

---

## 5. Test before activating

1. Run `Schedule Trigger` manually (▶️ on the node) to pull a full cycle end-to-end.
2. Check the **Smart Filter Score & Rank** node output — confirm articles are being scored sensibly for your niche.
3. Watch the **AI Story Worthiness Gate** output — tune the prompt if it's too strict/lenient for your audience.
4. Confirm a test post lands correctly in Telegram, then confirm the LinkedIn listener picks it up and posts within ~10 seconds.

---

## 6. Activate

Flip the workflow to **Active**. It will now run automatically every 3 hours via the Schedule Trigger — no further intervention needed.

---

## 🩹 Troubleshooting

| Symptom | Likely cause |
|---|---|
| No candidates ever pass the AI gate | Prompt is too strict for your source mix, or `SOURCE_WEIGHTS`/taxonomy in the Smart Filter code node doesn't match your topic focus — adjust the JS at the top of `V3: Smart Filter Score & Rank` |
| Research fallback triggers every run | Your RSS feeds may be stale/duplicated across cycles — check `rejectedTitles` dedup logic isn't over-rejecting |
| LinkedIn post never fires | Confirm the listener bot is admin on the channel and the `10s Post-Publish Delay` isn't too short for your channel's message propagation |
| Image missing on LinkedIn | The OG/Twitter meta fallback may be blocked by the source site's bot protection — check `V3: Fetch Article Page (OG Fallback)` response |
| Gemini rate limit errors | Split Gemini nodes across multiple API keys/credentials, as noted above |
