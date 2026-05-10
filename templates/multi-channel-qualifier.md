# Multi-Channel Lead Qualification Agent

## What It Does
Monitors prospects across email, LinkedIn, website visits, and intent signals — then qualifies and routes them to the right follow-up based on behavior and engagement.

## Target Clients
- SaaS companies with inbound + outbound leads
- Agencies managing multiple client campaigns
- Sales teams with long B2B cycles (30-90 days)
- Investment firms sourcing deals

## Workflow Flow
```
[Lead Source: Web Form / CRM / LinkedIn / Referral]
    ↓
[Data Enrichment: Company + Contact Info]
    ↓
[Intent Monitoring: Website visits, Content downloads, Job changes]
    ↓
[Engagement Scoring: Opens, Clicks, Replies, Social interactions]
    ↓
[Qualification: BANT / MEDDIC criteria]
    ↓
[Routing: SDR call, Demo booked, Nurture sequence, Archive]
```

## Components

### 1. Lead Intake
- Web form submissions (name, email, company, message)
- CRM import (HubSpot, Salesforce, Pipedrive)
- LinkedIn connections (parsed for company/title)
- Referral data (who referred them)

### 2. Data Enrichment Layer
For each lead, automatically fetch:
- Company: funding, headcount, tech stack, recent news
- Contact: verified email, phone, LinkedIn, job history
- Intent: G2 reviews, job postings, website engagement (if tracking installed)
- Social: recent posts, comments, engagement patterns

### 3. Multi-Channel Intent Monitor
Track across channels:
- Email: opens, clicks, replies, forwards
- LinkedIn: profile views, post engagement, connection status
- Web: pages visited, time on site, form submissions
- Intent tools: Bombora, G2, Clearbit signals

### 4. Engagement Scoring Engine
Score 0-100 based on:
| Signal | Weight | Points |
|--------|--------|--------|
| Opens email | 10% | 0-10 |
| Clicks link | 15% | 0-15 |
| Visits pricing page | 20% | 0-20 |
| Downloads content | 15% | 0-15 |
| LinkedIn engaged | 10% | 0-10 |
| Demo request | 30% | 0-30 |

### 5. Qualification Agent
Score leads against BANT or MEDDIC:
- **Budget**: Company size, funding, tech stack signals
- **Authority**: Job title, decision-maker indicators
- **Need**: Pain points from form/chat, industry challenges
- **Timeline**: Intent signals, job postings, expansion news

Output: Hot / Warm / Cool / Nurture / Disqualify

### 6. Smart Routing
Based on qualification output:
- **Hot (90+)**: Alert SDR via Slack, book demo immediately
- **Warm (60-89)**: Send high-value content, schedule call
- **Cool (30-59)**: Enter nurture sequence (3-touch over 2 weeks)
- **Nurture (0-29)**: Long-term drip campaign
- **Disqualify**: Archive, no follow-up

## n8n Workflow JSON

