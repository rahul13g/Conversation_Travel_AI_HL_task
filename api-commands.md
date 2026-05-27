# API Commands — LetsGo Raju Agent

Source `.env` before running any command:
```bash
source .env
```

---

## Agent Management

```bash
# Get current agent (saves to new-agent.json)
curl -s -X GET https://api.hoomanlabs.com/routes/v1/agents/$HL_AGENT_ID \
  -H "Authorization: $HL_API_KEY" | jq '.' > new-agent.json

# Push new agent version
curl -s -X POST https://api.hoomanlabs.com/routes/v1/agents/$HL_AGENT_ID \
  -H "Authorization: $HL_API_KEY" \
  -H "Content-Type: application/json" \
  -d @new-agent.json | jq '{version, versionName}'

# Activate a specific version
curl -s -X POST "https://api.hoomanlabs.com/routes/v1/agents/$HL_AGENT_ID/{VERSION_ID}/activate" \
  -H "Authorization: $HL_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}' | jq '.success'

# List all agent versions
curl -s -X GET "https://api.hoomanlabs.com/routes/v1/agents/$HL_AGENT_ID/versions" \
  -H "Authorization: $HL_API_KEY" | jq '.'
```

---

## Campaign Management

```bash
# Create Realtime campaign
curl -s -X POST https://api.hoomanlabs.com/routes/v1/campaigns/ \
  -H "Authorization: $HL_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"name\": \"LetsGo Realtime Campaign\", \"agent\": \"$HL_AGENT_ID\", \"slots\": 5}" | jq '{id, name}'

# List campaigns
curl -s -X GET https://api.hoomanlabs.com/routes/v1/campaigns/ \
  -H "Authorization: $HL_API_KEY" | jq '.'
```

---

## Task Management

```bash
# Create task (trigger a call)
curl -s -X POST https://api.hoomanlabs.com/routes/v1/tasks/ \
  -H "Authorization: $HL_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"phone\": \"+91XXXXXXXXXX\",
    \"agent\": \"$HL_AGENT_ID\",
    \"campaign\": \"$HL_CAMPAIGN_ID\",
    \"from\": \"$HL_FROM_NUMBER\",
    \"start\": 0,
    \"end\": 2359,
    \"timezone\": \"Asia/Kolkata\",
    \"retries\": 3,
    \"priorities\": [3, 2, 1],
    \"intervals\": [1800000, 3600000],
    \"context\": {
      \"name\": \"Lead Name\",
      \"leadId\": \"row_id\"
    }
  }" | jq '{id, status}'
```

---

## Testing

```bash
# POST fake connected-call payload to n8n post-call webhook
curl -s -X POST $N8N_WEBHOOK_POST_CALL \
  -H "Content-Type: application/json" \
  -d '{
    "leadId": "1",
    "name": "Rahul",
    "outcome": "callback_booked",
    "destination": "Bali",
    "destination_type": "international",
    "occasion": "honeymoon",
    "group_type": "couple",
    "travel_month": "December",
    "budget": 150000,
    "package_tier": "premium",
    "interest_level": "hot",
    "callback_day": "Saturday",
    "callback_time": "11 AM",
    "objection_type": "none",
    "discussion_note": "Comparing with Thailand"
  }'
```
