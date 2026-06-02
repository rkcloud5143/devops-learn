# AWS SES (Simple Email Service) — Deep Dive

---

## WHAT IS SES

```
SES = Send and receive emails at scale via AWS

┌─── SES Architecture ─────────────────────────────────────────┐
│                                                               │
│  SENDING:                                                    │
│  Your App ──► SES ──► Recipient's email                      │
│                                                               │
│  Methods:                                                    │
│  1. SMTP interface (standard email protocol)                 │
│  2. AWS SDK / API (ses:SendEmail)                            │
│  3. SES console (testing)                                    │
│                                                               │
│  RECEIVING:                                                  │
│  Incoming email ──► SES ──► S3 / Lambda / SNS                │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## SANDBOX vs PRODUCTION

```
┌─── Sandbox Mode (default) ───────────────────────────────────┐
│  - Can only send TO verified email addresses                 │
│  - 200 emails per 24 hours                                   │
│  - 1 email per second                                        │
│  - Must request production access to remove limits           │
└──────────────────────────────────────────────────────────────┘

┌─── Production Mode ──────────────────────────────────────────┐
│  - Send to any email address                                 │
│  - Higher sending limits (scales with reputation)            │
│  - Must verify domain ownership (DNS records)                │
│  - Must set up DKIM, SPF, DMARC for deliverability          │
└──────────────────────────────────────────────────────────────┘
```

---

## EMAIL AUTHENTICATION (Important for Deliverability)

```
┌──────────┬───────────────────────────────────────────────────┐
│ SPF      │ DNS TXT record listing authorized mail servers    │
│          │ "Only these IPs can send email for my domain"     │
├──────────┼───────────────────────────────────────────────────┤
│ DKIM     │ Cryptographic signature on emails                 │
│          │ Proves email wasn't tampered with in transit      │
├──────────┼───────────────────────────────────────────────────┤
│ DMARC    │ Policy for handling failed SPF/DKIM checks        │
│          │ "Reject/quarantine emails that fail verification" │
└──────────┴───────────────────────────────────────────────────┘

Without these: your emails go to SPAM folder
```

---

## COMMON USE CASES

```
- Transactional emails (order confirmations, password resets)
- Marketing emails (newsletters, promotions)
- Notification emails (alerts from CloudWatch via SNS → SES)
- Application emails (user signup verification)

Pricing: $0.10 per 1,000 emails sent
         Free if sending from EC2
```
