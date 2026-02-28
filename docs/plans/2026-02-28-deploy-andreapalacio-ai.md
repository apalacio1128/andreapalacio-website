# Deploy andreapalacio.ai Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Deploy the full andreapalacio.ai website (preview-v3.html with 48-question Lovable audit quiz) to live hosting, verify the email automation pipeline works end-to-end.

**Architecture:** Static HTML site on Namecheap LiteSpeed hosting. Quiz runs client-side, submits via fetch() to crewless-os.vercel.app/api/audit-submission which writes to Airtable, sends SMS via Twilio, and emails results via SMTP.

**Tech Stack:** Static HTML/CSS/JS, Namecheap cPanel/SFTP, GitHub, Vercel env vars (crewless-os)

---

### Task 1: Promote preview-v3.html to production index.html

**Files:**
- Modify: `/Users/andreapalacio/andreapalacio-website/preview-v3.html` → rename to `index.html`
- Remove from production: `preview.html`, `preview-v2.html`, `preview-v4.html`, old `style.css`, old `script.js`

**Step 1: Back up old index.html**

```bash
cd /Users/andreapalacio/andreapalacio-website
mv index.html index-old.html
```

**Step 2: Add SEO meta tags to preview-v3.html**

The current preview-v3.html is missing OG tags and description. Add these inside `<head>` after the `<title>` tag:

```html
<meta name="description" content="Andrea Palacio — AI Business Strategist, Speaker, Founder of Crewless. Take the free AI audit to find out how efficient your business really is.">
<meta property="og:title" content="Andrea Palacio | AI Strategist">
<meta property="og:description" content="I build AI systems that run businesses without the owner. Take the free 3-minute audit.">
<meta property="og:image" content="images/hero.jpeg">
<meta property="og:url" content="https://andreapalacio.ai">
<meta property="og:type" content="website">
<meta name="twitter:card" content="summary_large_image">
<link rel="canonical" href="https://andreapalacio.ai">
```

**Step 3: Rename to index.html**

```bash
mv preview-v3.html index.html
```

**Step 4: Verify in browser**

```bash
open index.html
```

Expected: Site loads with all sections, images, quiz visible.

**Step 5: Commit**

```bash
git add index-old.html index.html
git commit -m "Promote preview-v3 to production index.html with SEO meta tags"
```

---

### Task 2: Clean up preview files and create .gitignore

**Files:**
- Remove from production: `preview.html`, `preview-v2.html`, `preview-v4.html`, `style.css` (old), `script.js` (old)
- Create: `.gitignore`

**Step 1: Move preview files to an archive folder (keep for reference)**

```bash
cd /Users/andreapalacio/andreapalacio-website
mkdir -p _archive
mv preview.html preview-v2.html preview-v4.html index-old.html style.css script.js _archive/
```

**Step 2: Create .gitignore**

```
.DS_Store
_archive/
```

**Step 3: Verify only production files remain**

```bash
ls -la
```

Expected: `index.html`, `images/`, `docs/`, `.gitignore`, `.git/`

**Step 4: Commit**

```bash
git add .gitignore index.html images/
git commit -m "Clean up preview files, add .gitignore, stage production assets"
```

---

### Task 3: Push to GitHub

**Step 1: Create GitHub repo**

```bash
cd /Users/andreapalacio/andreapalacio-website
gh repo create apalacio1128/andreapalacio-website --public --source=. --push
```

**Step 2: Verify repo exists**

```bash
gh repo view apalacio1128/andreapalacio-website
```

Expected: Repo visible with index.html, images/, docs/

---

### Task 4: Upload to Namecheap hosting

**Context:** The domain andreapalacio.ai currently points to Namecheap LiteSpeed hosting (IP 162.0.232.241). The old site is already live there. We need to replace the files.

**Step 1: Ask user for Namecheap cPanel credentials or SFTP access**

