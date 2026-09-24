# SPEC-MOTION.md — the motion and graphics system

Replit Agent reads this in the motion prompt. Everything here is motivated (hierarchy, storytelling, feedback). The page is complete with JavaScript disabled and under `prefers-reduced-motion`.

## Hard rules

- Only `transform` and `opacity` animate. No width, height, top, left, margin, clip-path, stroke-dashoffset, filter, box-shadow (animate the opacity of a `::after` that carries the shadow) or background-position.
- Hidden pre-states exist only in CSS under `html.js.motion-ok:not(.reveal-failsafe)` and are removed by adding `.is-in`. Motion's `initial` prop never hides an element that contains text or an image.
- A claim and its qualification, caption, exclusion or no-fit counterpart reveal in the same frame (`data-reveal-unit`). Captions and the on-image disclosure never animate.
- Scroll-linked values come from `motion/react` `useScroll`; triggers come from one shared IntersectionObserver. No window scroll listeners.
- Never transform text columns or buttons. Never `overflow: hidden` on an ancestor of a sticky element (use `overflow-x: clip` on bleed wrappers).
- Every scroll-linked value returns 0 under `useReducedMotion()`; every CSS animation lives inside `@media (prefers-reduced-motion: no-preference)`; `<MotionConfig reducedMotion="user">` stays at the root as a backstop.
- No loops, pulsing, counters, cursor effects, magnetic buttons, glow, tilt, blur-in, typewriter, smooth-scroll library or GSAP.
- Budget: at most 6 elements animating at once in the viewport; `will-change` only on the hero layers during their animation and on the P04 image during a tween; nothing runs longer than 1200ms except scroll-linked values; every `transition-all` removed.

## Foundations

1. **Prerender.** `npm run build` produces `dist/index.html` containing the exact text of `t("TV.RUI.P01.h1")` and `t("TV.RUI.P16.faq-10-q")`: `client/src/entry-server.tsx` renders `<App/>` with `react-dom/server`, `scripts/prerender.mjs` injects it into `#root` after `vite build`, `main.tsx` uses `hydrateRoot`. Wire as `postbuild`.
2. **Head script.** In `index.html` `<head>`, before any stylesheet, an inline script adds `js` to `<html>`, adds `motion-ok` when `matchMedia('(prefers-reduced-motion: no-preference)').matches`, and after 3000ms adds `reveal-failsafe` unless `<html>` has `app-ready` (added after hydration). CSS: `html.reveal-failsafe [data-reveal], html.reveal-failsafe [data-reveal-unit] { opacity: 1 !important; transform: none !important }`.
3. **Reveal engine** (`client/src/components/motion/reveal.ts` + `reveal.css`): one IntersectionObserver (`rootMargin "0px 0px -12% 0px"`, threshold 0) adds `.is-in` once and unobserves. Opt-in via `data-reveal` (single element), `data-reveal-unit` (claim plus qualification, always one element) or `data-reveal-seq="<family>"` (children with `data-step` run in order via `--i` and `transition-delay: calc(var(--i) * step)`). Transition: opacity and transform only, 350ms `--ease-out`, y 12px unless the table says otherwise.
4. **Brackets** (`motion/Brackets.tsx`): four corners, each two `<span>` arms 2px thick and 16px (or 32px) long, tone `ink | paper | accent`, `aria-hidden`, corners positioned by `transform: translate`, arm `transform-origin` at the corner point; "draw" = each arm `scaleX`/`scaleY` 0→1 from the corner, 190–220ms, corners 30ms apart. Resting state fully drawn.
5. **Rule** (`motion/Rule.tsx`): a hairline `<span>`, orientation x|y, origin start|center|end, `scaleX`/`scaleY` draw; dashed variant = an inner strip translating inside an `overflow: clip` wrapper.
6. **Graphics contain no text** except approved strings or bare two-digit indices as separate `<span>` nodes; graphics are `aria-hidden`.

## Hero (P01) choreography, desktop ≥1024, pure CSS keyframes from first paint

