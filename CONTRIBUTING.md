# Contributing

The main ongoing maintenance need is keeping the lot list current. Insulet may add lots to the recall without notice.

## Updating the lot list

### 1. Check for new lots

Visit the recall page and compare against what's in the file:

```
https://www.omnipod.com/mdc/check-pod-lot?c=CC-5464839
```

The page organizes lots by product and region. The `AFFECTED_LOTS` set in `pod-recall-checker.html` combines all regions into a single deduplicated set (since a box is recalled regardless of where you bought it).

### 2. Add new lots to the HTML file

Open `lots.js` and add new lot numbers to the appropriate comment group:

```js
const AFFECTED_LOTS = new Set([
  // Omnipod Eros
  'L71480', ...

  // Omnipod DASH (all regions combined)
  'PD1U...', ...

  // Omnipod 5 — PH1U prefix
  'PH1U...', ...

  // Omnipod 5 — PR1U prefix
  'PR1U...', ...
]);
```

Lot number formats:
| Product | Format | Example |
|---|---|---|
| Omnipod Eros | `L` + 5 digits | `L72514` |
| Omnipod DASH | `PD1U` + 8 alphanumeric | `PD1U11202421` |
| Omnipod 5 | `PH1U` + 8 alphanumeric | `PH1U10212521` |
| Omnipod 5 Refill | `PR1U` + 8 alphanumeric | `PR1U12172521` |

### 3. Update the header date

In the `<header>` section, update the subtitle line to reflect when you last synced the list:

```html
<p>Recall CC-5464839 · Not affiliated with Insulet · Updated [Month Year]</p>
```


### 4. Recompute the SRI hash for lots.js

Any change to `lots.js` invalidates the existing SRI hash. Recompute it:

```bash
cat lots.js | openssl dgst -sha384 -binary | openssl base64 -A
```

Then update the `integrity` attribute on the `<script src="lots.js">` tag near the bottom of `pod-recall-checker.html`:

```html
<script src="lots.js" integrity="sha384-<new hash here>" crossorigin="anonymous"></script>
```

### 5. Update the source comment

Near the top of the `AFFECTED_LOTS` block, update the date comment:

```js
//  Source: omnipod.com/mdc/check-pod-lot?c=CC-5464839, [Month Year]
```

### 6. Open a pull request

Include in the PR description:
- The date you checked the Insulet recall page
- Which lots are new (copy/paste from the Insulet page is fine)
- A link to the Insulet page if the URL has changed

---

## Reporting issues

If OCR consistently misreads a particular lot format, or the partial search behaves unexpectedly, open a GitHub Issue with:
- The lot number involved (you can anonymize if preferred)
- What the app showed vs. what was expected
- Which device/browser you were using

---

## What this project does not accept

- Changes that add tracking, analytics, or any network requests beyond loading Tesseract
- Dependency additions (the point is that this is a single self-contained file)
- Any feature that requires a server or an account
