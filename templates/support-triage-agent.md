# Customer Support Triage Agent

## What It Does
Automatically handle tier-1 support tickets — answers common questions, classifies issues, and routes complex problems to the right team. Handles 60-80% of tickets without human intervention.

## Target Clients
- SaaS companies with high ticket volume
- E-commerce brands (pre/post-purchase questions)
- Agencies managing client support
- Any product with repeatable FAQ patterns

## Workflow Flow
```
[Incoming Ticket: Email / Chat / Helpdesk]
    ↓
[AI Ticket Classifier: Category + Priority + Sentiment]
    ↓
[Knowledge Base Lookup: Match to existing answer]
    ↓
[Response Generator: Draft reply based on context]
    ↓
[Quality Check: Is response accurate + helpful?]
    ↓
[Auto-Reply OR Route to Human: Based on complexity]
    ↓
[Follow-up Check: Did it resolve the issue?]
```

## Components

### 1. Ticket Ingestion
Connect to:
- Email (Gmail, Outlook, support@ inbox)
- Live chat (Intercom, Zendesk, Drift)
- Helpdesk (Freshdesk, HelpScout, Jira)
- Facebook/Instagram DMs
- WhatsApp business

### 2. AI Ticket Classifier
Analyze each ticket and assign:
- **Category**: Billing, Technical, Feature Request, Refund, General
- **Priority**: P1 (outage), P2 (blocked), P3 (inconvenient), P4 (question)
- **Sentiment**: Positive, Neutral, Frustrated, Angry
- **Complexity**: Simple (auto-reply), Medium (needs context), Complex (human needed)
- **Product Area**: Auth, Dashboard, Integrations, Billing, etc.

### 3. Knowledge Base Integration
- Search existing help docs
- Match to similar resolved tickets
- Pull product documentation
- Find related FAQ articles

### 4. Response Generator
Draft replies that:
- Answer the specific question (not generic)
- Match company tone/voice
- Include relevant links/docs
- Offer next steps if unresolved
- Add ticket reference number

### 5. Smart Routing
| Ticket Type | Action |
|-------------|--------|
| Refund request | Route to billing team |
| Bug report | Create Jira ticket, notify dev |
| Feature request | Log in product board, auto-reply |
| Billing question | Auto-reply with explanation |
| Complaint | Route to senior agent + flag |
| Simple FAQ | Auto-reply immediately |
| Complex technical | Route to support engineer |

### 6. Resolution Follow-up
- Auto-send satisfaction survey (after 24 hours)
- If "thumbs down", escalate to human
- Track resolution rate per category

## n8n Workflow JSON