Composition: the evidence panel (plate-dark, container left edge to viewport right edge) holds layer B = `heroDesk` region at ≈1.0× (the LCP image, opacity 1 at all times) inside the 12px frame with the single shadow; layer D = `heroDetail` (Session Duration and Active Work Time cards) as an `overflow: clip` window registered exactly over that region of B, holding a second `<img>` with the identical `src`/`srcset`/`sizes` (same cache entry). Accent brackets around D at a 10px outset. P01.caption under B, static.

| t (ms) | Element | Motion | Duration / easing |
|---|---|---|---|
| 0 | Nav, plate, caption, offer strip | none | |
| 0 | B | from `translateY(24px) scale(0.98)` (origin 50% 100%) to none; opacity stays 1 | 700ms `--ease-out` |
| 60 | Kicker | opacity 0→1, y 6→0 | 320ms `--ease-out` |
| 120 / 260 | H1 line 1 / line 2 (two `<span>` lines, never per-word) | opacity 0→1, y 10→0 | 420ms `--ease-out` |
| 420 | Lead | opacity 0→1, y 8→0 | 360ms |
| 540 | CTA + secondary + support as one group | opacity 0→1, y 8→0 | 360ms |
| 620 | D | opacity 0→1 at scale 1, registered | 80ms linear |
| 700 | D | to `translate(-3%, -5%) scale(1.06)`, `::after` shadow opacity 0→1 | 440ms `--ease-in-out` |
| 920→1200 | D brackets | arms draw, corners TL, TR, BR, BL 30ms apart | 190ms each |

Resting state (also reduced-motion, no-JS and first paint): B flat, D lifted at scale 1.06 with shadow, brackets drawn, all text visible. D at 1.06× of a 1.0× base is 0.53× of the 2× master file, so it stays crisp and inside the legibility band.

First scroll (desktop, motion-ok; `useScroll` target = beat one only, offset `["start start", "end start"]`, clamped): plate y 0→64px inside an `overflow: clip` wrapper sized to beat one and oversized upward (`top: -64px`); B with its caption y 0→32px; D y 0→−24px and scale 1.06→1.09; D brackets 6px inward. Text column: no transform. Nav progress line: opacity 0→1 when the hero bottom passes the header; `scaleX` = page `scrollYProgress`, origin left, never `width`.

Offer strip (beat two): its 2px ink top rule draws `scaleX` 0→1 from the left over 500ms; at +120ms the whole strip (P01.body, price, fit, descriptor, accounts, exclusions) reveals as one unit. The price is never visible without its qualifications.

Mobile (<1024): single layer B = `heroMobile`, settles from `translateY(16px)` over 600ms; brackets around the Session Duration card draw at 450–700ms; the H1 is static (it may be the LCP element); no D layer, no parallax.

## Family vocabulary (use exactly this; nothing generic)

