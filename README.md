# .fi Short Domain Finder

A single-page, no-build web tool that scans every short **.fi** domain and shows you which ones are still **free** — using public [RDAP](https://about.rdap.org/) data. Find available **2- and 3-character** combinations of letters, the Finnish letters **å ä ö**, and digits, fast.

> **Results are a strong hint, not a guarantee.** RDAP tells you whether a name exists in the registry right now — it does **not** tell you whether you can actually buy it (some short names are *reserved* or *premium-priced*). Always confirm on the official registry before you register.

**[▶ Live demo](#deploy-github-pages)** · No server, no API key, no dependencies — just one `index.html`.

---

## What it does

- **2 characters** → 1,521 combinations (39 × 39)
- **3 characters** → 59,319 combinations (39 × 39 × 39)
- Character set per position: `a–z` (26) + `å ä ö` (3) + `0–9` (10) = **39 characters**. A hyphen may not be the first or last character, so pure 2/3-char combos never contain one.
- Internationalized names (containing å/ä/ö) are automatically converted to their punycode `xn--…` form for the lookup. They're marked with a small violet dot.
- **Live "time remaining" timer** with elapsed time, current speed (requests/min), and a running ETA that adapts to your actual measured throughput.
- **One-click copy** of all free domains found.
- **Filters** to scan a slice at a time: only å/ä/ö, letters-only, with-digits, or only-free.

## The grid colors

| Color | Meaning |
|-------|---------|
| 🟦 Cyan (glowing) | **Free** — not in the registry |
| ⬛ Dim gray | Taken |
| 🟨 Yellow (pulsing) | Currently checking |
| 🟥 Red | Error / unsure (usually a rate limit — **not** a result) |
| 🟪 Violet dot | Contains å/ä/ö (IDN name) |

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

Public, free RDAP endpoints **throttle** you. This is the single most important thing to understand about the tool.

- **rdap.org** sits behind Cloudflare and allows roughly **10 requests per 10 seconds**. Go faster and you get HTTP `429` (rate limited).
- A **red "unsure" cell means the request failed** — almost always a rate limit. It does **not** mean the domain is taken or free.

### What the tool does for you automatically
- Runs requests **serially** by default (concurrency = 1), which is the safe choice for rdap.org.
- **Backs off and slows down** automatically when it sees `429` responses.
- **Re-queues** throttled cells and does a **second pass** so temporary rate limits get cleaned up on their own.
- Lets you press **"Retry unsure"** after a run to re-check whatever's left red.

### What you should do
- Keep the **delay at 1100 ms or higher** (the default).
- Leave **concurrency at 1** for rdap.org.
- **Scan in batches** using the filters instead of the whole set at once — especially for 3 characters.

### About the 3-character scan
59,319 combinations at a safe ~1 request/second is **many hours** for a full sweep — the on-screen ETA shows a live estimate. You almost never want the entire set. Use a filter (e.g. *with digits* or *only å/ä/ö*) and scan the slice you care about.

---

## Data sources

Switch between them in the dropdown at the bottom of the page:

| Source | How "free" is detected | Notes |
|--------|------------------------|-------|
| **rdap.org** (default) | HTTP `404` = free, `200` = taken | IETF bootstrap; routes to the authoritative registry. Cloudflare rate limit (~10 req / 10 s). |
| **who-dat.as93.net** | `isRegistered` field | CORS-friendly fallback. Small single instance — keep concurrency at 1 and be gentle. |

If rdap.org keeps rate-limiting you, switch to who-dat, or vice-versa.

---

## Where the free ones actually are

- Pure two-**letter** combos (`aa`–`zz`) are essentially **all registered**. Don't expect finds there.
- Your best odds: combos with **digits**, combos with **å/ä/ö**, and the much larger **3-character** space.

---

## Registering a free name

1. Confirm availability on the official search: **[registry.domain.fi](https://registry.domain.fi/search/fi/app/domainsearch)** (every free cell links here).
2. Register through an accredited **registrar** — e.g. Zoner, Louhi, Domainhotelli, and others. `.fi` domains are not sold directly by the registry.
3. Watch for **premium pricing** on desirable short names.

---

## How it works (technical)

- Pure client-side JavaScript. No framework, no build step, no backend.
- Domains are generated in-browser and each is looked up with a single `fetch()` to the selected RDAP endpoint.
- IDN labels are converted to punycode using the browser's own WHATWG `URL` parser (`new URL('http://äö.fi').hostname` → `xn--4ca0b.fi`), so the ASCII form sent to the API is always correct.
- The ETA blends your configured delay/concurrency with the **measured** per-request time once a handful of requests have completed, so it converges on a realistic estimate.
- Rendering is capped at 4,000 DOM cells for responsiveness on the 3-char set; **all** combinations are still scanned and counted regardless of what's drawn, and free results always appear in the list.

---

## Disclaimer

This tool queries third-party public RDAP/WHOIS services and reports what they return. Availability shown here is **indicative only**. Registry state can change at any moment, reserved and premium names can appear "available" via RDAP yet be unregisterable or costly, and rate limits can produce false "unsure" results. **Always verify on [registry.domain.fi](https://registry.domain.fi/search/fi/app/domainsearch) before relying on any result.** Provided as-is, with no warranty. Be respectful of the free public endpoints — don't hammer them.

## License

MIT — see [`LICENSE`](LICENSE). Do whatever you like; a link back is appreciated but not required.
