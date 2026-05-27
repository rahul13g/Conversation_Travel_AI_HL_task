# Raju — LetsGo Travel Outbound Voice Agent

An outbound AI voice agent that qualifies travel leads for LetsGo Travel in natural Hinglish. Built on HoomanLabs API, triggered via n8n, and backed by Google Sheets as the CRM.

---

## What It Does

When a new lead is added to the Google Sheet, n8n fires a task to HoomanLabs. Raju calls the lead, has a natural Hinglish discovery conversation to understand their trip intent, and books a callback with a human travel expert. Post-call data is written back to the sheet automatically.

Raju does not book trips, quote prices, or give visa details. His only job is to qualify interest and lock a callback.

### Required fields

| Field | Type | Required |
|---|---|---|
| ID | string | Yes |
| Name | string | Yes |
| Phone Number | string | Yes |
| Status | New | Yes |
---

## Architecture

```
Google Sheet (new lead row)
        ↓
    n8n Trigger
        ↓
POST /routes/v1/tasks/  →  HoomanLabs
        ↓
    Raju calls the lead
        ↓
    Call ends → callEndConnected webhook
        ↓
    n8n post-call workflow
        ↓
Google Sheet (row updated with call outcome)
```

---

## Agent Flow

```
START
  ├── engaged          → DISCOVERY
  ├── not_answered     → END_NOT_AVAILABLE
  └── wrong_number     → END_CALL

DISCOVERY
  ├── ready_to_pitch   → PITCH
  ├── callback_requested → CALLBACK_BOOK
  ├── not_interested   → END_NOT_INTERESTED
  └── dnd_requested    → END_DND

PITCH
  ├── interested       → CALLBACK_BOOK
  ├── objection        → OBJECTION
  ├── not_interested   → END_NOT_INTERESTED
  └── switch_destination → DISCOVERY

OBJECTION
  ├── recovered        → CALLBACK_BOOK
  └── lost             → END_NOT_INTERESTED

CALLBACK_BOOK
  ├── booked           → END_SUCCESS
  └── refused          → END_NOT_INTERESTED

END_SUCCESS / END_NOT_AVAILABLE / END_NOT_INTERESTED / END_DND
  └── all route →      END_CALL (terminal)
```

### Node Types
| Node | Type | Purpose |
|---|---|---|
| START | LLM | Identity confirmation, availability check |
| DISCOVERY | LLM | Qualify trip intent across 5 dimensions |
| PITCH | LLM | Present 2 tailored destination highlights |
| OBJECTION | LLM | Handle one concern, one attempt only |
| CALLBACK_BOOK | LLM | Lock in a specific callback day and time |
| END_SUCCESS | Fixed | Confirm callback, fire post-call webhook |
| END_NOT_AVAILABLE | Fixed | Polite exit — not answered or busy |
| END_NOT_INTERESTED | Fixed | Polite exit — declined |
| END_DND | Fixed | DNC acknowledgment |
| END_CALL | Fixed | Terminal node — no speech, no transitions |

---

## Discovery Order

Raju always asks in this order, one question at a time:
1. Occasion or purpose of the trip
2. Who is travelling — solo, couple, family, friends
3. Destination — specific or open to suggestions
4. When they want to travel
5. Budget — always last, never first

---

## Call Outcomes

| Outcome | Description |
|---|---|
| `callback_booked` | Callee agreed to a callback with confirmed day and time |
| `callback_requested_busy` | Callee was busy, asked to be called later |
| `interested_no_callback` | Callee showed interest but did not commit |
| `not_interested` | Callee clearly not interested |
| `dnd` | Callee asked not to be contacted again |
| `wrong_number` | Callee was not the intended recipient |

---

## Post-Call Data Extracted

Every connected call extracts the following via Gemini:

| Field | Type | Description |
|---|---|---|
| `destination` | string | Trip destination discussed |
| `destination_type` | string | domestic or international |
| `occasion` | string | Honeymoon, family vacation, solo break, etc. |
| `group_type` | string | solo, couple, family, friends |
| `travel_month` | string | When they want to travel |
| `budget` | number | Budget in INR as full integer |
| `package_tier` | string | budget, standard, premium, luxury |
| `interest_level` | string | hot, warm, cold |
| `callback_day` | string | Day agreed for callback |
| `callback_time` | string | Time agreed for callback |
| `objection_type` | string | price, trust, timing, comparison, none |
| `discussion_note` | string | What the caller is weighing |

---

## Triggering a Call (n8n Task API)

```http
POST https://api.hoomanlabs.com/routes/v1/tasks/
Authorization: YOUR_HL_API_KEY
Content-Type: application/json

{
  "phone": "+91XXXXXXXXXX",
  "agent": "YOUR_AGENT_ID",
  "campaign": "YOUR_CAMPAIGN_ID",
  "start": 1000,
  "end": 1900,
  "timezone": "Asia/Kolkata",
  "retries": 3,
  "priorities": [3, 2, 1],
  "intervals": [1800000, 3600000],
  "context": {
    "name": "Lead Name",
    "leadId": "row_id"
  }
}
```

---

## Repository Structure

```
├── new-agent.json              # HoomanLabs agent config (push via API)
├── flow-2.md                   # Agent flow spec (source of truth)
├── pre-call-workflow.json      # n8n pre-call workflow template
├── post-call-workflow.json     # n8n post-call workflow template
├── google-sheets-schema.md     # CRM sheet column definitions
├── .env.example                # Environment variable template
└── README.md                   # This file
```

---

## Key Guardrails

- Never quotes prices, visa timelines, or availability — defers to expert
- Never ends call directly — always uses the correct flow transition
- Never books or finalises a trip — only books a callback
- Honours DNC immediately with no second attempt
- Holds Hinglish 60/40 regardless of caller's language
- Accepts any callback time within 10 AM – 7 PM IST without questioning
