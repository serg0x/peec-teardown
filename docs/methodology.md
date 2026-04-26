# Methodology

Why the workflow is structured the way it is, and what each stage contributes.

## The core premise

AI search is a new retrieval surface with different failure modes than traditional SEO. Dashboards that treat it like classical SERP ranking miss things that matter.

Three failure modes specifically:

1. **Aggregate metrics compress cross-model divergence.** An "average visibility" across ChatGPT, Google AI Overview, and Perplexity hides that they often retrieve from different source pools and tell different stories. A 20% aggregate visibility could be 40% on Google and 0% on Perplexity.

2. **AI-attributed claims about competitors are routinely unverified.** LLMs amplify marketing language into harder claims. If a competitive analysis pipeline treats AI output as ground truth about what competitors actually say, it ships recommendations based on fiction.

3. **"Brand X beats us" has multiple possible causes, each with a different fix.** Owned-page citations, editorial listicles, UGC, reference content, and partner pages are all different moats. Treating them as a single "visibility" metric obscures what to actually do.

The six-stage workflow is designed around these three failure modes.

## Stage-by-stage rationale

### Stage 1 — Positioning Snapshot

Dimensioned by brand, tag, AND model. Not just brand.

The model dimension is the one that matters most. Aggregate visibility across models is misleading in almost every Peec project I've seen. Cross-model divergence reveals which engines favor which source types, which reveals what fixes will actually move which metric.

### Stage 2 — Battleground Identification

Ranked by gap, weighted by volume. The medium-volume battlegrounds are the only ones with commercial significance for most B2B brands. Low-volume prompts are noise unless they're strategically loaded (e.g., high-intent branded queries).

### Stage 3 — Perception Extraction

The feature density count is the key metric here. AI engines describe brands with whatever feature language the source material supplies. If Brand A's sources are wiki articles (educational tone, few features) and Brand B's sources are product pages (marketing tone, many features), the perception asymmetry is structural, not random.

Feature density is a proxy for "how much your own narrative is anchoring AI descriptions of you."

### Stage 4 — Moat Classification

Use Peec's native classifications. Don't invent a custom taxonomy.

The five moat types (owned-page, editorial, reference, UGC, partner) are structurally different markets with different operators and different economics. An owned-page moat means you're losing the SEO/content layer. An editorial moat means you're losing listicle outreach. A reference moat means you're losing Wikipedia/institutional authority. A UGC moat means you're losing Reddit/YouTube/G2. A partner moat means you're losing integration listings.

Conflating these produces generic advice. Separating them produces routed, specific, actionable recommendations.

### Stage 5 — Fact-Check and Verification (the critical stage)

This is the stage that separates this workflow from naïve analysis. It's non-negotiable.

Three specific checks:

**Page existence.** Before claiming "Brand X doesn't have a page for Y," verify by calling `get_url_content`. A surprising number of "missing" pages actually exist but aren't retrieved by AI engines. This is a discoverability problem, not a content problem. Different fix.

**Competitor claim verification.** When an AI engine attributes a specific capability or claim to a competitor ("offers liability protection," "guarantees 99.9% uptime," "supports 47 countries"), verify it against the competitor's actual published content. AI engines routinely amplify marketing language into harder claims. The reference teardown in this repo documents a case where Perplexity invented a liability guarantee the vendor never made.

**Retrieval vs. citation distinction.** For every competitor win, determine which of four states applies:
- Page not indexed → indexing problem
- Page indexed but not retrieved → discoverability problem
- Page retrieved but not cited → authority problem
- Page cited but brand not mentioned in the answer → narrative problem

Each has a different fix. A pipeline that conflates them produces wrong recommendations.

### Stage 6 — Response Plan and Measurement

Routed recommendations (on-page, editorial, UGC, sales enablement) because different teams execute different fixes. On-page changes go to a content team. Editorial outreach goes to PR. UGC seeding goes to community. Sales enablement goes to revenue ops.

Each recommendation ends with a measurement plan keyed to specific `prompt_id`s at 14/28/56 days. This closes the loop: the recommendations that were produced by Peec data get rechecked against Peec data. Without the recheck schedule, the workflow is open-loop. With it, the report becomes a system.

## Why cross-check against `get_actions`

Peec's native `get_actions` engine computes opportunity scores quantitatively. This workflow produces recommendations qualitatively through chat analysis.

When the two methods converge on the same top lever, you ship with confidence. When they diverge, one of them is wrong. The cross-check is a quality gate.

Don't ignore Peec's output in favor of your own opinions. If your qualitative analysis says "editorial outreach is the top lever" and `get_actions` says "owned product pages is the top lever with 3x the opportunity score," your analysis is probably missing something.

## What this workflow is not designed for

- **Real-time monitoring.** Teardowns are structural analysis. Run monthly, quarterly, or event-driven (repositioning, board meetings, competitor moves). Daily is overkill and noisy.
- **Single-keyword tracking.** The battleground concept assumes at least 15-20 tracked prompts in the Peec project. Fewer than that and Stage 2 produces weak rankings.
- **B2C brands with high volume.** The methodology works but the economics don't. Enterprise B2B with 5-20 named competitors and measurable queries is the sweet spot.
- **Press release copy.** The report is analytical, not promotional. It names weaknesses. Share with internal stakeholders, not with prospects.
