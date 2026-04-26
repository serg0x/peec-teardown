# Competitor Perception & Moat Teardown

A free, open-source workflow that turns your Peec AI data into a polished competitive intelligence report. Two install paths. Same workflow. Same output.

Not a screenshot of a dashboard. Not a generic SEO audit. A structured six-stage analysis that reads every AI search chat, verifies every claim against source material, classifies how competitors have built their moats, and outputs a routed action plan with a measurement schedule.

Built for the [Peec AI MCP Challenge](https://peec.ai/mcp-challenge).

---

## Which version should I install?

This repo ships in two flavors. Pick one based on how you work:

| | **Claude Project** | **Claude Code Desktop** |
|---|---|---|
| **Best for** | Marketers, SEOs, consultants who want the lowest-friction path | Developers, power users who want filesystem access and scheduling |
| **Runtime** | claude.ai (Pro or Team) | [Claude Code Desktop](https://code.claude.com/docs/en/desktop) |
| **Install time** | 5 minutes (paste prompt + upload template) | 5 minutes (clone repo + connect MCP) |
| **Output** | HTML rendered as Claude artifact, downloadable | HTML written directly to `reports/` folder |
| **Branding** | URL auto-extract, manual values, or upload logo as knowledge file | URL auto-extract, manual values, or local logo file |
| **Scheduling** | Manual reruns | Native scheduled tasks (e.g. weekly rechecks) |
| **Multi-MCP composition** | Limited to claude.ai connectors | Full (add Peec + GSC + GA4 + any MCP server) |
| **Source of instructions** | [`SYSTEM_PROMPT.md`](./SYSTEM_PROMPT.md) | [`CLAUDE.md`](./CLAUDE.md) |

If you're not sure: start with **Claude Project**. Lower friction. You can always switch later. The two versions share the same template, examples, and methodology docs.

---

## What it does

Given a Peec project and a named competitor, the workflow produces a branded HTML report that answers three questions a dashboard cannot:

1. **What are AI search engines actually saying about my brand versus the competitor?** Not just visibility numbers. The specific adjectives, use cases, and feature claims that anchor perception.

2. **Why is the competitor winning the battlegrounds they're winning?** Owned-page moat, editorial moat, reference moat, UGC moat, or partner moat. Each has a different fix.

3. **Which AI-attributed claims are actually true?** AI engines routinely amplify vendor marketing language into harder claims that the vendor never made. This workflow verifies every competitor claim against source material before it enters the response plan.

Output is routed to four action categories (on-page, editorial, UGC, sales enablement) with a 14/28/56-day recheck schedule that loops back into Peec.

---

## Why verification matters

Most AI-search competitive analyses accept what the AI says about competitors at face value. That's a mistake.

In the reference teardown shipped with this repo, Perplexity told prospective buyers that a competing SaaS offers legal liability protection through its platform. The vendor's actual product pages don't claim that. Anywhere. Perplexity had aggregated marketing language ("protect your company", "100% compliant") into a harder legal claim the vendor never made.

If your competitive analysis pipeline ingests that without verification, you ship recommendations based on things the competitor never actually said.

This workflow catches that. Stage 5 (Fact-Check) is non-negotiable in both versions.

---

## Branded reports

The HTML report renders in your client's visual identity. Three ways to set this up at the start of a session, identical in both versions:

**A) Auto-extract from a homepage URL.** Give the workflow a URL. It fetches the page, finds the brand color from CTA buttons and accent elements, identifies the font from Google Fonts or Fontshare loaders, locates the logo via favicon or og:image. It shows you the candidates with reasoning. You confirm or correct.

**B) Manual values.** You provide the brand color hex, font family name, and logo (URL or upload). The workflow applies them directly.

**C) Neutral default.** Use the template's stock indigo styling. Good for internal use or when client branding doesn't matter.

Option A is the fastest if your client's site uses standard CSS conventions. It works on most B2B SaaS sites. It can fail on heavily compiled Tailwind builds or JavaScript-rendered styles, in which case the workflow tells you so and asks for manual values.

---

## Install: Claude Project

You need a Claude.ai Pro or Team account.

**1. Connect the Peec AI MCP in Claude.ai:**

Settings → Connectors → Add custom connector
- Name: `Peec`
- URL: `https://api.peec.ai/mcp`
- Authorize via OAuth.

