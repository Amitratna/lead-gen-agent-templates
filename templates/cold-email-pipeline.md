# Cold Email Research & Personalization Pipeline

## What It Does
Takes a list of leads, research each one with AI, and generates personalized cold emails with subject lines — ready for sending through any email platform.

## Target Clients
- B2B sales teams
- Agency owners doing outreach for clients
- Recruiters sourcing candidates
- Real estate investors buying properties

## Workflow Flow
```
[Lead List CSV] → [Company Research] → [Contact Finder] → [AI Personalization] → [Email Drafts] → [CRM / Sending Queue]
```

## Components

### 1. Lead Import
- CSV upload with name, company, title, LinkedIn URL, or website
- Validates data, flags missing fields
- Deduplicates against existing database

### 2. Company Research Agent
For each company, gather:
- Recent news/press releases (last 90 days)
- Funding rounds, acquisitions, expansions
- Job postings (signal of growth/churn)
- Tech stack (from job posts or builtwith.com)
- Key challenges (from reviews, news)

### 3. Contact Discovery
- Find work email format (first.last@company.com)
- Verify email with ZeroBounce/Apollo API
- Find phone number if needed
- Return full contact record

### 4. Hyper-Personalization Engine
Generate per contact:
- Subject line (specific, not clickbait)
- Opening line (references specific trigger)
- Body (2-3 short paragraphs, value-driven)
- CTA (single clear next step)
- Signature line (optional personalized touch)

### 5. Quality Control
- AI checks for spam signals
- Reads for tone (not too salesy)
- Verifies personalization references are accurate
- Scores email 1-10 before sending

## n8n Workflow JSON

```json
{
  "name": "Cold Email Research Pipeline",
  "nodes": [
    {
      "name": "Trigger: New CSV Upload",
      "type": "n8n-nodes-base.trigger",
      "typeVersion": 1,
      "position": [250, 300]
    },
    {
      "name": "Parse CSV",
      "type": "n8n-nodes-base.readBinaryFile",
      "parameters": {}
    },
    {
      "name": "Company Research",
      "type": "HTTP Request",
      "parameters": {
        "url": "https://api.perplexity.ai/search",
        "method": "POST",
        "authentication": "genericCredentialType",
        "genericAuthType": "apiKey",
        "body": {
          "query": "Research {{ $json.company }}: recent news, funding, growth, hiring signals, challenges in the industry.",
          "recency_days": 90,
          "focus": "business intelligence"
        }
      }
    },
    {
      "name": "Find Contact Email",
      "type": "Apollo.io",
      "parameters": {
        "operation": "findEmail",
        "companyName": "{{ $json.company }}",
        "firstName": "{{ $json.firstName }}",
        "lastName": "{{ $json.lastName }}",
        "domain": "{{ $json.domain }}"
      }
    },
    {
      "name": "AI Email Writer",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "You are an expert B2B copywriter specializing in cold emails that get responses."},
          {"role": "user", "content": "Write a personalized cold email for:\n\nContact: {{ $json.firstName }} {{ $json.lastName }}, {{ $json.title }}\nCompany: {{ $json.company }}\nCompany Intel: {{ $json.companyResearch }}\nMy Value Prop: {{ $env.EMAIL_VALUE_PROP }}\n\nRequirements:\n- Subject line (under 50 chars, specific, curiosity-driven)\n- Opening line references specific company intel\n- 2-3 short paragraphs\n- Single clear CTA\n- Under 200 words total\n- Sound human, not AI-generated\n\nReturn JSON: {\"subject\": \"...\", \"body\": \"...\", \"personalization_note\": \"why this works\"}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Email Quality Score",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o-mini",
        "messages": [
          {"role": "system", "content": "Score this email 1-10 for: spam risk, personalization quality, clarity, tone. Return score and brief feedback."},
          {"role": "user", "content": "{{ $json.emailDraft }}"}
        ]
      }
    },
    {
      "name": "Filter Low Scores",
      "type": "IF",
      "parameters": {
        "conditions": {"options": {"caseSensitive": true}, "conditions": [{"id": "score", "leftValue": "{{ $json.qualityScore }}", "rightValue": 7, "operator": ">="}]}
      }
    },
    {
      "name": "Push to HubSpot",
      "type": "HubSpot",
      "parameters": {
        "operation": "createContact",
        "email": "{{ $json.email }}",
        "firstname": "{{ $json.firstName }}",
        "lastname": "{{ $json.lastName }}",
        "properties": {
          "email_personalized_body": "{{ $json.emailDraft.body }}",
          "email_subject": "{{ $json.emailDraft.subject }}",
          "lead_score": "{{ $json.qualityScore }}",
          "company_research": "{{ $json.companyResearch }}"
        }
      }
    }
  ]
}
```

## Key Prompts

### Company Research Prompt
```
Research [COMPANY] and return structured data:

1. NEWS: Any announcements, funding, product launches, or press from the last 90 days?

2. GROWTH SIGNALS: Hired recently? Expanding? New office locations?

3. CHALLENGES: Any negative reviews, layoffs, or industry headwinds mentioned in news?

4. TRIGGERS: What's the most specific, timely hook I could use in an outreach email?

Format response as:
{ "news": "...", "growth": "...", "challenges": "...", "best_trigger": "..." }
```

### Email Personalization Prompt
```
Write a cold email for {{ firstName }} {{ lastName }}, {{ title }} at {{ company }}.

Company Intel:
{{ companyResearch }}

My Offer:
{{ valueProp }}

Rules:
- Subject line must be under 50 characters and reference something specific
- Opening sentence must mention something unique about THEM or their company
- No "I hope this finds you well" openings
- Body is max 150 words
- CTA is ONE specific action
- Sign off with a real name (not a template)

Return:
{
  "subject": "...",
  "opening": "...",
  "body": "...",
  "cta": "..."
}
```

## Success Metrics
- Open rate: 25%+ (target: 35%+)
- Reply rate: 5%+ (target: 8%+)
- Meeting booked: 1-2% of emails sent

## Pricing for Clients
- Setup: $1,500-2,500
- Per 500 leads processed: $200 (or included in $300/mo retainer)
- Monthly management: $300-500 (unlimited leads within limit)

## Setup Time
- Template ready: ~3 hours
- Per client setup: ~1-2 hours (customize prompts + CRM connection)

## Pro Tips
- Always verify emails before sending (bounce rate kills deliverability)
- Test 3 subject lines, send to 50, pick winner, then scale
- Personalization beats automation — never lose the human touch
- Use Reply.io or Mailshake for sending + tracking