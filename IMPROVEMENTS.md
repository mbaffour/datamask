# Improvements

This document summarizes changes made to DataMask. All changes are client-side
only — no new runtime dependencies were added and the privacy posture is
unchanged (everything still runs entirely in the browser; no data leaves the
device).

## Fixes

### Corrected the overstated "Strong" mode claim

Strong mode applies a **monotonic** transform (min–max normalization → sigmoid
distortion → optional cross-column mixing → rescale). Because the sigmoid is
monotonic, Strong mode **largely preserves the rank order of values within a
column**. That means the output is still re-identifiable by rank, or by
correlating it against a known reference dataset — so describing it as
"practically irreversible" / "computationally infeasible to reverse" overstated
its protection.

The copy was reworded to be accurate: Strong mode is *hard to reverse without a
reference dataset, but preserves rank order and therefore remains
re-identifiable by rank*; for non-invertible synthetic values with no
mathematical link to the originals, users are pointed to **Vault mode**.

Vault-mode wording and all transform math were left untouched.

Locations changed:
- `data_anonymizer.html` — Strong mode card description + pill (Step 3 mode grid).
- `data_anonymizer.html` — Strong mode `⚡` warning notice (Step 3 options).
- `README.md` — reversibility table row and the "Strong" detail section.
- `blog.html` — Strong badge in the modes table and the "which mode" paragraph
  (same claim appeared in the portfolio blog; corrected for consistency).

## New features

Both features are additive and fully local. With no per-column overrides set,
the anonymization pipeline behaves exactly as before.

### 1. Downloadable run report / log

After every run (and after each re-anonymization), a report can be downloaded
from Step 4 as **JSON** or **plain text**. It records, for reproducibility and
audit trails:

- timestamp (ISO 8601), source filename, and sheet name
- global mode, Lite method (when used), and the full set of modes actually applied
- whether per-column overrides were in effect
- rows processed, total columns, columns anonymized, and values changed
- per-column detail: index, name, detected type, effective mode, and whether it
  was overridden

The report deliberately contains **no original values** — only metadata about
the run — so sharing it never leaks source data.

Implementation:
- `state.lastRunReport` holds the most recent run's metadata.
- `buildRunReport()` populates it; `reportToText()` renders the plain-text form;
  `exportReport()` handles the download.
- UI: "Download run report" section in Step 4 with `#dlReportJson` /
  `#dlReportTxt` buttons.

### 2. Per-column mode override

A collapsible **"Per-column mode overrides"** panel in Step 3 lists every
selected column with a small dropdown: *Global* (default) or an explicit
*Lite / Strong / Vault*. This lets different columns use different tiers in a
single run — e.g. Vault for identifiers, Lite for coarse measurements.

Implementation:
- `state.colModeOverrides` maps `{ colIndex: mode }`; empty means "use the
  global mode everywhere" (identical to prior behavior).
- The masking engines (`applyLite`, `applyStrong`, `applyVault`) and the text
  labeling helpers now accept an optional `cols` argument (defaulting to
  `state.selectedCols`), so each can operate on just its own column group.
- `runAnonymization()` groups selected columns by effective mode and runs the
  engines in sequence, feeding each engine's output into the next. Because every
  engine copies the row and writes only its own columns, sequential composition
  is lossless and order-independent for disjoint column groups.
- UI: `renderOverrideList()` builds the per-column dropdowns; the list is rebuilt
  whenever Step 3 is entered and prunes overrides for de-selected columns.

**Known minor limitation:** the HTTP/`crypto.subtle` warning banner is shown when
the *global* mode is Vault. If a column is individually overridden to Vault while
the global mode is not, and the page is served over plain HTTP (not HTTPS or
localhost), the run will fail loudly with an error toast rather than showing the
pre-emptive banner. This fails safe (no data is exposed); serving over HTTPS or
localhost — as the docs already recommend for Vault — avoids it.

## Notes

- No `LICENSE` change: an MIT `LICENSE` file already exists in the repo, so it
  was left untouched.
- Verification: the inline `<script>` was extracted and passed `node --check`;
  the new dispatch/override/report logic was additionally exercised with a
  Node smoke test (baseline global mode, mixed per-column overrides, and
  untouched-column checks all pass).
