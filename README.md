# Replit kit — what to upload and when

This folder is everything Replit Agent needs besides the prompts. Upload files by dragging them into the Replit file tree at the exact paths each prompt names.

| File | Upload to (in Replit) | When |
|---|---|---|
| `DESIGN.md` | project root | Prompt 1b |
| `SPEC-MOTION.md` | project root | Prompt 15 (or with DESIGN.md) |
| `SPEC-VALIDATOR.md` | project root | Prompt 2 |
| `SPEC-REGIONS.json` | project root | Prompt 3 |
| `content/site-content.json` (v1.1, 276 strings, 21 sections) | `client/src/content/site-content.json` | Prompt 2 |
| `content/COPY_VERIFICATION.json` (1.0 receipts) | `client/src/content/COPY_VERIFICATION.json` | Prompt 2 |
| `assets/teamvista-session-audit-2x.png` (2962×1782) | `client/src/assets/brand/` | Prompt 3 |
| `assets/teamvista-alerts-2x.png` (3172×1980) | `client/src/assets/brand/` | Prompt 3 |
| `assets/teamvista-screenshots-directory-2x.png` (3056×1728) | `client/src/assets/brand/` | Prompt 3 |
| `assets/teamvista-mark.png` (512×512) | `client/src/assets/brand/` | Prompt 3 |

Hashes for every file are in `SHA256SUMS.txt`; the prompts quote them so Replit verifies what it received.

Provenance: `site-content.json` here is v1.1, derived from Alan's approved 1.0 by the owner-authorised patch in `../content-proposal/copy-changes.v1.1.json` (adds, four minimal edits, six splits, two new sections; no owner-placeholder strings). Seven pending items that need owner facts (six strings and the P16a section) are parked in `../content-proposal/pending-owner-facts.v1.2.json`. The screenshots are the cleaned masters (approved mark swapped in, plate cropped) with a text-safe enhancement and 2× upscale, documented in `../generated-assets/TRANSFORMATIONS.md`.
