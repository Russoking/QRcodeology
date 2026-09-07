# QRcodeology

A network of four free, single-purpose QR code tools plus a hub landing page,
each targeting a specific long-tail search term. Fully client-side — no
backend, no build step. Localized into Spanish under /es/.

## File structure

```
/
├── index.html                    Hub — links to all 4 tools (EN)
├── no-expiration/index.html      Standard QR generator, no expiration angle
├── logo-watermark/index.html     QR generator with logo overlay, no watermark
├── wifi/index.html               WiFi credential QR generator
├── bulk/index.html               Up to 30 codes at once, zip download
├── es/                           Spanish versions of all 5 pages above
│   ├── index.html
│   ├── no-expiration/index.html
│   ├── logo-watermark/index.html
│   ├── wifi/index.html
│   └── bulk/index.html
├── robots.txt
└── sitemap.xml
```

Domain: **qrcodeology.com** — already baked into every canonical tag,
hreflang tag, robots.txt, and sitemap.xml. No placeholder swap needed this
time.

## Deploying via GitHub + Cloudflare Pages

1. **Push to GitHub**
   ```bash
   git init
   git add .
   git commit -m "Initial commit: QRcodeology tool network"
   git branch -M main
   git remote add origin https://github.com/Russoking/QRcodeology.git
   git push -u origin main
   ```

2. **Connect Cloudflare Pages**
   - Cloudflare dashboard → Workers & Pages → Create → Pages → Connect to Git
   - Select this repository
   - Build settings: static site, so leave **Build command** empty and set
     **Build output directory** to `/` (the repo root)
   - Deploy

3. **Attach the domain**
   - In the Pages project → Custom domains → add `qrcodeology.com` (and
     `www.qrcodeology.com` if you want both to resolve)
   - If the domain isn't already on Cloudflare, it'll prompt you to update
     nameservers at your registrar first

4. **After the domain is live**
   - Submit `sitemap.xml` in Google Search Console (add both the root
     property and, if you want separate tracking, the `/es/` path as a
     distinct property or just watch it under International Targeting)
   - Confirm Search Console's International Targeting report correctly
     picks up the 5 hreflang pairs

## AdSense

Not yet wired up on this site. Once the domain has been live a few days,
follow the same steps as before: apply at adsense.google.com with the live
domain, add the verification snippet to the `<head>` of every page (all 10
files), then add `ads.txt` at the root once approved.

## Adding a third language later

Duplicate the `es/` folder pattern into e.g. `/de/`, translate each page,
then add a `hreflang="de"` line to *all 10 existing files* (both English and
Spanish) plus the new German ones, and add 5 more `<url>` entries to
sitemap.xml.
