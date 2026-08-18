# SCHEMA.md

Reference for the ERA Professionals **IVY** cohort roster and the audit columns added by `scripts/sync_cohort.py`. LLMs reading this repo can derive everything they need about the data plane from this file. (Inherited unchanged from `butterfly-effect-club` — same column convention across every program on this platform.)

---

## Source spreadsheet

**Spreadsheet ID:** `1IrzM8z9X0bt-1Zp21s6DNxlL_1XaT-8Fq6e3YaQRcnU`
**URL:** https://docs.google.com/spreadsheets/d/1IrzM8z9X0bt-1Zp21s6DNxlL_1XaT-8Fq6e3YaQRcnU/edit
**Owner:** ERA Professionals (Bilal et al.)
**Service account with editor access:** `ivy-yoga-get-data-io-iam-gserv@get-data-io.iam.gserviceaccount.com`
**Local credentials file (gitignored):** `google_credentials.json` at repo root

## Tabs

| Tab name | Purpose | Owned by |
|---|---|---|
| `Cohort Roster` | Participant records — alumni + current cohort, discriminated by `Graduation Date` | ERA (source); sync_cohort.py writes audit columns |
| `Audit Trail` | Per-action log mirroring the "DApp Remarks" pattern (added by sync_cohort.py on first run) | sync_cohort.py |

## `Cohort Roster` columns

**Restructured 2026-08-18** from Bilal's original ad hoc `Sheet1` (Name / Studio / Location / Cohort /
Certification Date / Date of Last Renewal / Credential ID / Status / Email — one live row, "Shamshad Haider")
to the platform's standard shape: existing columns kept and preserved as-is, `Status` renamed to `Teaching Status`
to avoid colliding with the platform's own `status` audit column (same name, different meaning — his is "is this
instructor currently active," the platform's is "processed / pending / failed"), and the standard audit columns
appended starting at column I.

### Source columns (ERA-owned, do not overwrite)