```json
{
  "name": "Multi-Channel Lead Qualifier",
  "nodes": [
    {
      "name": "Webhook: New Lead",
      "type": "n8n-nodes-base.webhook",
      "parameters": {"path": "new-lead", "responseMode": "onReceived", "options": {}}
    },
    {
      "name": "Enrich Contact Data",
      "type": "Clearbit",
      "parameters": {
        "operation": "autocomplete",
        "email": "{{ $json.email }}"
      }
    },
    {
      "name": "Enrich Company Data",
      "type": "Clearbit",
      "parameters": {
        "operation": "company",
        "domain": "{{ $json.companyDomain }}"
      }
    },
    {
      "name": "Check Intent Signals",
      "type": "HTTP Request",
      "parameters": {
        "url": "https://api.apollo.io/v1/contacts/{{ $json.email }}/intent_signals",
        "authentication": "genericCredentialType",
        "genericAuthType": "apiKey"
      }
    },
    {
      "name": "Calculate Lead Score",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o-mini",
        "messages": [
          {"role": "system", "content": "Score this lead 0-100 based on engagement and intent signals. Return JSON with score, tier (hot/warm/cool/nurture/disqualify), and recommended action."},
          {"role": "user", "content": "Lead Data:\nName: {{ $json.name }}\nTitle: {{ $json.title }}\nCompany: {{ $json.company }}\nCompany Size: {{ $json.employeeCount }}\nFunding: {{ $json.companyFunding }}\nEmail Opens: {{ $json.emailOpens }}\nEmail Clicks: {{ $json.emailClicks }}\nWebsite Visits: {{ $json.pageViews }}\nPricing Page Visits: {{ $json.pricingPageViews }}\nContent Downloads: {{ $json.downloads }}\nLinkedIn Engagement: {{ $json.linkedInEngagement }}\n\nIntent Signals:\n{{ $json.intentSignals }}\n\nReturn: {\"score\": 0-100, \"tier\": \"hot/warm/cool/nurture\", \"action\": \"next step\", \"reason\": \"why\"}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Route by Tier",
      "type": "Switch",
      "parameters": {
        "dataType": "string",
        "valueInput": "{{ $json.tier }}",
        "cases": {
          "cases": [
            {"value": "hot", "output": 0},
            {"value": "warm", "output": 1},
            {"value": "cool", "output": 2},
            {"value": "nurture", "output": 3}
          ]
        },
        "fallbackOutput": 4
      }
    },
    {
      "name": "Hot: Slack Alert + CRM",
      "type": "Slack",
      "parameters": {
        "channel": "{{ $env.SLACK_CHANNEL }}",
        "text": "🔥 HOT LEAD: {{ $json.name }} ({{ $json.title }}) at {{ $json.company }}. Score: {{ $json.score }}. Intent: {{ $json.topIntent }}. Book demo now: [Link]"
      }
    },
    {
      "name": "Warm: Add to Sequence",
      "type": "HubSpot",
      "parameters": {
        "operation": "addToWorkflow",
        "workflowId": "{{ $env.HUBSPOT_WARM_WORKFLOW_ID }}",
        "email": "{{ $json.email }}"
      }
    },
    {
      "name": "Cool: Nurture Sequence",
      "type": "HubSpot",
      "parameters": {
        "operation": "addToWorkflow",
        "workflowId": "{{ $env.HUBSPOT_COOL_WORKFLOW_ID }}",
        "email": "{{ $json.email }}"
      }
    },
    {
      "name": "Update CRM Score",
      "type": "HubSpot",
      "parameters": {
        "operation": "updateContact",
        "email": "{{ $json.email }}",
        "properties": {
          "lead_score": "{{ $json.score }}",
          "lead_tier": "{{ $json.tier }}",
          "lead_action": "{{ $json.action }}",
          "lead_reason": "{{ $json.reason }}"
        }
      }
    }
  ]
}
```

## Key Prompts

### Lead Scoring Prompt
```
Score this lead from 0-100 for sales readiness.

Contact Info:
- Name: {{ name }}
- Title: {{ title }}
- Company: {{ company }}
- Company Size: {{ employees }}
- Industry: {{ industry }}

Engagement:
- Email opens: {{ opens }}
- Email clicks: {{ clicks }}
- Page views: {{ views }}
- Pricing page visits: {{ pricingVisits }}
- Content downloads: {{ downloads }}
- Demo requested: {{ demoRequested }}

Intent Signals:
- Recent funding: {{ funding }}
- Hiring growth: {{ hiringSignals }}
- Tech expansion: {{ techSignals }}
- G2/Review activity: {{ reviewActivity }}

Return JSON:
{
  "score": 0-100,
  "tier": "hot|warm|cool|nurture|disqualify",
  "top_reason": "why this score",
  "recommended_action": "specific next step",
  "urgency": "high|medium|low"
}
```

### Hot Lead Alert Prompt
```
Create a Slack alert for a hot lead. Include:
- Lead name and company
- Why they're hot (top 2 signals)
- Best talking points (from research)
- Suggested opener for the SDR
- One question to ask on the call
```

## Integration Setup

### Required
- n8n or Make.com
- OpenAI API (gpt-4o-mini sufficient)
- HubSpot, Salesforce, or Pipedrive
- Apollo.io or Clearbit for enrichment
- Slack for hot lead alerts

### Optional
- Bombora (intent data)
- G2 (intent signals)
- LinkedIn Sales Navigator (outreach triggers)
- ZeroBounce (email verification)

## Success Metrics
- Lead response time: <5 minutes for hot leads
- Qualified pipeline increase: 30-50%
- SDR time savings: 2-3 hours/week per rep
- Cold-to-booked rate: 5-10% improvement

## Pricing for Clients
- Setup: $3,000-5,000
- Monthly management: $800-1,200/month
- Enterprise (multiple users + complex routing): $2,000-5,000/month

## Setup Time
- Template ready: ~5 hours
- Per client setup: ~3 hours (CRM connections, routing rules, scoring weights)
- Calibration period: 2-4 weeks to tune thresholds