# Customization

How to adapt the HTML template to match your brand or your client's brand.

## The three knobs

The template at `templates/teardown-template.html` has a single config block at the top of the `:root` CSS rule:

```css
:root {
  /* 1. Brand color — your primary brand color (hex). Derives highlights, accents, charts. */
  --brand-color: #4338CA;

  /* 2. Brand font — must match family loaded via <link> in <head> above. */
  --brand-font: 'Plus Jakarta Sans', system-ui, -apple-system, sans-serif;

  /* 3. Brand logo — your logo URL (PNG or SVG on transparent background).
        Set to `none` to fall back to an auto-generated initials mark. */
  --brand-logo: none;
}
```

That's it. Everything else cascades automatically from these three inputs.

## Changing the brand color

Any valid CSS color works. Hex, HSL, named colors. The template uses `color-mix()` to derive shades:

- Deep brand (for hover states, dark accents)
- Soft brand (for backgrounds, tinted fills)
- Accent (for decorative highlights)
- Accent-soft (for subtle gradients)

So one hex value produces four coordinated shades. You don't need to pick them yourself.

The competitor's chart color stays fixed at neutral teal (`#0F766E`). This keeps competitors visually distinct from your brand regardless of what your brand color is. If your brand is green, the competitor bars stay teal. If your brand is red, same. Don't change this unless you have a specific reason.

## Changing the font

Two edits:

**1. Change the font URL in the `<head>` section.** The template loads Plus Jakarta Sans from Google Fonts by default:

```html
<link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&display=swap" rel="stylesheet">
```

Replace with any Google Fonts or Fontshare URL. Match the weight range (400;500;600;700;800) to keep typographic hierarchy intact.

**2. Update `--brand-font` to match the family name.**

```css
--brand-font: 'Inter', system-ui, -apple-system, sans-serif;
```

Always include `system-ui, -apple-system, sans-serif` as the fallback. If the Google Fonts CDN fails or the user is offline, the report still renders with a reasonable system sans-serif.

## Adding a logo

Set `--brand-logo` to a URL:

```css
--brand-logo: url('https://example.com/logo.svg');
```

The logo renders inside the masthead circle, cover-fitted. PNG or SVG works. Transparent background recommended. Square aspect ratio fits best (the container is a 32px circle).

If you set `--brand-logo: none`, the template shows auto-generated text initials instead (currently "AI").

## Font pairings that work well

If you want to experiment beyond Plus Jakarta Sans, these all work with the template's design system:

- **Inter** — neutral, clean, widely recognized
- **General Sans** (Fontshare) — similar to Plus Jakarta Sans, slightly more character
- **Satoshi** (Fontshare) — geometric, confident
- **Space Grotesk** — a bit more editorial
- **DM Sans** — rounded, friendly

Avoid:

- Serif fonts for the body (the template's hierarchy relies on sans weight contrast)
- Display fonts (e.g., Bebas Neue) — they break at the sizes used in headings
- Fonts without a 400/500/600/700 weight range

## Dark mode

Not currently supported. The template uses a warm cream canvas (`--canvas`). Dark mode would require inverting the entire color system, not just flipping a flag.

If you want it: fork the template, override `--canvas`, `--canvas-elevated`, `--ink`, `--ink-soft`, and `--ink-muted` with dark-mode equivalents, and test. It's doable but not a one-line change.

## Don't customize

Some things are deliberately fixed because they're part of the report's visual identity, not per-client branding:

- Typography hierarchy (sizes, weights, line heights)
- Spacing scale
- Component structure (stat cards, brand ranking bars, moat cards, citation donut)
- Code font (JetBrains Mono, for URL fragments and technical labels)
- Status colors (success green, warn orange, danger red)
- Competitor color (neutral teal)

Changing these breaks the "this is a teardown report" visual category. Different color palette, different logo — still reads as a teardown. Changed layout, different typography scale — reads as an unrelated document.

## Testing your customization

Open the modified template in a browser directly. Placeholders are clearly marked. If the spacing or hierarchy looks off, you probably changed something outside the three knobs.
