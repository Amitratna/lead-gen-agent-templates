# E-commerce Product Description Generator

## What It Does
Generate optimized product descriptions for online stores — takes product details and images, creates SEO-friendly descriptions, feature highlights, FAQs, and meta data for multiple platforms (Shopify, Amazon, WooCommerce, etc.)

## Target Clients
- E-commerce store owners (DTC brands)
- Amazon FBA sellers
- Dropshippers managing multiple suppliers
- Agencies managing client stores

## Workflow Flow
```
[Product Input: Name, Specs, Images, Category]
    ↓
[AI Product Analyzer: Extract features + benefits from specs]
    ↓
[Platform Optimizer: Generate for each channel]
    ├── Shopify (SEO + conversion)
    ├── Amazon (A+ content, search optimized)
    ├── Google Shopping (structured data ready)
    └── Social (Instagram, Facebook)
    ↓
[SEO Enhancer: Add keywords naturally]
    ↓
[Variant Generator: Size/color specific versions]
    ↓
[Review Synthesis: Include FAQ from customer questions]
```

## Components

### 1. Product Data Input
Multiple input methods:
- Manual entry (name, specs, category, price)
- CSV import (bulk products)
- URL scraping (from supplier/dropshipping source)
- Image upload (AI extracts details from product photos)

### 2. AI Product Analyzer
Extract and structure:
- Product category + subcategory
- Key features (from specs)
- Primary benefits (what problem does it solve?)
- Use cases / scenarios
- Target audience
- Price-to-value positioning
- Competitive differentiators

### 3. Platform-Specific Generators

#### Shopify / WooCommerce
- Meta title (60 chars, keyword-rich)
- Meta description (160 chars, compelling)
- Product title (SEO + conversion)
- Short description (bullet points)
- Full description (story + features + social proof)
- FAQ section
- Tags (for search/filter)

#### Amazon / E-commerce Marketplaces
- Title (up to 200 chars, keyword-stuffed but natural)
- Bullets (5-7 features, benefit-focused)
- A+ Content description (lifestyle language)
- Search terms / backend keywords
- Comparison table (vs competitors)

#### Google Shopping
- Structured data ready descriptions
- GTIN/UPC integration
- Price + availability language
- Condition (new/used/refurbished)

#### Social Media
- Instagram caption (hook + features + CTA)
- Facebook post (story-driven)
- Pinterest description (SEO-optimized)
- TikTok script hooks

### 4. SEO Enhancer
- Inject target keywords naturally (2-3% density)
- Add long-tail variations
- Include common search queries
- Structure for featured snippets

### 5. Variant Generator
For products with variations (size, color, material):
- Base description (shared across all)
- Variant-specific adjustments (e.g., "in premium leather")
- Size guide references
- Color-specific callouts

### 6. Review FAQ Integration
- Analyze common customer questions (from reviews, Q&A)
- Generate FAQ section
- Address objections proactively
- Include shipping/return info

## n8n Workflow JSON

```json
{
  "name": "E-commerce Product Description Generator",
  "nodes": [
    {
      "name": "Product Input Trigger",
      "type": "Webhook",
      "parameters": {"path": "new-product", "responseMode": "onReceived"}
    },
    {
      "name": "Scrape Product Data",
      "type": "HTTP Request",
      "parameters": {
        "method": "GET",
        "url": "{{ $json.productUrl }}"
      }
    },
    {
      "name": "AI Product Analyzer",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Analyze product details and structure for e-commerce listing."},
          {"role": "user", "content": "Analyze this product:\n\nName: {{ $json.name }}\nCategory: {{ $json.category }}\nPrice: {{ $json.price }}\nSpecs: {{ $json.specs }}\nSupplier Description: {{ $json.supplierDesc }}\n\nReturn JSON:\n{\n  \"category\": \"...\",\n  \"target_audience\": \"who buys this\",\n  \"primary_benefits\": [\"top 3 benefits\"],\n  \"key_features\": [\"feature list\"],\n  \"use_cases\": [\"when/how people use it\"],\n  \"differentiators\": \"what makes it better than alternatives\",\n  \"price_positioning\": \"budget|mid|premium\",\n  \"search_keywords\": [\"keywords for SEO\"],\n  \"common_objections\": [\"what buyers worry about\"],\n  \"lifestyle_hooks\": [\"story angles for marketing\"]\n}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Generate Shopify Description",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Write SEO-optimized e-commerce product descriptions."},
          {"role": "user", "content": "Write a Shopify/WooCommerce product listing for:\n\nProduct: {{ $json.name }}\nPrice: {{ $json.price }}\nCategory: {{ $json.category }}\nTarget Audience: {{ $json.targetAudience }}\nBenefits: {{ $json.primaryBenefits }}\nFeatures: {{ $json.keyFeatures }}\nDifferentiators: {{ $json.differentiators }}\nKeywords: {{ $json.searchKeywords }}\n\nReturn:\n{\n  \"meta_title\": \"(60 chars, keyword-rich)\",\n  \"meta_description\": \"(160 chars, compelling CTA)\",\n  \"product_title\": \"SEO + conversion optimized\",\n  \"short_description\": \"3-5 bullet points, benefit-focused\",\n  \"full_description\": \"Story-driven, 150-200 words, includes features + benefits + social proof\",\n  \"tags\": [\"tag1\", \"tag2\", \"tag3\", \"tag4\", \"tag5\"],\n  \"faq\": [\"Q: ...\", \"A: ...\"]\n}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Generate Amazon Listing",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Write Amazon-optimized product listings that rank and convert."},
          {"role": "user", "content": "Write an Amazon product listing:\n\nProduct: {{ $json.name }}\nPrice: {{ $json.price }}\nFeatures: {{ $json.keyFeatures }}\nBenefits: {{ $json.primaryBenefits }}\nKeywords: {{ $json.searchKeywords }}\nObjections: {{ $json.commonObjections }}\n\nReturn:\n{\n  \"title\": \"(200 chars, keywords naturally integrated)\",\n  \"bullets\": [\"5 feature bullets, benefit-focused, <200 chars each\"],\n  \"description\": \"A+ content style, lifestyle language, 100-150 words\",\n  \"backend_keywords\": \"keyword1, keyword2, keyword3 (avoid duplicates)\"\n}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Generate Social Content",
      "type": "OpenAI",
      "parameters": {
        "operation": "chat",
        "model": "gpt-4o",
        "messages": [
          {"role": "system", "content": "Write engaging social media product posts."},
          {"role": "user", "content": "Write social media posts for {{ $json.name }}:\n\nTarget Audience: {{ $json.targetAudience }}\nHook: {{ $json.lifestyleHooks[0] }}\nKey Benefit: {{ $json.primaryBenefits[0] }}\n\nReturn:\n{\n  \"instagram_caption\": \"Hook first line, benefits, CTA, 15 hashtags\",\n  \"facebook_post\": \"Story-driven, 50-100 words\",\n  \"pinterest_description\": \"SEO-focused, lifestyle keywords, 100 chars\"\n}"}
        ],
        "options": {"responseFormat": "json"}
      }
    },
    {
      "name": "Save to Airtable",
      "type": "Airtable",
      "parameters": {
        "operation": "create",
        "baseId": "{{ $env.AIRTABLE_BASE_ID }}",
        "table": "Products",
        "fields": {
          "Product Name": "{{ $json.name }}",
          "Shopify Description": "{{ $json.shopifyListing.metaTitle }}\n{{ $json.shopifyListing.shortDescription }}",
          "Amazon Listing": "{{ $json.amazonListing.title }}\n{{ $json.amazonListing.bullets }}",
          "Social Content": "{{ $json.socialPosts.instagramCaption }}",
          "Status": "Ready to Publish",
          "Source URL": "{{ $json.productUrl }}"
        }
      }
    },
    {
      "name": "Notify - Slack",
      "type": "Slack",
      "parameters": {
        "channel": "{{ $env.ECOMMERCE_SLACK }}",
        "text": "✅ Product listing ready:\n\n{{ $json.name }}\nPrice: {{ $json.price }}\nCategory: {{ $json.category }}\n\nShopify: Ready\nAmazon: Ready\nSocial: Ready\n\nReview: https://airtable.com/{{ $env.AIRTABLE_TABLE_ID }}"
      }
    }
  ]
}
```

