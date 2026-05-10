# SEO Audit & Content Brief Generator

## What It Does
Analyze any URL or keyword, generate comprehensive SEO audits, competitor analysis, and content briefs that rank. Outputs search intent analysis, keyword clusters, meta recommendations, and full article outlines.

## Target Clients
- Content marketers / SEO agencies
- Bloggers wanting to rank
- In-house marketing teams
- SaaS companies with content-driven growth

## Workflow Flow
```
[URL or Target Keyword]
    ↓
[Competitor Analysis: Top 10 ranking pages]
    ↓
[Search Intent Analysis: What searchers want]
    ↓
[Keyword Research: Primary + clusters + questions]
    ↓
[SEO Audit: On-page factors + gaps]
    ↓
[Content Brief Generator: Full outline + recommendations]
    ↓
[Optional: Direct to CMS or Writing Tool]
```

## Components

### 1. URL/Keyword Input
- Single URL analysis
- Bulk URL analysis (up to 50)
- Keyword-to-content mapping
- Competitor domain analysis

### 2. Competitor SERP Analysis
For target keyword, analyze:
- Top 10 ranking URLs
- Domain authority / page authority
- Content length and structure
- Word count and heading patterns
- Meta titles and descriptions
- Internal/external links
- Image optimization
- Schema markup presence

### 3. Search Intent Classification
Determine what searchers want:
- **Informational**: Learn something (how, what, why)
- **Navigational**: Find a specific site
- **Transactional**: Buy something
- **Commercial Investigation**: Compare options
- **Local**: Near me results

Output: Content type (blog, product page, landing page), Tone, Length recommendation

### 4. Keyword Research Engine
Generate:
- Primary keyword (with search volume + difficulty)
- 10-15 related keywords (semantic + LSI)
- Questions people ask (from People Also Ask, Quora, Reddit)
- Local keywords (if applicable)
- Long-tail opportunities

### 5. On-Page SEO Audit
For a given URL:
- Title tag (length, keyword presence, clickability)
- Meta description (length, keyword, CTA)
- H1-H6 structure (count, keywords used)
- Content quality (depth, uniqueness, freshness)
- Image optimization (alt text, file names, sizes)
- Internal linking opportunities
- External authority links
- Core Web Vitals indicators
- Mobile usability
- Schema markup check

### 6. Content Brief Generator
Create a complete brief for writers:
- Target keyword + synonyms
- Recommended word count
- Required sections/headings
- Key points to cover (from competitors)
- Questions to answer (from SERP)
- Stats/data to include
- Internal link recommendations
- External link sources
- Word count per section
- Tone and style guide
- Example intro paragraph

## n8n Workflow JSON

