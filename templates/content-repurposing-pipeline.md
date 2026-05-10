# Content Repurposing Automation Pipeline

## What It Does
Transform one piece of content (video, podcast, blog post) into a full month's worth of social posts, email newsletters, and blog variations — automatically.

## Target Clients
- Content creators / YouTubers (faceless channels especially)
- Agency owners managing client content
- Podcast hosts
- Thought leaders / coaches
- Newsletter publishers

## Workflow Flow
```
[Source: YouTube Video / Podcast / Blog Post]
    ↓
[Transcription: Extract full text / timestamps]
    ↓
[AI Analysis: Key themes, quotes, insights, CTAs]
    ↓
[Multi-Format Generator:]
    ├── LinkedIn post (hook + value + CTA)
    ├── Twitter thread (5-10 tweets)
    ├── Instagram caption (hook + value + hashtags)
    ├── Blog post rewrite (SEO optimized)
    ├── Email newsletter (intro + 3 key points + link back)
    └── YouTube description (timestamps + key moments)
    ↓
[Human Review Queue: Quick approval before posting]
    ↓
[Auto-Schedule: Push to respective platforms]
```

## Components

### 1. Content Ingestion
- YouTube video URL → auto-transcribe
- Podcast RSS → pull latest episode + transcribe
- Blog URL → scrape and parse
- Direct upload (mp3, mp4, text)

### 2. AI Content Analyzer
Extract and structure:
- Main topic and thesis
- Key quotes (3-5 shareable)
- Step-by-step processes (if tutorial)
- Statistics or data points
- Controversial/unpopular takes (engagement drivers)
- Actionable takeaways
- Recommended audience segments

### 3. Platform-Specific Generators

#### LinkedIn Post
- Hook line (stop scroll)
- 3-5 paragraph body (short, scannable)
- Embedded quote (visual)
- CTA (comment/DM/share)
- Length: 150-300 words

#### Twitter/X Thread
- Hook tweet (controversial take or stat)
- 5-10 thread tweets (numbered)
- Each tweet: single idea + context
- CTA tweet (follow, retweet, DM)

#### Instagram Caption
- Hook (first line must stop scroll)
- Value section (2-3 short paragraphs)
- Embedded quotes
- 10-15 hashtags (mixed reach + niche)
- CTA (save/comment/share)

#### YouTube Description
- Intro hook (160 chars)
- Timestamps (auto-generated)
- Key points summary
- Links mentioned in video
- Subscribe prompt
- SEO keywords

#### Email Newsletter
- Subject line (curiosity or urgency)
- Preview text (compelling hook)
- Opening paragraph (personal, conversational)
- 3 key points (from content)
- Link back to original
- P.S. (soft sell or engagement)

### 4. Human Review Layer
- Each piece goes to review queue
- Approve / Edit / Regenerate options
- Quick edit interface (no platform switching)
- Batch approve for fast content days

### 5. Auto-Scheduling
- LinkedIn: Buffer, Hypefury, or Publer
- Twitter/X: Hypefury, Circleboom
- Instagram: Later or Planoly
- Blog: WordPress, Webflow
- Email: Mailchimp, ConvertKit

## n8n Workflow JSON

