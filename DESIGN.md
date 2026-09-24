# DESIGN.md — Teamvista landing page design system (v2, locked)

Replit Agent: this file is the source of truth for every prompt. When a prompt and this file disagree, this file wins unless the prompt says "overrides DESIGN.md".

## Direction

"Real UI, editorially recomposed." An enterprise work-evidence review product presented like a magazine-grade product dossier. The real interface is the hero; the page chrome defers to it. Quality benchmarks only, never copied: ClickUp for section pacing and large product evidence, Stripe for tables and money, Linear for restraint. The page must look like a serious, well-funded enterprise software company, not a template.

## Motifs (the only decoration allowed)

1. **Viewfinder brackets** from the logo mark: four L-shaped corners, 2px stroke, 16px arms (32px on the P17 frame), drawn around screenshot detail regions, callouts and the closing frame. Tone: ink on light, paper on dark, accent only when marking an active detail. Brackets sit 8px outside a frame's corners, never on top of the 12px radius.
2. **Ink on cool paper with one accent**, used for meaning only.
3. **Qualifications as first-class content**, typeset directly under the claim they limit (see Qualification component). A red rule on every caveat is the most recognised AI-template tell, so the accent is reserved for the two Gates.

## Color (CSS custom properties; OKLCH with hex fallback)

Primitives: `--paper-0 #F9FAFC` page; `--paper-1 #F1F4F8` plates; `--line-1 #DDE2EA` hairlines; `--line-2 #C6CDD8` strong rules; `--ink-3 #5E6678` muted (5.5:1 on paper-0, 5.2:1 on paper-1); `--ink-2 #3A4257` secondary; `--ink-1 #141B2D` text (from the mark); `--ink-0 #0B1120` dark plate; `--accent #EA4224` (the product's own red-orange: rules, brackets, nav underline, focus ring, progress line; never behind text); `--action #D63619` (CTA fill; white label 4.77:1); `--action-hover #BF2D12`; `--accent-soft #FCEDE9`; `--ok #2E9E5B` (only mirrors the product's "active" status; never decorative).

Semantic roles: `--bg`, `--bg-plate`, `--bg-dark`, `--text`, `--text-2`, `--text-3`, `--border`, `--border-strong`, `--action`, `--action-hover`, `--action-text #FFFFFF`, `--focus` (= accent, 2px outline, 3px offset, never removed).

**Dark plates:** every surface on `--bg-dark` carries class `plate-dark`, which re-maps `--text` to `#F4F6FA`, `--text-2` to `#C6CDD8`, `--text-3` to `#9AA3B5`, `--border` to `rgb(255 255 255 / 0.12)`, `--border-strong` to `rgb(255 255 255 / 0.22)` and `--bg-plate` to `#121A2C`. No component references `--ink-2` or `--ink-3` directly. Dark is reserved for real UI and the final ask: exactly two full-width dark sections (P04 and P17), the hero evidence panel, and the Gate panels. Every other section is `--bg` or `--bg-plate`.

