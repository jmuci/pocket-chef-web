# Pocket Chef Web

This repo contains a static landing page for **Pocket Chef**.

Contents

- `index.html` — landing page (edit `Get on Play` CTA and the form action).
    
- `styles.css` — styles.
    
- `assets/` — images used on the page.
    

This README covers:

1. Deploying to **GitHub Pages** (root of `main` branch).
    
2. Configuring a custom domain with **Cloudflare** (DNS only).
    
3. Creating and wiring a **Formspree** email/waitlist form.
    
4. Quick troubleshooting.
    

---

## 1) Prepare the repo & push to GitHub

Run these commands locally (macOS / Linux / WSL / Git Bash):

`# in the directory containing index.html, styles.css, assets/ git init git add . git commit -m "Initial Pocket Chef landing page" # create repo on GitHub first, e.g. "pocket-chef-site" git remote add origin git@github.com:YOUR_USERNAME/pocket-chef-site.git git branch -M main git push -u origin main`

If you prefer HTTPS remote:

`git remote add origin https://github.com/YOUR_USERNAME/pocket-chef-site.git`

---

## 2) Enable GitHub Pages (main branch, root)

1. On GitHub, open your repository → **Settings** → **Pages**.
    
2. Under **Source**, choose:
    
    - Branch: `main`
        
    - Folder: `/ (root)`
        
3. Save. GitHub will publish at:
    
    `https://YOUR_USERNAME.github.io/pocket-chef-site/`
    

Wait a minute or two for the site to appear.

---

## 3) Add your custom domain (optional: `pocket-chef.app`)

If you want `pocket-chef.app` as the production domain, use Cloudflare for DNS.

### 3A — Add the domain to Cloudflare

- Add `pocket-chef.app` to Cloudflare and choose the Free plan.
    
- Cloudflare will give you two nameservers. Update those nameservers at the registrar where you bought the domain.
    

### 3B — DNS records for GitHub Pages (Cloudflare DNS, **proxy OFF**)

GitHub Pages requires you to point the domain to GitHub’s IPs. In Cloudflare DNS, create these **A** records for the root (apex) `pocket-chef.app`:

`Type: A Name: @ Value: 185.199.108.153 Proxy status: DNS only (grey cloud / OFF)  Type: A Name: @ Value: 185.199.109.153 Proxy status: DNS only  Type: A Name: @ Value: 185.199.110.153 Proxy status: DNS only  Type: A Name: @ Value: 185.199.111.153 Proxy status: DNS only`

And for `www` (optional), add:

`Type: CNAME Name: www Value: YOUR_USERNAME.github.io Proxy status: DNS only (OFF)`

**Why proxy OFF?**

- If Cloudflare proxy (orange cloud) is enabled for a GitHub Pages apex record, the TLS negotiation and certificate behavior becomes complicated. GitHub Pages issues and manages certificates for direct DNS. Setting Cloudflare to **DNS only** (grey cloud) keeps GitHub Pages in control of issuing and renewing the TLS certificate and avoids errors. If you require Cloudflare features (WAF, Workers) you’ll need extra steps — tell me and I’ll outline them, but DNS-only is the simplest, safest route.
    

### 3C — Configure the custom domain in GitHub Pages

1. In your repo → Settings → Pages → Custom domain: enter `pocket-chef.app` (or `www.pocket-chef.app`).
    
2. Save. GitHub will attempt to verify DNS and enable HTTPS. It can take a few minutes to a few hours for DNS propagation and certificate issuance.
    

**Tip:** GitHub shows a checkbox to enforce HTTPS; enable it once available.

---

## 4) CNAME file (alternative / convenience)

If you prefer to commit the custom domain into the repo, add a `CNAME` file at the repo root with your domain (single line):

`pocket-chef.app`

Commit & push. GitHub Pages will pick that up automatically.

---

## 5) Replace the Play Store & form placeholders

- Update the Play Store link: in `index.html` find the `Get it on Google Play` link and set `href` to your Play Store URL.
    
