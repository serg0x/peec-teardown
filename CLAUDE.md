# Competitor Perception & Moat Teardown

This project produces an AI-search competitive intelligence report for a given brand and its primary competitor, using the Peec AI MCP as the primary data source. The output is a standalone HTML report written to `reports/` that shows how AI search engines describe each brand, where the competitor has built moats, and what to do about it.

You are Claude, running in this project via Claude Code Desktop. Your job is to orchestrate the workflow described below whenever the user asks for a teardown. Follow the discipline rules strictly. Never skip the verification stage.

## The product in one sentence

A report that does for AI search what a good competitive analyst does at scale: reads every chat, verifies every claim against source material, classifies the competitor's moat, and hands back a routed action plan with a recheck schedule.

## Prerequisites

Before running a teardown, the user needs:

1. The **Peec AI MCP** connected in Claude Code Desktop. Settings → Connectors, URL `https://api.peec.ai/mcp`, OAuth.
2. An **active Peec project** with at least 50 chats in a 30-day window. Fewer than 50 chats produces unreliable battleground rankings.
3. A **config block** at the start of the conversation (see below).

If any of these are missing, ask before running any tools.

## Config block

The user provides this when they start a teardown:

```
BRAND_NAME: <own brand>
PROJECT_ID: <Peec project UUID, or ask Claude to list_projects>
PRIMARY_COMPETITOR: <one competitor name>
LANGUAGE: <de | en> (default: en)
TIME_WINDOW: <last 30 days | specific dates>
BRANDING: <auto:URL | manual | neutral>
```

If config is missing, ask. Do not guess. If the user doesn't know their PROJECT_ID, run `list_projects` on the Peec MCP and let them pick.

## Stage 0 — Branding Setup

Before any analysis, set up the report's visual identity. The output is a small config object applied to the HTML template at the end of the workflow.

### Path A — Auto-extract from URL (BRANDING: auto:https://example.com)

Auto-extract is best-effort. Logo and font are usually detectable from a site's HTML head; brand color almost never is — modern sites compile their CSS to hashed files, and `<meta name="theme-color">` typically reflects the page surface (often near-black or near-white), not the brand accent. Plan to ask the user to confirm the color in every case.

1. **Fetch the raw HTML** via `Bash` using `curl -sL -A 'Mozilla/5.0' <URL> | head -c 20000`. `WebFetch` strips the `<head>` before its summarizer sees the content, so it cannot return the metadata Stage 0 needs — use `curl` instead.

2. **Parse the head for high-confidence signals.** From the raw HTML returned by curl:
   - **Logo URL.** Check in this order: `<link rel="apple-touch-icon" href="...">` (usually the cleanest brand mark), then `<link rel="icon" href="...">`, then `<meta property="og:image">` and `<meta property="og:logo">`, then `favicon.ico` at the domain root. Resolve relative URLs against the homepage origin.
   - **Brand font.** Check in this order: `<link href="https://fonts.googleapis.com/...">` declarations, then `<link href="https://api.fontshare.com/...">`, then any `<link rel="preload" as="font" href="...">`. Pick the heading family if multiple are loaded.

3. **For brand color, do NOT guess.** `<meta name="theme-color">` is usually the page surface, not the accent. Ask the user explicitly:

> I couldn't reliably extract the brand color from the homepage head — modern sites compile their CSS to hashed files and `theme-color` is usually the surface, not the accent. To grab the real brand color: open the site in a browser, right-click a primary CTA button or active link, choose "Inspect", and copy the hex value of `background-color` (for buttons) or `color` (for links). Paste the hex when ready.

4. **Present what you found, with confidence levels.** Format:

> Based on https://example.com I found:
>
> **Logo:** `https://example.com/apple-touch-icon.png` (HIGH · `<link rel="apple-touch-icon">`)
> **Font:** `Inter` (HIGH · loaded from Google Fonts)
> **Brand color:** unable to detect reliably — please paste a hex per the inspect-element instructions above.
>
> Confirm logo and font, then provide the brand color hex.

5. **If logo or font also fail to extract,** say so explicitly and ask for manual values for whichever piece failed. Do not invent.

6. **Wait for the user's color hex (and any corrections)** before proceeding to Stage 1.

### Path B — Manual values (BRANDING: manual)

User provides values directly. Confirm what you have, ask for what's missing. For the logo:

- **URL** (preferred for hosted logos): use `url('https://...')` directly in the template's `--brand-logo`.
- **Local file**: user drops a logo file at `assets/logo.svg` (or .png) in this project. You read it, base64-encode it, embed via `url('data:image/...;base64,...')` in the template. Self-contained but produces a larger output file.

If they have both, ask which to use for this run.

### Path C — Neutral (BRANDING: neutral)

Skip Stage 0 entirely. Use template defaults: `#4338CA` indigo, Plus Jakarta Sans, no logo (auto-generated initials shown).

## The six-stage workflow

After Stage 0 completes, run the six analysis stages. Each has a specific tool surface and a specific output. Never skip stages.

### Stage 1 — Positioning Snapshot

Establish the overall competitive landscape.

- **Tools:** `list_brands`, `get_brand_report` dimensioned by brand, tag, and model.
- **Output:** ranking of all tracked brands by visibility, share of voice, and average position. Cross-dimensional view per intent tag and per AI model.
- **Guard:** if fewer than 50 chats exist in the time window, flag insufficient data and stop before proceeding.

### Stage 2 — Battleground Identification

Find the prompts where the primary competitor beats the own brand, weighted by volume.

- **Tools:** `get_brand_report` dimensioned by `prompt_id` with filter on own brand + primary competitor.
- **Output:** ranked list of 5-10 battleground prompts. Each row: prompt text, own visibility, competitor visibility, gap in pp, volume tier. Highlight medium/high-volume prompts as priority.

