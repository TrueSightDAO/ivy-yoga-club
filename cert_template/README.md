# cert_template/

Visual assets for the IVY (Indus Valley Yoga) teacher-training certificate PDF, built from the real design
Bilal/Shahbaz shared in the ERA DAO WhatsApp thread on 2026-08-17 (`IVY certificate - blank - v1.2.pdf`).

| File | Purpose |
|---|---|
| `cert_config.json` | Overlay field positions (name, dates, signatures, QR, certificate ID), font references, color. Coordinates reverse-engineered from the blank + sample PDFs — see the file's `_comment` for methodology. |
| `cert_template.pdf` | Base certificate PDF — Shahbaz's v1.2 design, unedited. |
| `reference_sample_filled.pdf` | The filled example ("Ayesha Khan") Shahbaz sent — used to calibrate `cert_config.json`'s coordinates and confirm font choices. Not consumed by the renderer; kept for reference. |
| `logo.png` | The IVY icon + wordmark, extracted from the certificate PDF's embedded image (the "INDUS VALLEY YOGA" tagline text below it is a separate text element, not part of this image). |
| `fonts/` | Full family files for the three Google Fonts used on the certificate — see below. |

## Fonts

Identified from the embedded PDF font-subset names (`CG` / `IN` / `GV` prefixes) and confirmed against
[google/fonts](https://github.com/google/fonts):

| Font | Used for | File |
|---|---|---|
| **Cormorant Garamond** (upright, variable weight) | Body copy, "CERTIFICATE" title (semibold instance) | `CormorantGaramond-VariableFont_wght.ttf` |
| **Cormorant Garamond Italic** (variable weight) | Recipient name (medium-italic instance, 34pt), dates | `CormorantGaramond-Italic-VariableFont_wght.ttf` |
| **Inter** (variable weight) | Small-caps labels ("DATE OF CERTIFICATION", "VERIFY"), the "26 postures..." bold phrase, certificate ID | `Inter-VariableFont_opsz-wght.ttf` |
| **Great Vibes** (script) | The two co-signer signatures (rendered once each attests) | `GreatVibes-Regular.ttf` |

All three are open-source under the SIL Open Font License — full license text for each family is included as
`fonts/OFL-<Family>.txt`. Vendoring the real family files (not the PDF's subsetted glyph set) so the renderer can
draw arbitrary recipient names, not just the glyphs that happened to appear in the sample.

## Two overlay fields beyond what the platform currently renders

`cert_config.json` documents `signature_bilal`, `signature_olivia`, and `certificate_id` with real coordinates,
but **lineage-engine's `cert_overlay.py` doesn't support these yet** — Butterfly Effect only ever needed
`recipient_name` + `date` + `qr`. Wiring up dual independent signatures and a cosmetic per-year certificate-ID
sequence is tracked as **PR3** in `agentic_ai_context/plans/IVY_YOGA_COHORT_ONBOARDING_PLAN.md`, gated on two open
decisions (fee/branding model; whether Olivia re-signs on every renewal or only Bilal does). This config exists now
so that PR3 has the exact target coordinates to implement against, rather than re-deriving them later.

## How these get used (once PR3 lands)

1. **Central rendering** — `lineage-engine/scripts/build_cv_cache.py` fetches these via the URLs in
   `../config.json::cert_template.base_url + cert_template.*` whenever a credential is rendered for this program.
2. **Per-program QR** — `logo.png` becomes the centre logo of the per-program QR.
3. **PDF overlay** — `cert_overlay.py` in lineage-engine reads `cert_config.json`, opens `cert_template.pdf` as the
   base, applies overlays, embeds the QR.

## Editing the cert design

Update the file(s) here, commit, push. If a recipient name overflows or a position looks off once real credentials
start rendering, edit `overlay_fields` in `cert_config.json` — no code change needed for position/font tweaks;
adding the two new field *types* (signatures, certificate ID) to the renderer is the PR3 code change.

## Cross-references

- `../config.json` — exposes the stable URLs the central handler fetches these from.
- `../SCHEMA.md` — the `Config URL` field on each `[CREDENTIALING ATTESTATION EVENT]` points at the program config.
- `agentic_ai_context/plans/IVY_YOGA_COHORT_ONBOARDING_PLAN.md` — this program's plan of record, PR3 detail.