Ask the user: "I need access to your Namecheap hosting to upload files. Can you either:
(a) Share cPanel login URL + credentials, or
(b) Share SFTP host/username/password, or
(c) Upload the files yourself — I'll tell you exactly which files to upload where"

**Step 2: Upload files via SFTP (if credentials provided)**

```bash
# Upload index.html and images/ to public_html (or the web root)
sftp user@andreapalacio.ai << 'EOF'
cd public_html
put index.html
put -r images/
bye
EOF
```

OR via scp:

```bash
scp index.html images/* user@server:/home/user/public_html/
```

**Step 3: Verify live site**

```bash
curl -sI https://andreapalacio.ai
curl -s https://andreapalacio.ai | head -10
```

Expected: New `<title>Andrea Palacio | AI Strategist</title>` (not the old title)

**Step 4: Open in browser to visually confirm**

```bash
open https://andreapalacio.ai
```

---

### Task 5: Verify SMTP environment variables on Vercel (crewless-os)

**Context:** The audit email is sent by `crewless-os.vercel.app/api/audit-submission` using Nodemailer. It needs SMTP env vars set in the Vercel project.

**Step 1: Check which env vars are set**

```bash
cd /Users/andreapalacio/crewless-os
vercel env ls
```

If Vercel CLI not installed:

```bash
npm i -g vercel
vercel login
vercel link
vercel env ls
```

**Step 2: Verify these 4 SMTP vars exist:**

- `SMTP_HOST` — should be `mail.andreapalacio.ai`
- `SMTP_PORT` — should be `465`
- `SMTP_USER` — should be `audits@andreapalacio.ai`
- `SMTP_PASSWORD` — the password for that mailbox

If any are missing, add them:

```bash
echo "mail.andreapalacio.ai" | vercel env add SMTP_HOST production
echo "465" | vercel env add SMTP_PORT production
echo "audits@andreapalacio.ai" | vercel env add SMTP_USER production
# For password, ask user
```

**Step 3: Also verify these existing vars:**

- `AIRTABLE_API_KEY` — needed for lead capture
- `AIRTABLE_BASE_ID` — should be `app059nFTunVtcf8X`
- `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_PHONE_NUMBER` — for SMS alerts

**Step 4: Verify the audits@andreapalacio.ai mailbox exists**

Ask user to confirm: "Does the email account `audits@andreapalacio.ai` exist on your Namecheap email hosting? If not, we need to create it in cPanel → Email Accounts."

**Step 5: Redeploy crewless-os to pick up new env vars**

```bash
cd /Users/andreapalacio/crewless-os
vercel --prod
```

---

### Task 6: End-to-end test

**Step 1: Open the live site and complete the audit**

```bash
open https://andreapalacio.ai
```

Complete all 48 questions with test data. On the lead form, use:
- Name: Test User
- Email: (user's real email to verify delivery)
- Phone: 555-000-0000
- Business: Test Business

**Step 2: Verify Airtable record**

Check the "Free Audits" table in Airtable base `app059nFTunVtcf8X`. A new record should appear with:
- Name: Test User
- Email: (test email)
- Status: New
- Source: andreapalacio.ai

**Step 3: Verify SMS received**

Check phone +19544486394 for SMS notification about the new lead.

**Step 4: Verify email received**

Check the test email inbox for the audit results email from `audits@andreapalacio.ai`. Verify:
- Subject line contains score and grade
- Email has score circle, financial impact boxes, department breakdowns
- CTA button links to cal.com/andrea-palacio/20min
- Email renders correctly on mobile

**Step 5: If email NOT received, check Vercel logs**

```bash
vercel logs crewless-os --since 5m | grep -i "audit\|smtp\|email\|error"
```

Debug any SMTP connection issues.

**Step 6: Commit and tag as deployed**

```bash
cd /Users/andreapalacio/andreapalacio-website
git tag v1.0-live
git push origin main --tags
```
