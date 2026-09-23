# SPEC-VALIDATOR.md — content:check rules

`content:check` runs in development at app start (`import.meta.env.DEV`) and as an npm script before dev and build. It fails with a non-zero exit on any violation. `client/src/content/site-content.json` is never modified by any script.

## 1. File identity

- SHA-256 of `client/src/content/site-content.json` equals `56fb1d484312f70ae847b6df074ae02db00f4e072dcd25a2cf5d1d48600b0a2d`.
- `copy.sections` has exactly 21 sections in this order: P00, P01, P02, P03, P04, P05, P06, P07, P07a, P08, P09, P10, P11, P12, P13, P13a, P14, P15, P16, P17, P18.
- Exactly 276 items, every `id` unique, no empty `text`, no text containing `[OWNER`.

## 2. Receipts (optional, when `client/src/content/COPY_VERIFICATION.json` exists)

The receipt file describes the 1.0 base. Walk `protected_reuses` (96 objects with `id` and `selected_text_sha256`) and separately the single object in `additional_exact_reuses` (`id` and `source_text_sha256`). For every receipt whose `id` still exists in site-content.json and is NOT in the v1.1 change list below, sha256(utf8(text)) must equal the receipt hash. Receipts for IDs that were edited or split in v1.1 are skipped and listed in the report, never treated as failures. Report counts: checked / skipped / missing.

v1.1 change list (skip receipts): TV.RUI.P16.faq-10-q, TV.RUI.P01.fit, TV.RUI.P16.faq-06-a, TV.RUI.P03.scope, TV.RUI.P03.sequence, TV.RUI.P05.prompt, TV.RUI.P09.body, TV.RUI.P10.formula, TV.RUI.P11.cost-body, TV.RUI.P12.body.

## 3. Assets

`assets[].sha256` in the JSON describes the pre-cleanup originals, not the files in the workspace. Do not compare them to on-disk files. Check only that each entry has a 64-hex `sha256`, a `caption_id` that resolves to a real copy ID (where present) and an `alt`. On-disk image identity is verified once in the asset prompt. `asset(id).url` is provenance metadata, never a valid `src`; a dev assertion fails if any `<img>` or `<source>` references it.

## 4. Coverage (dev only, in the browser after render)

Walk visible text nodes inside `header`, `main`, `footer`, excluding `<script>` and `<style>` subtrees and `aria-hidden` decorative graphics. Normalise whitespace. Rules per item type:

- Plain items: the exact `text` must appear verbatim somewhere in the DOM.
- `table-head` and every `row-*` item (P10 and P07a): split the raw pipe row on unescaped `|`, trim each cell, reduce `[label](url)` to `label`, and require each cell's plain text verbatim. The raw pipe string never appears.
- `rich()` items (P10.source-details): reduce links to labels and require the plain text verbatim.
- `table-rule` items (P10, P07a) are consumed structurally; assert `table()` derived the right number of alignment values (5 for P10, 3 for P07a) and exclude them from the coverage count.
- Nav strings may render per viewport (header at ≥1280, drawer below). Coverage for P00 is satisfied if each P00 string appears in the header OR the drawer DOM.

Coverage report: "274 rendered as visible text + 2 table-rule items consumed structurally = 276/276 accounted for". Any visible text node longer than 3 characters that is not a substring of an approved text (numbers, punctuation, single glyphs and bare two-digit indices exempt) is logged as an error: it is invented copy.

## 5. FAQ JSON-LD

Parse the rendered `<script type="application/ld+json">`. For each of the 10 `mainEntity` entries, `question.name` equals `t("TV.RUI.P16.faq-0N-q")` and `acceptedAnswer.text` equals `t("TV.RUI.P16.faq-0N-a")` exactly.

## 6. CTA

`t("TV.RUI.P00.cta")` equals "Request a fit-and-technical evaluation" and the same string is P01.cta, P08.cta, P11.cta and P17.cta. In the DOM the string appears exactly five times at every width: at ≥1280 in the header nav, P01, P08, P11 and P17; from 768 to 1279 the drawer holds the nav instance (count the drawer DOM even when closed); below 768 the drawer node, repositioned into the bottom bar when appropriate, is that same single nav instance. If `CTA_HREF` still equals the placeholder `#evaluation`, the QA pass reports FAIL, not a warning.

## 7. Type generation

`scripts/generate-copy-ids.mjs` writes `client/src/content/copy-ids.generated.ts` with literal unions `CopyId` (276 members) and `SectionId` (21) before typecheck; a misspelled ID must fail `tsc`. Wire: at Prompt 2, `"predev"` and `"prebuild"` are both `"node scripts/generate-copy-ids.mjs && npm run content:check"`. Prompt 3 appends `&& npm run images` to both once `scripts/images.mjs` exists. Never reference `npm run images` before Prompt 3.
