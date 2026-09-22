# Stephora Website — Deployment Guide

A single-page, responsive, SEO-optimised site for **GitHub Pages + Cloudflare** (custom domain `stephora.org`).

---

## 1. Repository structure

Put these files in your repo **`sarbanidance/stephora-website`** exactly like this (paths matter — the site uses root-absolute paths like `/images/...`, which work because you serve from the domain root):

```
stephora-website/
├── index.html                 ← the website (already includes CSS + JS)
├── 404.html                   ← branded "page not found"
├── CNAME                       ← contains: stephora.org
├── robots.txt
├── sitemap.xml
├── site.webmanifest
├── favicon.ico
└── images/
    ├── logo.png                ← your official logo (unchanged)
    ├── mandala.png             ← decorative background (generated)
    ├── og-image.png            ← social-share preview (1200×630)
    ├── about.jpg               ← ★ REPLACE with a photo of Sarbani (square works best)
    ├── icon-16.png  icon-32.png  icon-48.png
    ├── icon-192.png  icon-512.png  apple-touch-icon.png
    ├── social-facebook.png  social-instagram.png  social-youtube.png  social-google.png
    └── gallery/
        ├── photo-1.jpg  … photo-6.jpg   ← ★ REPLACE with your real photos
```

### Files YOU should replace with real photos
| File | What to put there |
|------|-------------------|
| `images/about.jpg` | A portrait of Sarbani Ghosh (or a favourite dance photo). ~800×800px. |
| `images/gallery/photo-1.jpg` … `photo-6.jpg` | 6 performance / recital / class photos. ~900×675px (4:3), landscape looks best. |

Keep the **same file names** and everything just works. (You can download your existing photos from the old Wix site’s Gallery/Press pages and drop them in.) Optimise them (e.g. squoosh.app) so each is < 300 KB for fast loading + better SEO.

Everything else (logo, favicons, og-image, social icons) is already generated and ready.

---

## 2. Push to GitHub

```bash
git clone https://github.com/sarbanidance/stephora-website.git
cd stephora-website
# copy all the deployment files into this folder (see structure above)
git add .
git commit -m "New responsive, SEO-optimised Stephora site"
git push origin main
```

Then in **GitHub → repo → Settings → Pages**:
- **Source:** Deploy from a branch
- **Branch:** `main` / root (`/`)
- Under **Custom domain**, enter `stephora.org` and Save. (The `CNAME` file already sets this too.)
- Tick **Enforce HTTPS** once it becomes available.

---

## 3. Cloudflare DNS (domain bought on Cloudflare)

In the Cloudflare dashboard → your `stephora.org` zone → **DNS → Records**, add:

| Type  | Name              | Content                          | Proxy      |
|-------|-------------------|----------------------------------|------------|
| A     | `stephora.org` (@)| `185.199.108.153`                | Proxied 🟠 |
| A     | @                 | `185.199.109.153`                | Proxied 🟠 |
| A     | @                 | `185.199.110.153`                | Proxied 🟠 |
| A     | @                 | `185.199.111.153`                | Proxied 🟠 |
| CNAME | `www`             | `sarbanidance.github.io`         | Proxied 🟠 |

(These four IPs are GitHub Pages’ official addresses.)

**SSL/TLS settings (Cloudflare):**
- SSL/TLS → Overview → mode: **Full** (not Flexible).
- SSL/TLS → Edge Certificates → **Always Use HTTPS: On**, **Automatic HTTPS Rewrites: On**, **Minimum TLS 1.2**.

> Tip: While first verifying the domain in GitHub Pages, you can temporarily set the DNS records to **DNS-only (grey cloud)**; once GitHub issues its certificate and “Enforce HTTPS” is available, switch back to **Proxied (orange)**.

---

## 4. Security hardening (do this in Cloudflare)

GitHub Pages can’t set custom HTTP headers, but Cloudflare can. Go to **Rules → Transform Rules → Modify Response Header → Create rule** (apply to `hostname equals stephora.org`) and add these response headers:

| Header | Value |
|--------|-------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` |
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `SAMEORIGIN` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | `geolocation=(), microphone=(), camera=()` |

Also enable: **Security → Settings → Bot Fight Mode: On**, and **Speed → Optimization → Auto Minify** (HTML/CSS/JS) + **Brotli: On**.

The page already ships with a `Content-Security-Policy` and `referrer` **meta** tag as a baseline. For a stricter policy you can move the CSP to a Cloudflare header instead (recommended long-term). The current CSP allows: your own files, Google Fonts, and the Google Maps embed only.

---

## 5. Contact

There is **no contact form** (kept simple — no mail server needed). Visitors reach you directly via the **email, call/text, and WhatsApp** buttons and the embedded **Google Map** in the Contact section. If you ever want a form later, a free service like Formspree can be added without a server — just ask.

---

## 6. SEO — after launch
- **Google Search Console** (search.google.com/search-console): add `stephora.org`, verify (Cloudflare DNS TXT is easiest), then **submit `https://stephora.org/sitemap.xml`**.
- **Google Business Profile:** you already have one (the review link is on the site). Make sure the name, address (10290 Chapel Hill Rd, Suite 200, Morrisville, NC) and website match the site exactly — consistency boosts local ranking.
- Test with: **PageSpeed Insights**, Google **Rich Results Test** (it will read the LocalBusiness/Course structured data already embedded), and the **Mobile-Friendly Test**.
- Ask a few happy parents to leave Google reviews — local reviews are one of the strongest ranking signals for “Bharatanatyam classes near me”.

### What’s already built in for SEO
- Descriptive `<title>` + meta description + local keywords
- Open Graph + Twitter cards (with the branded share image)
- **JSON-LD structured data**: `DanceSchool`/`LocalBusiness` (name, address, geo area, hours, phone, social profiles) and a `Course` for Bharatanatyam
- Canonical URL, `robots.txt`, `sitemap.xml`, semantic HTML5, image `alt` text, lazy-loaded images, mobile-responsive layout, fast single-file load.

---

## 7. Updating content later
Everything is in `index.html` (text, sections, links). Edit, commit, push — Cloudflare serves the update within minutes (you can **Purge Cache** in Cloudflare → Caching if you don’t see changes immediately).
