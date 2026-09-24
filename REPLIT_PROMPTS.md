# Teamvista landing page — Replit prompt pack v2

Paste one prompt at a time into Replit Agent in the existing Teamvista workspace (prompts 0, 1a, 1b, 2, 3, 4a, 4b, 5 to 17, and an optional 18). Wait for each verification report before pasting the next. Detailed specifications live in the public kit repo (https://github.com/ahmadashfq/teamvista-replit-kit), which the prompts fetch by commit-pinned URL; the prompts stay short on purpose because Replit's own research shows long rule lists decay over a session.

Before you start:

1. Set the Replit project to **Private**. The `*.replit.dev` preview link is reachable by anyone who has it.
2. Every supporting file is fetched by the prompts themselves from a public, commit-pinned GitHub repo (https://github.com/ahmadashfq/teamvista-replit-kit, commit 9cdf5b9), so nothing needs uploading by hand. Each fetch is hash-verified inside the prompt.
3. Prompt 4 is split into 4a (navigation) and 4b (hero).
4. `CTA_HREF` is yours to set. Until you do, the QA prompt will report it as a blocking failure by design. The placeholder still scrolls to the closing section, so the preview is usable meanwhile.
5. If a prompt fails twice on the same problem, roll back to the last checkpoint and re-paste it with the failure described in one sentence at the top.

Every build prompt ends with this line, scoped to its own risk:

> GUARDRAILS: render only exact copy from site-content.json via t()/section()/table(), never paraphrase or shorten; keep every boundary, qualification and caption directly adjacent to its claim or image; no analytics, forms, chat widgets, cookie banners or third-party scripts (ask first); no publish or deploy.

---

## Prompt 0 — Read-only audit (Plan mode)

```
Use Plan mode. Do not change any file and do not create a checkpoint. Read the workspace and report, in this order:
1. package.json in full (every dependency with its exact version); the Vite config; whether Tailwind is v3 (tailwind.config.*) or v4 (@tailwindcss/vite and CSS @theme); whether shadcn/ui is installed; the TypeScript, ESLint and Prettier setup; the exact scripts for dev, build, typecheck, lint and test.
2. The folder tree of client/, server/ and shared/ two levels deep, and which files render the current landing page.
3. Any images, fonts or font <link> tags in use and where they load from.
4. Git: branch, clean or dirty, last five commits.
5. `node -v`, `npm -v`, and `npm ls sharp` (if absent, say whether `npm install sharp@0.34.3` should work here; do not install).
6. Whether this project is Public or Private.
7. One honest paragraph: is the existing frontend worth keeping, or should client/src be rebuilt from a clean slate, given that the goal is an original enterprise-grade long-form marketing page and the previous output was judged not good enough?
Do not build anything. End with "AUDIT COMPLETE".
```

---

## Prompt 1a — Checkpoint, clean slate, dependencies, prerender

```
Create a checkpoint named "pre-redesign" and confirm it exists before anything else.

Keep the framework (Vite + React + TypeScript + Tailwind, plus the existing thin server that serves the client; leave the server alone). Remove every previous landing-page component, page and shadcn marketing block from client/src so nothing from the old design leaks into the new one; keep only the app entry, router and global CSS, emptied.

Install pinned (exact versions, no caret): motion@13.4.1, @phosphor-icons/react@2.1.10, @fontsource-variable/instrument-sans, @fontsource/ibm-plex-sans, @fontsource/ibm-plex-mono, sharp@0.34.3, png-to-ico. Never install GSAP or Lenis at any point in this build. Show me the resulting package.json dependency lines.

Tailwind mapping: if Prompt 0 found v3, tokens live as CSS custom properties on :root in the global CSS and tailwind.config.js references them under theme.extend as 'var(--token)' strings; if v4, declare them inside an @theme block in the global CSS and do not create a tailwind.config.js.

Prerender pipeline, so the page is complete with JavaScript disabled: add client/src/entry-server.tsx (react-dom/server render of <App/>), scripts/prerender.mjs (after vite build, inject the markup into #root of dist/index.html), switch main.tsx to hydrateRoot, wire "postbuild": "node scripts/prerender.mjs". In index.html <head>, before any stylesheet, add an inline script that adds class "js" to <html>, adds "motion-ok" when matchMedia('(prefers-reduced-motion: no-preference)').matches, and after 3000ms adds "reveal-failsafe" unless <html> has "app-ready" (the app adds "app-ready" after hydration). Add a visually-hidden-until-focused skip link as the first element in <body> pointing to <main id="main">. Set lang="en" and <title>Teamvista</title>.

GUARDRAILS: no analytics, forms, chat widgets, cookie banners or third-party scripts; no publish or deploy.

Verify: typecheck and build succeed; `grep -c "<main" dist/index.html` is 1; Preview shows an empty main with zero console errors; checkpoint "foundation-deps".
```

---

## Prompt 1b — Design system

```
Fetch https://raw.githubusercontent.com/ahmadashfq/teamvista-replit-kit/9cdf5b99b972a10a73dccf06da41ae0655660b8c/DESIGN.md and save it byte-for-byte as DESIGN.md at the project root (if the fetch fails outright, stop and tell me; do not retry more than once). Its SHA-256 must be c3f9396ed34d81f74693e19ca9aa84097c789093065e6b88a96a18be13fe843c; verify and stop if it differs.

Read DESIGN.md at the project root. It is the locked design system and wins over any later instruction unless that instruction says "overrides DESIGN.md". Implement its foundations now, nothing else:

1. Global CSS: every colour primitive and semantic role as custom properties (OKLCH with hex fallback), the plate-dark re-map (a .plate-dark class that redefines --text, --text-2, --text-3, --border, --border-strong, --bg-plate), the type scale tokens including --t-price and the pivot size, spacing and radius tokens, the single shadow token, motion tokens, and scroll-margin-top: 80px on every [id] section. Load the three fonts from the fontsource packages with font-display: swap and add size-adjust fallback metrics so the swap does not shift layout. Set the Phosphor default weight to "regular".
2. Components in client/src/components/ui/: Button (primary: --action fill, --action-text label, hover --action-hover, active scale(0.97) 120ms, focus-visible 2px --focus outline offset 3px, 44px minimum height, white-space: nowrap; secondary: text link, weight 600, underline offset 4px); SectionHeading with variants "margin" and "stacked" exactly as DESIGN.md defines (kicker in columns 1–2 on the H2's first baseline for the margin variant; H2 is always a semantic <h2>; H2 uses weight 560 inside .plate-dark); Qualification with variants "neutral", "strong" and "gate" exactly as DESIGN.md defines (gate = plate-dark panel, 3px --accent top rule, Phosphor Warning icon, max 60ch); Container (1280px, responsive gutters, 12/8/4 column grid helpers); Brackets (four corners built from spans, tone ink|paper|accent, arm length 16 or 32, aria-hidden, resting state drawn); Rule (hairline span, x|y, origin start|center|end, dashed variant).
3. Root: <MotionConfig reducedMotion="user"> around the app; <main id="main"> with a temporary type specimen (H1, H2, H3, lead, body, small, caption, kicker, price, a primary and secondary button, one of each Qualification variant, Brackets) so I can see the system rendering. Remove the specimen in Prompt 4a.

GUARDRAILS: no copy of your own except the specimen placeholders, which must be removed later; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, build; Preview at 1440 and 390: fonts load from the bundle (no external font request in the Network tab), the specimen shows every state, white-on-#D63619 buttons, zero console errors; compute and report the contrast of --text-2 and --text-3 on --paper-0 and --paper-1 and of the plate-dark --text-2 on --bg-dark; checkpoint "design-system".
```

---

## Prompt 2 — Content layer, typed helpers, validator

```
Work only in client/src/content/, scripts/ and package.json scripts. Fetch these three files byte-for-byte (if any fetch fails outright, stop and tell me; do not retry more than once) and verify each SHA-256, stopping if any differs:
- https://raw.githubusercontent.com/ahmadashfq/teamvista-replit-kit/9cdf5b99b972a10a73dccf06da41ae0655660b8c/content/site-content.json → client/src/content/site-content.json (SHA-256 56fb1d484312f70ae847b6df074ae02db00f4e072dcd25a2cf5d1d48600b0a2d)
- https://raw.githubusercontent.com/ahmadashfq/teamvista-replit-kit/9cdf5b99b972a10a73dccf06da41ae0655660b8c/content/COPY_VERIFICATION.json → client/src/content/COPY_VERIFICATION.json (SHA-256 d609b9985955eb93b32dbd9fa0900e4ca36d4a1078e62b32879cf1a9f0f3ca7a)
- https://raw.githubusercontent.com/ahmadashfq/teamvista-replit-kit/9cdf5b99b972a10a73dccf06da41ae0655660b8c/SPEC-VALIDATOR.md → SPEC-VALIDATOR.md at the project root (SHA-256 f6afc3aa0b7055f89de706fe351a4fe567c170fa5ee94f4e4cb238e6ee6c203f)

Shape: copy.sections[] (21 sections: P00–P18 plus P07a after P07 and P13a after P13), each with id, title and items[] of { id, text }; 276 items; IDs like "TV.RUI.P04.boundary". Some strings carry markdown: P10 and P07a encode tables as pipe rows in items table-head, table-rule and row-* (with inline [label](url) links in P10), and P10.source-details is prose with inline links. Strings contain em dashes and curly apostrophes that must render exactly.

Create:
1. scripts/generate-copy-ids.mjs writing client/src/content/copy-ids.generated.ts with literal union types CopyId (276 members) and SectionId (21). A plain typeof on a JSON import widens to string, so this codegen is required for misspelled IDs to fail tsc.
2. client/src/content/index.ts exporting: t(id: CopyId): string (throws in dev on unknown id); section(id: SectionId): { id, title, items, byId }; table(sectionId): { columns: string[]; align: ("left"|"right"|"center")[]; rows: { cells: RichText[] }[] } parsed from table-head, table-rule and every row-* item, where "---:" means right-align (colon after the dashes), ":---" left, ":---:" center; rich(id): RichText (segments { text } | { text, href } parsed from [label](url)); asset(id): the manifest entry plus a local src filled in by Prompt 3. asset(id).url is provenance only and must never be used as an image src.
3. client/src/content/validate.ts implementing SPEC-VALIDATOR.md exactly (file hash, section order and count, 276 unique non-empty items, the two receipt arrays with two different hash field names and the v1.1 skip list, the assets rule, the dev-only DOM coverage walk that excludes <script>, <style> and aria-hidden graphics and uses the per-type coverage rules, the FAQ JSON-LD equality check, the CTA rules). Expose it as npm script content:check (non-zero exit on any violation) and run it in the browser in development after render.
4. client/src/content/links.ts: export const CTA_HREF = "#evaluation" (placeholder I will replace) and the anchor ids: product→"product" (P03), how-review-works→"how-review-works" (P04), pricing→"pricing" (P10), storage-access→"storage-access" (P12), fit→"fit" (P13), readiness→"readiness" (P14), questions→"questions" (P16), evaluation→"evaluation" (P17). Map nav strings explicitly: P00.product→product, P00.workflow→how-review-works, P00.pricing→pricing, P00.storage→storage-access, P00.fit→fit, P00.readiness→readiness, P00.faq→questions, P00.evaluation→evaluation; footer P18.product, P18.workflow, P18.pricing, P18.storage, P18.fit, P18.readiness, P18.faq, P18.evaluation likewise.
5. package.json: "predev" and "prebuild" both equal "node scripts/generate-copy-ids.mjs && npm run content:check" (Prompt 3 will append the image step).

GUARDRAILS: no copy is typed by hand anywhere; everything renders through these helpers; no third-party scripts.

Verify: run content:check and paste its output (expect 276 items, receipts checked/skipped/missing counts, zero violations); typecheck passes and a deliberately misspelled t("TV.RUI.P01.nope") fails tsc (show the error, then remove it); t("TV.RUI.P01.cta") returns "Request a fit-and-technical evaluation"; table("P10").rows.length is 6 with the Hubstaff first cell carrying an href; table("P07a").rows.length is 4; checkpoint "content-layer".
```

---

## Prompt 3 — Assets, image pipeline, regions, Evidence component

```
Fetch these five files byte-for-byte (if any fetch fails outright, stop and tell me; do not retry more than once), verify each SHA-256 and the pixel sizes, and stop if any differ:
- https://raw.githubusercontent.com/ahmadashfq/teamvista-replit-kit/9cdf5b99b972a10a73dccf06da41ae0655660b8c/assets/teamvista-session-audit-2x.png → client/src/assets/brand/teamvista-session-audit-2x.png (2962×1782, SHA-256 0de7a2cd64cf3abde90b739fb0c59da188a5b4df6b22d7e3ac2342afee935c8a)
- https://raw.githubusercontent.com/ahmadashfq/teamvista-replit-kit/9cdf5b99b972a10a73dccf06da41ae0655660b8c/assets/teamvista-alerts-2x.png → client/src/assets/brand/teamvista-alerts-2x.png (3172×1980, SHA-256 dfe761fa94b1fb66f53e25fd163ba220f4aa84978effd55524b00cf7d84b61aa)
- https://raw.githubusercontent.com/ahmadashfq/teamvista-replit-kit/9cdf5b99b972a10a73dccf06da41ae0655660b8c/assets/teamvista-screenshots-directory-2x.png → client/src/assets/brand/teamvista-screenshots-directory-2x.png (3056×1728, SHA-256 7e29004c245959bbf1666fdc29e7aa9cf56f3810593b29e01165a5555b3b6ae5)
- https://raw.githubusercontent.com/ahmadashfq/teamvista-replit-kit/9cdf5b99b972a10a73dccf06da41ae0655660b8c/assets/teamvista-mark.png → client/src/assets/brand/teamvista-mark.png (512×512, SHA-256 bcc98898236f2b6297eb4a5a85c885546a5501850594f4a5fbbbab2bdcac0ab2)
- https://raw.githubusercontent.com/ahmadashfq/teamvista-replit-kit/9cdf5b99b972a10a73dccf06da41ae0655660b8c/SPEC-REGIONS.json → SPEC-REGIONS.json at the project root (SHA-256 56479e51ae518dbc2a734e3c2ea4c7b936653d7adf645e2165eabc16b34a8d9b) These are the only product images allowed: real-interface illustrations with fictional records, already cleaned and enhanced at 2×. Never alter their content or generate substitute UI.

Work only in client/src/assets/, client/src/content/regions.ts, client/src/components/media/, scripts/, client/public/ and package.json scripts.

1. scripts/images.mjs with sharp: for each master write AVIF (q60), WebP (q82) and PNG to client/public/evidence/ at widths 480, 800, 1200, the 1× native width (1481 / 1586 / 1528) and the 2× file width (2962 / 3172 / 3056), always downscaling from the 2× file (never upscale), plus a JSON manifest of intrinsic sizes. Add client/public/evidence/ to .gitignore. Change the scripts so "predev" and "prebuild" are "node scripts/generate-copy-ids.mjs && npm run content:check && npm run images"; run images once now so Preview works from Prompt 4.
2. client/src/content/regions.ts: a typed registry seeded from SPEC-REGIONS.json (x0, y0, x1, y1 in percent of the master). No other file may contain crop percentages.
3. <Picture>: <picture> with AVIF and WebP sources, PNG fallback, srcset, a required sizes prop, explicit width and height, loading and fetchpriority props, alt from the manifest, --bg-plate background while loading.
4. <Evidence asset region captionId brackets? callouts? badge?>: renders the master cropped to the named region with the crop mechanism in DESIGN.md (frame with overflow: clip and aspect-ratio = region pixel aspect; the full <img> sized 100/(x1−x0)% and translated by −x0/(x1−x0)% and −y0/(y1−y0)%, as inline custom properties; sizes describing the enlarged image width); a 12px radius, 1px --border hairline, the single shadow; Brackets 8px outside the corners when brackets is set; callouts = an array of region names from the same asset, each drawn as a bracket set positioned over that region; badge = a neutral bracket set over the asset's "badge" region; the caption t(captionId) directly beneath in the caption style, always. Write one helper that computes the crop values and unit-test it against the "full", "sidebar" and "heroDetail" regions (the last one starts away from 0 on both axes and catches a frame-relative versus image-relative mix-up). Add a mobileRegion prop; below 768px (below 1024px for the hero) the component uses it. Region "full" never uses a cropped mobile variant: on mobile it shows the whole image at a smaller size.
5. <Mark> from teamvista-mark.png; favicons: 32/180/512 PNGs with sharp and favicon.ico via png-to-ico, wired in index.html. Do not create any other logo or wordmark.

GUARDRAILS: no other images, no AI imagery, no third-party scripts, no publish or deploy.

Verify: run images and list the generated files; typecheck and build; a temporary test page showing heroDesk, heroDetail, timing, apps, review, content and directory regions at 1440 and 390 with each frame's computed scale (rendered img width ÷ 1× native width) printed beside it; confirm every scale is within DESIGN.md's bands and every caption sits under its frame; zero console errors; remove the test page; checkpoint "assets".
```

---

## Prompt 4a — P00 navigation

```
Build P00 in client/src/sections/Nav.tsx and mount it; remove the Prompt 1b specimen from <main>. Use DESIGN.md and the content helpers only.

P00 at ≥1280px: a 64px sticky header (position: sticky from first paint), paper, hairline bottom border, containing: <Mark> plus t("TV.RUI.P00.brand") in the display face; six anchor links in this order, P00.workflow, P00.pricing, P00.storage, P00.fit, P00.readiness, P00.faq, with hrefs from links.ts and a 1px accent underline on the active anchor driven by one IntersectionObserver; the primary Button t("TV.RUI.P00.cta") to CTA_HREF (nowrap). All on one line at 1280 and up. A 1px accent scroll-progress line along the header's bottom edge (transform scaleX, origin left; wired in Prompt 15). P00.product and P00.evaluation render in the drawer only.
P00 below 1280px: brand left; a button labelled t("TV.RUI.P00.menu-open") right that opens a full-height drawer listing all eight anchors (product, workflow, pricing, storage, fit, readiness, faq, evaluation) and the CTA; open-state label t("TV.RUI.P00.menu-close"); focus trapped, Escape and backdrop close it, focus returns to the toggle, body scroll locked. Below 768px the drawer's CTA link is the single CTA node for this breakpoint: while the drawer is closed and the hero's own CTA has scrolled out of view, reposition that same node into a fixed bottom bar (full width, 44px target, safe-area padding); hide it again whenever the drawer is open or any in-flow CTA is in the viewport. Never more than one nav CTA node in the DOM.

GUARDRAILS: exact strings via helpers; no extra items in the header; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check, build; Preview at 1440, 1280, 1024, 768 and 390: nav on one line at 1280 and 1440, drawer opens and closes by keyboard with focus returning to the toggle, body scroll locks while open, the active-anchor underline moves as you scroll (test after Prompt 4b), zero console errors; checkpoint "nav".
```

---

## Prompt 4b — P01 hero

```
Build P01 in client/src/sections/Hero.tsx and mount it after the nav. Use DESIGN.md and the content helpers only.

P01 desktop (≥1024): beat one, top padding 72px, no min-height. SectionHeading stacked: kicker P01.eyebrow, then the H1 P01.h1 across columns 1–10 at the full H1 token so it sets in exactly two balanced lines at 1440 and 1280 (each line wrapped in a <span>). Columns 1–7 below it: P01.lead (the 16-word lead), then the primary Button P01.cta to CTA_HREF beside the secondary link P01.secondary (scrolls to #how-review-works), then P01.support in the small style directly under the buttons. Then, still in beat one and above the fold at 1440×900 (at least 300px of real UI visible on load): the evidence panel, plate-dark, running from the container's left edge to the right viewport edge with 48px padding top and left and a square right edge, holding <Evidence asset="session-audit" region="heroDesk" captionId="TV.RUI.P01.caption" fetchpriority="high" eager> at ≈1.0× (never the sidebar), plus the static detail layer: a second <img> window over the heroDetail region (Session Duration and Active Work Time cards) at scale 1.06, translated (−3%, −5%), positioned so its brackets never overlap the screen header text, with a ::after shadow and accent Brackets at a 10px outset; the resting state is fully composed without JavaScript. Add <link rel="preload" as="image"> for the hero image. The caption sits under the frame on the panel.
Beat two, directly below: the offer strip between a 2px --ink-1 top rule and a hairline bottom rule, three columns each ≤ 46ch: P01.body (the full paragraph) then P01.price at --t-price in the display face in columns 1–4; P01.fit (weight 600) and P01.descriptor in columns 5–8; P01.accounts and P01.exclusions as neutral Qualification blocks in columns 9–12. One inseparable data-reveal-unit.
P01 mobile: kicker, H1 (static), lead, CTA, secondary link, support, then the panel edge to edge with square corners using region heroMobile (session header, profile card, Session Duration card) with its caption and brackets around the Session Duration card, then the offer strip stacked in the same order. Complete copy.

GUARDRAILS: exact strings via helpers; no extra taglines, badges, trust strips or stats in the hero; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check, build; Preview at 1440, 1280, 1024, 768 and 390: H1 two lines at 1440 and 1280 with no orphan, nav on one line at 1280 and 1440, drawer keyboard-operable with focus return, mobile CTA bar rule holds, at least 300px of the hero UI visible without scrolling at 1440×900, evidence scale within band, zero horizontal overflow, zero console errors; checkpoint "hero".
```

---

## Prompt 5 — P02 review problem and P03 product story

```
Build P02 in client/src/sections/Problem.tsx and P03 in Story.tsx (id="product"), mounted after the hero. Two different families: P02 is a ruled index; P03 is a typographic staircase.

P02 (paper, 96px): SectionHeading margin variant with P02.eyebrow and P02.h2; P02.body and P02.body-two in the lead style, 66ch, columns 3–10. Then three hairline-ruled bands, unnumbered (no 01/02/03): the label (attendance-label, alert-label, screenshot-label) in the display face at H3 size in columns 1–4 and the question (attendance-question, alert-question, screenshot-question) at the lead size in columns 5–10. Convergence graphic (aria-hidden, static now, animated in Prompt 15): from each band's rule, a short vertical elbow drops toward a single vertical spine in column 11 that runs down to P02.bridge. P02.bridge sits under the spine in the display face at H3 size in columns 5–11 with a neutral Qualification rule.

P03 (--bg-plate, 96px): SectionHeading margin variant with P03.eyebrow and P03.h2; P03.body in the measure. Then the staircase: an <ol> of the four steps P03.step-1, P03.step-2, P03.step-3, P03.step-4 in order, each an <li> in the display face at clamp(1.75rem, 1rem + 2.4vw, 3rem) weight 500, one per line, each line starting one step further right (columns 1, 3, 5, 7) with a hairline above it that spans from its start column to column 12; list markers hidden, no numbers, arrows or labels beyond the approved strings; on mobile the four lines stack flush left. Below the last step, P03.scope and P03.boundary stacked as neutral Qualification blocks in columns 7–12.

GUARDRAILS: helpers only, exact strings; no cards, no icons; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check, build; Preview at 1440, 768, 390: no horizontal overflow, the staircase reads as four steps, the bridge and both qualifications adjacent; zero console errors; checkpoint "problem-story".
```

---

## Prompt 6 — P04 session context (sticky viewfinder)

```
Build P04 in client/src/sections/SessionContext.tsx (id="how-review-works", class plate-dark, full-width --bg-dark, 96px), mounted after P03. This is the canonical Session Audit placement and the page's one sticky viewfinder chapter. CSS sticky only; no GSAP, no pinning.

Heading group across columns 1–8 (SectionHeading stacked: P04.eyebrow, H2 P04.h2 in two lines, P04.body in the measure). Below it a split: the left rail (columns 1–4) holds four chapter blocks in order, each a label in the display face at H3 size and its body: (timing-label, timing-body), (apps-label, apps-body), (breaks-label, breaks-body), (screenshots-label, screenshots-body); then P04.boundary as a neutral Qualification directly under the last chapter. Right (columns 5–12, extending to the right viewport edge): a viewfinder window, position: sticky; top: 96px; 560px tall; overflow: clip; 12px radius on its left corners; 1px --border hairline; containing the full session-audit master at scale clamp(0.75, windowWidth ÷ 940, 1) relative to 1× native (never above 1.0×, so text stays crisp), moved only by transform: translate(). Four stations from regions.ts, each a bracket target: timingTarget (heroDetail), appsTarget, breaksCard, screenshotsTab. For each station the translate centres the target in the window, clamped so the window never shows the app's top bar or cuts the "Session Audit Details" heading (the timing station aligns the content area's left edge with the window's left edge), and paper-tone Brackets (16px arms, 12px outset) frame the target. The active chapter is set from an IntersectionObserver (rootMargin "-45% 0px -55% 0px"); active label --text, others --text-3 at opacity 0.55, with a paper-tone indicator bar using layoutId="p04-active". Only under (min-width: 1024px) and (prefers-reduced-motion: no-preference) does JS add class pan-on, which gives each chapter block min-height: 65vh so there is scroll distance; the station tween itself is wired in Prompt 15 (for now, stations switch instantly). Without pan-on (reduced motion, no JS) the chapters stay compact and the window rests on the timing station with its brackets drawn, still sticky: complete, no dead zone. P04.caption sits directly under the window at all times, in the plate-dark caption colour. Never put overflow: hidden on an ancestor of the sticky window.

Below 1024px: no sticky. Kicker, H2, body, then for each chapter: label, body, then its own <Evidence> with the mobile region (mobileTiming, mobileApps, mobileBreaks, mobileScreenshotsTab) with brackets and the full caption P04.caption under each image; boundary last.

GUARDRAILS: helpers only, exact strings; caption and boundary never detach; crops only reveal what is in the master; no GSAP; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check, build; confirm gsap is absent from package.json; Preview at 1440 and 1280: the window stays sticky, each chapter switches the station with brackets on the Session Duration and Active Work Time cards, the activity log, the Non-Work Time card and the Screenshots tab, the caption stays under the window, the section releases with no layout shift; Preview at 1024, 768 and 390: static layout, legible crops within band, captions under every image; reduced-motion emulation: compact section resting on the timing station; zero console errors; checkpoint "session-context".
```

---

## Prompt 7 — P05 alerts and P06 screenshot directory

```
Build P05 in client/src/sections/Alerts.tsx and P06 in ScreenshotsDirectory.tsx, mounted after P04. Two families: a mirrored headline-over-split, then an evidence breakout.

P05 (paper, 96px): SectionHeading stacked in columns 5–12 (offset right): P05.eyebrow, H2 P05.h2. At ≥1280px, below the heading, the evidence on the LEFT runs from 32px off the left viewport edge to the end of column 9 (≈990px at 1440): <Evidence asset="alerts" region="review" captionId="TV.RUI.P05.caption" callouts={["allStatuses","severityColumn"]} badge> (the filter row Today / All Statuses / All Severities / All Employees / All Teams plus the full Security & Policy Violations Log, the exact things P05.body names; never the counters, never the full image in a half-width column). Columns 10–12: P05.body, then P05.prompt-1, P05.prompt-2 and P05.prompt-3 in order at body size weight 500 as three short lines, then P05.boundary as a neutral Qualification. From 1024 to 1279px the section stacks: heading, evidence at full container width, text in columns 1–8. Mobile: text first, then <Evidence region="mobileFilters"> with the caption, then <Evidence region="mobileLog"> with the caption, boundary last.

P06 (--bg-plate, 96px): the one centred heading on the page: P06.eyebrow and H2 P06.h2 centred over a 66ch measure, P06.body beneath. Then <Evidence asset="screenshots-directory" region="full" captionId="TV.RUI.P06.caption" callouts={["filters","directory"]} badge> as a breakout at width min(100vw − 64px, 1376px) at ≥1210px (frame, hairline and radius intact; never true edge-to-edge); from 768 to 1209px use region "content" at container width; under the caption, P06.image-scope as a second caption line. Below the image, two columns: P06.prompt (columns 2–6, lead style) and P06.boundary (columns 7–11, neutral Qualification). Mobile: heading, body, <Evidence region="mobile"> plus both caption lines, prompt, boundary.

GUARDRAILS: helpers only, exact strings; captions adjacent to every image instance; no cards; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check, build; Preview at 1440, 1280, 1024, 768, 390: report each Evidence scale (must be within band), no horizontal overflow at the breakout, the on-image "Illustrative demo data" pill visible in P06, captions never separate from images; zero console errors; checkpoint "alerts-directory".
```

---

## Prompt 8 — P07 rules and patterns, P07a record types, P08 human follow-up

```
Build P07 in client/src/sections/Rules.tsx, P07a in RecordTypes.tsx and P08 in FollowUp.tsx, mounted after P06 in that order.

P07 (paper, 96px): SectionHeading margin variant with P07.eyebrow, P07.h2; P07.intro in the measure. Three hairline-ruled bands, not cards: title (attendance-title / rules-title / patterns-title) in the display face at H3 size in columns 1–4; body (attendance-body / rules-body / patterns-body) in columns 5–8; the matching boundary (attendance-boundary / rules-boundary / patterns-boundary) as a neutral Qualification in columns 9–12. Each boundary stays in the same band as its body on every viewport (mobile stacks title, body, boundary). No image in P07.

P07a (--bg-plate, 64px): H2 P07a.h2 with the margin heading variant (no kicker exists; leave the kicker slot empty). Then table("P07a") as a real <table> on a --paper-0 surface with a 1px --border-strong outline and 8px radius: <th scope="col"> headers from the parsed columns; four rows in authored order with the first cell a <th scope="row">; cells at the small size; hairline row rules; on mobile a real table with the first column sticky and the other columns scrolling inside the table container only. No icons, ticks or highlights.

P08 (paper, 128px, a 2px --ink-1 rule across the container at the section top; left-aligned, never centred): SectionHeading stacked with P08.eyebrow and H2 P08.h2 (semantic <h2> at the H2 token, two lines max); P08.body in the measure; then P08.question in the display face at the pivot size, weight 500, across columns 1–10 (this line is the section's visual); then P08.boundary and P08.proof side by side on desktop as neutral Qualification blocks (the page's one side-by-side qualification pair, one data-reveal-unit); then P08.bridge; then the primary Button P08.cta to CTA_HREF, identical in size and colour to the hero button. Mobile: stacked in order.

GUARDRAILS: helpers only, exact strings; boundaries adjacent; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check, build; Preview at 1440, 768, 390: each P07 boundary shares a band with its body, the P07a table scrolls only inside its container on mobile, the P08 CTA matches the hero button; zero console errors; checkpoint "rules-records-followup".
```

---

## Prompt 9 — P09 the switching case

```
Build P09 in client/src/sections/Switching.tsx, mounted after P08, on --bg-plate with 96px padding. Family: a sticky heading beside a question stack (no grid, no cards, no icons).

Left (columns 1–5, position: sticky; top: 96px): SectionHeading stacked with P09.eyebrow and H2 P09.h2, then P09.body-lead, P09.body-q1, P09.body-q2 and P09.body-close in order (lead, then the two questions at the lead size, then the close). Right (columns 7–12): the four decision factors in authored order, (context-title, context-body), (followup-title, followup-body), (operating-title, operating-body), (cost-title, cost-body); each title as a 0.9375rem weight-600 label in --text-2, each body at the lead size in --text, 48px between factors, no rules, no boxes. Below the split: P09.best-fit at the lead size in columns 3–10 as a neutral Qualification (≤ 68ch; it names what is not established and must not be subordinate); then P09.decision in the display face at H3 size (it is a 40-word sentence; the pivot size is for one-line questions) in columns 3–10; then P09.fit-link as a secondary text link to #fit. Mobile: single column in authored order.

GUARDRAILS: helpers only, exact strings; no invented labels; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check, build; Preview at 1440, 768, 390; the sticky heading releases cleanly; zero console errors; checkpoint "switching".
```

---

## Prompt 10 — P10 dated list-price comparison

```
Build P10 in client/src/sections/Comparison.tsx (id="pricing"), mounted after P09, on --bg-plate with 64px padding (not dark: a dated, footnoted disclosure reads as a financial statement on paper). It is the page's most sensitive section: one block, no winner treatment.

SectionHeading margin variant with P10.eyebrow and P10.h2; P10.body; then P10.accounts and P10.exclusions together as one neutral Qualification block directly above the table (one data-reveal-unit with the table).
The table from table("P10"): a real <table> on a --paper-0 surface with a 1px --border-strong outline and 8px radius; a visible <caption> = P10.table-title above the table in the mono face (it carries the capture date); every header cell a <th scope="col"> in IBM Plex Sans 500 at 0.8125rem in --text-2; six <tbody> rows in authored order with the first cell a <th scope="row"> rendering its RichText (vendor links open in a new tab with rel="noopener", a visually-hidden " (opens in a new tab)" suffix and a 12px Phosphor ArrowSquareOut after the text; Teamvista plain); numeric columns right-aligned in IBM Plex Sans 500 with tabular-nums (not mono), cents exactly as authored; every row identical (no highlight, accent, badge, ticks, strike-throughs or "best" language); hairline row rules; sticky header. Where a note concerns a row (note-teamvista, note-storage, note-activtrak, note-teramind, note-insightful, note-hubstaff, note-timedoctor), add a small mono superscript index after the relevant first cell as an <a href="#note-N"> to that note's id (layout only; no copy reordered). Mobile (<768px): still a real table; first column sticky; the numeric columns scroll horizontally inside the table container only, with a subtle non-gradient edge cue; never cards or a carousel.
Under the table: P10.formula-monthly, P10.formula-annual, P10.formula-teramind and P10.formula-assumption as four short lines in the small style; then P10.notes-heading as an H3; then the ELEVEN notes (note-date, note-parity, note-costs, note-teamvista, note-storage, note-activtrak, note-teramind, note-insightful, note-hubstaff, note-timedoctor, note-decision) as an ordered list in authored order, complete, each with id="note-N", in the small style (--text-2) with a hanging mono index, set in two CSS columns at ≥1024px (≤ 64ch each, break-inside: avoid) and one column below; then P10.source-details rendered with rich() so its links are live (same new-tab treatment). P10.table-rule is layout data and is never rendered as text.

GUARDRAILS: helpers only, exact strings and figures; no reordering, summarising or savings language; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check, build; Preview at 1440 and 390: six rows and all eleven notes visible, superscripts jump to their notes, the mobile table scrolls only inside its container, the caption date visible at both widths; zero console errors; checkpoint "comparison".
```

---

## Prompt 11 — P11 proposed pricing and P12 storage and access

```
Build P11 in client/src/sections/Pricing.tsx and P12 in StorageAccess.tsx (id="storage-access"), mounted after P10.

P11 (paper, 96px): an offer statement, not a pricing card. Left (columns 1–5, position: sticky; top: 96px): SectionHeading stacked with P11.eyebrow, H2 P11.h2 (wrap "per-seat" in a nowrap span), P11.body. Right (columns 7–12): one offer sheet (no fill, no radius; a 2px --ink-1 top rule, hairline rules between rows, hairline bottom rule) containing in order and inseparable: P11.price at --t-price (display face, no mono); P11.accounts; P11.exclusions as a neutral Qualification; P11.scope; P11.mcp-scope as a neutral Qualification. One data-reveal-unit. Below the split, a full-width band: P11.cost-heading as an H3, then P11.cost-include, P11.cost-ai and P11.cost-writing as three lines in the measure. Then the primary Button P11.cta to CTA_HREF (same component and weight) with P11.support in the small style directly beneath. Mobile: heading, sheet intact, cost band, CTA and support.

P12 (--bg-plate, 96px): SectionHeading margin variant with P12.eyebrow and H2 P12.h2 (wrap "customer-controlled" in a nowrap span); P12.status immediately under the H2 as a mono status line with a neutral Qualification rule (prominent: it says proposed, not verified); then P12.body-model, P12.body-control and P12.bridge in the measure. Then three hairline-ruled bands, unnumbered: (storage-title, storage-body), (access-title, access-body), (retention-title, retention-body with retention-boundary directly beneath as a neutral Qualification inside the same band). Then the Gate: P12.authorization rendered with the Qualification "gate" variant (the one dark object in the section), followed immediately by P12.authorization-status as a small line with a text link to #readiness. Boundary schematic (aria-hidden, static now): beside the bands at ≥1024px in columns 9–12, a dashed rectangle (Rule dashed variant) representing the customer boundary with four nodes as Phosphor icons and bare two-digit indices only: 01 HardDrives and 03 ClockCounterClockwise inside the boundary, 02 Key and 04 Warning straddling its right edge, the 04 node framed by accent Brackets. No text in the graphic.

GUARDRAILS: helpers only, exact strings; the price never separates from its qualifications; status and the Gate stay prominent on mobile; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check, build; Preview at 1440, 768, 390: the sheet intact at 390, the Gate is the strongest object in P12, the schematic hides on mobile without leaving a gap; zero console errors; checkpoint "pricing-storage".
```

---

## Prompt 12 — P13 fit and no-fit, P13a what to bring, P14 readiness

```
Build P13 in client/src/sections/Fit.tsx (id="fit"), P13a in Bring.tsx and P14 in Readiness.tsx (id="readiness"), mounted after P12.

P13 (paper, 96px, framed by a 1px --border-strong rule top and bottom so it reads with the same confidence as the product chapters): SectionHeading margin variant with P13.eyebrow and P13.h2; P13.body in the measure. Two equal columns, each with a 1px --border-strong left rule: left headed by P13.fit-title (H3) with fit-one, fit-two, fit-three; right headed by P13.no-fit-title with no-fit-one, no-fit-two, no-fit-three; items at body size separated by hairlines, no icons, no tints; pair n in both columns shares one data-reveal-unit. Both lists fully visible stacked on mobile. Beneath, P13.boundary as the Qualification "strong" variant (lead size, 2px --ink-1 rule, max 60ch). Then the role row: five columns at ≥1280px separated by vertical 1px --border rules, each with the role title (owners-title, managers-title, hr-title, it-title, finance-title) in the display face at H3 size over its body (owners-body, managers-body, hr-body, it-body, finance-body) at 0.9375rem; two columns from 768 to 1279px; stacked below. Each role stays with its guidance in the same column on every viewport.

P13a (--bg-plate, 64px): H2 P13a.h2 with the margin heading variant (no kicker); P13a.intro in the lead style; then a checklist of three hairline-ruled rows, P13a.item-question, P13a.item-requirements, P13a.item-cost, each with a Phosphor CheckSquare icon (regular, --text-2) before the text at body size. No numbers, no cards.

P14 (--bg-plate, 96px): SectionHeading margin variant with P14.eyebrow and P14.h2; P14.body. Bento with exactly four cells: the access Gate spans columns 1–7 and rows one and two (Qualification "gate" panel) with access-heading and access-body, then P14.controls and P14.tests below a hairline at 0.9375rem; Windows in columns 8–12 row one (windows-heading, windows-body, windows-detail beneath as a neutral Qualification, one unit); workforce in columns 8–12 row two (workforce-heading, workforce-body); operations across columns 1–12 row three with a 2px --ink-1 top rule, no fill, text ≤ 68ch (operations-heading, operations-body). Windows and workforce cells on --paper-0 with a hairline. No row of three equal cells, no empty cell. Mobile: cells stack in authored order.

GUARDRAILS: helpers only, exact strings; the Windows detail stays adjacent; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check, build; Preview at 1440, 1280, 1024, 768, 390: no empty bento cell, both fit lists fully visible at 390, the role row reflows without orphans; zero console errors; checkpoint "fit-bring-readiness".
```

---

## Prompt 13 — P15 AI/MCP maturity and P16 FAQ

```
Build P15 in client/src/sections/AiMcp.tsx and P16 in Faq.tsx (id="questions"), mounted after P14.

P15 (paper, 96px, framed by a 1px --border-strong rule top and bottom so it reads as separate from the core offer): SectionHeading margin variant with P15.eyebrow and P15.h2; P15.body. Two sub-sections divided by a hairline, never merged:
(a) P15.mcp-label as the sub-section H3 (no chip or pill), P15.mcp-body; then P15.tools-title as an H4-styled label and the six tools (tool-1 to tool-6) as a mono-face list on a --bg-plate band; then P15.tools-boundary and P15.read-only-boundary as two neutral Qualification blocks directly under the list; then P15.requirements-heading (H3), P15.requirements-body, and P15.scope as a neutral Qualification.
(b) P15.assistant-label as the sub-section H3, P15.assistant-body, then P15.request-boundary as a neutral Qualification.
No "AI-ready", "included" or capability language may be added; only the approved strings.

P16 (--bg-plate, 64px): SectionHeading stacked in a sticky left column (columns 1–4, top: 96px) with P16.eyebrow and H2 P16.h2; in columns 5–12 the ten question/answer pairs (faq-01-q/faq-01-a through faq-10-q/faq-10-a) as native <details>/<summary> elements in authored order, each independently expandable, summary in IBM Plex Sans 600 at 1.125rem with a Phosphor CaretDown rotating 180° in 150ms, answers always in the DOM, hairline rules between items, 44px minimum hit area, keyboard operable, focus-visible ring. Add a FAQPage JSON-LD <script> whose question and answer texts come from the helpers (exact strings). Mobile: heading, then the list full width.

GUARDRAILS: helpers only, exact strings; the two P15 sub-sections stay separate; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check (it now asserts the JSON-LD equals the approved strings), build; Preview at 1440 and 390: open three FAQ items at once and confirm all stay open; tab through by keyboard; zero console errors; checkpoint "ai-faq".
```

---

## Prompt 14 — P17 closing invitation, P18 footer, document head

```
Build P17 in client/src/sections/Closing.tsx (id="evaluation", class plate-dark, full-width --bg-dark, 128px, left-aligned, never centred) and P18 in Footer.tsx, mounted last.

P17: four paper-tone viewfinder Brackets (32px arms) frame the content box at a 48px outset: the page opens with brackets on real UI and closes with brackets around the buyer's question. SectionHeading stacked with P17.eyebrow, then H2 P17.h2 as a semantic <h2> at the H1 type token in columns 1–9, two lines (each line in a <span>). Below, a split mirroring the hero: columns 1–6 hold P17.body in the lead style and then the primary Button P17.cta (same component and weight) to CTA_HREF; columns 8–12 hold P17.preparation and P17.support stacked as neutral Qualification blocks (one unit), top-aligned with the body. Mobile: body, preparation, support, then the CTA. Nothing below the CTA row.

P18 (paper, 48px, hairline top): left, <Mark> and P18.brand in the display face with P18.descriptor beneath in the small style; right, the complete index in this order: P18.product, P18.workflow, P18.pricing, P18.storage, P18.fit, P18.readiness, P18.faq, P18.evaluation, to the anchors from links.ts. Nothing else: no legal line, no social icons, no copyright, no version label.

Head: <title>Teamvista</title>, meta description = t("TV.RUI.P18.descriptor"), theme-color #F9FAFC, the favicons from Prompt 3, and an Open Graph image: render a standalone 1200×630 HTML page (same self-hosted fonts) showing the mark and P18.brand on --paper-0 and screenshot it with headless Chromium at build time into client/public/og.png (do not use sharp's SVG text path; it substitutes fonts). No external resources of any kind in the head.

GUARDRAILS: helpers only, exact strings; no invented footer copy; no third-party scripts; no publish or deploy.

Verify: typecheck, lint, content:check, build; Preview at 1440 and 390: every nav, drawer and footer anchor lands with the heading fully visible under the sticky header; a link check over all in-page hrefs; zoom into og.png and confirm the letterforms are Instrument Sans; zero console errors; checkpoint "closing-footer".
```

---

## Prompt 15 — Motion and graphics system

```
Fetch https://raw.githubusercontent.com/ahmadashfq/teamvista-replit-kit/9cdf5b99b972a10a73dccf06da41ae0655660b8c/SPEC-MOTION.md and save it byte-for-byte as SPEC-MOTION.md at the project root (if the fetch fails outright, stop and tell me; do not retry more than once); its SHA-256 must be 19d9a1b8292bc1185399e31d25b163b757dd2dcfc5b3471db0d2ed0436af1219, verify and stop if it differs. Then read it and implement it completely in one pass. Files you may touch: the section files, client/src/components/motion/, client/src/components/media/Evidence.tsx, client/src/content/regions.ts, the global CSS, client/src/main.tsx, client/src/entry-server.tsx, scripts/prerender.mjs, index.html and package.json scripts. Nothing else.

Order of work: (0) confirm the foundations in SPEC-MOTION.md exist (prerender producing the H1 and FAQ-10 strings in dist/index.html; the head script with js, motion-ok, app-ready and reveal-failsafe; Brackets and Rule components); build any that are missing before animating. (1) The reveal engine (one IntersectionObserver, class-driven, data-reveal / data-reveal-unit / data-reveal-seq) and delete any earlier Reveal that used whileInView or initial. (2) Apply the family vocabulary table to every section exactly, including the units that keep a claim with its qualification. (3) The hero choreography (CSS keyframes from first paint, two-line H1 stagger on desktop only, the detail layer lift and brackets, the first-scroll parallax in pixels, the offer strip as one unit). (4) The P04 sticky viewfinder tweens with img.decode(), interruptible retargeting, the layoutId indicator, and the reduced-motion crossfade. (5) The P06 dolly. (6) Chrome: nav layoutId underline, the scroll-progress line as scaleX, button states, P10 hover tint. (7) The budget: remove every transition-all, cap simultaneous animations at 6, scope will-change.

GUARDRAILS: no new copy, no new images, no GSAP or Lenis, no loops or counters or cursor effects, no third-party scripts; graphics contain no text except bare two-digit indices; content:check must stay clean; no publish or deploy.

Verify: run every step in the "Verification" list at the end of SPEC-MOTION.md and paste the evidence for each (build greps, the animation greps, the hero and P04 descriptions at 1440 and 390, reduced-motion, JavaScript disabled, the failsafe, Lighthouse or its fallback with the LCP element named, zero console and hydration warnings); checkpoint "motion".
```

---

## Prompt 16 — Full QA pass and anti-slop gate

```
Run a complete QA pass and fix what you find, in at most three fix-and-recheck loops; if something still fails after three, stop and report it.

A. Widths 1440, 1280, 1024, 768, 390, 320. At each: nav on one line (≥1280) or drawer works; exactly one <h1>, then <h2> per section, <h3> inside; hero CTA visible without scrolling; every Evidence placement's scale (rendered img width ÷ 1× native width) reported and within DESIGN.md's bands; every caption and qualification adjacent to its image or claim; the P10 table with all six rows and eleven notes and the P07a table complete; document.documentElement.scrollWidth equals clientWidth (no horizontal overflow); FAQ items open independently; the mobile CTA bar appears only when no in-flow CTA is visible.
B. Keyboard and accessibility: tab through the page and list the focus order; the skip link works; every interactive element shows a focus ring; the drawer traps focus and returns it; landmarks header, nav, main, footer; all images have manifest alt text; every <table> has a caption or heading, th scope="col" and scope="row"; targets ≥ 44px for buttons, bar, drawer links and FAQ summaries; run axe-core (npm install --no-save axe-core if needed) against the preview and report; compute contrast for every text/background token pair in use and report any under 4.5:1 (text) or 3:1 (UI).
C. Reduced motion, JavaScript disabled (built output) and the failsafe, as in SPEC-MOTION.md.
D. Content: content:check output; the CTA string appears exactly five times at 1440 and five at 390 (drawer/bar node is the nav instance); no "TV.RUI." visible; if CTA_HREF still equals "#evaluation", report FAIL and list it first as the single blocking item.
E. Anti-slop gate, mechanical; report counts and fix any hit: "transition-all"; "backdrop-blur"; "bg-gradient" or "linear-gradient" or "radial-gradient" outside the screenshots; "rounded-2xl"; "lucide"; a font-family value containing Inter as a face (not the string IntersectionObserver); Tailwind "uppercase" together with "tracking-" in one className (expected 0); emoji characters; "Trusted by" or "Testimonial"; hex literals in client/src not in the DESIGN.md token list; "<path" under client/src/sections and client/src/components excluding components/media and Phosphor; elements with an --accent left or top rule (allowed: the two Gates only); sibling groups of three or more equal-width boxes (allowed: 0); full-width plate-dark sections (must be exactly 2: P04 and P17); any text block wider than 72ch. Report the layout family per section and confirm no family repeats consecutively.
F. Network and console: zero errors and warnings on load and after scrolling to the end; no request leaves the app origin; zero 404s.
G. Build: typecheck, lint, the full test script, content:check, build and the prerender; paste the outputs.

GUARDRAILS: fixes must not change any visible string; no publish or deploy.

Verify: every check above has a pass/fail entry with evidence. Report as a table of checks with pass/fail and what you changed; checkpoint "qa-pass".
```

---

## Prompt 17 — Completion report

```
Do not change any file. Write COMPLETION_REPORT.md at the project root with: (1) the design thesis in three sentences and the layout family per section P00–P18 plus P07a and P13a; (2) every file added or changed since the "pre-redesign" checkpoint; (3) the image pipeline: sources and hashes verified in Prompt 3, derivatives generated, and a note that logo replacement, cropping and the 2× enhancement were done upstream and are documented in the client's generated-assets/TRANSFORMATIONS.md; (4) the content:check output with the coverage line (274 visible + 2 structural = 276) and where each CTA instance is; (5) every command from Prompt 16 with its result and the checkpoint it ran against; (6) how to view the preview at desktop and mobile widths, noting the *.replit.dev URL is not access-controlled; (7) accessibility, reduced-motion, no-JS, responsive, console and overflow results; (8) open items for the owner: CTA_HREF destination; the seven pending items awaiting owner facts (four "what happens next" lines, the access status line, the P13a people item and the P16a evaluation-steps section); any string that did not fit a layout and how layout solved it; (9) a clear statement that nothing was published or deployed. End with the current checkpoint name.
```

---

## Prompt 18 (later, optional) — Add the owner-fact strings

Use only after Ahmad has replaced every `[OWNER: …]` placeholder in `content-proposal/pending-owner-facts.v1.2.json` and Alan has approved the wording. A new `site-content.json` (v1.2) and hash must be produced first; then:

```
I have uploaded a new client/src/content/site-content.json (v1.2, SHA-256 <paste>). Update the hash and counts in validate.ts and SPEC-VALIDATOR.md. Render the new strings where the file places them: P01.next, P08.next, P11.next and P17.next as the second line of each CTA's small support block (one unit with the CTA); P14.access-status directly under access-heading inside the Gate; P13a.item-people as the fourth checklist row; section P16a (h2, four steps as an ordered staircase in the P03 style, boundary as a neutral Qualification) between P16 and P17 with id "evaluation-steps", and point the nav and footer "Evaluation" anchors at it. Change nothing else. Verify: content:check, typecheck, build, Preview at 1440 and 390; checkpoint "owner-facts".
```

---

## After the pack

- To set the CTA destination: "Set CTA_HREF in client/src/content/links.ts to <value>; change nothing else; verify all five CTAs and checkpoint." Recommended compliant destination: a monitored `mailto:` with a prefilled subject and the address shown as visible text beside the button; a calendar embed remains out of scope unless the owner lifts that constraint.
- Nothing here authorizes Publish or Deploy. For external review use Replit's Private Publishing (Only you or Workspace only), not the raw preview link.
