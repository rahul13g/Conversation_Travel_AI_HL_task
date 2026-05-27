# SafarSathi Travels — Google Sheets CRM Schema

## Sheet Name: `Leads`

Create one sheet with the following columns in this exact order.
The workflows match on the **ID** column — it must be unique per lead.

---

## Columns

| # | Column Name | Type | Filled By | Notes |
|---|---|---|---|---|
| 1 | **ID** | Number | You (manually or CRM export) | Unique lead ID. Used to match rows for updates. |
| 2 | **Name** | Text | You | Full name. Agent uses first name only. |
| 3 | **Phone Number** | Text | You | 10-digit Indian mobile number (no country code needed — n8n adds +91) |
| 4 | **Status** | Text | You → n8n | Start as `New`. n8n updates to `Call Scheduled` → `Called` |
| 5 | **Source** | Text | You (optional) | Where lead came from: Website, Facebook Ad, Referral, etc. |
| 6 | **Created At** | Date/Text | You (optional) | When lead was added |
| 7 | **HL Task ID** | Text | n8n (pre-call) | Hooman Labs task ID returned after task creation |
| 8 | **Scheduled At** | DateTime | n8n (pre-call) | Timestamp when HL task was created |
| 9 | **Call Picked Up** | Text | n8n (post-call) | TRUE / FALSE |
| 10 | **Qualification Status** | Text | n8n (post-call) | Hot / Warm / Cold / Not Interested / No Answer / Wrong Number / Callback Requested |
| 11 | **Destination** | Text | n8n (post-call) | e.g. "Bali", "Goa, Kerala", "Not decided" |
| 12 | **Travel Month** | Text | n8n (post-call) | e.g. "December", "Q1 2026", "Flexible" |
| 13 | **Group Type** | Text | n8n (post-call) | solo / couple / family / friends / corporate / unknown |
| 14 | **No. of Travelers** | Text | n8n (post-call) | e.g. "4", "2-3", "Not shared" |
| 15 | **Budget Per Person** | Text | n8n (post-call) | e.g. "₹50k-1L", "Under ₹30k", "Not shared" |
| 16 | **Trip Type** | Text | n8n (post-call) | honeymoon / family / adventure / pilgrimage / leisure / corporate / not_specified |
| 17 | **Callback Time** | Text | n8n (post-call) | e.g. "Tomorrow 5 PM IST" or blank |
| 18 | **Call End Reason** | Text | n8n (post-call) | Raw HL end reason: customer-ended-call, no-answer, voicemail, etc. |
| 19 | **Call Duration (s)** | Number | n8n (post-call) | Duration in seconds |
| 20 | **Notes** | Text | n8n (post-call) | Any key context extracted from transcript |
| 21 | **Updated At** | DateTime | n8n (post-call) | Timestamp of last update |

---

## Sample Lead (Pre-Call — what you add manually)

| ID | Name | Phone Number | Status | Source |
|---|---|---|---|---|
| 1 | Rahul Sharma | 9876543210 | New | Website |
| 2 | Priya Mehta | 9123456789 | New | Facebook Ad |

---

## After Call Completes (n8n fills these in)

| ID | Status | Call Picked Up | Qualification Status | Destination | Travel Month | Group Type | No. of Travelers | Budget Per Person | Trip Type | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Called | TRUE | Hot | Bali | December | couple | 2 | ₹80k-1L | honeymoon | Wants WhatsApp follow-up |
| 2 | Called | FALSE | No Answer | N/A | N/A | unknown | N/A | N/A | not_specified | no-answer |

---

## Status Flow

```
New
  ↓ (n8n pre-call workflow fires)
Call Scheduled
  ↓ (Hooman Labs calls the lead)
Called
  ↓ (n8n post-call workflow fires)
[Qualification Status is filled in]
```

---

## Tips

- **Do not change column names** — the n8n workflows reference them exactly by name
- **ID must be unique** — duplicate IDs will cause the wrong row to be updated
- Keep phone numbers as plain text (not formatted as numbers) to avoid losing leading zeros
- You can add extra columns after column 21 — n8n won't touch them