## Key Prompts

### Product Analyzer
```
Analyze this product for e-commerce listing:

Name: {{ productName }}
Category: {{ category }}
Price: {{ price }}
Specs: {{ specs }}
Description: {{ supplierDescription }}
Images: {{ imageUrls }}

Extract and return:
{
  "target_audience": "who is this for?",
  "primary_benefits": "top 3 benefits (not features)",
  "key_features": "product specs and attributes",
  "use_cases": "when/how people use this",
  "differentiators": "why choose this over competitors",
  "price_positioning": "budget/mid-range/premium",
  "search_keywords": "15-20 keywords for SEO",
  "common_questions": "3-5 FAQs from customer perspective",
  "objection_handlers": "how to address buyer concerns",
  "lifestyle_hooks": "story angles for marketing copy"
}
```

### Shopify Description Writer
```
Write product listing content for:

Product: {{ name }}
Category: {{ category }}
Price: {{ price }}
Audience: {{ targetAudience }}
Benefits: {{ primaryBenefits }}
Features: {{ keyFeatures }}
Differentiation: {{ differentiators }}
Keywords: {{ searchKeywords }}

Write:
1. **Meta Title** (60 chars max, include primary keyword)
2. **Meta Description** (160 chars, compelling + CTA)
3. **Product Title** (SEO optimized, conversion-focused)
4. **Short Description** (5 bullet points, benefits not features)
5. **Full Description** (150-200 words, story + features + proof)
6. **Tags** (10 tags for search/filter)
7. **FAQ** (3 Q&As addressing common questions)

Tone: Conversational, not salesy. Focus on transformation, not specs.
```

### Amazon Bullet Points
```
Write 5 Amazon bullet points for:

Product: {{ name }}
Features: {{ keyFeatures }}
Benefits: {{ primaryBenefits }}
Keywords: {{ searchKeywords }}

Rules:
- Each bullet: max 200 characters
- Start with the benefit, not the feature
- Use numbers for specifics ("lasts 50% longer")
- Capitalize key words (not everything)
- Address buyer concerns
- No: "I", "we", "our", "the best", "premium quality"

Format:
[KEY BENEFIT]: [specific detail about this benefit]
```

## Success Metrics
- Time to list: Reduce from 15 min to 2 min per product
- SEO ranking: Track keyword positions after publishing
- Conversion rate: Compare vs previous listings
- Bulk output: Track products processed per day

## Pricing for Clients
- Per product: $5-15 (bulk pricing available)
- Monthly subscription: $100-300/month (unlimited products)
- Agency pricing: White-label, unlimited use
- Setup fee: $500-1,000 (including bulk import setup)

## Best For
- Dropshippers with 100+ products
- Amazon FBA sellers (scale listings fast)
- DTC brands with frequent new products
- Agencies managing multiple client stores

## Setup Requirements
- Shopify/ WooCommerce API (or manual export)
- OpenAI API (gpt-4o for quality)
- Airtable or Google Sheets for product database
- Bulk CSV import capability

## Setup Time
- Template ready: ~4 hours
- Per client setup: ~1 hour (platform connections + category defaults)
- Training: 1 week (tune voice for industry)