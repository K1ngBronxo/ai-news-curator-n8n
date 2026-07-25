# 📜 Changelog — V1 → V3

## V3 — Intelligence + Research Fallback + Multi-Platform

A ground-up expansion of the original pipeline: **18 nodes → 64 nodes**.

### 🧠 Editorial intelligence
- Added a full **Smart Filter Score & Rank** engine: weighted source authority + 5-domain taxonomy matching + clickbait/sponsored-content suppression, replacing V1's flat NLP taxonomy filter.
- Added an **AI Story Worthiness Gate** — a Gemini call acting as a senior editor, returning a structured verdict (`worthPosting`, `confidence`, `clickbaitRisk`, `factCheck`) before anything is allowed to publish.
- Added a **Fallback Router**: if the AI editor rejects a story, V3 automatically retries the *next*-ranked candidate instead of giving up or posting the rejected story anyway.

### 🔬 Research fallback (new in V3)
- If every RSS candidate is rejected, V3 pivots to **live research** via the Hacker News Algolia search API.
- A dedicated **AI Research Curator** picks the best live story from that corpus.
- Failed research runs are logged (`V3: Log Research Failed`) rather than silently failing.

### 📈 Sources
- Expanded from **3 RSS feeds** (HackerNews, TechCrunch AI, CyberSecurity News) to **9**: HackerNews, TechCrunch AI, The Hacker News, MIT Technology Review, Ars Technica, VentureBeat AI, The Verge, OpenAI Blog, Wired.

### ✨ Content enrichment (new in V3)
- **Content Enrichment Engine**: sentiment analysis, audience segmentation, named entity extraction, credibility scoring, key quote pulling, and **A/B headline generation** with an AI-selected winner.
- A parallel **Research Enrichment Engine** applies the same treatment to stories sourced from the research fallback path.
- **Breaking Priority Flag**: stories flagged as breaking news get an `instant_priority` publish mode instead of the default scheduled flow.

### 🖼️ Image pipeline
- V1 posted whatever image (if any) came through in the RSS payload.
- V3 adds a fallback chain: RSS-provided image → **Open Graph / Twitter meta tag scraping** from the source article → merged image pipeline that picks whichever source actually resolved, with a clean skip path when the RSS image was already good.

### ♻️ Repurposing (new in V3)
- A **Repurpose Engine** takes the final published Telegram copy and regenerates it into a 3-tweet X thread, a Discord embed, a 2-sentence newsletter blurb, and a poll question — all from a single structured Gemini call.

### 📊 Analytics (new in V3)
- **Analytics Collector** node tracks run-level metrics in workflow static data for later inspection.

### 📤 Publishing
- V1: Telegram → LinkedIn (two platforms).
- V3: Telegram → LinkedIn → **Discord** (three platforms), plus the repurposed thread/newsletter/poll assets ready for manual or future automated distribution.

### 🧩 Shared foundation (kept from V1)
- Core loop: RSS → filter → AI writer → Telegram → 10s delay → scrape published message → AI rewrite → LinkedIn.
- Dual-bot Telegram pattern (publisher bot + listener bot) to catch the live channel message and repurpose it.
- Gemini as the sole LLM provider throughout.

---

## V1 — `TELE-LINKEDIN` (baseline)

The original 5-phase build:

1. **Phase 1** — NLP 5-domain taxonomy filter on 3 RSS feeds
2. **Phase 2** — Gemini writes the Telegram post
3. **Phase 3** — Payload/image formatting
4. **Phase 4** — Broadcast to Telegram channel
5. **Phase 5** — Secondary bot listens for the published message, scrapes it, rewrites it for LinkedIn via a second Gemini call, appends a brand signature/watermark, and publishes

No AI worthiness gating, no fallback logic, no enrichment, no repurposing — every filtered article that made it through the taxonomy filter got posted.