**2. Create a new Claude Project.**

Projects → New Project. Name it whatever you like.

**3. Paste the system prompt.**

Copy the contents of [`SYSTEM_PROMPT.md`](./SYSTEM_PROMPT.md) into the Project's instruction field.

**4. Upload the HTML template as a knowledge file.**

Upload [`templates/teardown-template.html`](./templates/teardown-template.html) to the Project's knowledge files.

**5. Start a new conversation in the Project.**

Claude asks for your config: brand name, Peec project ID, primary competitor, and how you want to brand the report. Answer those, then say:

> Run the teardown.

Claude walks through the six stages with periodic checkpoints. The final HTML report renders as an artifact at the end. Download from the artifact panel.

---

## Install: Claude Code Desktop

You need [Claude Code Desktop](https://code.claude.com/docs/en/desktop) installed.

**1. Clone this repo:**

```bash
git clone https://github.com/serg0x/peec-teardown.git
cd peec-teardown
```

**2. Connect the Peec AI MCP in Claude Code Desktop:**

Settings → Connectors → Add custom connector
- Name: `Peec`
- URL: `https://api.peec.ai/mcp`
- Authorize via OAuth.

**3. Open the project in Claude Code Desktop:**

File → Open Project → select the cloned `teardown` folder.

Claude reads `CLAUDE.md` automatically and knows how to run the workflow.

**4. Run your first teardown:**

In a new session, paste a config block:

```
BRAND_NAME: Acme
PROJECT_ID: or_abc123...
PRIMARY_COMPETITOR: Rival Co
LANGUAGE: en
TIME_WINDOW: last 30 days
BRANDING: auto:https://acme.com
```

Then say:

> Run the teardown.

Claude walks through Stage 0 (branding setup) and the six analysis stages with periodic checkpoints. The final HTML lands in `reports/`.

---

## The workflow

1. **Stage 0 — Branding setup.** Auto-extract from URL, manual values, or neutral default.
2. **Stage 1 — Positioning snapshot.** Visibility and share-of-voice ranking, dimensioned by intent tag and AI model.
3. **Stage 2 — Battleground identification.** The prompts where the competitor beats you by the widest margin, ranked by volume.
4. **Stage 3 — Perception extraction.** How AI engines actually describe each brand, including specific adjectives and feature claims.
5. **Stage 4 — Moat classification.** Owned-page, editorial, reference, UGC, or partner, using Peec's native URL and domain classifications.
6. **Stage 5 — Fact-check and verification.** Distinguishes page-not-indexed from page-indexed-but-not-retrieved from retrieved-but-not-cited. Three different diagnoses, three different fixes.
7. **Stage 6 — Response plan and measurement.** Routed recommendations with 14/28/56-day recheck schedules keyed to specific prompt IDs.

See [`docs/methodology.md`](./docs/methodology.md) for the deeper rationale behind each stage.

---

## Run time

Full workflow on a Peec project with 5-10 battleground prompts: 30-50 tool calls, roughly 5-8 minutes wall time. Fits inside a standard Claude session.

Partial runs (specific battlegrounds, rechecks, single-stage queries) are faster. Use "rerun the teardown on prompts 4 and 7 only" for month-over-month measurement.

---

## Examples

[`examples/`](./examples/) contains anonymized reference teardowns. Each includes the final HTML report plus a markdown summary of the key findings and what the workflow caught that a dashboard would have missed.

---

## What this workflow is not

- **Not a dashboard.** Dashboards compress hundreds of chats into three numbers. This workflow reads them all.
- **Not a replacement for Peec.** This runs on top of Peec. You still need a Peec subscription for the data.
- **Not hosted.** Everything runs in your Claude environment. No infrastructure to maintain, no subscription to this tool.
- **Not automatic.** Claude checkpoints at each stage. Review findings, ask for drill-downs, course-correct.

---

## License

MIT. Fork freely, adapt the template, rebrand for your agency, publish teardowns under your own name.

---

## Built with

- [Claude.ai](https://claude.ai) and [Claude Code Desktop](https://code.claude.com/docs/en/desktop) as the runtimes
- [Peec AI MCP](https://docs.peec.ai) as the data source
- Plus Jakarta Sans (Google Fonts) and JetBrains Mono (Fontshare) for the report typography by default; any web font can be substituted via Stage 0 branding setup

#BuiltWithPeec
