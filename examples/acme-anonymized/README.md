# Example: Brand A vs Brand B

An anonymized teardown produced with this workflow against a real live Peec project. Brand A and Brand B are two competitors in a German-speaking B2B SaaS category. 877 AI search chats over 30 days, three active models (ChatGPT, Google AI Overview, Perplexity).

Open [`teardown.html`](./teardown.html) in a browser to see the full report.

## What the dashboard showed

Brand A sits at 22% visibility, second place, two points behind Brand B's 24%. Average position 1.7 versus 1.4. Sentiment is tied at 53 vs 54. A dashboard-only analyst would call this a close race and move on.

## What the teardown surfaced

**Three structurally different problems hiding behind the two-point gap.** Each needs a different fix.

### 1. The A1 product page exists but AI can't find it

The most commercially painful battleground is a medium-volume query where Brand A loses by 13 percentage points. Initial hypothesis: Brand A lacks a dedicated product page for this topic, only has wiki content.

**Stage 5 (Fact-Check) caught the mistake.** Brand A does have a dedicated product page at `/loesungen/features/<topic>`. Peec has it indexed as `PRODUCT_PAGE`. But it appears in zero of the seven analyzed chats for this query. AI engines retrieve Brand A's wiki pages instead.

This is a discoverability problem, not a content problem. Different fix: audit URL depth, H1 phrasing, schema markup, internal link density. Not "build a new page."

### 2. AI engines amplified vendor marketing into a legal claim

Perplexity told prospective buyers that Brand B guarantees legal liability protection through its platform. A verbatim quote attributed to Brand B: *"Haftung durch das System"* (liability through the system).

`get_url_content` on Brand B's actual product page revealed: no such claim exists. The page uses language about "protection," "100% compliance," "global coverage" — all marketing language. None of it commits to liability transfer.

Perplexity had aggregated soft marketing into a hard legal claim the vendor never made. Without Stage 5 verification, a naïve pipeline would have accepted this as established competitive intelligence and shipped recommendations based on it.

### 3. The "close race" is a cross-model illusion

The aggregate numbers are close. The per-model numbers are not.

- Google AI Overview: Brand A 48% vs Brand B 40%. Brand A dominates by 8 points.
- Perplexity: Brand A 8% vs Brand B 19%. Brand B dominates by 11 points, a factor of 2.4x.
- ChatGPT: roughly tied (14% vs 16%).

The aggregate 2-point gap is an average of two opposite realities. Brand A wins Google hard and loses Perplexity hard. The fix for each is different. The aggregate number hides both.

## Methodology validation

A useful check: did the qualitative deep-dive arrive at the same top recommendation as Peec's native `get_actions` engine computed independently?

- Qualitative analysis from chat content: top lever is Owned / Product Page improvement
- Peec Actions `opportunity_score`: OWNED / PRODUCT_PAGE at 0.33 (HIGH), gap 43%, highest score in the entire analysis

Two methods, same answer. When qualitative and quantitative converge, you ship with confidence. When they diverge, one of them is wrong and it's worth investigating which before making recommendations.

## What this proves

The value of this workflow is not "AI reads a lot of data fast." LLMs do that trivially.

The value is the verification discipline. Stage 5 catches three things that a naïve pipeline misses: (1) that pages exist but aren't retrieved, (2) that AI engines amplify marketing language into harder claims, and (3) that aggregate metrics hide cross-model asymmetries.

Without Stage 5, all three of these findings would have been shipped wrong.