```json
{
  "name": "Support Triage Agent",
  "nodes": [
    {
      "name": "Email Trigger",
      "type": "Email Trigger",
      "parameters": {"mailboxId": "{{ $env.IMAP_ID }}"}
    },
    {
      "name": "Classify Ticket",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Classify this support ticket and determine the best action. Return JSON."},
          {"role": "user", "content": "Classify ticket:\n\nSubject: {{ $json.subject }}\nBody: {{ $json.body }}\nFrom: {{ $json.from }}\n\nCategories: billing, technical, feature_request, refund, general\nPriority: p1, p2, p3, p4\nSentiment: positive, neutral, frustrated, angry\nComplexity: simple, medium, complex\n\nReturn: {\"category\": \"...\", \"priority\": \"...\", \"sentiment\": \"...\", \"complexity\": \"...\", \"route_to\": \"...\", \"reason\": \"why classification\"}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Knowledge Base Search",
      "type": "HTTP Request",
      "parameters": {
        "url": "https://api.openai.com/v1/chat/completions",
        "method": "POST",
        "authentication": "genericCredentialType",
        "genericAuthType": "apiKey",
        "body": {
          "model": "gpt-4o",
          "messages": [
            {"role": "system", "content": "You are a support knowledge base. Answer based on provided docs. If no answer exists, say 'NO_MATCH'."},
            {"role": "user", "content": "Customer question: {{ $json.body }}\n\nFind matching article from this knowledge base:\n{{ $json.knowledgeBase }}\n\nReturn: {\"match\": true/false, \"article\": \"...\", \"answer\": \"...\"}"}
          ]
        }
      }
    },
    {
      "name": "Route by Complexity",
      "type": "Switch",
      "parameters": {
        "dataType": "string",
        "valueInput": "{{ $json.complexity }}",
        "cases": {
          "cases": [
            {"value": "simple", "output": 0},
            {"value": "medium", "output": 1},
            {"value": "complex", "output": 2}
          ]
        }
      }
    },
    {
      "name": "Simple: Auto Reply",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Write a friendly, helpful support reply. Be conversational, not robotic. Include relevant links. End warmly."},
          {"role": "user", "content": "Customer: {{ $json.fromName }}\nQuestion: {{ $json.body }}\nKB Article: {{ $json.answer }}\nProduct: {{ $env.PRODUCT_NAME }}\nTone: {{ $env.SUPPORT_TONE }}\n\nWrite a reply that:\n- Answers their specific question\n- References their exact issue\n- Includes helpful links\n- Has a signature: {{ $env.SUPPORT_SIGNATURE }}\n- Under 150 words"}
        ]
      }
    },
    {
      "name": "Send Auto Reply",
      "type": "Email",
      "parameters": {
        "to": "{{ $json.from }}",
        "subject": "Re: {{ $json.subject }}",
        "body": "{{ $json.draftResponse }}"
      }
    },
    {
      "name": "Complex: Route to Human",
      "type": "Slack",
      "parameters": {
        "channel": "{{ $env.SUPPORT_SLACK_CHANNEL }}",
        "text": "🎫 Human ticket needed:\n\nFrom: {{ $json.fromName }}\nSubject: {{ $json.subject }}\nPriority: {{ $json.priority }}\nSentiment: {{ $json.sentiment }}\nCategory: {{ $json.category }}\n\nIssue:\n{{ $json.body | truncate(500) }}\n\nKB Answer (didn't fully resolve): {{ $json.answer }}"
      }
    },
    {
      "name": "Log to Helpdesk",
      "type": "Zendesk",
      "parameters": {
        "operation": "createTicket",
        "subject": "{{ $json.subject }}",
        "comment": "{{ $json.body }}",
        "priority": "{{ $json.priority }}",
        "tags": ["ai-triaged", "{{ $json.category }}"]
      }
    }
  ]
}
```

## Key Prompts

### Ticket Classifier
```
Classify this support ticket:

Subject: {{ subject }}
Body: {{ body }}

Return JSON:
{
  "category": "billing|technical|feature_request|refund|general|cancellation",
  "priority": "p1|p2|p3|p4",
  "sentiment": "positive|neutral|frustrated|angry",
  "complexity": "simple|medium|complex",
  "product_area": "auth|dashboard|billing|integrations|api|general",
  "route_to": "auto_reply|billing_team|support_agent|dev_team|customer_success|manager",
  "reason": "brief explanation of classification"
}
```

### Auto-Reply Generator
```
Write a support reply for:

Customer name: {{ customerName }}
Their question: {{ body }}
Best KB answer: {{ kbAnswer }}
Product name: {{ productName }}
Our tone: {{ supportTone }}

Rules:
- Start with their name
- Address their specific question (not generic)
- If KB has answer, reference it specifically
- Include relevant links (docs, pricing, etc.)
- If their issue isn't fully resolved, explain next steps
- End warmly, offer to help more
- Add signature: {{ signature }}
- Under 150 words
- No corporate speak
```

## Success Metrics
- Ticket deflection rate: 60-80% (target: 75%+)
- Response time: <2 minutes (vs hours)
- CSAT score: Track auto-reply vs human reply
- Resolution rate: Track whether auto-replies actually solve issues

## Pricing for Clients
- Setup: $2,000-3,500
- Monthly management: $400-800/month
- Enterprise (complex routing + multiple products): $1,500-3,000/month

## Setup Requirements
- Helpdesk integration (Zendesk, Intercom, Freshdesk)
- Knowledge base (existing docs or we'll create structure)
- Email/chat channels connected
- 2-4 weeks to train on company voice

## Setup Time
- Template ready: ~4 hours
- Per client setup: ~3 hours (connections + KB setup)
- Training period: 2 weeks (tune accuracy)
- Total to full auto: ~4 weeks