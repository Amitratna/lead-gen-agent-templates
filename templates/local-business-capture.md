# Local Business Lead Capture Agent

## What It Does
Automatically capture, qualify, and follow up with leads from local service businesses (plumbers, dentists, HVAC, real estate agents) via website forms, phone calls, and SMS — without the business owner lifting a finger.

## Target Clients
- Home service businesses: plumbers, electricians, HVAC, locksmiths
- Health/beauty: dentists, chiropractors, salons
- Real estate: agents, property managers
- Auto services: mechanics, body shops
- Any local business with "near me" search traffic

## Workflow Flow
```
[Lead Source: Website Form / Phone Call / Google Local / Facebook Lead]
    ↓
[AI Lead Qualifier: Service needed, Budget, Location, Urgency]
    ↓
[Auto Text/Email Response: Instant acknowledgment + next step]
    ↓
[CRM Entry: All details + qualification score]
    ↓
[Route to Owner: Text/Slack with lead summary + suggested response]
    ↓
[Follow-up Sequence: 3 texts + 2 emails over 7 days]
```

## Components

### 1. Multi-Channel Lead Capture
- **Website form**: Name, phone, service needed, zip code, message
- **Google Local**: Integrate with Google Business Messages
- **Facebook Messenger**: Auto-reply + collect info
- **Phone**: Voicemail transcription → AI qualifies
- **SMS**: Text keyword to opt-in (e.g., "PLUMBING" to 555-1234)

### 2. AI Qualification Bot
Instantly qualify via text/chat:
1. What service do you need? (dropdown or keywords)
2. When do you need it? (today / this week / this month / later)
3. What's your zip code? (service area check)
4. Best number to reach you? (if not captured)

Output: Hot (book now), Warm (schedule), Cool (info only), Out of Area

### 3. Auto-Response Engine
Send immediately (<30 seconds):
- Confirmation of receipt
- What happens next (e.g., "We'll text you within 15 min")
- Option to call/text directly
- Social proof (e.g., "Serving Phoenix since 2010")

### 4. Owner Alert System
Send to business owner via SMS/Slack:
```
New Lead: Sarah M.
Service: Water heater replacement
Zip: 85001 (in-service area)
Urgency: This week
Score: 🔥 HOT
Quote: $800-1,500
Suggested reply: "Hi Sarah! We can be there Monday. Want to schedule?"
```

### 5. Follow-up Sequences
| Tier | Day 1 | Day 2 | Day 3 | Day 5 | Day 7 |
|------|-------|-------|-------|-------|-------|
| Hot | Call + Text | Text | Call | Text | Final call |
| Warm | Text | Text | Email | Text | Email |
| Cool | Email | - | Email | - | Final email |

## n8n Workflow JSON