- Replace the waitlist form action placeholder:
    
    `<form id="signup" action="https://formspree.io/f/your-form-id" method="POST">`
    
    Replace `https://formspree.io/f/your-form-id` with the real Formspree endpoint you will create below.
    

---

## 6) Create a Formspree form (waitlist) — quick steps

Formspree provides a simple endpoint you can POST to from static pages.

### 6A — Sign up

1. Go to [Formspree](https://formspree.io) and create an account (free tier available).
    
2. After login, **Create a new form** (or “Add form” / “New endpoint”).
    

### 6B — Copy the endpoint URL

- Formspree will give you an endpoint like:
    
    `https://formspree.io/f/ABCDEFGHI`
    
- Copy that entire URL.
    

### 6C — Update `index.html`

- Replace `action="https://formspree.io/f/your-form-id"` with:
    
    `action="https://formspree.io/f/ABCDEFGHI"`
    

### 6D — (Optional) Add a hidden field for subject or tags

You can include hidden inputs to help filter messages:

`<input type="hidden" name="_subject" value="Pocket Chef waitlist signup" /> <input type="hidden" name="source" value="landing-page" />`

### 6E — Test the form locally or on the live site

You can test with `curl`:

`curl -X POST "https://formspree.io/f/ABCDEFGHI" \   -H "Accept: application/json" \   -d "email=test@example.com" \   -d "name=Jose"`

Formspree will return JSON confirming success. Then check the Formspree dashboard / your email for the submission.

### 6F — Spam protection & CORS

- Formspree handles CORS for basic forms; POSTing from a static page directly to the endpoint is standard and supported.
    
- Use their built-in CAPTCHA or honeypot options provided in the Formspree dashboard if you see spam.
    
- Free tier has limits — check Formspree plan details if you expect heavy traffic.
    

---

## 7) Netlify forms alternative (no third party signups)

If you prefer not to use Formspree, Netlify Forms captures form submissions automatically if you:

- Deploy to Netlify (connect the repo).
    
- Add `netlify` attributes to the form:
    

`<form name="waitlist" method="POST" data-netlify="true">   <input type="email" name="email" required /> </form>`

Netlify will capture and store the submissions in their dashboard. Let me know if you prefer this and I’ll convert the form.

---

## 8) Verify HTTPS and site

After DNS and Pages setup:

- Visit `https://pocket-chef.app` — check the certificate (browser lock icon).
    
- If HTTPS not yet active, GitHub Pages will typically issue TLS cert within 5–60 minutes but occasionally longer (up to several hours). Wait and check again.
    

Troubleshoot:

- DNS propagation: use `dig pocket-chef.app +short` to verify the A records point to GitHub IPs.
    
- If you see a Cloudflare error or GitHub certificate error, confirm Cloudflare proxy is **OFF** for the A records.
    
- Clear browser cache or test in incognito.
    

---

## 9) Quick commands summary

Push to GitHub:

`git add . git commit -m "Deploy landing page" git push origin main`

Make CNAME file:

`echo "pocket-chef.app" > CNAME git add CNAME git commit -m "Add CNAME for custom domain" git push origin main`

Test the Formspree endpoint with `curl`:

`curl -X POST "https://formspree.io/f/ABCDEFGHI" \   -H "Accept: application/json" \   -d "email=you@yourdomain.com"`

---

## 10) Notes, gotchas & next steps

- If you later want Cloudflare **proxy ON** for CDN/WAF, you'll need to manage TLS between Cloudflare and GitHub — GitHub Pages doesn't allow installing custom origin certs, so the simplest safe approach remains **DNS only** for GitHub Pages. If you want Cloudflare features (cache, WAF, Workers), ask me and I’ll outline a secure configuration (involves using an alternate host or Cloudflare Pages).
    
- If you want no third-party form provider, I can convert the form to Netlify Forms (free) or give you a tiny serverless function that stores emails in a Google Sheet or a simple Postgres table.
    
- Remember to replace the Play Store CTA before launch and to export app icons/favicons (I can generate those from your logo).

