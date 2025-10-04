
# Index
> Custom Email Setup
>
> - Introduction
> - Custom Domain Email Setup Guide
> > 1. Overview
> > 2. Cloudflare Email Routing Setup
> > 3. Receiving Emails in Gmail
> > 4. Sending Emails (SMTP Configuration)
> > 5. Email Authentication (SPF, DKIM, DMARC)
> > 6. Common Issues & Fixes
> > 7. Understanding the Protocols
> > 8. Optional:  Advanced Security (Cloudflare Zero Trust)
> > 9. Example Final DNS Records (Simplified)
> > 10. Recommendations


<br>

# Custom Email Setup
This repository contains all the documentation and files associated during the Custom Email Setup procedure ...  
All the associated chats, guidence and helps taken [ChatGPT](https://chatgpt.com/g/g-p-68e081a5a8148191aff5ccfc0d100ee1-custom-email-setup/project "ChatGPT Project Link") can be seen there for review.

 

<br>

## Introduction
The Custom Email Setup process begin shortly after I have created and setup my custom portfolio domain [Roman Shrestha](https://romanstha.com.np)  
The Github repo of the website is [Github Repo](https://github.com/Technozamazing/web.portfolio) <br>

> Initially, I aimed to ...
>
> - Create a Portfolio website for myself.
> - Create a custom domain email with that Portfolio domain.
> - Host my Achievements and Improvements through the Portfolio website.
>
> > _Everything_ is going according to ~~**the plan**~~

<br>

<img width="1165" height="682" alt="image" src="https://github.com/user-attachments/assets/5b9f9072-4f12-4f9b-9810-5d30c4c5cd22" />

> Snapshot of the Portfolio website ...

<div align="right">
  <b><a href="#index">↥ Back To Top</a></b>
</div>

<br>

<br>

## 📧 Custom Domain Email Setup Guide  
**For:** romanstha.com.np  
**Author:** Roman Shrestha  
**Purpose:** Use `contact@romanstha.com.np` for sending and receiving emails through Gmail, Cloudflare, and SMTP (SendGrid/Brevo).

<br>

## 1. Overview

This guide explains how to:
- Set up **custom domain email routing** using **Cloudflare Email Routing**.
- Send and receive those emails through **Gmail**.
- Configure **SPF**, **DKIM**, and **DMARC** for email authentication.
- Integrate **SendGrid** or **Brevo (formerly Sendinblue)** for reliable SMTP sending.
- Troubleshoot DNS and delivery issues.

<div align="right">
  <b><a href="#index">↥ Back To Top</a></b>
</div>

<br>

## 2. Cloudflare Email Routing Setup

###  Step 1: Create a Custom Email Address
1. Go to **Cloudflare Dashboard → Email Routing**.  
2. Under “Custom addresses”,
   > create: `contact@romanstha.com.np`
3. Under “Destination addresses”,
   > add your Gmail: `romanstha306@gmail.com`
4. Verify your Gmail address by clicking the confirmation link.

### Step 2: Add Cloudflare’s DNS Records
Cloudflare provides:
- MX records (for routing)
- TXT record (for SPF)
- DKIM and DMARC (optional but recommended)

Example records:

| Type | Name | Value | Proxy |
|------|------|--------|--------|
| MX | romanstha.com.np | mx1.mail.routing.cloudflare.net | DNS only |
| MX | romanstha.com.np | mx2.mail.routing.cloudflare.net | DNS only |
| TXT | romanstha.com.np | v=spf1 include:_spf.mx.cloudflare.net ~all | DNS only |

> ⚠️ Don’t proxy MX, TXT, or CNAME records used for mail (orange cloud must be **off**).

<div align="right">
  <b><a href="#index">↥ Back To Top</a></b>
</div>

<br>

## 3. Receiving Emails in Gmail

Your Cloudflare routing already forwards all incoming emails from  
`contact@romanstha.com.np`  →  `romanstha306@gmail.com`.

To view them separately:

1. In Gmail, go to **Settings → Inbox → Multiple Inboxes**.  
2. Add a section with the search query:
   > to:contact@romanstha.com.np
3. Save changes — now your domain emails appear in a separate pane.

To prevent duplicate inbox entries, create a Gmail filter:
- **Filter:** `to:contact@romanstha.com.np`
- **Action:** “Skip Inbox (Archive it)” + “Apply label: Contact Email”

<div align="right">
  <b><a href="#index">↥ Back To Top</a></b>
</div>

<br>

## 4. Sending Emails (SMTP Configuration)

You can send from Gmail using **SendGrid** or **Brevo** SMTP.

### Option A — Using SendGrid
1. Create a SendGrid account.
2. Add and verify your domain (`romanstha.com.np`).
3. Under **Settings → API Keys**, create an API key with “Full Access”.
4. Add SendGrid DNS records (CNAME) to Cloudflare.
5. Wait for domain verification.

#### Gmail Configuration
1. In Gmail → **Settings → Accounts and Import → Send mail as**  
2. Add:
   > - Name: Roman Shrestha
   > - Email: contact@romanstha.com.np
   > - SMTP Server: smtp.sendgrid.net
   > - Port: 587
   > - Username: apikey
   > - Password: <Your `SendGrid API Key`>
3. Choose “TLS” and save.
4. Verify by clicking the confirmation link sent to your Gmail.

---

### Option B — Using Brevo (Sendinblue)
1. Create a **Brevo** account.
2. Go to **Senders & Domains → Domains → Add a domain**.
3. Enter your domain (`romanstha.com.np`) and verify it by adding DNS records in Cloudflare:
   > - SPF: `v=spf1 include:spf.brevo.com -all`
   > - DKIM: add Brevo’s provided CNAME
4. Once verified, go to **SMTP & API → Generate SMTP Key**.

#### Gmail Configuration
1. In Gmail → **Settings → Accounts and Import → Send mail as**
2. Add:
   > - SMTP Server: smtp-relay.brevo.com
   > - Port: 587
   > - Username: something@brevo.com
   > - Password: <Your `Brevo SMTP Key`>
3. Use TLS, click Add Account, then verify via the Gmail link.

> ⚠️ Do **not** use the API key as the password — use the SMTP key provided under “Transactional → SMTP & API”.

<div align="right">
  <b><a href="#index">↥ Back To Top</a></b>
</div>

<br>

## 5. Email Authentication (SPF, DKIM, DMARC)

### SPF Record
Defines which mail servers are allowed to send email for your domain.

Example:
```
v=spf1 include:_spf.mx.cloudflare.net include:spf.brevo.com include:sendgrid.net -all
```

### DKIM Record
Adds a digital signature that proves your email wasn’t modified in transit.

Brevo/SendGrid both provide CNAME records like:
```
Type: CNAME
Name: mail._domainkey
Value: mail.domainkey.brevo.com (or sendgrid.net)
```

### DMARC Record
Protects your domain from `spoofing` and `phishing`.

Example:
```
v=DMARC1; p=quarantine; rua=mailto:dmarc@romanstha.com.np; ruf=mailto:dmarc@romanstha.com.np; sp=none; aspf=r;
```

#### Parameters Explained
- `p=` → Policy (`none`, `quarantine`, or `reject`)
- `rua=` → Aggregate reports (sent to this email)
- `ruf=` → Forensic reports
- `aspf=r` → Relaxed alignment (recommended)

<div align="right">
  <b><a href="#index">↥ Back To Top</a></b>
</div>

<br>

## 6. Common Issues & Fixes

| Problem | Cause | Fix |
|----------|--------|-----|
| Gmail says “Invalid username or password” | Using API key instead of SMTP key | Use Brevo SMTP key |
| “DMARC Quarantine/Reject policy not enabled” | DMARC set to `p=none` | Change to `p=quarantine` or `p=reject` |
| Cloudflare warns “Exposes IP address” | Mail record proxied (orange cloud on) | Turn proxy **off (grey cloud)** |
| Emails from Gmail show in main inbox | Filter not applied properly | Edit filter → ensure “Skip Inbox” checked |

<div align="right">
  <b><a href="#index">↥ Back To Top</a></b>
</div>

<br>

## 7. Understanding the Protocols

### SPF
Validates **who** is allowed to send emails for your domain.

### DKIM
Validates **what** — that the content hasn’t changed in transit.

### DMARC
Validates **how** — sets policies if SPF/DKIM fail and reports activity.

They work together as:
```
SPF + DKIM → DMARC alignment → Trustworthy email
```

<div align="right">
  <b><a href="#index">↥ Back To Top</a></b>
</div>

<br>


## 8. Optional:  Advanced Security (Cloudflare Zero Trust)

- **Not required** for your portfolio or email routing.
- Used for corporate VPNs, secure tunnels, and identity protection.
- Safe to skip for personal domains.

<div align="right">
  <b><a href="#index">↥ Back To Top</a></b>
</div>

<br>

## 9. Example Final DNS Records (Simplified)

| Type | Name | Value |
|------|------|--------|
| MX | romanstha.com.np | mx1.mail.routing.cloudflare.net |
| MX | romanstha.com.np | mx2.mail.routing.cloudflare.net |
| TXT | romanstha.com.np | v=spf1 include:_spf.mx.cloudflare.net include:spf.brevo.com -all |
| TXT | _dmarc | v=DMARC1; p=quarantine; rua=mailto:dmarc@romanstha.com.np; aspf=r;  |
| CNAME | mail._domainkey | mail.domainkey.brevo.com |

<div align="right">
  <b><a href="#index">↥ Back To Top</a></b>
</div>

<br>

## 10. Recommendations

- Keep **proxy off** for all mail DNS entries.
- Test configuration at [https://mxtoolbox.com](https://mxtoolbox.com)
- Monitor DMARC reports weekly.
- Don’t delete your SendGrid or Brevo DNS entries even if switching temporarily.
- Always keep backup of your DNS record list before making changes.

<div align="right">
  <b><a href="#index">↥ Back To Top</a></b>
</div>

---

**Last Updated:** October 2025  
**Maintained by:** Roman Shrestha  






# 
<mark>Learn Markdown</mark> from [Codecademy](https://www.codecademy.com/resources/docs/markdown/blockquotes) <br>
<mark>Hope this Documentation have help you (:</mark>
