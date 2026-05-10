# Recruiting & Candidate Sourcing Agent

## What It Does
Automate candidate sourcing for recruiters and hiring managers — finds qualified candidates, enriches their profiles, scores them against job requirements, and drafts personalized outreach messages.

## Target Clients
- Recruiters / Headhunters (agency or in-house)
- HR teams at startups and scale-ups
- Hiring managers doing direct sourcing
- Freelance sourcers working on contingency

## Workflow Flow
```
[Job Requirements: Role + Skills + Location + Experience]
    ↓
[Boolean Search Builder: Generate optimized search queries]
    ↓
[Multi-Platform Search: LinkedIn, Indeed, GitHub, etc.]
    ↓
[Candidate Enrichment: Profile data, company history, social signals]
    ↓
[AI Candidate Scorer: Match against job requirements]
    ↓
[Outreach Generator: Personalized messages per candidate]
    ↓
[CRM Update: Add to talent pipeline with scores]
```

## Components

### 1. Job Requirements Parser
Input:
- Job title (with alternatives)
- Required skills (hard + soft)
- Experience level / years
- Location / remote preference
- Salary range (if public)
- Must-have vs nice-to-have

Output: Structured requirements for search + scoring

### 2. Boolean Search Builder
Generate optimized searches for:
- LinkedIn (title, skills, location, current company)
- GitHub (language, repos, contributions)
- Indeed / Glassdoor (keywords, title variations)
- Twitter (tech hiring hashtags)
- Resume databases (RocketReach, Zoominfo)

Include:
- Boolean strings ready to paste
- Alternative queries for variety
- Site-specific search syntax

### 3. Multi-Platform Candidate Search
For each candidate found, collect:
- Name + current title
- Company (size, industry, growth signals)
- Experience (years, roles, companies)
- Skills (from profile + endorsements)
- Education (school, degree, year)
- Social signals (posts, engagement, job changes)
- Contact info (email, phone if available)
- Availability signals (open to work, recent activity)

### 4. AI Candidate Scorer
Score candidates 0-100 on:
- **Skills Match**: % of required skills present (40% weight)
- **Experience Fit**: Years + relevance (25% weight)
- **Company Fit**: Target companies or similar (15% weight)
- **Location**: Relocation possibility, timezone (10% weight)
- **Engagement**: Active on LinkedIn, recent activity (10% weight)

Output: Tier (A/B/C/D), Match score, Missing qualifications, Red flags

### 5. Outreach Message Generator
Generate per candidate:
- Subject line (personalized, not "Job Opportunity")
- Opening (reference something specific from their profile)
- Body (why this role fits their background)
- CTA (specific next step, not "let's chat")
- Signature (genuine name, real title)

Different versions for:
- Cold outreach (LinkedIn message)
- Email (if email found)
- Referral request (if mutual connection exists)

### 6. Talent Pipeline CRM
Log all candidates with:
- Profile link + photo
- Match score + tier
- Outreach status (sent, opened, replied, scheduled, hired)
- Notes from screening
- Interview feedback

## n8n Workflow JSON

