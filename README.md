# DataMask

**A privacy-first data anonymization tool for researchers, scientists, and technical users.**

DataMask lets you anonymize sensitive datasets entirely in your browser — before uploading them to AI tools, cloud scripts, or external collaborators. No data ever leaves your device.

---

## Why This Exists

Modern AI-assisted workflows are powerful, but they come with a real privacy risk: when you upload a dataset to an AI tool, that data may be logged, retained, or used for model training. For researchers working with unpublished results, experimental cohorts, sample identifiers, or proprietary measurements, this is not an acceptable trade-off.

DataMask was built to solve this. It gives you a clean, fast way to anonymize sensitive data locally — so you can safely use AI tools, share files with collaborators, or hand off data to scripts without exposing anything real.

---

## Features

### Three anonymization tiers

| Mode | Method | Reversibility |
|------|--------|--------------|
| **Lite** | Random noise, shuffle, replace, or scale & shift | Reversible with effort |
| **Strong** | Normalize → sigmoid distortion → cross-column mixing | Practically irreversible |
| **Vault** | SHA-256 per-cell seeding → PRNG synthetic values | Cryptographically irreversible |

### Text label substitution
All selected text columns (sample names, strain IDs, group labels, metadata) are replaced with consistent `SAMPLE_001`, `SAMPLE_002`, … labels across every row.

### Export formats
Download anonymized data as `.xlsx`, `.csv`, `.tsv`, `.txt`, or `.json` — with a custom filename prompt before every download.

### Additional capabilities
- Drag-and-drop or click-to-browse file upload (`.xlsx`, `.xls`, `.csv`, `.txt`)
- Paste raw CSV or TSV directly from clipboard
- Copy anonymized output to clipboard as CSV
- Export the original → label mapping as CSV or JSON (for traceability)
- Re-anonymize output for additional rounds of obfuscation
- Vault mode progress bar for large datasets
- Multi-sheet Excel support
- Keyboard shortcuts (`Enter` to advance, `Escape` to go back)

### Privacy-first architecture
- 100% local — all processing happens in your browser
- No backend, no server, no telemetry
- No data retained between sessions
- No hidden network calls (CDN only for SheetJS and fonts)
- Vault mode warns when running on HTTP (requires HTTPS or localhost for `crypto.subtle`)

---

## Usage

DataMask is a single self-contained HTML file. No installation, no build step, no dependencies to manage.

**Option 1 — Open directly:**
Download `data_anonymizer.html` and open it in any modern browser.

**Option 2 — Serve locally (recommended for Vault mode):**
```bash
# Python 3
python -m http.server 8080
# Then open: http://localhost:8080/data_anonymizer.html
```

**Option 3 — Clone and open:**
```bash
git clone https://github.com/mbaffour/datamask.git
cd datamask
open data_anonymizer.html   # macOS
start data_anonymizer.html  # Windows
```

---

## Workflow

1. **Upload** — drag & drop a file or paste CSV from clipboard
2. **Select columns** — check which columns to anonymize (numeric, text, or both)
3. **Choose protection mode** — Lite, Strong, or Vault
4. **Download** — export in your preferred format; optionally download the label mapping

---

## Anonymization Modes in Detail

### Lite
Applies one of four simple transformations per selected numeric column:
- **Random Noise** — adds a ±% delta to each value
- **Shuffle** — randomly reorders values within the column
- **Random Replace** — replaces each value with a random value in the column's min/max range
- **Scale & Shift** — multiplies by a random factor and adds a random offset

Controls: noise intensity, decimal places, sign preservation.

### Strong
Three-pass pipeline per numeric column:
1. Normalize to [0, 1]
2. Apply a random sigmoid distortion (`1 / (1 + e^(-k*(v-x0)))`) with random `k` and `x0`
3. Cross-mix with adjacent selected columns at a configurable blend ratio
4. Rescale to a randomized fake output range

All distortion parameters are generated fresh each run and immediately discarded — making reversal computationally infeasible even with the full output.

### Vault
For each numeric cell:
1. Compute `SHA-256(userSeed + ":" + colIndex + ":" + rowIndex)`
2. Seed a mulberry32 PRNG with the first 32 bits of the hash
3. Generate a synthetic value: either from a normal distribution matching the column's mean/std (shape-preserving), or uniform within the column range

No key is ever stored. Even with full knowledge of the algorithm, all output values, and the column statistics — the originals cannot be recovered.

---

## Text Label Substitution

When text columns are selected, DataMask builds a deterministic mapping of unique original values to sequential labels (`SAMPLE_001`, `SAMPLE_002`, …) in order of first appearance. This mapping is:
- Consistent across all rows (the same original value always maps to the same label)
- Applied in all three modes (Lite, Strong, and Vault)
- Exportable as a CSV or JSON file for traceability

---

## Browser Compatibility

Requires a modern browser (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+).

Vault mode requires `crypto.subtle`, which is available on:
- `https://` origins
- `http://localhost` (local development)

---

## License

MIT — see [LICENSE](LICENSE).

---

## Roadmap ideas

- Configurable label prefix (SAMPLE_, ID_, STRAIN_, …)
- Save and reload anonymization configurations
- Undo/redo history
- Batch file processing
- Column-level mode selection (different tiers per column)
- Reproducible anonymization via exportable seed files

---

*Built with passion for science and discovery.*