```json
{
  "name": "Content Repurposing Pipeline",
  "nodes": [
    {
      "name": "YouTube Trigger",
      "type": "Webhook",
      "parameters": {"path": "content-source", "responseMode": "onReceived"}
    },
    {
      "name": "Transcribe Video",
      "type": "HTTP Request",
      "parameters": {
        "url": "https://api.openai.com/v1/audio/transcriptions",
        "method": "POST",
        "authentication": "genericCredentialType",
        "genericAuthType": "apiKey",
        "body": {
          "model": "whisper-1",
          "file": "{{ $binary.data }}"
        }
      }
    },
    {
      "name": "AI Content Analyzer",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "You are a content strategist. Analyze this transcript and extract structured content assets."},
          {"role": "user", "content": "Analyze this transcript and return JSON:\n\n{\n  \"title\": \"main topic\",\n  \"thesis\": \"one sentence main argument\",\n  \"quotes\": [\"3-5 shareable quotes\"],\n  \"key_points\": [\"3-5 actionable insights\"],\n  \"timestamps\": [\"0:00 intro\", \"2:30 main point\", etc],\n  \"audience\": \"who this resonates with\",\n  \"controversial_take\": \"if any\",\n  \"stats\": [\"any numbers or data\"],\n  \"cta\": \"recommended action for audience\"\n}\n\nTranscript:\n{{ $json.transcript }}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Generate LinkedIn Post",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "You are a LinkedIn content expert. Write posts that get high engagement."},
          {"role": "user", "content": "Write a LinkedIn post based on:\n\nTitle: {{ $json.title }}\nKey Points: {{ $json.keyPoints }}\nQuote: {{ $json.quotes[0] }}\n\nFormat:\n- First line: HOOK (stop the scroll)\n- 3-4 short paragraphs\n- Include the quote in visual format\n- End with a question or CTA\n\nUnder 300 words. No emojis in hook. Conversational but professional."}
        ]
      }
    },
    {
      "name": "Generate Twitter Thread",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "You write viral Twitter threads. Each tweet is standalone and drives engagement."},
          {"role": "user", "content": "Write a Twitter thread from:\n\nTopic: {{ $json.title }}\nThesis: {{ $json.thesis }}\nKey Points: {{ $json.keyPoints }}\n\nFormat:\n- Tweet 1: Hook (controversial or surprising stat)\n- Tweets 2-8: Each covers one key point with context\n- Tweet 9: CTA (follow for more, retweet, DM)\n\nEach tweet: <280 chars. Thread should be 8-10 tweets total. No hashtag spam. Sound like a human."}
        ]
      }
    },
    {
      "name": "Generate Email Newsletter",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "You write newsletters people actually read. Conversational, punchy, valuable."},
          {"role": "user", "content": "Write an email newsletter:\n\nTopic: {{ $json.title }}\nKey Points: {{ $json.keyPoints }}\nOriginal Link: {{ $json.sourceUrl }}\n\nFormat:\n- Subject: curiosity-driven, under 50 chars\n- Preview text: 90 chars max\n- Opening: personal, conversational hook\n- 3 sections: one per key point (2-3 sentences each)\n- Link back paragraph\n- P.S.: soft ask or engagement hook\n\nUnder 500 words. No fluff."}
        ]
      }
    },
    {
      "name": "Review Queue - Airtable",
      "type": "Airtable",
      "parameters": {
        "operation": "create",
        "baseId": "{{ $env.AIRTABLE_BASE_ID }}",
        "table": "Content Review",
        "fields": {
          "Source": "{{ $json.sourceUrl }}",
          "Title": "{{ $json.title }}",
          "LinkedIn Post": "{{ $json.linkedInPost }}",
          "Twitter Thread": "{{ $json.twitterThread }}",
          "Email Draft": "{{ $json.emailDraft }}",
          "Status": "Pending Review",
          "Date": "{{ $now }}"
        }
      }
    },
    {
      "name": "Notify Creator - Slack",
      "type": "Slack",
      "parameters": {
        "channel": "{{ $env.SLACK_CHANNEL }}",
        "text": "📝 New content ready for review:\n\n{{ $json.title }}\n\nLinkedIn: {{ $json.linkedInPost | truncate(100) }}\nTwitter: {{ $json.twitterThread | truncate(100) }}\n\nReview here: https://airtable.com/{{ $env.AIRTABLE_TABLE_ID }}"
      }
    }
  ]
}
```

## Key Prompts

### Content Analyzer
```
Analyze this content and extract for repurposing:

Content: {{ full_transcript_or_article_text }}
Format: {{ video | podcast | blog }}

Return structured JSON:
{
  "meta": {
    "title": "...",
    "thesis": "...",
    "duration": "...",
    "platform": "..."
  },
  "quotes": ["3-5 shareable, visual quotes"],
  "key_points": ["3-7 main takeaways"],
  "steps": ["if tutorial, step-by-step process"],
  "timestamps": ["for video/podcast: key moments"],
  "hooks": ["3 opening hooks for different platforms"],
  "stats": ["any numbers mentioned"],
  "controversial_take": "one polarizing opinion from content",
  "audience": "who this resonates with most",
  "cta": "natural call-to-action for audience"
}
```

### LinkedIn Generator
```
Turn this content into a LinkedIn post:

Title: {{ title }}
Key Points: {{ keyPoints }}
Best Quote: {{ quotes[0] }}
Thesis: {{ thesis }}

Requirements:
- Hook: First line stops the scroll (no questions, no "I", no "This")
- Body: 3-4 short paragraphs, 1-2 sentences each
- Quote: Embed one visual quote in the middle
- CTA: End with a question that sparks comments
- Length: 200-280 words
- Tone: Conversational, no corporate speak
- Emoji: Max 3-5 in body, none in hook
```

### Twitter Thread Generator
```
Write a Twitter thread from:

Topic: {{ title }}
Main Thesis: {{ thesis }}
Key Points: {{ keyPoints }}
Controversial Take: {{ controversial_take }}

Rules:
- Thread: 8-12 tweets
- Tweet 1: Hook (stat, hot take, or "Unpopular opinion:")
- Tweets 2-9: One idea per tweet, build momentum
- Tweet 10-12: CTA + payoff
- No emoji headers, minimal hashtag use
- Each tweet: <260 chars
- Sound like a real person, not a brand
- End with a question or call-to-action
```

## Success Metrics
- Content output: 30x (1 video → 30 pieces)
- Time per piece: <5 minutes (just review + approve)
- Engagement rate: track per platform vs manual posts
- Consistency score: % of scheduled posts delivered

## Pricing for Clients
- Setup: $1,500-2,500
- Monthly management: $300-600/month
- Includes: 4-8 source pieces/month, all formats generated

## Who This Works Best For
- Podcasters: 1 episode = 1 week of social content
- YouTubers: 1 video = 1 month of content
- Coaches: Reuse webinar content endlessly
- Agencies: Scale client content 10x without hiring writers

## Setup Time
- Template ready: ~4 hours
- Per client setup: ~2 hours (platform connections + review queue)
- Initial calibration: 1-2 weeks (tune voice + format preferences)