# Pod Recall Checker

A lightweight, mobile-friendly web app that uses your phone's camera and OCR to check Omnipod pod lot numbers against the Insulet Corporation recall list (CC-5464839, May 2026).

> **Not affiliated with or endorsed by Insulet Corporation.**

---

## Why this exists

Insulet issued a Medical Device Correction (recall CC-5464839) covering hundreds of lot numbers across Omnipod Eros, DASH, and Omnipod 5 pods. Checking a lot number means reading a small alphanumeric code off a box label and cross-referencing it against a long list on Insulet's website — a task that's tedious for anyone and particularly difficult for people with low vision.

This tool lets you point your phone camera at a box, tap a button, and immediately hear and see whether the lot is recalled.

---

## Features

- **Camera OCR** — point at the lot number on the pod box or tray label and tap Scan
- **Audio readout** — result is spoken aloud via the device's text-to-speech
- **Partial manual entry** — type as few as 4–5 characters and see all matching recalled lots in real time; no need to type the full lot number
- **Offline capable** — works without internet after the first load (Tesseract OCR engine is cached by the browser)
- **No install, no account, no data collection** — a single static HTML file; nothing leaves your device
- **Covers all regions** — the embedded lot database includes all globally affected lots from the Insulet recall page

---

## How to use it on iPhone

1. **Get the file onto your phone.** Options:
   - AirDrop `pod-recall-checker.html` from your Mac
   - Email it to yourself and open the attachment
   - If hosted on GitHub Pages, just open the URL in Safari

2. **Open in Safari.** Camera access requires Safari on iOS — other browsers on iPhone do not support `getUserMedia`.

3. **Optional: Add to Home Screen** for app-like access.
   Share → Add to Home Screen → Add

4. **First use requires internet** to load the Tesseract OCR library (~4 MB, cached after that).

### Scanning a lot number

1. Open the app and point the camera at the lot number on the box or pod tray lid.
2. Tap **📷 Scan Lot Number**.
3. The result appears immediately — **✅ NOT RECALLED** (green) or **🚨 RECALL** (red) — and is read aloud.
4. Tap **↩ Retake** to scan another box.

### Where to find the lot number

- On the **pod tray lid** (printed near the barcode)
- On the **side of the 5-pack box**
- On the **flat side of the pod itself**

### Manual entry (partial lot numbers)

If OCR doesn't pick up the number, expand **"Type lot number manually"** and type any portion of the lot number — the last 5 or 6 digits is usually enough. Matching recalled lots appear as a list as you type; tap one to confirm.

---

## How to host on GitHub Pages

1. Fork or clone this repo.
2. Rename `pod-recall-checker.html` to `index.html`.
3. Go to **Settings → Pages** in your repo.
4. Set Source to `main` branch, root folder (`/`).
5. Your app will be live at `https://yourusername.github.io/your-repo-name/`.

---

## Updating the lot list

Insulet may add lots to the recall at any time. When that happens:

1. Visit [omnipod.com/mdc/check-pod-lot?c=CC-5464839](https://www.omnipod.com/mdc/check-pod-lot?c=CC-5464839) and compare the current tables against the `AFFECTED_LOTS` set in the HTML file.
2. Add any new lots to the appropriate comment section inside the `AFFECTED_LOTS` `new Set([...])` block.
3. Update the date in the header subtitle (`<p>Recall CC-5464839 · ...`).
4. Update the source comment above the Set.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full update procedure.

---

## Technical notes

### Stack
- **Single static HTML file** — no build step, no framework, no server required
- **[Tesseract.js v5.1.1](https://github.com/naptha/tesseract.js)** — in-browser OCR via WebAssembly; loaded from jsDelivr CDN with a pinned Subresource Integrity (SRI) hash
- **Web Speech API** — native browser text-to-speech for the audio readout
- **MediaDevices.getUserMedia** — live camera viewfinder; falls back to `<input type="file" capture>` if unavailable

### Security
- CDN script is pinned to an exact version with an SRI `integrity` hash — the browser refuses to run it if the file doesn't match
- A `Content-Security-Policy` meta tag restricts script execution to `'self'` and the specific CDN origin
- All DOM manipulation uses `textContent` and `createTextNode` — no `innerHTML` with dynamic content
- User input is stripped to `[A-Z0-9]` before any processing or speech synthesis
- No data is transmitted anywhere; all processing is local

### Updating the Tesseract SRI hash (if you upgrade the version)

```bash
curl -sL https://cdn.jsdelivr.net/npm/tesseract.js@X.Y.Z/dist/tesseract.min.js \
  | openssl dgst -sha384 -binary | openssl base64 -A
```

Paste the output (prefixed with `sha384-`) into the `integrity` attribute on the `<script>` tag.

---

## Disclaimer

This tool is for **informational convenience only**. It is not a medical device and does not constitute medical advice. The embedded lot list was accurate as of May 2026 but may not reflect subsequent updates to the recall.

**Always verify with Insulet directly:**
- Website: [omnipod.com/insulet-alerts](https://www.omnipod.com/insulet-alerts)
- Phone: 1-800-641-2049 (24/7)

---

## License

MIT — see [LICENSE](LICENSE).

This software was written with substantial AI assistance. See the LICENSE file for notes on authorship and copyright status.
