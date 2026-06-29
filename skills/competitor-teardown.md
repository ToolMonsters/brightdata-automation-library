---
name: Competitor Teardown (Bright Data)
description: Takes one competitor (a domain or a name) and returns a sharp, opinionated battlecard built entirely from Bright Data's structured endpoints — no other tools. It reads what the competitor is BUILDING from their open job listings (their roadmap, leaked through hiring), what customers actually HATE from their reviews (your wedge), how they POSITION today from their recent LinkedIn posts and site, and where they STAND from Crunchbase + headcount trajectory + news. Output is a battlecard covering Snapshot, Where they're investing, Current messaging, The cracks (your opening), and Win themes. Fully self-contained — the battlecard IS the deliverable, nothing gets enriched or pushed. Triggers on "tear down [competitor]", "battlecard for [company]", "what is [competitor] building", "how do I beat [company]", "competitor teardown", "what do [company]'s customers hate", or "/teardown".
---

# Competitor Teardown (Bright Data)

A battlecard generator that reads a competitor at Bright-Data depth and tells you three things sales/marketing pay analysts to find: **what they're building, what their customers hate, and how they sell themselves right now.** All three come from public data the competitor can't hide — they just never get read together. This skill reads them together, from a single input, using only the Bright Data MCP.

The two non-obvious moves that make this sharp:
- **Open roles = roadmap.** A company's job listings leak its priorities before any announcement. 12 of 18 open reqs on the data platform team → they're rebuilding the core. 6 new enterprise AEs → they're moving upmarket. Read the hiring, predict the next two quarters.
- **Negative reviews = your wedge.** The pattern in 1–3★ reviews is the durable weakness — the thing they *can't* fix fast. That's where you position.

## Required MCP

- **Bright Data MCP** — the only dependency. Everything below is a `web_data_*`, `discover`, `search_engine`, or `scrape_as_markdown` call. If Bright Data is missing, stop and say so; there is no fallback.

No FullEnrich, no HubSpot, no Slack, no push. The output is a document, full stop.

---

## Input

- **One competitor.** Domain (preferred — unambiguous) or name. If a name resolves to multiple companies, use `search_engine` to disambiguate and confirm with the user before scanning.
- **Optional lens:** *"from the POV of <my product>"*. If given (or stored in memory), the wedge + win themes are framed against the user's offer. If not, the teardown is neutral — still useful, just not pre-aimed.

---

## What it pulls (Bright Data only)

Fire only what the battlecard needs; don't burn every endpoint blindly.

| Section it feeds | Bright Data call(s) |
|---|---|
| Snapshot (size, funding, trajectory) | `web_data_crunchbase_company`, `web_data_linkedin_company_profile` |
| Roadmap (what they're building) | `web_data_linkedin_job_listings` — the core signal; read role areas, counts, and tech named in descriptions |
| Current messaging / positioning | `web_data_linkedin_posts` + `scrape_as_markdown` on their homepage/product pages |
| The cracks (customer pain) | reviews — `web_data_amazon_product_reviews` (ecom) **or** `search_engine` → G2/Capterra/Trustpilot pages → `scrape_as_markdown`; `web_data_google_maps_reviews` for local/physical |
| Triggers / momentum | `web_data_reuter_news` + `search_engine` (recent) |

Run the independent pulls in parallel where the surface allows. Respect cache inside the freshness window — don't re-fetch the same competitor's profile twice in one session.

---

## Flow

### 1. Resolve the competitor
Domain → straight in. Name → `search_engine` to confirm the right entity (watch for same-name collisions across regions/industries). Confirm once if ambiguous.

### 2. Scan
Pull the endpoints above. For the **roadmap** read, this is the work: bucket the open roles by function and area, count them, and extract any tools/tech named in descriptions. For the **cracks** read, pull the actual 1–3★ verbatims, not just the star average — the pattern is the point.

### 3. Synthesize the battlecard
Five sections, in this order. Be opinionated; a battlecard that hedges is useless.

> ## Teardown — <Competitor>
> *built from public data · checked <date>*
>
> **1. Snapshot**
> <Size, HQ, funding (amount + date + investors), headcount trajectory (+/-% if visible), one-line "what they are">. Flag up or down — are they ascending or stalling?
>
> **2. Where they're investing → roadmap**
> Read from <N> open roles. e.g. *"14 of 21 reqs are eng on the ingestion/AI side → they're rebuilding the data layer; expect feature parity on <X> within ~2 quarters. 4 enterprise AE reqs → moving upmarket. Zero on support → the support gap below won't close soon."* This section is the value — be concrete.
>
> **3. How they sell themselves now**
> Current messaging from posts + site. The promise they lead with, the proof they cite, the audience they're targeting today (vs. who they used to target).
>
> **4. The cracks → your opening**
> Top 2–3 recurring complaints from reviews, each with a short real verbatim and a count/share ("~60% of negative reviews hit onboarding"). Separate *fixable* gripes from *structural* ones — the structural ones are the wedge.
>
> **5. Win themes**
> 2–4 ways to position against them, each tied to a specific crack above. If a "my product" lens is set, aim these at it. If not, keep them as raw angles.

### 4. Deliver
Return the battlecard inline (or as a doc if the user wants to keep it). That's the end of the skill — nothing to enrich, nobody to contact. Offer a re-run cadence (see Tracking) if it's a competitor they'll watch.

---

## Freshness

Bright Data's edge is recency, and a battlecard goes stale fast (roadmaps and messaging shift monthly). Default windows:

- Job listings: last **30 days** (the roadmap read must be current)
- Posts / messaging: last **45 days**
- News: last **30 days**
- Funding: last **180 days** (slower-moving, but flag if older)

Stamp the card with *checked <date>*. If a section's data is older than its window, label it *"as of <date> — may have shifted"* rather than presenting it as current.

---

## Memory model

- `Teardown skill: my product = "<one-line offer>"` *(optional — frames the wedge + win themes)*
- `Teardown skill: tracked competitors = "<comp A>, <comp B>, ..."` *(optional — for "re-run my teardowns")*
- `Teardown skill: review sources = "<G2 / Capterra / Trustpilot / Google Maps / Amazon>"` *(optional — where this user's category actually gets reviewed, so the cracks read goes straight to the right place)*

---

## Tracking (optional)

If the user wants to watch a competitor over time: save them to `tracked competitors` and, on request ("re-run my teardowns"), re-scan and **diff against last time** — new roles appeared (roadmap shift), messaging changed, new complaints emerged, funding moved. The diff is often more interesting than the snapshot.

---

## Edge cases

- **Private / stealth company** — no Crunchbase depth, thin LinkedIn. Lean on jobs + site + news, and mark Snapshot *"partial"* rather than guessing numbers.
- **Thin or no reviews** — B2B tools with few public reviews: widen the `search_engine` pass (Reddit, forums, comparison posts) and say plainly if the cracks section is light. Never invent complaints.
- **Name collision** — same name, different company/region. Always confirm the entity before scanning.
- **No open roles** — hiring freeze is itself a signal (stalling / cost-cutting); say so instead of leaving the roadmap section blank.
- **Huge company** — hundreds of roles. Sample/bucket by the function areas relevant to the user's category instead of listing everything.

---

## What this skill does NOT do

- ❌ Use any MCP other than Bright Data — it's fully self-contained
- ❌ Enrich or contact anyone — the battlecard is the deliverable, there's no lead step
- ❌ Present stale data as current — out-of-window sections are labeled, not hidden
- ❌ Hedge — a battlecard that won't name the wedge isn't worth making
- ❌ Invent complaints or roadmap items — every claim traces to a real Bright Data pull