| Family / section | Sequence | Step / duration |
|---|---|---|
| Offer strip P01, offer sheet P11 | top rule scaleX 500ms, then the whole group as ONE unit | 350ms, y 12, +120ms |
| Ruled index / bands (P02, P07, P12) | band rule scaleX from start 400ms, then band text | 60ms between bands |
| P02 convergence graphic | three row rules → two elbows scaleY 200ms → central spine scaleY 300ms → bridge rule scaleY + bridge text 250ms | ≈1100ms total |
| P03 staircase | each line's hairline draws scaleX 300ms, then the sentence y 8→0 300ms, top to bottom | 120ms between lines |
| P04 sticky viewfinder | see below | |
| P05 / P06 evidence | frame y 16→0 + opacity 450ms; station 1 brackets at +300ms; station 2 at +700ms; badge bracket at +1000ms; P05.prompt reveals with station 2 | arms 220ms, corners 30ms |
| P06 dolly | inner `<img>` scale 1.06→1 (origin 50% 50%) as the section goes from entering to 40% in view (`useScroll`, clamped); frame, caption and image-scope never move; brackets draw after the dolly settles | zero under reduced motion |
| P07a, P10 tables | qualification + caption + table as ONE unit, then the thead rule scaleX 400ms; never per row | |
| P08 pivot, P17 close | heading block, then pivot/body +150ms, then both qualification blocks as ONE unit, then CTA | 350ms each |
| P09 question stack | heading, then factors top to bottom | 70ms step |
| P12 schematic | boundary sides clockwise 4 × 220ms (dashed strip translate); nodes 01 and 03, then 02, then 04 with its bracket | 60ms step, scale 0.92→1 |
| P13 fit / no-fit | the two column rules 450ms; list pairs 1, 2, 3 revealed together across both columns; the no-fit item never lags its fit counterpart | 70ms between pairs |
| P13a checklist | items top to bottom | 60ms |
| P14 bento | cells in reading order; Windows body + detail as one unit; the Gate's accent top rule scaleX after its cell | 50ms step |
| P15 | block reveals only; the two sub-section headings never animate specially | 350ms |
| P16 FAQ | caret rotate 150ms; answer opacity + y 6 over 200ms; height never animates | CSS only |
| All H2 heading groups | whole block, no word split | 350ms |
| P17 | brackets start 24px further out and travel in over 600ms `--ease-in-out`; H2 lines stagger 120ms (the only heading besides the H1 with a line split) | |

## P04 sticky viewfinder (≥1024 and motion-ok)

Active chapter from an IntersectionObserver with `rootMargin "-45% 0px -55% 0px"`. On change: `await img.decode()`, then animate the master image `transform` (translate only, constant zoom) over 550ms `--ease-in-out` so the station's target region is centred in the window, clamped to the master's edges; brackets travel to the target region 60ms later over 500ms; `will-change: transform` only during the tween; tweens retarget from the current value (interruptible). The active chapter label is `--text`, inactive chapters `--text-3` at opacity 0.55, with a paper-tone indicator bar using `layoutId="p04-active"` (300ms). The dark plate scales X 0.96→1 as the section top travels the last 30% of the viewport. Reduced motion: the rail stays sticky, crops change by a 150ms opacity crossfade between two pre-positioned images, brackets jump, no plate scale, no dimming. Below 1024: no sticky; each inline crop's brackets draw on enter.

## Chrome

Nav active underline uses `layoutId` (≤200ms). Buttons: background-color 150ms, `:active scale(0.97)` 120ms, focus-visible ring never removed; no lift, glow or magnetic pull. P10 rows: row and column `::after` tint on hover, opacity 120ms.

## Verification (paste the evidence)

1. typecheck, lint, content:check, build; grep the built `dist/index.html` for the H1 and FAQ-10 strings.
2. grep: zero hits for `transition-all`, `addEventListener('scroll'`, `clip-path` inside any transition/animation, `stroke-dashoffset`, `initial={{ opacity: 0` on text or image elements, `gsap`, `lenis`; no `width`/`height` inside Motion animate objects.
3. Preview 1440 and 390 with motion on: describe the hero sequence, confirm the hero image is visible at first paint, step through the four P04 stations and confirm the brackets frame the Session Duration and Active Work Time cards, the activity log, the Non-Work Time card and the Screenshots tab; confirm P05 stations frame the filter row and the severity column and P06 the filters and directory, never the counters.
4. Reduced-motion emulation: nothing hidden, nothing moves with scroll, P04 crops crossfade only, hero flat and complete.
5. JavaScript disabled: every section and string visible; P04 rests on the timing station.
6. Block the main bundle in the Network panel: everything visible within 3s (failsafe).
7. Lighthouse if available, else `npx lighthouse http://localhost:<port> --output=json --quiet --chrome-flags='--headless'`; report LCP (target < 2.0s desktop) and which element is LCP, CLS (< 0.05), TBT (< 150ms). If neither runs, confirm manually: hero image has `fetchpriority="high"` and a `<link rel="preload" as="image">`, is never opacity-gated, and no synchronous script blocks first paint.
8. Zero console errors, warnings or hydration warnings.
