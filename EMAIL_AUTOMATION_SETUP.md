# Email Automation Setup Guide

This repository now has automated email sending integrated from Google Sheets. Emails are sent hourly to contacts in your Leads and Referrals sheets.

## Quick Start

### 1. Google Sheets Setup

Create a Google Sheet with the following worksheets:
- **Leads** - contains leads to email
- **Referrals** - contains referral partners to email
- **Outreach_Log** - logs all email sends (created automatically)

**Required columns in Leads/Referrals sheets:**
- `lead_id` - unique identifier
- `business_name` - company name (used in email template)
- `email` - email address to send to
- `outreach_status` - should be "new" for unsent leads
- `date_contacted` - populated automatically when email sent
- `city` - city (for logging)
- `state` - state (for logging)

### 2. GitHub Secrets Configuration

Add these secrets to your repository (Settings → Secrets and variables → Actions):

#### Gmail Credentials:
- `SENDER_EMAIL` - Your Gmail address (e.g., b2bleadsguy@gmail.com)
- `GMAIL_PASSWORD` - Your Gmail password (or app-specific password if 2FA enabled)

#### Google Authentication:
- `WORKLOAD_IDENTITY_PROVIDER` - Google Cloud workload identity provider
- `SERVICE_ACCOUNT_EMAIL` - Google Cloud service account email

#### Google Sheets:
- `SHEET_ID` - Your Google Sheet ID (from the URL: `https://docs.google.com/spreadsheets/d/{SHEET_ID}/`)

#### Notifications (Optional):
- `DISCORD_WEBHOOK_URL` - Discord webhook for email send summaries

### 3. Gmail Setup

**If your Gmail account has 2-factor authentication:**
1. Go to myaccount.google.com/apppasswords
2. Select "Mail" and "Windows Computer" (or your device)
3. Copy the app password provided
4. Use this app password as `GMAIL_PASSWORD` secret

**If your Gmail account does NOT have 2FA:**
- Use your regular Gmail password as `GMAIL_PASSWORD`
- Optional: Enable "Less secure app access" in Gmail security settings

### 4. Google Cloud Setup

1. Create a Google Cloud service account
2. Enable "Workload Identity Federation" 
3. Create a workload identity provider for GitHub Actions
4. Grant the service account permission to edit your Google Sheet

### 5. Workflow Behavior

The workflow runs **every hour** automatically:
- Reads "new" leads from Leads and Referrals sheets
- Validates emails (checks format, MX records, blocked domains)
- Sends personalized emails via Outlook
- Logs all attempts to Outreach_Log sheet
- Updates outreach_status to "contacted" on success
- Monitors bounce rate and stops if it exceeds 5%
- Has gradual ramp-up: starts at 150 emails/day, increases by 25 each day to max 300/day

### 6. Email Templates

The script uses two templates:
- **Leads** - "Administrative Support" pitch for businesses
- **Referrals** - "Connection" pitch for referral partners

Templates include:
- Business name personalization
- HTML formatting with logo
- Professional signature with contact info
- Unsubscribe notice

### 7. Safety Features

- **Bounce rate monitoring** - stops if >5% of emails hard bounce
- **Domain validation** - checks for valid MX records
- **Email validation** - rejects role addresses, blocked domains, invalid formats
- **Rate limiting** - max 10 emails to same domain per run, retry on MS Graph 429 errors
- **Consecutive failure protection** - stops if 5+ emails fail in a row

### 8. Monitoring

Enable Discord notifications by setting `DISCORD_WEBHOOK_URL`. Each run sends:
- Number of emails sent (leads + referrals breakdown)
- Status of email validation
- List of recipients
- Any alerts or errors

### 9. Testing

To test the email sending locally:
```bash
export SENDER_EMAIL="b2bleadsguy@gmail.com"
export GMAIL_PASSWORD="your-gmail-password-or-app-password"
export SHEET_ID="your-sheet-id"
export TEST_MODE="true"
export TEST_EMAIL="your-personal-email@gmail.com"

python send_emails.py
```

This will send one test email to `TEST_EMAIL` and mark it as "sent_test" in the log.

## Troubleshooting

**Workflow not running?**
- Check GitHub Actions are enabled (Settings → Actions → General)
- Verify all required secrets are set
- Check workflow permissions (Settings → Actions → General → Workflow permissions)

**Emails not being sent?**
- Check Discord notifications for errors
- Verify outreach_status = "new" in your sheet
- Check email addresses are in the "email" column
- Review Outreach_Log for validation errors

**High bounce rate?**
- Check for typos in email addresses
- Verify you're targeting the right contacts
- Review bounced emails in Outreach_Log for patterns

## Support

For issues, check:
1. GitHub Actions logs (Actions tab → click the failed run)
2. Discord notifications (if enabled)
3. Outreach_Log sheet for detailed email status
4. send_emails.py comments for implementation details