```json
{
  "name": "Recruiting Sourcing Agent",
  "nodes": [
    {
      "name": "Job Requirements Trigger",
      "type": "Webhook",
      "parameters": {"path": "new-job", "responseMode": "onReceived"}
    },
    {
      "name": "Parse Job Requirements",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Parse job requirements into structured format for recruiting search."},
          {"role": "user", "content": "Parse these job requirements:\n\nTitle: {{ $json.title }}\nDescription: {{ $json.description }}\nLocation: {{ $json.location }}\nSalary: {{ $json.salary }}\n\nReturn:\n{\n  \"title_variations\": [\"...\"],\n  \"required_skills\": [\"...\"],\n  \"preferred_skills\": [\"...\"],\n  \"experience_years\": \"min # years\",\n  \"target_companies\": [\"...\"],\n  \"target_schools\": [\"...\"],\n  \"job_level\": \"junior|mid|senior|lead|executive\",\n  \"location_requirements\": \"...\",\n  \"dealbreakers\": [\"...\"],\n  \"compensation_alignment\": \"competitive|below-market|above-market\"\n}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Generate Boolean Searches",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Generate optimized Boolean search strings for LinkedIn recruiter."},
          {"role": "user", "content": "Create Boolean search strings for:\n\nTitle variations: {{ $json.titleVariations }}\nSkills: {{ $json.requiredSkills }}\nExperience: {{ $json.experienceYears }}\nLocation: {{ $json.location }}\n\nReturn:\n{\n  \"linkedin_searches\": [\"string1\", \"string2\", ...],\n  \"indeed_searches\": [\"...\"],\n  \"github_searches\": [\"...\"],\n  \"twitter_searches\": [\"...\"],\n  \"job_boards\": [\"...\"],\n  \"tips\": [\"how to use these searches more effectively\"]\n}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Enrich Candidate Profile",
      "type": "HTTP Request",
      "parameters": {
        "url": "https://api.apollo.io/v1/contacts/enrich",
        "method": "POST",
        "authentication": "genericCredentialType",
        "genericAuthType": "apiKey",
        "body": {
          "email": "{{ $json.email }}",
          "first_name": "{{ $json.firstName }}",
          "last_name": "{{ $json.lastName }}",
          "company": "{{ $json.currentCompany }}"
        }
      }
    },
    {
      "name": "Score Candidate",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Score candidate against job requirements. Return detailed match analysis."},
          {"role": "user", "content": "Score this candidate for:\n\nJob: {{ $json.jobTitle }}\nRequired Skills: {{ $json.requiredSkills }}\nExperience: {{ $json.experienceYears }}+ years\n\nCandidate:\n- Name: {{ $json.candidateName }}\n- Title: {{ $json.candidateTitle }}\n- Company: {{ $json.candidateCompany }}\n- Experience: {{ $json.candidateExperience }}\n- Skills: {{ $json.candidateSkills }}\n- Education: {{ $json.candidateEducation }}\n- LinkedIn Activity: {{ $json.linkedInActivity }}\n- Tenure: {{ $json.jobTenure }}\n\nReturn:\n{\n  \"overall_score\": 0-100,\n  \"tier\": \"A|B|C|D\",\n  \"skills_match\": {\"score\": #, \"matched\": [], \"missing\": []},\n  \"experience_fit\": {\"score\": #, \"fit_notes\": \"...\"},\n  \"red_flags\": [\"...\"],\n  \"strengths\": [\"...\"],\n  \"availability_signals\": \"...\",\n  \"recommendation\": \"contact|maybe|skip\"\n}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Filter Tier C+",
      "type": "IF",
      "parameters": {
        "conditions": [{"id": "score", "leftValue": "{{ $json.overallScore }}", "rightValue": 60, "operator": ">="}]
      }
    },
    {
      "name": "Generate Outreach",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Write personalized recruiting outreach messages that get responses."},
          {"role": "user", "content": "Write outreach for candidate:\n\nCandidate: {{ $json.candidateName }}, {{ $json.candidateTitle }} at {{ $json.candidateCompany }}\nRole: {{ $json.jobTitle }} at {{ $env.COMPANY_NAME }}\nKey hook: {{ $json.hookFromProfile }}\n\nRules:\n- Subject: Specific, not generic \"job opportunity\"\n- Opening: Reference something from their profile (not \"I found your profile\")\n- Body: Why THIS role fits THEM specifically\n- Length: Under 150 words\n- CTA: Single specific action\n- Signature: Use \"{{ $env.RECRUITER_NAME }}\" from {{ $env.COMPANY_NAME }}\n\nReturn:\n{\n  \"linkedin_subject\": \"...\",\n  \"linkedin_message\": \"...\",\n  \"email_subject\": \"...\",\n  \"email_body\": \"...\",\n  \"why_this_person\": \"1 sentence on why they're a good fit\"\n}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Add to Talent Pipeline",
      "type": "Airtable",
      "parameters": {
        "operation": "create",
        "baseId": "{{ $env.AIRTABLE_BASE_ID }}",
        "table": "Candidates",
        "fields": {
          "Name": "{{ $json.candidateName }}",
          "Current Title": "{{ $json.candidateTitle }}",
          "Company": "{{ $json.candidateCompany }}",
          "Match Score": "{{ $json.overallScore }}",
          "Tier": "{{ $json.tier }}",
          "Skills Match": "{{ $json.skillsMatch }}",
          "LinkedIn URL": "{{ $json.linkedInUrl }}",
          "Outreach Message": "{{ $json.outreachMessage }}",
          "Job": "{{ $json.jobTitle }}",
          "Status": "Ready for Outreach",
          "Red Flags": "{{ $json.redFlags }}"
        }
      }
    },
    {
      "name": "Slack Notification",
      "type": "Slack",
      "parameters": {
        "channel": "{{ $env.RECRUITING_SLACK }}",
        "text": "🎯 Tier A candidate found for {{ $json.jobTitle }}:\n\n{{ $json.candidateName }} - {{ $json.candidateTitle }}\nCompany: {{ $json.candidateCompany }}\nScore: {{ $json.overallScore }}/100\nMatched Skills: {{ $json.matchedSkills }}\n\n{{ $json.linkedInUrl }}"
      }
    }
  ]
}
```