| Col | Label | Type | Required | Notes |
|-----|-------|------|----------|-------|
| A | `Name` | string | yes | Display name → `identity.json.names[0]` |
| B | `Studio / Location` | string | yes | e.g. "IVY Islamabad" → `identity.json.metadata.studio` |
| C | `Cohort` | string | yes | e.g. "Yoga - Nov 2027" |
| D | `Certification Date` | date (e.g. "14 November 2027") | yes | Maps to the cert's `DATE OF CERTIFICATION` overlay field |
| E | `Date of Last Renewal` | date, blank until first renewal | no | Maps to the cert's `DATE OF LAST RENEWAL` overlay field (renders "—" when blank) — see PR3, not yet wired |
| F | `Credential ID` | string, e.g. `IVY-TT-2027-0001` | no | Cosmetic `IVY-TT-<year>-<seq>` ID printed on the cert; no generator exists yet for the sequence, see `cert_template/cert_config.json` |
| G | `Teaching Status` | string, e.g. "Active" | yes | ERA's own instructor-status tracking — renamed from `Status` 2026-08-18 to avoid colliding with the audit column below |
| H | `Email` | string | yes | Instructor contact — not currently used for admin-panel auth (that's the sheet-editor list, a separate mechanism) |

### Audit columns (added by the central tokenomics handler / admin panel)

| Col | Label | Filled when | Notes |
|-----|-------|-------------|-------|
| I | `public_key` | row first processed | Full base64 SPKI of the admin-minted placeholder pubkey. Public half only — private is never persisted. |
| J | `pk_hash` | row first processed | Canonical: `pk-` + first 12 chars of `base64url(SHA-256(decoded pubkey bytes))`. Matches `tokenomics/google_app_scripts/tdg_credentialing/practice_event_processing.gs::deriveSlug()`. |
| K | `attestation_tx_id` | after Edgar 200 | Edgar's `Request Transaction ID` (RSA-SHA256 signature, base64). Cryptographic audit anchor. |
| L | `qualification_tx_id` | live-cohort path only | Optional second tx_id for the admission event. Empty for alumni rows. |
| M | `profile_url` | after Edgar 200 | `https://truesight.me/programs/ivy-yoga/credentials/#<pk_hash>` |
| N | `credential_pdf_url` | after `build-cv-cache.yml` lands the file | `https://cdn.jsdelivr.net/gh/TrueSightDAO/lineage-credentials@main/_cache/cv/<pk_hash>__ivy-yoga.pdf` |
| O | `certificate_url` | live-cohort completion event lands | Only populated post-completion attestation for live-cohort path; alumni rows leave empty. |
| P | `status` | each run | `pending` / `profile_created` / `certificate_issued` / `failed` — **not** the same as column G (`Teaching Status`) |
| Q | `processed_at` | each successful step | ISO 8601 UTC |
| R | `github_commit_sha` | after GAS commit observable | Secondary breadcrumb to `attestation_tx_id` (which is the canonical anchor). |
| S | `notes` | on failure | Human-readable error message; cleared on successful retry |
| T | `public_listable_override` | manual ERA action | Default empty (= use program default `true`). Set to `false` to keep this participant off the searchable directory at `truesight.me/programs/ivy-yoga/members.html`. Individual profile URL still resolves. |

**No private key column. Anywhere.**

## `Audit Trail` tab columns

Mirrors the "DApp Remarks" pattern (`1eiqZr3LW-qEI6Hmy0Vrur_8flbRwxwA7jXVrbUnHbvc`).

| Col | Label | Notes |
|-----|-------|-------|
| A | `processed_at` | ISO datetime |
| B | `name` | Participant name (copy from `Cohort Roster.Name`) |
| C | `action` | `profile_created` / `certificate_issued` / `failed` / `key_generated` |
| D | `github_commit_sha` | lineage-credentials commit SHA from the GAS handler |
| E | `profile_url` | |
| F | `credential_pdf_url` | |
| G | `certificate_url` | |
| H | `error_message` | Empty on success |
| I | `triggered_by` | pk_hash of the operator from `admins.json` |

## Related sheets the script reads

| Spreadsheet | Purpose | Access |
|---|---|---|
| `1GE7PUq-UT6x2rBN-Q2ksogbWpgyuh2SaxJyG_uEK6PU` (Main Ledger) | Contributors contact information + Contributors Digital Signatures tabs — looked up for Bilal's lineage key + admin pubkey resolution | **Read access needed** — grant `ivy-yoga-get-data-io-iam-gserv@get-data-io.iam.gserviceaccount.com` viewer rights |
| `1qbZZhf-_7xzmDTriaJVWj6OZshyQsFkdsAV8-pyzASQ` (Telegram Chat Logs) | Universal DAO event ledger | **NOT directly accessed.** Edgar mediates. Scaling principle for future programs: program scripts never read Telegram Chat Logs directly. |

## Lineage / event mapping

The canonical flow goes via the admin panel (browser) + the central tokenomics handler. `scripts/sync_cohort.py` is the dev-side `--dry-run` tool that previews + tests the same event shape without firing live events.

```
ERA Cohort Roster row
    └── Admin clicks "Attest" on ivy-yoga-club.truesight.me
            ├── Browser builds [CREDENTIALING ATTESTATION EVENT] with routing fields:
            │       - Roster Source URL: this sheet
            │       - Roster Source Row: <N>
            │       - Schema URL: link to this SCHEMA.md
            ├── Browser signs with admin's localStorage RSA key
            ├── POST → Edgar /dao/submit_contribution
            ├── Edgar appends row to Telegram Chat Logs (universal ledger)
            └── tokenomics central handler picks up the event:
                ├── Verifies signature
                ├── Verifies Attestor Public Key ∈ program manifest authorized_attestors[]
                ├── Commits to lineage-credentials/programs/ivy-yoga/<pk_hash>/:
                │       - identity.json
                │       - attestations/<timestamp>-program-completion.json
                └── Back-fills the source sheet (using Roster Source URL + Row):
                        - Cohort Roster row N: status, pk_hash, attestation_tx_id, profile_url, processed_at, ...
                        - Appends to Audit Trail tab on same spreadsheet
```

### Required routing fields on each `[CREDENTIALING ATTESTATION EVENT]`

| Field | Example | Why |
|---|---|---|
| `Program` | `ivy-yoga` | Routes to lineage-credentials subfolder + program manifest |
| `Attestor Public Key` | base64 SPKI | Verified against program's `authorized_attestors[]` |
| `Attestor Name` | "Bilal Musharraf" | Audit display |
| `Attestee Public Key` | base64 SPKI | Derives `pk-hash` folder name |
| `Attestee Name` | "Maria Santos" | identity.json + display |
| `Attestation Type` | `program-completion` | Distinguishes profile-creation vs cert-attestation events |
| `Captured At` | ISO 8601 | identity.json `linked_at` |
| `Program Year` | "2027-2028" (computed at attest-time from the current date, see `programYear()` in `index.html`) | Cert template variable |
| `Source URL` | this admin panel URL | Where the event originated (UI/CLI/etc.) |
| `Roster Source URL` | sheet URL | Tells the handler which sheet to back-fill |
| `Roster Source Row` | `14` | Tells the handler which row to update |
| `Schema URL` | link to this SCHEMA.md | Documentation reference for LLMs / future programs |
| `Config URL` | `https://raw.githubusercontent.com/TrueSightDAO/ivy-yoga-club/main/config.json` | Tells the central tokenomics handler where to fetch program bootstrap config (GAS proxy URL, roster sheet ID, lineage-credentials path, etc.) |
| `Payload JSON` | `{"school": "...", "learner_type": "...", "graduation_date": "..."}` | Program-specific metadata |

## `identity.json` shape (written by GAS handler into lineage-credentials)

```json
{
  "primary_public_key": "<base64 SPKI of admin-minted placeholder pubkey>",
  "names": ["Shamshad Haider"],
  "emails": [],
  "linked_at": "2026-08-18T14:00:00Z",
  "metadata": {
    "studio": "IVY Islamabad",
    "cohort": "Yoga - Nov 2027",
    "program_year": "2027-2028",
    "certification_date": "2027-11-14"
  },
  "alternate_public_keys": [],
  "former_pk_hashes": [],
  "public_listable": true
}
```

- `alternate_public_keys[]` populated when the instructor later self-claims (deferred platform §13 flow).
- `former_pk_hashes[]` populated only if a participant is re-onboarded after key loss.
- `public_listable` mirrors `Cohort Roster.public_listable_override`. Defaults to `true`; ERA flips per-record by writing `false` to column T (not column P — see the audit-column table above; IVY's roster has more source columns than butterfly-effect's, so every audit column shifted right by 4 letters).