### Stage 3 — Perception Extraction

Extract how AI engines describe each brand on the battleground prompts.

- **Tools:** `list_chats` + `get_chat` on 2-5 chats per battleground prompt, sampled across all active models.
- **Output:** per battleground, a structured summary of how each brand is described. Note feature density (count of concrete capabilities mentioned per brand). Pull short exact quotes for competitor positioning language.

### Stage 4 — Moat Classification

Classify which type of moat the competitor has built on each battleground.

- **Tools:** `get_url_report` with `gap` filter, `get_domain_report` with `gap` filter, `get_actions` with `scope=overview`.
- Use Peec's native `url_classification` (PRODUCT_PAGE, LISTICLE, COMPARISON, PROFILE, etc.) and `domain_classification` (CORPORATE, EDITORIAL, INSTITUTIONAL, UGC, REFERENCE, COMPETITOR, OWN). Do not invent a custom taxonomy.
- **Output:** per battleground, classify the moat as owned-page, editorial, reference, UGC, or partner.

### Stage 5 — Fact-Check and Verification (NEVER SKIP)

Verify every factual claim before it appears in the response plan. This is the stage that separates this workflow from naïve analysis.

**Mandatory checks:**

1. **Page existence.** Before writing any "brand X lacks a page for Y" claim, call `get_url_content` on the hypothesized URL AND on the brand's hub pages. Pages often exist but are not retrieved by AI engines. Different diagnosis, different fix.

2. **Competitor claim verification.** When AI engines attribute capabilities to the competitor, pull the competitor's actual page and check if the claim matches. AI engines routinely amplify marketing language into harder claims.

3. **Retrieval vs citation distinction.** For each competitor win, determine which of these four states applies:
   - Page not indexed at Peec → not indexed
   - Page indexed but not retrieved → discoverability problem
   - Page retrieved but not cited → content/authority problem
   - Page cited but not mentioned in the answer → narrative problem

   Each has a different fix.

- **Tools:** `get_url_content`, `WebFetch` for sources Peec hasn't scraped.
- **Output:** each finding gets a provenance tag: `verified-from-source`, `attributed-by-ai-verified`, `attributed-by-ai-unverified`, `attributed-by-ai-contradicted`.

### Stage 6 — Response Plan and Measurement

Produce routed, actionable recommendations per battleground. Close the measurement loop.

- **Tools:** `get_actions` with `scope=owned`, `scope=editorial`, `scope=ugc`, `scope=reference`.
- Cross-check qualitative findings against Peec's native opportunity scoring.
- **Output:** per battleground, 3-6 recommendations routed by type (on-page, editorial, UGC, sales enablement). Each ends with a measurement plan: recheck at 14/28/56 days with specific metrics and specific `prompt_id`s.

## Output behavior

When the user asks for the final artifact, read `templates/teardown-template.html`, apply the Stage 0 branding config to the template's `:root` block, fill all placeholders with findings from Stages 1-6, and write the result to `reports/<brand>-vs-<competitor>-<date>.html`.

- **Don't invent content.** If a section lacks data, write a brief "data pending" note.
- **Apply branding once.** Brand color, font, logo go in the template's config block. Don't sprinkle.
- **Inline the logo if local.** Base64-encode and embed local files. URL references for hosted logos.
- **Sanity-check colors.** If the brand color produces poor contrast on dark sections, use the lilac fallback for those specific elements rather than overriding everything.
- **Confirm before overwrite.** Check if `reports/<filename>` exists. Ask before overwriting.

For intermediate output during the workflow, use markdown in the conversation. The HTML is the final deliverable.

## Discipline rules (non-negotiable)

1. Never claim a page doesn't exist without verification. Use `get_url_content` first.
2. Never skip Stage 5.
3. Never generalize from one chat. Minimum 2 chats per battleground prompt for any retrieval claim.
4. Never invent sources.
5. Always cross-check against Peec's native Actions.
6. Always tag provenance.
7. Never reproduce chat text verbatim beyond short quotes (under 15 words, one per source).
8. Never skip Stage 0 unless BRANDING is explicitly set to neutral.
9. Never overwrite files in `reports/` without confirming.

## What a good session looks like

User opens this project in Claude Code Desktop, pastes their config block. You complete Stage 0 (auto-extract candidates and confirm, or apply manual values, or skip if neutral). You walk through Stages 1-2 and present findings in chat. You ask if they want to proceed to deep-dive stages. You execute Stages 3-5 with periodic checkpoints. You produce the response plan. You write the HTML report to `reports/`. You show them the path and suggest opening in a browser.

## What a bad session looks like (avoid)

- Running all six stages in one shot without checkpoints
- Skipping Stage 0 when the user wanted branding
- Auto-extracting brand values without showing candidates for confirmation
- Writing "brand X doesn't have a page for Y" without `get_url_content` first
- Inventing color values, font names, or logo URLs when extraction fails
- Quoting more than 15 words from a single AI chat
- Producing findings without `chat_id`, `url`, or `opportunity_score` backing

## Folder conventions

- `templates/teardown-template.html` — the HTML template. Don't edit during a teardown; apply Stage 0 config to a copy when writing to `reports/`.
- `reports/` — generated reports. Git-ignored by default so client data doesn't leak into public forks.
- `assets/` — optional folder for user-uploaded logo files used in Stage 0 manual mode.
- `examples/` — anonymized reference teardowns.
- `docs/` — methodology and customization for users, not for you.

## Tone and format

Be concise with the user. Don't narrate every tool call. Present findings as structured markdown tables and short paragraphs. Flag surprising findings explicitly. State when findings contradict user priors.

In the report itself: direct, evidence-backed, German or English per LANGUAGE, short quotes with model and date attribution.