## Key Prompts

### Boolean Search Generator
```
Generate recruiting search strings for:

Role: {{ jobTitle }}
Skills: {{ requiredSkills }}
Experience: {{ experienceLevel }}+ years
Location: {{ location }}

Create:
1. LinkedIn searches (3 variations)
2. Indeed/Job board searches
3. GitHub searches (if tech role)
4. Twitter searches (hashtags, tech communities)
5. General tips for finding candidates faster

Format as ready-to-copy strings.
```

### Candidate Scorer
```
Score candidate for {{ jobTitle }} position.

Requirements:
- Skills: {{ requiredSkills }}
- Experience: {{ experienceYears }}+ years
- Location: {{ location }}

Candidate Profile:
{{ candidateData }}

Evaluate:
1. Skills match (which required skills do they have?)
2. Experience relevance (same industry? similar role?)
3. Career trajectory (moving up or lateral?)
4. Company quality (target companies or similar?)
5. Activity/signals (active job seeker? happy where they are?)
6. Red flags (job hopping? employment gaps?)

Return score 0-100, tier A-D, and specific reasoning.
```

### Outreach Writer
```
Write recruiting outreach for:

Candidate: {{ name }}, {{ title }} at {{ company }}
My role: {{ recruiterRole }} at {{ companyName }}
Job: {{ jobTitle }}

Hook (from their profile): {{ personalSignal }}

Rules:
- Don't start with "I came across your profile" or "I'm a recruiter"
- Reference something specific from their background
- Show you did research (their company, role, or content)
- Tie their experience to THIS specific role
- Ask one specific question, don't just say "let's chat"
- Keep under 150 words
- Sound like a human, not a template

Return LinkedIn message (under 300 chars) and email version.
```

## Success Metrics
- Time to fill: Track reduction in sourcing time
- Response rate: Target 15-25% for personalized outreach
- Quality of hire: Score candidates who get interviewed/hired
- Pipeline velocity: Candidates from source to offer in X days

## Pricing for Clients
- Setup: $1,500-2,500
- Monthly subscription: $300-600/month
- Per placement fee: $500-1,000 (contingency model)
- Agency license: $2,000-3,000 (unlimited use)

## Best For
- Startup hiring (high volume, fast cycles)
- Technical recruiting (hard-to-find skills)
- Agency recruiters (scale without adding headcount)
- In-house recruiting (reduce time-to-shortlist)

## Setup Requirements
- LinkedIn Sales Navigator (for searches)
- Apollo.io or Clearbit (for enrichment)
- Airtable (talent pipeline)
- OpenAI API

## Setup Time
- Template ready: ~4 hours
- Per client setup: ~1-2 hours (job templates + CRM)
- Training: 1 week (tune scoring weights)