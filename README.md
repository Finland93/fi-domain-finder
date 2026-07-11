# .fi Short Domain Finder

A single-page, no-build web tool that scans every short **.fi** domain and shows you which ones are still **available** — using public [RDAP](https://about.rdap.org/) data. Find free **2- and 3-character** combinations of letters, digits, the Finnish letters **å ä ö**, and the full Sámi set, fast.

> **"Available" = not currently in the registry.** RDAP tells you whether a name exists right now — it does **not** guarantee you can buy it (some short names are *reserved* or *premium-priced*). Always confirm on the official registry before you register.

**[▶ Live demo](https://finland93.github.io/fi-domain-finder/)** · No server, no API key, no dependencies — just one `index.html`.

---

## What it does

Covers **every combination `.fi` permits** at 2 and 3 characters. The character set is built from toggleable groups so you scan exactly what you want:

| Group | Characters | Default |
|-------|-----------|---------|
| Base (always on) | `a–z` (26) + `0–9` (10) = 36 | ✅ locked on |
| Finnish | `å ä ö` (3) | ✅ on |
| Sámi | `á â č đ ǥ ǧ ǩ ŋ õ š ŧ ž ʒ ǯ` (14) | ⬜ off |
| Hyphen | `-` (middle position of 3-char names only) | ⬜ off |

Resulting combination counts (shown live next to each tab as you toggle):

| Character set | 2 characters | 3 characters |
|---------------|-------------:|-------------:|
| Finnish (default) | 1,521 | 59,319 |
| + Sámi | 2,809 | 148,877 |
| + Sámi + hyphen | 2,809 | 151,686 |

- A **hyphen** may never be first or last, so it can only appear in the **middle of a 3-character** name — never in a 2-character one. The tool enforces this automatically.
- **Internationalized (IDN)** names — anything with a national character — are converted to their punycode `xn--…` form for the lookup, using the browser's own URL parser. They're marked with a small violet dot.
- **Live "time remaining" timer** with elapsed time, current speed (requests/min), and a running ETA that adapts to your actual measured throughput.
- **One-click copy** of all available domains found.
- **Filters** to scan a slice at a time: only-national, letters-only, with-digits, or only-available.

## The grid colors

| Color | Meaning |
|-------|---------|
| 🟦 Cyan (glowing) | **Available** — not in the registry |
| ⬛ Dim gray | Taken |
| 🟨 Yellow (pulsing) | Currently checking |
| 🟥 Red | Error / unsure (usually a rate limit — **not** a result) |
| 🟪 Violet dot | Contains a national character (IDN name) |

---

## How to run it

### Option A — just open it
Download `index.html` and double-click it. That's it. Everything runs in your browser.

> One caveat: opening via `file://` works, but some browsers apply stricter CORS rules to local files. If a scan returns only red "unsure" cells, serve it over HTTP instead (Option B) or deploy it (Option C).

### Option B — local web server
```bash
# clone your repo
git clone https://github.com/<your-username>/fi-domain-finder.git
cd fi-domain-finder

# serve it (pick one)
python3 -m http.server 8080
# or
npx serve .
```
Then open http://localhost:8080

### Option C — deploy (GitHub Pages) <a id="deploy-github-pages"></a>
1. Push this repo to GitHub.
2. **Settings → Pages → Build and deployment → Source: Deploy from a branch**, pick `main` and `/ (root)`, **Save**.
3. Your tool is live at `https://<your-username>.github.io/fi-domain-finder/`.

Because it's fully static, it also drops straight onto Netlify, Cloudflare Pages, Vercel, or any static host.

---

## ⚠️ Rate limits — please read

Public, no-cost RDAP endpoints **throttle** you. This is the single most important thing to understand about the tool.

- **rdap.org** sits behind Cloudflare and allows roughly **10 requests per 10 seconds**. Go faster and you get HTTP `429` (rate limited).
- A **red "unsure" cell means the request failed** — almost always a rate limit. It does **not** mean the domain is taken or available.

### What the tool does for you automatically
- Runs requests **serially** by default (concurrency = 1), which is the safe choice for rdap.org.
- **Backs off and slows down** automatically when it sees `429` responses.
- **Re-queues** throttled cells and does a **second pass** so temporary rate limits get cleaned up on their own.
- Lets you press **"Retry unsure"** after a run to re-check whatever's left red.

### What you should do
- Keep the **delay at 1100 ms or higher** (the default).
- Leave **concurrency at 1** for rdap.org.
- **Scan in batches** using the filters instead of the whole set at once — especially for 3 characters.

### About scan size
The full Finnish 3-character set is 59,319 combinations; enabling Sámi pushes it to ~150k. At a safe ~1 request/second a complete sweep is **many hours to days** — the on-screen ETA shows a live estimate. You almost never want the entire set at once. Use a filter (e.g. *with digits* or *only-national*) and scan the slice you care about, and only enable the Sámi group if you actually want those exotic names.

---

## Data sources

Switch between them in the dropdown at the bottom of the page:

| Source | How "available" is detected | Notes |
|--------|------------------------|-------|
| **rdap.org** (default) | HTTP `404` = available, `200` = taken | IETF bootstrap; routes to the authoritative registry. Cloudflare rate limit (~10 req / 10 s). |
| **who-dat.as93.net** | `isRegistered` field | CORS-friendly fallback. Small single instance — keep concurrency at 1 and be gentle. |

If rdap.org keeps rate-limiting you, switch to who-dat, or vice-versa.

---

## Where the available ones actually are

- Pure two-**letter** combos (`aa`–`zz`) are essentially **all registered**. Don't expect finds there.
- Your best odds: combos with **digits**, combos with **national characters** (å ä ö / Sámi), and the much larger **3-character** space.

---

## Registering an available name

1. Confirm availability on the official search: **[registry.domain.fi](https://registry.domain.fi/search/fi/app/domainsearch)** (every available cell links here).
2. Register through an accredited **registrar** — e.g. Zoner, Louhi, Domainhotelli, and others. `.fi` domains are not sold directly by the registry.
3. Watch for **premium pricing** on desirable short names.

---

## How it works (technical)

- Pure client-side JavaScript. No framework, no build step, no backend.
- Domains are generated in-browser and each is looked up with a single `fetch()` to the selected RDAP endpoint.
- IDN labels (including Sámi characters) are converted to punycode using the browser's own WHATWG `URL` parser (`new URL('http://äö.fi').hostname` → `xn--4ca0b.fi`), so the ASCII form sent to the API is always correct.
- Combinations are generated position-aware: edge positions (first/last) draw from the letters + digits + enabled national groups; the middle position of a 3-char name additionally allows a hyphen when that group is on. This guarantees the hyphen rule is never violated and no combination is missed or duplicated.
- The ETA blends your configured delay/concurrency with the **measured** per-request time once a handful of requests have completed, so it converges on a realistic estimate.
- Rendering is capped at 4,000 DOM cells for responsiveness on the large sets; **all** combinations are still scanned and counted regardless of what's drawn, and available results always appear in the list.

---

## Disclaimer

This tool queries third-party public RDAP/WHOIS services and reports what they return. Availability shown here is **indicative only**. Registry state can change at any moment, reserved and premium names can appear "available" via RDAP yet be unregisterable or costly, and rate limits can produce false "unsure" results. **Always verify on [registry.domain.fi](https://registry.domain.fi/search/fi/app/domainsearch) before relying on any result.** Provided as-is, with no warranty. Be respectful of the no-cost public endpoints — don't hammer them.

## License

MIT — see [`LICENSE`](LICENSE). Do whatever you like; a link back is appreciated but not required.