Rules: no gradients anywhere (the product's gradient stays inside the untouched screenshots); no purple or violet; no pure #000/#FFF text or backgrounds; `--ink-3` never below 0.9375rem; caption-size text on paper uses `--text-2`.

## Typography (self-hosted via fontsource packages, WOFF2, font-display swap, no Google Fonts link)

| Role | Face | Weights |
|---|---|---|
| Display: H1, H2, H3, price, pivot lines | Instrument Sans (variable) | 500, 600 |
| Body, UI, table figures | IBM Plex Sans | 400, 500, 600 |
| Machine text only: IDs, dates, the P10 dateline, the P15 tool names | IBM Plex Mono | 400, 500 |

Scale: H1 `clamp(2.5rem, 1.5rem + 3.2vw, 4.25rem)` / 1.02 / -0.022em / 600. H2 `clamp(2rem, 1.35rem + 1.9vw, 3rem)` / 1.08 / -0.018em / 600 (560 on plate-dark). H3 1.375rem / 1.25 / -0.008em / 600. Pivot (P08.question, P09.decision) `clamp(1.75rem, 1.1rem + 2vw, 2.75rem)` / 1.15 / 500. Price `--t-price clamp(3rem, 2rem + 3.5vw, 5rem)` / 1 / -0.03em / 600, display face, proportional lining figures, never mono. Lead 1.25rem / 1.5. Body 1.0625rem / 1.6. Small 0.9375rem / 1.5. Caption 0.875rem / 1.45. Kicker 0.875rem / 500 / sentence case in `--text-2` (the approved "eyebrow" strings render as sentence-case kickers, never uppercase tracked labels). FAQ summaries: IBM Plex Sans 600 at 1.125rem. Table figures: IBM Plex Sans 500 with `font-variant-numeric: tabular-nums`.

Headings use `text-wrap: balance`. Prose measure max 68ch; qualifications max 60ch. Compound words that would break at a hyphen in headings (P11.h2 "per-seat", P12.h2 "customer-controlled") are wrapped in a `white-space: nowrap` span; never change the string. Approved copy contains em dashes and curly apostrophes; render them exactly.

## Section heading grammar

One `<SectionHeading>` component. Variant `margin` (default for text chapters at ≥1024px): the kicker sits in columns 1–2 on the H2's first baseline as a marginal note; the H2 and body start at column 3 and end by column 10. Variant `stacked` (kicker above the H2) only when the heading lives inside a column (P01, P04, P05, P09, P11, P16, P17) and everywhere below 1024px. Content below a heading may use all 12 columns. Every H2 is a semantic `<h2>`; the page has exactly one `<h1>`, in P01. "H1-size" always means the type token, never the tag.

## Qualification component (one component, three variants)

- `neutral` (default for every boundary, exclusion, scope, caption-style note): 0.9375rem, `--text-2`, a 1px `--border-strong` left rule, 16px indent, max 60ch. Wherever a prompt says "qualification block", use this.
- `strong` (P13.boundary only): lead size, 2px `--ink-1` left rule, no fill, max 60ch.
- `gate` (only P12.authorization and the P14 access cell, both the open authorization finding): a `plate-dark` panel, `--paper-0` text at lead size, 3px `--accent` top rule, Phosphor `Warning` icon, max 60ch, 32px padding.

Screenshot captions: 0.875rem, `--text-3`, directly under the frame, part of the Evidence component, never animated.

## Layout

Container 1280px; gutters 24px at ≥1024, 20px at 768, 16px at ≤390. Columns 12 / 8 / 4. At 1440 one column is 80.7px and the content is 1232px. Section padding: hero and closing 128px; chapters 96px; dense sections (comparison, FAQ) 64px; footer 48px; at ≤768px use 80/64/48/40. Nav 64px, `position: sticky` from first paint. Every anchor target has `scroll-margin-top: 80px`. Breakpoints 640 / 768 / 1024 / 1280 / 1440. Desktop and mobile compositions are authored independently as ONE DOM tree per section laid out responsively, never two parallel markup blocks. Mobile carries complete copy.

Layout family per section (no family repeats consecutively; no row of three or more equal-width boxes anywhere): P01 headline over evidence · P02 ruled index · P03 typographic staircase · P04 sticky viewfinder (dark) · P05 mirrored split · P06 evidence breakout · P07 ruled bands · P07a comparison-style table · P08 pivot statement · P09 sticky question stack · P10 table · P11 sticky offer sheet · P12 unnumbered bands + schematic · P13 two columns + role row · P13a checklist · P14 four-cell bento · P15 rule-framed stack · P16 sticky FAQ · P17 framed close (dark) · P18 footer index.

## Radius, depth

Base 4px; buttons and inputs 6px; panels 8px; screenshot frames 12px. Hairlines before shadows. One shadow token, screenshot frames only: `0 24px 48px -24px rgb(11 17 32 / 0.35)`. Every Evidence frame carries a 1px `--border` hairline on every surface. No glass, no blur, no cards for text that spacing can separate.

## Evidence (screenshots)

Sources: three enhanced 2× masters (session-audit 2962×1782, alerts 3172×1980, screenshots-directory 3056×1728) plus the 512px mark. Regions are named rectangles in percent of the master, stored only in `client/src/content/regions.ts` (seeded from SPEC-REGIONS.json). Crop mechanism, everywhere: a frame with `overflow: clip` and `aspect-ratio` = region width ÷ region height in master pixels, containing the full master `<img>` with `width = 100 / (x1 − x0) × 100%` of the frame and `transform: translate(−x0%, −y0%)` (CSS translate percentages are relative to the image's own box, so the region's top-left lands on the frame's top-left; the equivalent with `left`/`top`, which are frame-relative, is `left = −x0 / (x1 − x0) × 100%`, `top = −y0 / (y1 − y0) × 100%`; never mix the two), passed as inline custom properties so prerender and no-JS render exact crops. Unit-test the helper against a region that does not start at 0 (for example `heroDetail`), not only `full` and `sidebar`. Never `object-fit`/`object-position` for regions. `sizes` describes the enlarged image width. Scale = rendered `<img>` CSS width ÷ the 1× native width (1481 / 1586 / 1528). Bands: 0.75–1.10 at ≥1024px; 0.6–1.25 at 768–1023 and on phones ≥375px; floor 0.5 at 320px. Every frame has `max-width` = 1.25 × its region's 1× native width. Derivatives are generated by downscaling the 2× masters (never upscaled). Brackets are drawn around the smaller region the copy names; crops are chosen for resolution. The on-image "Illustrative demo data" badge stays visible in each full presentation and gets a neutral bracket.

## Motion (summary; SPEC-MOTION.md is the full spec)

Only `transform` and `opacity` animate. Durations `--dur-micro 150ms`, `--dur-press 120ms`, `--dur-entrance 350ms`, `--dur-morph 450ms`; easing `--ease-out cubic-bezier(0.23, 1, 0.32, 1)`, `--ease-in-out cubic-bezier(0.77, 0, 0.175, 1)`; stagger 50–70ms. Reveals are CSS-class driven from one shared IntersectionObserver (`rootMargin "0px 0px -12% 0px"`, once); hidden pre-states exist only under `html.js.motion-ok:not(.reveal-failsafe)`; Motion's `initial` prop never hides text or images. A claim and its qualification reveal as one unit; captions never animate. Parallax exists only in the hero (pixel travel, plate ≤ 64px) and the P06 dolly. No GSAP, no Lenis, no loops, no counters, no cursor effects, no tilt, no glass. Complete without JavaScript (prerendered HTML) and under `prefers-reduced-motion`.

## Icons and states

Phosphor only, weight `regular`, one family; no hand-drawn SVG icons; no emoji. Every interactive element designs default, hover, focus-visible (2px `--focus` outline, 3px offset), active (`scale(0.97)`, 120ms) and disabled. Minimum targets: 44×44px for buttons, the mobile CTA bar, drawer links and FAQ summaries; 24×24px CSS floor for inline links with ≥8px spacing. A skip link is the first focusable element. Drawer: focus trapped, Escape closes, focus returns to the toggle.

## Closed scope

The page contains exactly the sections in `client/src/content/site-content.json` (P00–P18 plus P07a and P13a) and nothing else. No analytics, tracking, cookie banners, forms, calendar embeds, chat widgets, testimonials, logos, badges, stats, or third-party runtime scripts. No publishing or deployment.
