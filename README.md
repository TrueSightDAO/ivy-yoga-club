# ivy-yoga-club

Operational repo for ERA Professionals' **IVY** cohort onboarding into the TrueSight DAO credentialing platform.
IVY is a yoga-teacher-training certification based on Olivia's "Liv for Yoga" curriculum, rooted in Bikram
Chaudhry's 26-posture regimen. Second program on this infrastructure, after
[`butterfly-effect-club`](https://github.com/TrueSightDAO/butterfly-effect-club) — same architecture, no
platform-side changes required (`agentic_ai_context/plans/IVY_YOGA_COHORT_ONBOARDING_PLAN.md`).

**Live surfaces:**
- Public credential pages → `https://truesight.me/programs/ivy-yoga/credentials/#<pk_hash>`
- Admin console → `https://ivy-yoga.truesight.me/` (GitHub Pages from this repo's `main` root)

**This repo holds:**
- `scripts/sync_cohort.py` — dev-side `--dry-run` tool that previews the event shape against IVY's Cohort Roster.
  Live attestations flow through the admin panel (browser-signed) → Edgar → central tokenomics handler.
- `index.html` — admin console. Boot fetches `./config.json`, runtime-auth-resolves admins against the Cohort
  Roster sheet's editor list (via GAS proxy). No per-program edits needed — it auto-rebrands from `config.json`.
- `config.json` — program bootstrap config: roster sheet URL (indirect, via the public manifest), GAS proxy URL,
  schema URL, lineage-credentials path, Edgar endpoint.
- `SCHEMA.md` — Cohort Roster sheet schema + the `[CREDENTIALING ATTESTATION EVENT]` field reference (inherited
  unchanged from the template — same column convention across every program).
- `PROPOSAL.md` — architecture decision record, inherited from `butterfly-effect-club` (the v4 design this program
  reuses verbatim; kept for reference, not IVY-specific).

**Trust circle:** whoever is an editor on the Cohort Roster sheet
(`https://docs.google.com/spreadsheets/d/1IrzM8z9X0bt-1Zp21s6DNxlL_1XaT-8Fq6e3YaQRcnU/edit`). Currently: Gary,
Bilal, Shahbaz, Danesh, Arshad (live-verified via the central tokenomics endpoint 2026-08-18). To grant or revoke
admin access: share/unshare the sheet. No static admin file to maintain.

**This repo does NOT hold:**
- Per-participant credential data (lives in `TrueSightDAO/lineage-credentials`, `programs/ivy-yoga/` once the
  first attestation lands).
- Cache artifacts / PDFs (rendered by `TrueSightDAO/lineage-engine` on push to `lineage-credentials`).
- Secrets: any local service-account JSON is `.gitignore`'d; CI uses base64-encoded GitHub Actions secrets
  (`GOOGLE_CREDENTIALS_JSON_B64`, same pattern as `butterfly-effect-club`).

## Cert template

`cert_template/` ships the real v1.2 design (Shahbaz, 2026-08-17 — hand-delivered by Gary 2026-08-18 after the
WhatsApp export dropped the media). Overlay coordinates and the three Google Fonts it uses (Cormorant Garamond,
Inter, Great Vibes) were reverse-engineered from the PDF and are vendored in full under `cert_template/fonts/` —
details in `cert_template/README.md`. Two of the design's overlay fields (the Bilal + Olivia dual signatures, and
the cosmetic `IVY-TT-<year>-<seq>` certificate ID) aren't wired into lineage-engine's renderer yet — that's PR3 in
the plan of record, gated on two open decisions.

## Quick start (local dry-run)

```bash
cd /opt/claude_workspace/ivy_yoga_club

# 1. Install deps
pip install -r scripts/requirements.txt

# 2. Point at the local service-account credentials
export GOOGLE_APPLICATION_CREDENTIALS=~/ivy_yoga_google_private_key.json

# 3. Dry-run the sync — walks every pending row in the Cohort Roster, prints plan
python3 scripts/sync_cohort.py --dry-run
```

## Documents

- [`PROPOSAL.md`](PROPOSAL.md) — architecture decision record (inherited from butterfly-effect-club)
- [`SCHEMA.md`](SCHEMA.md) — Cohort Roster schema + back-fill column glossary
- [`scripts/README.md`](scripts/README.md) — operator runbook

## References

- `agentic_ai_context/plans/IVY_YOGA_COHORT_ONBOARDING_PLAN.md` — this program's plan of record, incl. UAT phase
- `agentic_ai_context/credentials/CREDENTIALING_COHORT_PROGRAM_ONBOARDING.md` — the canonical onboarding playbook
- `agentic_ai_context/credentials/CREDENTIALING_PLATFORM.md` — overall credentialing data model
- [`butterfly-effect-club/PROPOSAL.md`](https://github.com/TrueSightDAO/butterfly-effect-club/blob/main/PROPOSAL.md) — the v4 architecture decisions this program reuses
