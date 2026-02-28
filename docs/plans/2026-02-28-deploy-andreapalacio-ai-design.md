# Deploy andreapalacio.ai — Full Site with Lovable Audit Quiz

**Date:** 2026-02-28
**Status:** Approved

## Goal

Replace the old simple site at andreapalacio.ai with the full preview-v3.html (2,145-line site with the 48-question AI audit quiz built from the Lovable "crewless-ai-audits" project). Verify the full lead capture and email automation pipeline works end-to-end.

## Current State

- **Domain:** andreapalacio.ai → Namecheap LiteSpeed hosting (IP 162.0.232.241), SSL active
- **Live site:** Old index.html (simple contact form, no quiz)
- **New site:** preview-v3.html (full site with 48-question audit, scoring, results)
- **Backend:** crewless-os.vercel.app/api/audit-submission (deployed, handles Airtable + Twilio + email)
- **Git:** Not initialized

## Architecture

```
andreapalacio.ai (Namecheap/LiteSpeed - static HTML)
  └─ index.html (quiz engine + all sections)
  └─ images/ (hero, headshot, about, speaking, podcast, instagram photos)
        │
        │ fetch() POST on quiz submit
        ▼
crewless-os.vercel.app/api/audit-submission (Vercel serverless)
  ├─ Airtable: base app059nFTunVtcf8X, table "Free Audits"
  ├─ Twilio: SMS alert to +19544486394
  └─ Nodemailer: SMTP via mail.andreapalacio.ai:465 from audits@andreapalacio.ai
```

## Implementation Steps

### 1. Prepare production files
- Copy preview-v3.html → index.html (replacing old version)
- Verify all image paths, meta tags, OG tags, favicon
- Ensure CORS origin in API matches andreapalacio.ai
- Remove preview files from production deployment

### 2. Initialize git repo
- git init, create .gitignore
- Push to GitHub (apalacio1128/andreapalacio-website)

### 3. Upload to Namecheap hosting
- Access cPanel/file manager or SFTP
- Replace old files with new production files
- Verify site loads at https://andreapalacio.ai

### 4. Verify SMTP environment variables on Vercel (crewless-os)
- Confirm SMTP_HOST, SMTP_PORT, SMTP_USER, SMTP_PASSWORD are set
- Confirm TWILIO_* vars are set
- Confirm AIRTABLE_API_KEY and AIRTABLE_BASE_ID are set

### 5. End-to-end test
- Complete the 48-question audit on the live site
- Verify: Airtable record created, SMS received, email delivered to test address
- Check email renders correctly (score, financials, department breakdowns, CTA)

## Risks

- **SMTP vars missing on Vercel:** Email silently fails (.catch only). Mitigated by explicit testing.
- **Namecheap access required:** Need cPanel credentials to upload files.
- **audits@andreapalacio.ai mailbox:** Must exist on mail.andreapalacio.ai for SMTP to work.

## Success Criteria

- [ ] andreapalacio.ai loads the new design with full audit quiz
- [ ] Quiz completes all 48 questions and shows results
- [ ] Lead captured in Airtable "Free Audits" table
- [ ] SMS alert received on Andrea's phone
- [ ] Personalized results email delivered to lead's inbox