```json
{
  "name": "Local Business Lead Capture",
  "nodes": [
    {
      "name": "Webhook: New Lead",
      "type": "n8n-nodes-base.webhook",
      "parameters": {"path": "local-lead", "responseMode": "immediate", "options": {}}
    },
    {
      "name": "AI Qualifier - Service",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o-mini",
        "messages": [
          {"role": "system", "content": "You are a friendly service coordinator. Classify the service request and extract key details."},
          {"role": "user", "content": "Lead info: Name={{ $json.name }}, Phone={{ $json.phone }}, Message='{{ $json.message }}'\n\nClassify: plumbing|electrical|HVAC|dental|real_estate|other\nExtract: service_type, urgency (now/this_week/this_month/later), budget_range (if mentioned)\n\nReturn JSON: {\"service\": \"...\", \"urgency\": \"...\", \"budget\": \"...\", \"hot_score\": 0-100}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Zip Code Service Area Check",
      "type": "HTTP Request",
      "parameters": {
        "url": "https://zipcodeapi.com/rest/{{ $env.ZIP_API_KEY }}/info.json/{{ $json.zip }}/degrees",
        "method": "GET"
      }
    },
    {
      "name": "Route by Qualification",
      "type": "IF",
      "parameters": {
        "conditions": [
          {"id": "inServiceArea", "leftValue": "{{ $json.inServiceArea }}", "rightValue": true, "operator": "equals"},
          {"id": "hotScore", "leftValue": "{{ $json.hotScore }}", "rightValue": 70, "operator": "greater"}
        ],
        "options": {"combineType": "AND"}
      }
    },
    {
      "name": "Auto Reply SMS",
      "type": "Twilio",
      "parameters": {
        "operation": "sendSMS",
        "from": "{{ $env.TWILIO_NUMBER }}",
        "to": "{{ $json.phone }}",
        "message": "Thanks {{ $json.name }}! We got your {{ $json.service }} request. Our team will text you within 15 min. Need it urgent? Call us now: {{ $env.BUSINESS_PHONE }}"
      }
    },
    {
      "name": "Owner Alert - Hot Lead",
      "type": "Twilio",
      "parameters": {
        "operation": "sendSMS",
        "from": "{{ $env.TWILIO_NUMBER }}",
        "to": "{{ $env.OWNER_PHONE }}",
        "message": "🔥 HOT LEAD: {{ $json.name }} needs {{ $json.service }}. Zip: {{ $json.zip }}. Urgency: {{ $json.urgency }}. Phone: {{ $json.phone }}. Score: {{ $json.hotScore }}/100"
      }
    },
    {
      "name": "Log to Airtable",
      "type": "Airtable",
      "parameters": {
        "operation": "create",
        "baseId": "{{ $env.AIRTABLE_BASE_ID }}",
        "table": "Leads",
        "fields": {
          "Name": "{{ $json.name }}",
          "Phone": "{{ $json.phone }}",
          "Service": "{{ $json.service }}",
          "Urgency": "{{ $json.urgency }}",
          "Zip": "{{ $json.zip }}",
          "Score": "{{ $json.hotScore }}",
          "Status": "New",
          "Source": "{{ $json.leadSource }}"
        }
      }
    },
    {
      "name": "Schedule Follow-up Sequence",
      "type": "n8n-nodes-base.scheduleTrigger",
      "parameters": {
        "rule": {"recurring": {"interval": 1}}
      }
    }
  ]
}
```

## Key Prompts

### Initial Auto-Reply (Text)
```
Hi {{ firstName }}! 👋
Got your request for {{ service }}. Here's what happens next:
1. Our team reviews your request (next 15 min)
2. We'll text you a personalized quote
3. Book a time that works for you

Need it urgent? Call us: {{ businessPhone }}
- {{ businessName }}
```

### Owner Alert
```
🚨 {{ tier }} LEAD: {{ name }}
📞 {{ phone }}
📍 Zip: {{ zip }}
🔧 Service: {{ service }}
⏰ Urgency: {{ urgency }}
💰 Est. Budget: {{ budget }}
📊 Score: {{ score }}/100

Suggested reply: "{{ suggestedMessage }}"
```

### Follow-up Sequence
```
Day 1 (if no response):
"Hi {{ firstName }}, just checking in on your {{ service }} request from {{ date }}. We're still available today if you need help!"

Day 3:
"Hi {{ firstName }}, following up on your request. We've got openings this {{ nextAvailableDay }}. Want to lock in a time?"

Day 7:
"Hi {{ firstName }}, last text from us about your {{ service }} request. If you still need help, we're here: {{ phone }}"
```

## Setup Requirements

### Tools Needed
- n8n (free tier works for <1,000 leads/month)
- Twilio ($1/month phone number + $0.01/ SMS)
- OpenAI API (~$5-20/month)
- Airtable (free tier works)

### Total Monthly Cost
- $0-50/month for most local businesses
- Scales with lead volume

## Success Metrics
- Lead response time: <30 seconds (vs. hours)
- Capture rate improvement: 30-50%
- Follow-up completion: 80%+ (vs. 20% manual)
- Quote-to-job conversion: track monthly

## Pricing for Clients
- Setup: $500-1,500
- Monthly management: $150-300/month
- Add-on: SMS campaigns ($50/mo extra)

## Best Niches (by $ per lead)
1. **Plumbers**: $150-500 job value, high volume
2. **HVAC**: $200-800 job value, seasonal
3. **Electricians**: $100-600 job value, urgent needs
4. **Real estate agents**: $5K-50K commission, competitive
5. **Dentists**: $500-5K procedure value

## Setup Time
- Template ready: ~3 hours
- Per client setup: ~1 hour (phone number, CRM setup, owner notification)
- Full automation: ~2 hours