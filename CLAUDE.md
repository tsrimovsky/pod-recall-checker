# Pod Recall Checker — Claude Context

A single-file static web app for checking Omnipod pod lot numbers against the Insulet recall CC-5464839 (May 2026). Uses in-browser OCR (Tesseract.js) and the Web Speech API. No server, no build step, no dependencies to install.

## Files

| File | Purpose |
|---|---|
| `pod-recall-checker.html` | Main app — camera, OCR, manual entry, result display |
| `lots.js` | `AFFECTED_LOTS` Set — all recalled lot numbers |
| `README.md` | User-facing docs and GitHub Pages setup |
| `CONTRIBUTING.md` | Lot update procedure (edit lots.js → recompute SRI → PR) |
| `LICENSE` | MIT with AI authorship note and medical disclaimer |

## Branch structure

- `main` — single-file baseline (lots embedded in HTML); kept as a clean rollback point
- `split-lots-file` — active development branch; `lots.js` split out; **GitHub Pages serves this branch**

Always work on `split-lots-file`. Do not merge into `main` without thinking about it — it's the intentional rollback point.

## Key technical decisions

- **No CSP meta tag** — a `Content-Security-Policy` header was removed because it blocked Tesseract's runtime fetch of language data from `tessdata.projectnaptha.com`. Do not re-add it without testing on iOS Safari with a real scan.
- **`langPath` set to `tessdata.projectnaptha.com`** — `createWorker` sets `langPath: 'https://tessdata.projectnaptha.com/4.0.0'`. The jsDelivr path (`tesseract.js-data@4.0.0/tessdata_fast`) caused the download to hang indefinitely; switching to projectnaptha (Tesseract.js's native host) fixed it. No CSP restriction since the meta tag was removed.
- **SRI on both scripts** — `tesseract.min.js` and `lots.js` have `integrity` attributes. Any edit to `lots.js` requires recomputing the hash (see below).
- **`AFFECTED_LOTS` is a `Set`** — O(1) lookup; lot numbers stored as uppercase strings.
- **DOM-only manipulation** — all dynamic content uses `textContent`/`createElement`; no `innerHTML` with user data.
- **Input sanitized to `[A-Z0-9]`** — applies before lookup and before passing to speech synthesis.
- **Blob URLs revoked** — camera captures create a blob URL that is revoked after OCR completes to prevent memory leaks.

## Lot number formats

| Product | Format | Example |
|---|---|---|
| Omnipod Eros | `L` + 5 digits | `L72514` |
| Omnipod DASH | `PD1U` + 8 alphanumeric | `PD1U11202421` |
| Omnipod 5 | `PH1U` + 8 alphanumeric | `PH1U10212521` |
| Omnipod 5 Refill | `PR1U` + 8 alphanumeric | `PR1U12172521` |

## Updating the lot list

1. Edit `lots.js` — add new lots to the appropriate comment group
2. Update the source date comment near the top of `lots.js`
3. Update the header date in `pod-recall-checker.html` (`<p>Recall CC-5464839 · ...`)
4. Recompute the SRI hash:
   ```bash
   cat lots.js | openssl dgst -sha384 -binary | openssl base64 -A
   ```
5. Update the `integrity` attribute on the `<script src="lots.js">` tag in `pod-recall-checker.html`
6. Commit both files together; push to `split-lots-file`

## Updating the Tesseract SRI hash (on version bump)

```bash
curl -sL https://cdn.jsdelivr.net/npm/tesseract.js@X.Y.Z/dist/tesseract.min.js \
  | openssl dgst -sha384 -binary | openssl base64 -A
```

Paste result (prefixed `sha384-`) into the `integrity` attribute on the Tesseract `<script>` tag.

## Current SRI hashes (v1.5)

- `tesseract.js@5.1.1`: `sha384-GJqSu7vueQ9qN0E9yLPb3Wtpd7OrgK8KmYzC8T1IysG1bcvxvIO4qtYR/D3A991F`
- `lots.js`: `sha384-OVZaEI1xaHZ5+vZEobZxKNyGtAkY8W81AsUSS07teu8PlIdydLVaNwNm1XQs/zzr`

## Deployment

GitHub Pages serves `split-lots-file` branch, root `/`. URL: `https://tsrimovsky.github.io/pod-recall-checker/`

After pushing, Pages typically propagates in 1–2 minutes. The version string in the app header (e.g. `v1.4`) is the fastest way to confirm a new version is live.

## Known constraints

- **iOS Safari only** for camera scanning — other iOS browsers don't expose `getUserMedia`
- **First scan requires internet** to fetch Tesseract language data (~10 MB, cached by browser after that)
- **Git and gh commands work normally** — project is accessed via Claude Code CLI, so `git` and `gh` can be run directly without user intervention