```json
{
  "name": "SEO Audit & Content Brief Generator",
  "nodes": [
    {
      "name": "Webhook Input",
      "type": "Webhook",
      "parameters": {"path": "seo-brief", "responseMode": "onReceived"}
    },
    {
      "name": "Fetch Competitor URLs",
      "type": "HTTP Request",
      "parameters": {
        "url": "https://api.semrush.com",
        "method": "POST",
        "authentication": "genericCredentialType",
        "genericAuthType": "apiKey",
        "body": {
          "type": "phrase_organic",
          "phrase": "{{ $json.keyword }}",
          "database": "us",
          "export_columns": "Ph,Ur,Rl,Rd,Tc"
        }
      }
    },
    {
      "name": "Scrape Top Pages",
      "type": "HTTP Request",
      "parameters": {
        "method": "GET",
        "url": "{{ $json.competitorUrl }}"
      }
    },
    {
      "name": "AI SERP Analyzer",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "You are an SEO expert. Analyze these top-ranking pages to extract content strategy insights."},
          {"role": "user", "content": "Analyze these top-ranking pages for \"{{ $json.keyword }}\":\n\nPages:\n{{ $json.competitorData }}\n\nFor each page extract:\n- Content length (word count)\n- Heading structure (H2, H3)\n- Key topics covered\n- Word count per section\n- Unique angles not covered\n- CTAs used\n- Data/stats mentioned\n\nReturn: {\"avg_word_count\": #, \"required_sections\": [], \"missing_angles\": [], \"content_gaps\": []}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Search Intent Analysis",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Determine the search intent for this keyword and recommend content strategy."},
          {"role": "user", "content": "Analyze intent for: {{ $json.keyword }}\n\nWhat does a searcher want when they type this?\nWhat format should the content be?\nHow long should it be?\nWhat tone is appropriate?\n\nReturn JSON: {\"intent\": \"informational|transactional|navigational|commercial\", \"content_type\": \"blog|product|landing|video\", \"recommended_length\": \"# words\", \"tone\": \"...\", \"required_elements\": []}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Keyword Cluster Generator",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Generate keyword clusters for SEO content planning."},
          {"role": "user", "content": "Generate keyword clusters for: {{ $json.keyword }}\n\nPrimary keyword: {{ $json.keyword }}\nIntent: {{ $json.intent }}\n\nReturn 15 related keywords organized as:\n- Cluster 1 (semantic variations)\n- Cluster 2 (questions)\n- Cluster 3 (long-tail)\n\nFormat:\n{\n  \"primary\": \"{{ $json.keyword }}\",\n  \"semantic\": [\"keyword1\", \"keyword2\", ...],\n  \"questions\": [\"how to...\", \"what is...\", ...],\n  \"long_tail\": [\"keyword phrase1\", \"keyword phrase2\", ...],\n  \"volume_estimate\": \"low|medium|high\",\n  \"difficulty\": \"easy|medium|hard\"\n}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "On-Page SEO Audit",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Perform an on-page SEO audit with specific recommendations."},
          {"role": "user", "content": "Audit this URL: {{ $json.url }}\nTarget keyword: {{ $json.keyword }}\n\n{{ $json.pageContent }}\n\nCheck and return:\n{\n  \"title_tag\": {\"current\": \"...\", \"issue\": \"...\", \"recommendation\": \"...\"},\n  \"meta_description\": {\"current\": \"...\", \"issue\": \"...\", \"recommendation\": \"...\"},\n  \"headings\": {\"current\": \"...\", \"issues\": [], \"recommendations\": []},\n  \"content\": {\"word_count\": #, \"depth_score\": 1-10, \"gaps\": [], \"improvements\": []},\n  \"images\": {\"alt_text_issues\": [], \"recommendations\": []},\n  \"internal_links\": {\"current\": #, \"recommendations\": []},\n  \"schema\": {\"present\": true/false, \"type\": \"...\", \"recommendations\": []}\n}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Generate Content Brief",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Create a comprehensive content brief for writers."},
          {"role": "user", "content": "Create a content brief for:\n\nTarget keyword: {{ $json.keyword }}\nSearch intent: {{ $json.intent }}\nWord count: {{ $json.recommendedLength }}\nCompetitor gaps: {{ $json.missingAngles }}\nRequired sections: {{ $json.requiredSections }}\nRelated keywords: {{ $json.keywordClusters }}\n\nInclude:\n1. Meta title (under 60 chars, keyword included, clickable)\n2. Meta description (under 160 chars)\n3. H1 (with keyword)\n4. Introduction hook (1-2 sentences)\n5. Required H2s with descriptions\n6. Key points per section\n7. Questions to answer\n8. Stats to include\n9. Internal link targets\n10. External link sources\n11. Word count per section\n12. Example intro paragraph\n13. Tone/style notes\n\nFormat as a structured brief writers can follow."}
        ],
        "options": {"responseFormat": "markdown"}
      }
    },
    {
      "name": "Save to Airtable",
      "type": "Airtable",
      "parameters": {
        "operation": "create",
        "baseId": "{{ $env.AIRTABLE_BASE_ID }}",
        "table": "Content Briefs",
        "fields": {
          "Keyword": "{{ $json.keyword }}",
          "Intent": "{{ $json.intent }}",
          "Brief": "{{ $json.contentBrief }}",
          "Competitor Analysis": "{{ $json.serpAnalysis }}",
          "Status": "Ready for Writer",
          "Target Publish Date": "{{ $json.targetDate }}"
        }
      }
    }
  ]
}
```

## Key Prompts

### SERP Analysis Prompt
```
Analyze top 10 ranking pages for "{{ keyword }}" and extract:
1. Average content length of top 3
2. Common heading structure (what H2s do they all use?)
3. Content format (listicle, guide, tutorial, comparison?)
4. Unique angles or topics not covered by all
5. CTAs they use
6. Data points or statistics mentioned

URLs to analyze:
{{ competitor_urls }}

Return: Structured analysis with specific recommendations.
```

### Content Brief Generator
```
Create a comprehensive content brief for keyword: {{ keyword }}

Context:
- Intent: {{ intent }}
- Target length: {{ word_count }} words
- Audience: {{ audience }}
- Competitor gaps: {{ gaps }}

Format:
# Content Brief: {{ keyword }}

## Meta Info
- Primary keyword: ...
- Secondary keywords: ...
- Target word count: ...
- Search intent: ...

## Structure
### Introduction
[Hook requirements + what to cover]

### Main Sections (H2)
For each H2:
- H2 title
- What to cover (3-5 bullet points)
- Word count target
- Keywords to include
- Questions to answer

### Data/Stats to Include
[List of stats or research to cite]

### Internal Link Targets
[3-5 relevant internal pages to link]

### External Sources
[2-3 authoritative sources to reference]

## Style Guide
- Tone: ...
- Reading level: ...
- Structure preferences: ...

## Example Intro
[Write 2-3 opening sentences]
```

## Success Metrics
- Content ranking: Track keyword positions over 90 days
- Organic traffic lift: Measure vs pre-brief baseline
- Time to publish: Reduce briefing time by 80%
- Content consistency: More content meeting quality threshold

## Pricing for Clients
- Per brief: $50-150 (depending on depth)
- Monthly retainer (10 briefs): $400-600
- SEO audit (full site): $500-1,500
- Agency pricing: White-label brief generation service

## Best For
- Content agencies scaling output
- In-house teams needing faster briefs
- Bloggers competing with established competitors
- SaaS companies with content-driven SEO strategy

## Setup Requirements
- Semrush/Ahrefs API (optional, can use free alternatives)
- Airtable or Google Sheets for brief storage
- OpenAI API (gpt-4o for best quality)

## Setup Time
- Template ready: ~5 hours
- Per client setup: ~1 hour (connections + preferences)
- Integration with CMS: ~2 hours additional