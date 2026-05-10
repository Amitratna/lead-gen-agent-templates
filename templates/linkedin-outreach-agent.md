# LinkedIn Outreach Agent Template

## What It Does
Automatically finds prospects, researches their profile, and generates personalized outreach messages for LinkedIn connection requests + follow-ups.

## Target Clients
- B2B SaaS founders
- Sales teams at agencies
- Recruiters and headhunters
- Real estate agents

## Workflow Flow
```
[LinkedIn Search] → [Profile Scraper] → [AI Research Agent] → [Personalization Engine] → [Outreach Queue]
```

## Components

### 1. Prospect Finder
- Search queries by industry/job title/location
- Filter by: company size, job level, time at company
- Output: list of profile URLs

### 2. Profile Research Agent
Research each prospect and extract:
- Recent posts/content they've shared
- Job changes/promotions
- Company news
- Mutual connections
- Pain points from their bio/headline

### 3. Personalization Engine
Generate connection request message (max 300 chars) that:
- References specific trigger (post, job change, etc.)
- Shows genuine interest
- Has clear CTA (not "let's grab coffee")
- Uses their first name

### 4. Follow-up Sequence
- Day 3: Engage with their latest post
- Day 7: Second connection request (different angle)
- Day 14: If no response, move to cold email

## n8n Workflow JSON

```json
{
  "name": "LinkedIn Outreach Agent",
  "nodes": [
    {
      "name": "Schedule Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "parameters": {
        "rule": {"interval": [{"field": "day", "hours": [9, 10, 11]]]}
      }
    },
    {
      "name": "LinkedIn Search",
      "type": "LinkedIn API",
      "parameters": {
        "operation": "searchProfiles",
        "query": "{{ $json.searchTerms }}",
        "filters": {"industry": "{{ $json.industry }}", "location": "{{ $json.location }}"}
      }
    },
    {
      "name": "Profile Research",
      "type": "HTTP Request",
      "parameters": {
        "url": "https://api.perplexity.ai/search",
        "method": "POST",
        "body": {
          "query": "Research this LinkedIn profile: {{ $json.profileUrl }}. Find recent activity, job history, posts, and any notable achievements.",
          "recency_days": 90
        },
        "authentication": "genericCredentialType",
        "genericAuthType": "apiKey"
      }
    },
    {
      "name": "AI Personalization",
      "type": "OpenAI Node",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o-mini",
        "messages": [
          {"role": "system", "content": "You are a B2B sales copywriter. Write personalized LinkedIn connection requests under 300 chars that feel genuine, not spammy."},
          {"role": "user", "content": "Prospect: {{ $json.name }}, Job: {{ $json.title }} at {{ $json.company }}. Recent activity: {{ $json.researchData }}. Write 3 connection request variations."}
        ]
      }
    },
    {
      "name": "LinkedIn Send Message",
      "type": "LinkedIn API",
      "parameters": {
        "operation": "sendConnectionRequest",
        "profileId": "{{ $json.profileId }}",
        "message": "{{ $json.personalizedMessage }}"
      }
    },
    {
      "name": "Airtable Logger",
      "type": "Airtable",
      "parameters": {
        "operation": "create",
        "baseId": "{{ $env.AIRTABLE_BASE_ID }}",
        "table": "Prospects",
        "fields": {
          "Name": "{{ $json.name }}",
          "Status": "Outreach Sent",
          "Profile URL": "{{ $json.profileUrl }}",
          "Personalized Message": "{{ $json.personalizedMessage }}"
        }
      }
    }
  ]
}
```

## Key Prompts

### Research Prompt
```
Research this LinkedIn profile and return a JSON object with:
- company_info: recent news, funding, growth
- profile_activity: recent posts, engagement, topics
- personal_signals: job changes, promotions, milestones
- conversation_starters: 3 specific things to mention

Profile: {{ prospect_url }}
```

### Personalization Prompt
```
Write a LinkedIn connection request (under 300 characters) for:
Name: {{ name }}
Title: {{ title }}
Company: {{ company }}
Industry: {{ industry }}
Personalization trigger: {{ trigger_from_research }}

Rules:
- Mention something specific (post, achievement, company news)
- No generic "I'd love to connect" openings
- End with a simple question or clear reason to connect
- Sound like a human, not a template
```

## Success Metrics
- Connection acceptance rate: 30%+ (target: 40%+)
- Response rate: 10%+ (target: 15%+)
- Meeting booked rate: 3%+

## Pricing for Clients
- Setup: $2,000-3,000
- Monthly management: $500-800
- Includes: 50 prospects/week, monitoring, optimization

## Setup Time
- Template ready: ~4 hours
- Per client customization: ~2 hours