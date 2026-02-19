# GenAI Assignment:

**Evaluation Criteria**

We will score your submission on:

* Clarity and practicality of architecture
* Robust JSON schema design
* Prompt quality (zero-shot, reliable, minimal hallucination risk)
* Handling of ambiguity + user review flow
* Bulk generation thinking (errors, naming, report)

## Problem 1: **Proposal for “Video-to-Notes”**

We have a local folder of long videos (3–4 hours each, 200MB+). Watching them fully is slow. We need an automated way to generate a “summary package” per video: **Summary.md** + highlight clips + screenshots, all organized per video. [READ MORE ABOUT THE PROJECT](./Video-summary-platform.md)

### **Task**

Prepare a **pre-processed solution proposal** comparing  **three approaches** **:**

1. **Online/Cloud-Based (Already Available Solutions)**
2. **Build Our Own Using LLM APIs (Hybrid: local media processing + cloud LLM)**
3. **Build Fully Offline Using Open-Source Models (Local transcription + local LLM + pipeline)**

No code required. We want a **clear, practical proposal** with architecture and tradeoffs.

### Your Solution for problem 1:

## Video-to-Notes Platform: Comparative Analysis of Three Approaches

### Executive Summary

After analyzing the requirements for processing long videos (3-4 hours, 200MB+) into structured summary packages, I recommend **Approach 2 (Hybrid)** as the optimal solution. It balances cost-efficiency, quality, and practicality for production use.

---

## Approach 1: Online/Cloud-Based Solutions (Already Available)

### Description
Use existing SaaS platforms that offer video summarization, transcription, and highlight extraction capabilities.

### Available Solutions
| Platform | Transcription | Summarization | Clip Extraction | Pricing |
|----------|--------------|---------------|-----------------|---------|
| **Otter.ai** | ✅ Excellent | ✅ Basic summaries | ❌ No | $16.99/mo |
| **Descript** | ✅ Excellent | ⚠️ Manual | ✅ Yes | $24/mo |
| **Fireflies.ai** | ✅ Good | ✅ AI summaries | ❌ No | $18/mo |
| **Summarize.tech** | ✅ Good | ✅ Video-focused | ⚠️ Limited | $10/mo |
| **Vizard.ai** | ✅ Yes | ✅ AI clips | ✅ Auto-highlight | $30/mo |

### Architecture
```
User → Upload Video to Platform → Platform Processing → Download Results
                                    ↓
                         [Transcription + AI Summary + Clips]
```

### Pros
- **Zero development effort** - Ready to use immediately
- **No infrastructure management** - Fully managed service
- **Regular updates** - Models improve automatically
- **User-friendly interfaces** - Built-in UI for review/editing

### Cons
- **Limited customization** - Cannot tailor output format to our exact `Summary.md` structure
- **Privacy concerns** - Sensitive videos uploaded to third-party servers
- **Cost scales with usage** - Per-minute pricing becomes expensive for batch processing
- **No batch folder processing** - Manual upload per video required
- **Output format mismatch** - Cannot generate our specific folder structure with clips/screenshots aligned to timestamps
- **API limitations** - Most have rate limits unsuitable for large batch jobs

### Cost Estimation (100 videos × 3 hours each)
- Average: ~$0.10-0.25 per minute of video
- Total: **$1,800 - $4,500/month** for processing alone

### Verdict
❌ **Not Recommended** - Does not meet requirements for batch processing, custom output structure, or privacy constraints for sensitive content.

---

## Approach 2: Hybrid (Local Media Processing + Cloud LLM) ✅ RECOMMENDED

### Description
Process media locally using FFmpeg for video operations, use cloud APIs (OpenAI Whisper + GPT-4) for transcription and summarization.

### Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        VIDEO PROCESSING PIPELINE                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Input       │    │  Local       │    │  Cloud APIs  │    │  Output      │
│  Folder      │───▶│  Processing  │───▶│  (OpenAI)    │───▶│  Package     │
│  (Videos)    │    │  (FFmpeg)    │    │              │    │  Generation  │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
                           │                   │
                           ▼                   ▼
                    ┌──────────────┐    ┌──────────────┐
                    │ Audio        │    │ Whisper API  │
                    │ Extraction   │    │ (Transcribe) │
                    │ Frame        │    │ GPT-4        │
                    │ Extraction   │    │ (Summarize)  │
                    └──────────────┘    └──────────────┘
```

### Detailed Pipeline Steps

**Step 1: Video Preprocessing (Local - FFmpeg)**
```bash
# Extract audio for transcription
ffmpeg -i video.mp4 -vn -acodec mp3 -ar 16000 audio.mp3

# Extract frames at intervals (every 30 seconds for screenshot candidates)
ffmpeg -i video.mp4 -vf fps=1/30 frames/frame_%04d.png

# Get video metadata
ffprobe -v quiet -print_format json -show_format -show_streams video.mp4
```

**Step 2: Transcription (Cloud - OpenAI Whisper API)**
- Send audio to Whisper API for accurate transcription with timestamps
- Whisper provides word-level timestamps essential for clip alignment
- Cost: $0.006/minute → $1.08 for 3-hour video

**Step 3: Highlight Detection & Summarization (Cloud - GPT-4)**
```
Input to GPT-4:
- Full transcript with timestamps
- Video metadata (duration, filename)
- Frame descriptions (optional: use GPT-4 Vision for key frames)

Output from GPT-4:
- High-level summary (2-3 paragraphs)
- Key highlights with precise timestamps
- Takeaways/action items
- Suggested screenshot timestamps
```

**Step 4: Asset Generation (Local - FFmpeg)**
```bash
# Extract highlight clips based on GPT-4 timestamps
ffmpeg -i video.mp4 -ss 00:15:30 -t 00:02:00 -c copy clips/highlight_1.mp4

# Extract screenshots at specified timestamps
ffmpeg -i video.mp4 -ss 00:15:45 -vframes 1 screenshots/frame_001.png
```

**Step 5: Summary.md Generation (Local)**
- Template-based Markdown generation using GPT-4 output
- Embed links to clips and screenshots with relative paths

### Output Folder Structure
```
output/
├── video_name_1/
│   ├── Summary.md
│   ├── clips/
│   │   ├── highlight_1_intro_topic.mp4
│   │   ├── highlight_2_key_demo.mp4
│   │   └── highlight_3_conclusion.mp4
│   └── screenshots/
│       ├── screenshot_1_opening.png
│       ├── screenshot_2_diagram.png
│       └── screenshot_3_results.png
├── video_name_2/
│   └── ...
```

### Pros
- **Cost-effective** - Only pay for API calls, not infrastructure
- **Privacy-preserving** - Video files stay local; only audio/text sent to cloud
- **Fully customizable** - Output format, folder structure, naming conventions
- **Batch processing** - Script entire folder automatically
- **High quality** - Whisper accuracy ~95%+, GPT-4 summarization excellent
- **Scalable** - Easy to parallelize across multiple machines

### Cons
- **Requires development** - Pipeline orchestration code needed
- **API dependency** - Requires internet connection and API keys
- **Rate limits** - OpenAI has rate limits (can be mitigated with batching)

### Cost Estimation (100 videos × 3 hours each)
| Component | Cost per Video | Total Cost |
|-----------|---------------|------------|
| Whisper API | $1.08 | $108 |
| GPT-4 (summarization) | $0.50 | $50 |
| GPT-4 Vision (optional frames) | $0.30 | $30 |
| **Total** | **$1.88** | **$188** |

### Verdict
✅ **RECOMMENDED** - Best balance of cost, quality, customization, and privacy. Meets all requirements including batch processing, custom output structure, and handling large files.

---

## Approach 3: Fully Offline (Open-Source Models)

### Description
Complete local processing using open-source models: Whisper (local), LLaMA/Mistral for summarization, FFmpeg for media operations.

### Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FULLY OFFLINE PROCESSING PIPELINE                         │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Input       │    │  Whisper     │    │  Local LLM   │    │  Output      │
│  Videos      │───▶│  (Local)     │───▶│  (LLaMA/     │───▶│  Package     │
│              │    │              │    │  Mistral)    │    │              │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ GPU Required │
                    │ (16GB+ VRAM) │
                    │ for Whisper  │
                    │ Large-v3     │
                    └──────────────┘
```

### Required Infrastructure

| Component | Minimum Spec | Recommended Spec |
|-----------|-------------|------------------|
| **GPU** | RTX 3060 (12GB) | RTX 4090 (24GB) |
| **RAM** | 32GB | 64GB |
| **Storage** | 50GB SSD | 100GB NVMe |
| **CPU** | 8 cores | 16 cores |

### Model Selection

| Task | Model | Size | Quality |
|------|-------|------|---------|
| **Transcription** | Whisper Large-v3 | 3GB | Excellent (~95% accuracy) |
| **Summarization** | LLaMA-3-70B-Quantized | 40GB | Good (slightly below GPT-4) |
| **Alternative LLM** | Mistral-7B | 14GB | Decent (faster, lower quality) |
| **Frame Analysis** | LLaVA-1.6 | 13GB | Good for visual descriptions |

### Pros
- **Complete privacy** - Nothing leaves local machine
- **No API costs** - Free after hardware investment
- **No rate limits** - Process as fast as hardware allows
- **No internet required** - Works in air-gapped environments
- **Full control** - Can fine-tune models for specific domains

### Cons
- **High hardware cost** - $3,000-5,000 for recommended GPU setup
- **Lower transcription accuracy** - Whisper local slightly less accurate than API
- **Lower summarization quality** - Open-source LLMs trail GPT-4 in reasoning
- **Maintenance overhead** - Model updates, dependency management
- **Slower processing** - Local inference slower than cloud APIs
- **Technical complexity** - Requires ML engineering expertise

### Cost Estimation

| Item | Cost |
|------|------|
| GPU (RTX 4090) | $1,999 |
| RAM (64GB) | $200 |
| Storage (2TB NVMe) | $150 |
| Electricity (100 videos) | ~$5 |
| **Total Initial Investment** | **~$2,350** |

### Processing Time Comparison
| Approach | Time per 3-hour video |
|---------|----------------------|
| Whisper API | ~5 minutes |
| Whisper Local (RTX 4090) | ~15 minutes |
| Whisper Local (RTX 3060) | ~45 minutes |

### Verdict
⚠️ **Viable for specific use cases** - Choose this if:
- Strict data privacy requirements (healthcare, legal, defense)
- Processing thousands of videos regularly (ROI on hardware)
- No reliable internet connectivity
- Already have GPU infrastructure

---

## Final Comparison Matrix

| Criterion | Approach 1 (SaaS) | Approach 2 (Hybrid) | Approach 3 (Offline) |
|-----------|-------------------|---------------------|------------------------|
| **Development Effort** | None | Medium | High |
| **Initial Cost** | $0 | $0 | $2,350+ |
| **Per-Video Cost** | $18-45 | $1.88 | $0.05 (electricity) |
| **Privacy** | ❌ Low | ✅ High | ✅ Maximum |
| **Customization** | ❌ None | ✅ Full | ✅ Full |
| **Batch Processing** | ❌ Manual | ✅ Automated | ✅ Automated |
| **Output Format Control** | ❌ Fixed | ✅ Custom | ✅ Custom |
| **Quality** | ⚠️ Varies | ✅ Excellent | ⚠️ Good |
| **Scalability** | ⚠️ Limited | ✅ Easy | ⚠️ Hardware-bound |
| **Maintenance** | ✅ None | ⚠️ Low | ❌ High |

---

## Recommended Implementation: Approach 2 (Hybrid)

### Why Hybrid Wins
1. **Meets all requirements**: Batch processing, custom output structure, large file handling
2. **Cost-effective**: ~$2/video vs $20-45 for SaaS alternatives
3. **Privacy-conscious**: Video files never leave local storage
4. **High quality**: Leverages best-in-class models (Whisper + GPT-4)
5. **Fast iteration**: Easy to adjust prompts, templates, and output formats

### Implementation Roadmap

**Phase 1: Core Pipeline (Week 1)**
- FFmpeg integration for audio/frame extraction
- Whisper API transcription with timestamp handling
- GPT-4 summarization prompt engineering

**Phase 2: Asset Generation (Week 2)**
- Clip extraction based on timestamps
- Screenshot capture at key moments
- Summary.md template generation

**Phase 3: Batch Processing (Week 3)**
- Folder watching and queue management
- Parallel processing with rate limit handling
- Error recovery and retry logic

**Phase 4: Refinement (Week 4)**
- User review interface for highlights
- Quality metrics and confidence scoring
- Performance optimization

## Problem 2: **Zero-Shot Prompt to generate 3 LinkedIn Post**

Design a **single zero-shot prompt** that takes a user’s persona configuration + a topic and generates **3 LinkedIn post drafts** in **3 distinct styles**, each aligned to the user’s voice and constraints. The output must be structured so the app can: show 3 drafts to the user. Assume we are consuming **OpenAI API / Gemini API** with **one prompt call** (no fine-tuning). Your prompt must reliably produce valid, structured output. [READ MORE ABOUT THE PROJECT](./linkedin-automation.md)

**TASK:** Write a prompt that can work.

### Your Solution for problem 2:

## Zero-Shot Prompt for LinkedIn Post Generation

### Prompt Design Strategy

This prompt is designed for **OpenAI GPT-4** with the following goals:
1. **Structured JSON output** - Reliable parsing for the application
2. **Three distinct styles** - Concise Insight, Story-Based, Actionable Checklist
3. **Persona preservation** - Maintains user's voice, tone, and guidelines
4. **Minimal hallucination** - Explicit constraints and validation rules
5. **Single API call** - All three posts generated in one request

---

### The Zero-Shot Prompt

```
SYSTEM PROMPT:
You are an expert LinkedIn content strategist who creates authentic, engaging posts tailored to each user's unique voice. Your task is to generate 3 LinkedIn post drafts in distinct styles while strictly adhering to the user's persona configuration and content guidelines.

CRITICAL RULES:
1. Output MUST be valid JSON - no markdown, no code blocks, just pure JSON
2. Each post MUST be LinkedIn-ready (proper formatting, emoji use, line breaks)
3. Posts MUST differ meaningfully in style while maintaining the user's voice
4. NEVER violate the user's do/don't guidelines
5. NEVER fabricate facts, statistics, or quotes not provided in the input
6. Each post should be 150-300 words (LinkedIn optimal length)
7. Include 3-5 relevant hashtags per post

---

USER INPUT FORMAT (JSON):

{
  "persona": {
    "name": "User's full name",
    "background": "Professional background, experience, expertise areas",
    "industry": "User's industry/domain",
    "tone": "Preferred tone (e.g., professional, conversational, inspirational, witty)",
    "language_style": "Writing style preferences (e.g., uses emojis, bullet points, storytelling, direct)",
    "dos": ["List of things to do/include"],
    "donts": ["List of things to avoid"],
    "signature_phrases": ["Optional: phrases the user commonly uses"],
    "target_audience": "Who the user typically writes for"
  },
  "topic": {
    "subject": "The main topic/subject for the post",
    "context": "Optional: additional context or specific angle",
    "goal": "Optional: what the user wants to achieve (engagement, thought leadership, etc.)"
  }
}

---

OUTPUT FORMAT (JSON):

{
  "posts": [
    {
      "style": "concise_insight",
      "style_description": "A focused, punchy insight that delivers value quickly",
      "content": "The actual LinkedIn post text with proper formatting",
      "hashtags": ["hashtag1", "hashtag2", "hashtag3"],
      "hook": "The opening line designed to grab attention",
      "cta": "The call-to-action if present"
    },
    {
      "style": "story_based",
      "style_description": "A narrative-driven post that uses storytelling to make the point",
      "content": "The actual LinkedIn post text with proper formatting",
      "hashtags": ["hashtag1", "hashtag2", "hashtag3"],
      "hook": "The opening line designed to grab attention",
      "cta": "The call-to-action if present"
    },
    {
      "style": "actionable_checklist",
      "style_description": "A practical, list-based post with actionable takeaways",
      "content": "The actual LinkedIn post text with proper formatting",
      "hashtags": ["hashtag1", "hashtag2", "hashtag3"],
      "hook": "The opening line designed to grab attention",
      "cta": "The call-to-action if present"
    }
  ],
  "persona_adherence": {
    "tone_match": "Brief note on how the tone matches user preferences",
    "guidelines_followed": ["List of specific guidelines that were applied"]
  }
}

---

STYLE SPECIFICATIONS:

STYLE 1 - CONCISE INSIGHT:
- Lead with a strong, contrarian or surprising statement
- 1-2 short paragraphs maximum
- Focus on ONE key insight
- End with a thought-provoking question or statement
- Minimal emojis (0-2)
- Punchy, direct language

STYLE 2 - STORY-BASED:
- Begin with "When I..." or a relatable scenario
- Include a challenge/conflict and resolution
- Weave in the topic naturally through the narrative
- End with a lesson learned or reflection
- More conversational tone
- Moderate emojis (2-4)

STYLE 3 - ACTIONABLE CHECKLIST:
- Start with a promise: "X things I learned about..." or "Here's how to..."
- Use bullet points or numbered lists
- Each point should be specific and actionable
- Include a "save this for later" nudge
- Practical, value-driven
- Moderate emojis (3-5)

---

VALIDATION CHECKLIST (apply before outputting):
□ All 3 posts are meaningfully different in structure and approach
□ User's tone preferences are reflected in all posts
□ No "don't" items from persona are present
□ At least 2 "do" items from persona are incorporated per post
□ No fabricated statistics or quotes
□ Each post is 150-300 words
□ Hashtags are relevant and not overused
□ Posts are ready to publish (no placeholders or brackets)

Now, process the user input and generate the JSON response.
```

---

### Example Usage

**Input to the prompt:**

```json
{
  "persona": {
    "name": "Priya Sharma",
    "background": "Product Manager at a fintech startup with 8 years experience in B2B SaaS. Previously worked at enterprise companies.",
    "industry": "Fintech / B2B SaaS",
    "tone": "Professional but approachable, occasionally witty",
    "language_style": "Uses bullet points frequently, includes relevant emojis, asks questions to engage readers",
    "dos": [
      "Share practical lessons from experience",
      "Use specific examples",
      "Include actionable takeaways",
      "Ask questions to spark discussion"
    ],
    "donts": [
      "Don't use buzzwords like 'synergy', 'pivot', 'leverage'",
      "Don't be overly promotional about my company",
      "Don't share confidential information",
      "Don't use more than 5 hashtags"
    ],
    "signature_phrases": ["Here's what I learned", "The reality is"],
    "target_audience": "Product managers, startup founders, tech professionals"
  },
  "topic": {
    "subject": "Feature prioritization in resource-constrained startups",
    "context": "Recently had to cut 70% of our roadmap due to budget constraints",
    "goal": "Share learnings and establish thought leadership"
  }
}
```

---

### Expected Output

```json
{
  "posts": [
    {
      "style": "concise_insight",
      "style_description": "A focused, punchy insight that delivers value quickly",
      "content": "The reality is: most feature roadmaps are wishlists, not strategies.\n\nLast month, we cut 70% of our roadmap. Painful? Yes. Revealing? Absolutely.\n\nWhat survived:\n→ Features tied directly to revenue\n→ Items with validated user demand\n→ Tech debt that blocked everything else\n\nWhat didn't:\n→ 'Nice-to-have' enhancements\n→ Features for hypothetical users\n→ Projects that needed 6+ months to ship\n\nConstraint breeds clarity.\n\nWhat would your roadmap look like if you could only keep 30%?",
      "hashtags": ["ProductManagement", "Startups", "Prioritization", "ProductStrategy"],
      "hook": "The reality is: most feature roadmaps are wishlists, not strategies.",
      "cta": "What would your roadmap look like if you could only keep 30%?"
    },
    {
      "style": "story_based",
      "style_description": "A narrative-driven post that uses storytelling to make the point",
      "content": "When I joined my current startup, our roadmap had 47 features.\n\n47.\n\nWe had 4 engineers.\n\nYou can imagine how that planning session went. 😅\n\nFast forward to last month: budget cuts. We had to reduce our roadmap by 70%.\n\nHere's what I learned through the process:\n\n1. The features I fought hardest for weren't always the most important ones. My attachment was emotional, not strategic.\n\n2. Saying 'no' to stakeholders got easier when I had data. 'This feature serves 3% of users' is hard to argue with.\n\n3. The team actually moved faster. Fewer context switches, clearer priorities.\n\n4. Our users didn't notice most of what we cut. They were waiting for the 2-3 things that actually mattered to them.\n\nThe constraint we feared became the clarity we needed.\n\nSometimes the best product decision isn't what to build—it's what to stop building.\n\nAnyone else experienced this 'constraint clarity'?",
      "hashtags": ["ProductManagement", "StartupLife", "Roadmap", "Leadership"],
      "hook": "When I joined my current startup, our roadmap had 47 features.",
      "cta": "Anyone else experienced this 'constraint clarity'?"
    },
    {
      "style": "actionable_checklist",
      "style_description": "A practical, list-based post with actionable takeaways",
      "content": "5 questions that saved our roadmap (and my sanity):\n\nWe recently cut 70% of our feature roadmap. Here's the framework I used to decide what stayed:\n\n✅ 1. Revenue tie\nDoes this feature directly impact revenue in the next 2 quarters?\nIf yes → Keep\nIf no/unclear → Question it\n\n✅ 2. Validated demand\nHave users explicitly asked for this? (Not assumed, not extrapolated—actually asked)\nIf yes → Keep\nIf no → Cut\n\n✅ 3. Dependency check\nDoes anything else depend on this feature?\nIf yes → Consider keeping\nIf no → Lower priority\n\n✅ 4. Effort-to-impact ratio\nCan we ship this in <4 weeks with meaningful impact?\nIf yes → Strong candidate\nIf no → Needs stronger justification\n\n✅ 5. Strategic alignment\nDoes this move our core metric?\nIf yes → Keep\nIf no → Cut\n\nThe result? A focused roadmap our team can actually deliver.\n\nSave this for your next planning session. 🔖\n\nWhich question would you add to this list?",
      "hashtags": ["ProductManagement", "Prioritization", "Startups", "ProductStrategy", "Roadmap"],
      "hook": "5 questions that saved our roadmap (and my sanity):",
      "cta": "Which question would you add to this list?"
    }
  ],
  "persona_adherence": {
    "tone_match": "Professional yet approachable; used wit in story-based post; maintained practical focus throughout",
    "guidelines_followed": [
      "Shared practical lessons from experience",
      "Used specific examples (47 features, 70% cut)",
      "Included actionable takeaways (5-question framework)",
      "Asked questions to spark discussion in all posts",
      "Avoided buzzwords",
      "Kept hashtags under 5 per post"
    ]
  }
}
```

---

### Why This Prompt Works

| Criterion | How It's Addressed |
|-----------|-------------------|
| **Structured Output** | JSON schema with explicit field definitions |
| **Minimal Hallucination** | Explicit rules against fabricating facts; validation checklist |
| **Style Differentiation** | Detailed style specifications with structural requirements |
| **Persona Preservation** | dos/donts enforcement; persona_adherence section in output |
| **App Integration** | Clean JSON parsing; metadata fields (hook, cta) for UI display |
| **Zero-Shot Reliability** | Comprehensive instructions eliminate need for examples in prompt |
| **User Review Flow** | Separate posts array allows app to display drafts individually |

---

### Integration Notes for Application

```python
# Pseudocode for API integration
response = openai.chat.completions.create(
    model="gpt-4",
    messages=[
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": json.dumps(user_input)}
    ],
    response_format={"type": "json_object"},  # Enforce JSON output
    temperature=0.7  # Balance creativity with consistency
)

posts = json.loads(response.choices[0].message.content)["posts"]

# Display each post to user for selection
for i, post in enumerate(posts):
    print(f"Style: {post['style']}")
    print(f"Content: {post['content']}")
    print(f"Hashtags: {post['hashtags']}")
```

## Problem 3: **Smart DOCX Template → Bulk DOCX/PDF Generator (Proposal + Prompt)**

Users have many Word documents that act like templates (offer letters, certificates, invoices, contracts). They repeatedly change only a few fields (name, date, amount, address, role, etc.). Doing this manually is slow and error-prone, especially in bulk. [READ MORE ABOUT THE PROJECT](./docs-template-output-generation.md)

We want a system that:

1. Converts an uploaded **DOCX** into a reusable **template** by identifying editable fields.
2. Supports **single generation** (form-fill → DOCX/PDF download).
3. Supports **bulk generation** via **Excel/Google Sheet** rows.

### **Task (No coding)**

Submit a **proposal** for building this system using GenAI (OpenAI/Gemini) for “template field detection” and “field schema generation”. We want a practical design, not code.

### Your Solution for problem 3:

## Smart DOCX Template → Bulk DOCX/PDF Generator: System Proposal

### Executive Summary

This proposal outlines a practical system that transforms ordinary Word documents into reusable templates using GenAI for intelligent field detection. The system supports both single-document generation via form UI and bulk generation via Excel/Google Sheets, with robust error handling and predictable file naming.

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    DOCX TEMPLATE GENERATOR - SYSTEM ARCHITECTURE                 │
└─────────────────────────────────────────────────────────────────────────────────┘

┌────────────────┐     ┌────────────────┐     ┌────────────────┐     ┌────────────┐
│   DOCX Upload  │────▶│  Field         │────▶│  Schema        │────▶│  Template  │
│   (User Input) │     │  Detection     │     │  Generation    │     │  Storage   │
└────────────────┘     │  (GPT-4)       │     │  (JSON Schema) │     │  (DB)      │
                       └────────────────┘     └────────────────┘     └────────────┘
                                                      │
                       ┌──────────────────────────────┼──────────────────────────────┐
                       │                              │                              │
                       ▼                              ▼                              ▼
              ┌────────────────┐             ┌────────────────┐             ┌────────────┐
              │  Single Gen    │             │  Bulk Gen      │             │  Download  │
              │  (Form UI)     │             │  (Excel/Sheets)│             │  Package   │
              └────────────────┘             └────────────────┘             └────────────┘
                       │                              │
                       ▼                              ▼
              ┌────────────────┐             ┌────────────────┐
              │  DOCX/PDF      │             │  ZIP Bundle    │
              │  Generation    │             │  + Report      │
              │  (python-docx  │             │  Generation    │
              │   + reportlab) │             │                │
              └────────────────┘             └────────────────┘
```

---

## Component 1: Template Field Detection (GenAI-Powered)

### Purpose
Automatically identify editable fields in an uploaded DOCX file (names, dates, amounts, addresses, etc.)

### Process Flow

```
DOCX File → Text Extraction → GPT-4 Analysis → Field Suggestions → User Confirmation
```

### Field Detection Prompt

```
SYSTEM PROMPT:
You are a document template analyst. Your task is to analyze document text and identify all fields that appear to be variable/placeholder content suitable for templating.

INPUT: Raw text extracted from a Word document

OUTPUT: JSON schema of detected fields

FIELD TYPES TO DETECT:
- text: Names, addresses, company names, titles
- date: Dates in various formats
- number: Quantities, ages, counts
- currency: Money amounts, salaries, prices
- email: Email addresses
- phone: Phone numbers
- select: Fields with limited options (e.g., Mr/Mrs/Ms, Yes/No)

DETECTION RULES:
1. Look for patterns like [Name], {{name}}, <<NAME>>, or obvious placeholders
2. Identify context-based variables (e.g., "Dear ______" suggests a name field)
3. Detect repeated patterns that change per document
4. Preserve surrounding context for each field
5. Suggest field names in snake_case format

OUTPUT FORMAT:
{
  "template_name": "suggested_template_name",
  "fields": [
    {
      "field_id": "candidate_name",
      "field_type": "text",
      "original_text": "John Doe",
      "context": "Dear Mr./Ms. ______,",
      "position_hint": "Appears in greeting section",
      "required": true,
      "validation": {
        "min_length": 2,
        "max_length": 100
      },
      "suggested_label": "Candidate Name"
    }
  ],
  "document_type": "offer_letter",
  "confidence_score": 0.92
}

Analyze the following document text and output the field schema:
```

### Example Field Detection

**Input Document (Offer Letter):**
```
OFFER LETTER

Date: January 15, 2025

Dear John Doe,

We are pleased to offer you the position of Software Engineer at TechCorp Inc. 
Your starting salary will be ₹12,00,000 per annum.
You will report to Rajesh Kumar on your start date of February 1, 2025.
Your work location will be our Bangalore office at 123 Tech Park, Koramangala.

Please sign and return this letter by January 20, 2025.

Sincerely,
Priya Sharma
HR Manager
```

**Output Schema:**
```json
{
  "template_name": "offer_letter",
  "fields": [
    {
      "field_id": "letter_date",
      "field_type": "date",
      "original_text": "January 15, 2025",
      "context": "Date: ______",
      "position_hint": "Top of document",
      "required": true,
      "validation": {"format": "YYYY-MM-DD"},
      "suggested_label": "Letter Date"
    },
    {
      "field_id": "candidate_name",
      "field_type": "text",
      "original_text": "John Doe",
      "context": "Dear ______,",
      "position_hint": "Greeting line",
      "required": true,
      "validation": {"min_length": 2, "max_length": 100},
      "suggested_label": "Candidate Name"
    },
    {
      "field_id": "position",
      "field_type": "text",
      "original_text": "Software Engineer",
      "context": "position of ______ at",
      "position_hint": "First paragraph",
      "required": true,
      "validation": {},
      "suggested_label": "Position/Role"
    },
    {
      "field_id": "company_name",
      "field_type": "text",
      "original_text": "TechCorp Inc.",
      "context": "at ______",
      "position_hint": "First paragraph",
      "required": true,
      "validation": {},
      "suggested_label": "Company Name"
    },
    {
      "field_id": "salary",
      "field_type": "currency",
      "original_text": "₹12,00,000",
      "context": "salary will be ______ per annum",
      "position_hint": "First paragraph",
      "required": true,
      "validation": {"currency": "INR"},
      "suggested_label": "Annual Salary"
    },
    {
      "field_id": "reporting_manager",
      "field_type": "text",
      "original_text": "Rajesh Kumar",
      "context": "report to ______ on",
      "position_hint": "Second paragraph",
      "required": true,
      "validation": {},
      "suggested_label": "Reporting Manager"
    },
    {
      "field_id": "start_date",
      "field_type": "date",
      "original_text": "February 1, 2025",
      "context": "start date of ______",
      "position_hint": "Second paragraph",
      "required": true,
      "validation": {"format": "YYYY-MM-DD"},
      "suggested_label": "Start Date"
    },
    {
      "field_id": "work_location",
      "field_type": "text",
      "original_text": "Bangalore office at 123 Tech Park, Koramangala",
      "context": "location will be ______",
      "position_hint": "Second paragraph",
      "required": true,
      "validation": {},
      "suggested_label": "Work Location"
    },
    {
      "field_id": "response_deadline",
      "field_type": "date",
      "original_text": "January 20, 2025",
      "context": "by ______",
      "position_hint": "Third paragraph",
      "required": true,
      "validation": {"format": "YYYY-MM-DD"},
      "suggested_label": "Response Deadline"
    },
    {
      "field_id": "hr_name",
      "field_type": "text",
      "original_text": "Priya Sharma",
      "context": "Sincerely,\\n______\\nHR Manager",
      "position_hint": "Signature block",
      "required": true,
      "validation": {},
      "suggested_label": "HR Name"
    }
  ],
  "document_type": "offer_letter",
  "confidence_score": 0.95
}
```

---

## Component 2: Template Storage Schema

### Database Model

```json
{
  "template_id": "tpl_abc123",
  "template_name": "offer_letter_v1",
  "created_at": "2025-01-15T10:30:00Z",
  "created_by": "user_xyz",
  "original_docx_path": "/storage/originals/tpl_abc123.docx",
  "field_schema": {
    "fields": [...],
    "version": "1.0"
  },
  "user_modifications": {
    "fields_added": [],
    "fields_removed": [],
    "field_renames": {}
  },
  "usage_stats": {
    "times_used": 0,
    "last_used": null
  }
}
```

---

## Component 3: Single Document Generation

### Workflow

```
1. User selects template from saved templates
2. System renders form based on field schema
3. User fills form fields
4. System validates input
5. System generates DOCX using python-docx
6. System optionally converts to PDF using reportlab
7. User downloads file(s)
```

### Form Generation Logic

| Field Type | Form Input | Validation |
|------------|------------|------------|
| `text` | Text input | Min/max length |
| `date` | Date picker | Valid date format |
| `number` | Number input | Min/max range |
| `currency` | Number + currency selector | Positive value |
| `email` | Email input | Email regex |
| `phone` | Tel input | Phone format |
| `select` | Dropdown | Predefined options |

---

## Component 4: Bulk Document Generation

### Workflow

```
1. User selects template
2. System generates Excel template with column headers matching field_ids
3. User downloads Excel, fills multiple rows
4. User uploads filled Excel (or connects Google Sheet)
5. System validates all rows
6. System generates documents in parallel
7. System creates ZIP bundle + generation report
8. User downloads package
```

### Excel Template Format

| candidate_name | position | salary | start_date | work_location | ... |
|----------------|----------|--------|------------|---------------|-----|
| Rahul Verma | Frontend Developer | ₹10,00,000 | 2025-02-15 | Mumbai Office | ... |
| Sneha Patel | Backend Developer | ₹14,00,000 | 2025-02-20 | Bangalore Office | ... |
| Amit Singh | Data Analyst | ₹8,00,000 | 2025-03-01 | Delhi Office | ... |

### File Naming Convention

```
Format: {primary_field}_{template_name}_{date}.{ext}

Examples:
- Rahul_Verma_offer_letter_2025-01-15.pdf
- Sneha_Patel_offer_letter_2025-01-15.pdf
- INV-2025-001_invoice_2025-01-15.docx
```

### Bulk Generation Report

```json
{
  "job_id": "job_xyz789",
  "template_name": "offer_letter_v1",
  "total_rows": 100,
  "successful": 97,
  "failed": 3,
  "started_at": "2025-01-15T10:00:00Z",
  "completed_at": "2025-01-15T10:05:32Z",
  "output_files": {
    "zip_path": "/output/jobs/job_xyz789/bundle.zip",
    "pdf_folder": "/output/jobs/job_xyz789/pdfs/",
    "docx_folder": "/output/jobs/job_xyz789/docxs/"
  },
  "errors": [
    {
      "row_number": 23,
      "field": "start_date",
      "error": "Invalid date format: 'Feb 30, 2025'",
      "suggestion": "Use YYYY-MM-DD format"
    },
    {
      "row_number": 56,
      "field": "salary",
      "error": "Currency value cannot be negative",
      "suggestion": "Enter positive salary value"
    },
    {
      "row_number": 89,
      "field": "candidate_name",
      "error": "Required field is empty",
      "suggestion": "Enter candidate name"
    }
  ],
  "summary": {
    "documents_generated": 97,
    "total_pages": 194,
    "total_size_mb": 12.4
  }
}
```

---

## Component 5: Error Handling Strategy

### Validation Layers

| Layer | When | What | Action |
|-------|------|------|--------|
| **Pre-generation** | Before processing | Field completeness, format validation | Reject with clear error message |
| **Row-level** | During bulk processing | Per-row validation | Skip row, log error, continue |
| **Document-level** | During generation | DOCX structure integrity | Retry with fallback |
| **Output-level** | After generation | File size, corruption check | Regenerate if failed |

### Error Categories

| Error Type | Example | User Message | Recovery |
|------------|---------|--------------|----------|
| **Missing Required Field** | Empty `candidate_name` | "Row 5: Candidate Name is required" | Highlight in Excel |
| **Invalid Format** | Wrong date format | "Row 12: Date should be YYYY-MM-DD" | Show expected format |
| **Type Mismatch** | Text in currency field | "Row 8: Salary must be a number" | Suggest correction |
| **Constraint Violation** | Negative salary | "Row 20: Salary cannot be negative" | Show valid range |
| **Template Corrupted** | Original DOCX damaged | "Template file is corrupted. Please re-upload." | Request re-upload |

### Partial Success Handling

For bulk jobs with mixed success/failure:
1. Generate all valid documents
2. Create detailed error report
3. Provide "Retry Failed Rows" option
4. Download only failed rows as new Excel for correction

---

## Component 6: Technology Stack

### Backend (Python)

| Component | Library | Purpose |
|-----------|---------|---------|
| **DOCX Manipulation** | `python-docx` | Read/write Word documents, preserve formatting |
| **PDF Generation** | `reportlab` + `docx2pdf` | Convert DOCX to PDF |
| **Excel Processing** | `openpyxl` + `pandas` | Read/write Excel files |
| **Google Sheets** | `gspread` | API integration with Google Sheets |
| **GenAI Integration** | `openai` SDK | Field detection via GPT-4 |
| **File Storage** | Local filesystem or S3 | Store templates and outputs |
| **Job Queue** | `Celery` + `Redis` | Async bulk processing |

### Frontend

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Form Builder** | React + Formik | Dynamic form generation from schema |
| **File Upload** | react-dropzone | DOCX/Excel upload |
| **Preview** | react-file-viewer | Document preview before download |
| **Progress Tracking** | WebSocket | Real-time bulk job progress |

---

## Component 7: Security Considerations

| Concern | Mitigation |
|---------|------------|
| **Document Privacy** | Encrypt stored files; auto-delete after configurable period |
| **Sheet Access** | OAuth for Google Sheets; scoped permissions |
| **API Security** | Rate limiting; API key rotation |
| **Input Validation** | Sanitize all inputs; prevent injection attacks |
| **Audit Trail** | Log all generation requests with user ID and timestamp |

---

## Implementation Roadmap

### Phase 1: Core Template System (Week 1-2)
- DOCX upload and text extraction
- GPT-4 field detection integration
- Field schema storage
- Basic template management API

### Phase 2: Single Generation (Week 3)
- Dynamic form generation from schema
- python-docx template filling
- PDF conversion pipeline
- Download functionality

### Phase 3: Bulk Generation (Week 4-5)
- Excel template generation
- Bulk upload and validation
- Parallel document generation
- ZIP bundling and report generation

### Phase 4: Polish & Edge Cases (Week 6)
- Google Sheets integration
- Retry failed rows
- Progress tracking UI
- Error recovery improvements

---

## Success Metrics

| Metric | Target |
|--------|--------|
| **Field Detection Accuracy** | >90% fields correctly identified |
| **Single Generation Time** | <3 seconds per document |
| **Bulk Generation Speed** | >100 documents/minute |
| **Error Rate** | <2% for valid inputs |
| **User Satisfaction** | Template creation <5 minutes |

---

## Summary

This system leverages GenAI (GPT-4) to automatically detect template fields, eliminating manual field marking. The JSON schema approach ensures flexibility for various document types. Bulk processing with comprehensive error handling and reporting makes it production-ready for enterprise use cases like HR offer letters, invoice generation, and certificate creation.

## Problem 4: Architecture Proposal for 5-Min Character Video Series Generator

We want to build a system that helps a user create a short video series (around **5 minutes per episode**) using predefined characters. The user defines characters (image + personality) and relationships once, then provides a short story/situation for an episode. The system outputs a complete episode package and optionally a final video. [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

### **Task**

Create a **small, clear architecture proposal** (no code, no prompts) describing how you would design and build this system.

### Your Solution for problem 4:

## Character-Based Short Video Series Generator: Architecture Proposal

### Executive Summary

This architecture proposal describes a modular system for generating consistent 5-minute video episodes using predefined characters. The system maintains character consistency across episodes through a "Series Bible" and leverages modern AI tools (Runway/Pika for visuals, ElevenLabs for audio) to produce production-ready episode packages.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│              CHARACTER VIDEO SERIES GENERATOR - SYSTEM ARCHITECTURE                 │
└─────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                    INPUT LAYER                                      │
├──────────────────────┬──────────────────────┬───────────────────────────────────────┤
│   Series Bible       │   Episode Request    │   Output Preferences                  │
│   (One-time setup)   │   (Per episode)      │   (Format, duration, style)           │
└──────────────────────┴──────────────────────┴───────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              ORCHESTRATION LAYER                                    │
├──────────────────────────────────────────────────────────────────────────────────────┤
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐       │
│  │ Story Engine   │─▶│ Script         │─▶│ Scene          │─▶│ Asset          │       │
│  │ (GPT-4)        │  │ Generator      │  │ Planner        │  │ Orchestrator   │       │
│  └────────────────┘  └────────────────┘  └────────────────┘  └────────────────┘       │
└─────────────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              GENERATION LAYER                                       │
├──────────────────────┬──────────────────────┬──────────────────────┬────────────────┤
│   Visual Generator   │   Audio Generator    │   Duration Controller│   Consistency   │
│   (Runway/Pika)     │   (ElevenLabs)      │   (Timing Engine)    │   Manager       │
└──────────────────────┴──────────────────────┴──────────────────────┴────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              OUTPUT LAYER                                           │
├──────────────────────┬──────────────────────┬───────────────────────────────────────┤
│   Episode Package    │   Final Video        │   Iteration Support                   │
│   (Script + Assets)  │   (Rendered MP4)     │   (Edit + Regenerate)                 │
└──────────────────────┴──────────────────────┴───────────────────────────────────────┘
```

---

## Core Components

### 1. Series Bible Manager

**Purpose:** Store and manage all character definitions, relationships, and world-building data.

**Data Model:**

```
SERIES BIBLE
├── series_id
├── series_name
├── world_settings
│   ├── location
│   ├── time_period
│   ├── tone
│   └── recurring_themes
├── characters[]
│   ├── character_id
│   ├── name
│   ├── visual_profile
│   │   ├── reference_images[]
│   │   ├── physical_description
│   │   ├── typical_clothing
│   │   ├── color_palette
│   │   └── age_appearance
│   ├── personality
│   │   ├── traits[]
│   │   ├── speaking_style
│   │   ├── catchphrases
│   │   └── behavioral_rules
│   └── voice_profile
│       ├── voice_id (ElevenLabs)
│       ├── pitch
│       ├── speed
│       └── emotion_range
├── relationships[]
│   ├── character_1_id
│   ├── character_2_id
│   ├── relationship_type
│   └── interaction_rules
└── style_guidelines
    ├── visual_style
    ├── narration_ratio
    └── content_rating
```

**Character Consistency Strategy:**

| Aspect | Consistency Mechanism |
|--------|----------------------|
| **Visual Identity** | Reference image embeddings stored; used for image generation prompts |
| **Voice** | Fixed ElevenLabs voice_id per character |
| **Personality** | Trait embeddings injected into all script generation |
| **Relationships** | Interaction rules enforced during dialogue generation |

---

### 2. Episode Generation Pipeline

**Step-by-Step Flow:**

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  EPISODE    │     │  STORY      │     │  SCRIPT     │     │  SCENE      │
│  REQUEST    │────▶│  EXPANSION  │────▶│  GENERATION │────▶│  BREAKDOWN  │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
      │                   │                   │                   │
      ▼                   ▼                   ▼                   ▼
  User Input:        GPT-4 expands       Scene-by-scene       Timing +
  - Situation        story into          dialogue +           shot list
  - Characters       narrative           narration            per scene
  - Tone             structure
  - Goal

                          │
                          ▼
              ┌─────────────────────┐
              │  DURATION CONTROL   │
              │  Target: ~5 minutes │
              │  - Scene count: 8-12│
              │  - Avg scene: 25-40s│
              └─────────────────────┘
```

**Duration Control Mechanism:**

| Target | 5 minutes (300 seconds) |
|--------|-------------------------|
| **Opening** | 15-20 seconds |
| **Main Content** | 240-260 seconds |
| **Closing** | 20-25 seconds |
| **Scene Count** | 8-12 scenes |
| **Dialogue Pacing** | ~150 words/minute |

**Scene Structure Template:**

```
SCENE {
  scene_id: "scene_001",
  scene_type: "dialogue" | "narration" | "action",
  characters_present: ["char_1", "char_2"],
  location: "living_room",
  duration_seconds: 35,
  dialogue_lines: [...],
  narration: "...",
  visual_prompt: "...",
  camera_angle: "medium_shot",
  background_music: "upbeat"
}
```

---

### 3. Visual Asset Generation

**Architecture:**

```
┌────────────────────────────────────────────────────────────────────┐
│                     VISUAL GENERATION PIPELINE                      │
└────────────────────────────────────────────────────────────────────┘

┌────────────────┐     ┌────────────────┐     ┌────────────────┐
│  Character     │     │  Scene         │     │  Image/Video   │
│  Reference     │────▶│  Prompt        │────▶│  Generator     │
│  Images        │     │  Builder       │     │  (Runway/Pika) │
└────────────────┘     └────────────────┘     └────────────────┘
                              │                       │
                              ▼                       ▼
                       ┌────────────────┐     ┌────────────────┐
                       │  Consistency   │     │  Output        │
                       │  Injection     │     │  Validation    │
                       └────────────────┘     └────────────────┘
```

**Visual Prompt Construction:**

```
BASE_PROMPT = character_visual_profile + scene_context + style_guidelines

Example:
"[Character: Maya, 28, professional woman, dark hair, blue blouse, 
confident expression] [Scene: Modern office, morning light through 
windows] [Action: Looking at laptop screen, slight smile] [Style: 
Cinematic, warm colors, medium shot] [Consistency: Reference image #3]"
```

**Tool Integration:**

| Tool | Use Case | Output |
|------|----------|--------|
| **Runway Gen-3** | High-quality character shots | 4-6 second clips |
| **Pika Labs** | Quick scene variations | 3-4 second clips |
| **Midjourney** | Static backgrounds/keyframes | PNG images |
| **D-ID** | Talking head animations | Lip-synced video |

**Character Consistency Techniques:**

1. **Reference Image Seeding:** Use same seed image across all generations
2. **LoRA Fine-tuning:** Train lightweight adapter for each character
3. **IP-Adapter:** Use image prompts instead of text for character consistency
4. **Face Swap Post-processing:** Apply consistent face overlay if needed

---

### 4. Audio Generation Pipeline

**Architecture:**

```
┌────────────────────────────────────────────────────────────────────┐
│                      AUDIO GENERATION PIPELINE                      │
└────────────────────────────────────────────────────────────────────┘

┌────────────────┐     ┌────────────────┐     ┌────────────────┐
│  Script        │     │  Voice         │     │  Audio         │
│  Segments      │────▶│  Synthesis     │────▶│  Assembly      │
│  (per line)    │     │  (ElevenLabs)  │     │  (FFmpeg)      │
└────────────────┘     └────────────────┘     └────────────────┘
                              │
                              ▼
                       ┌────────────────┐
                       │  Character     │
                       │  Voice Mapping │
                       └────────────────┘
```

**Voice Configuration:**

```
CHARACTER_VOICE {
  character_id: "maya",
  elevenlabs_voice_id: "voice_abc123",
  voice_settings: {
    stability: 0.7,
    similarity_boost: 0.8,
    style: "conversational",
    use_speaker_boost: true
  },
  emotion_presets: {
    happy: {style_exaggeration: 0.3},
    serious: {style_exaggeration: 0.1},
    excited: {style_exaggeration: 0.5}
  }
}
```

**Audio Components:**

| Component | Source | Duration Handling |
|-----------|--------|-------------------|
| **Dialogue** | ElevenLabs per-character voices | Variable (script-dependent) |
| **Narration** | Single narrator voice | Bridges scenes |
| **Background Music** | Royalty-free library / AI-generated | Looped, faded per scene |
| **Sound Effects** | Sound library | Triggered by scene events |

---

### 5. Assembly & Rendering Engine

**Final Video Assembly:**

```
┌────────────────────────────────────────────────────────────────────┐
│                     VIDEO ASSEMBLY PIPELINE                         │
└────────────────────────────────────────────────────────────────────┘

┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Visual     │     │  Audio      │     │  Timing     │     │  Final      │
│  Clips      │────▶│  Tracks     │────▶│  Sync       │────▶│  Render     │
│  (per scene)│     │  (dialogue+ │     │  (align to  │     │  (FFmpeg)   │
└─────────────┘     │  music+sfx) │     │  script)    │     └─────────────┘
                    └─────────────┘     └─────────────┘
```

**Rendering Configuration:**

| Parameter | Options |
|-----------|---------|
| **Resolution** | 1080p (1920x1080) or 720p for 9:16 mobile |
| **Aspect Ratio** | 16:9 (landscape) or 9:16 (vertical/Reels) |
| **Frame Rate** | 30 fps |
| **Codec** | H.264 for compatibility |
| **Audio** | AAC 192kbps |

---

## Data Models

### Episode Package Structure

```
EPISODE_PACKAGE/
├── episode_meta.json
│   ├── episode_id
│   ├── episode_number
│   ├── title
│   ├── duration_seconds
│   ├── characters_featured[]
│   └── generation_timestamp
├── script.md
│   ├── scene_breakdown
│   ├── full_dialogue
│   └── narration_text
├── storyboard.json
│   ├── scenes[]
│   │   ├── scene_id
│   │   ├── duration
│   │   ├── shot_type
│   │   └── visual_description
├── assets/
│   ├── visuals/
│   │   ├── scene_001.mp4
│   │   ├── scene_002.mp4
│   │   └── ...
│   ├── audio/
│   │   ├── dialogue/
│   │   │   ├── char_1_scene_1.wav
│   │   │   └── ...
│   │   ├── music/
│   │   │   └── background.mp3
│   │   └── sfx/
│   └── references/
│       └── character_refs/
├── final_video.mp4
└── generation_report.json
    ├── scenes_generated
    ├── total_duration
    ├── consistency_score
    └── issues_detected[]
```

---

## Iteration & Editing Support

**User Iteration Workflow:**

```
┌────────────────────────────────────────────────────────────────────┐
│                     ITERATION WORKFLOW                              │
└────────────────────────────────────────────────────────────────────┘

1. REVIEW
   └── User watches generated episode

2. IDENTIFY CHANGES
   ├── Edit dialogue line
   ├── Change scene visual
   ├── Adjust timing
   ├── Swap character expression
   └── Modify background music

3. TARGETED REGENERATION
   ├── Only affected scenes regenerated
   ├── Unchanged assets reused
   └── Partial re-assembly

4. COMPARE & APPROVE
   ├── Side-by-side comparison
   └── Final approval
```

**Supported Edit Operations:**

| Edit Type | Regeneration Scope | Time Impact |
|-----------|-------------------|-------------|
| **Dialogue change** | Single audio file | ~30 seconds |
| **Scene visual change** | Single video clip | ~2 minutes |
| **Character swap** | All scenes with character | ~5-10 minutes |
| **Story modification** | Script + affected scenes | ~10-15 minutes |
| **Style change** | All visuals | ~15-20 minutes |

---

## Technology Stack

### Backend Services

| Service | Technology | Purpose |
|---------|------------|---------|
| **API Layer** | FastAPI / Node.js | REST endpoints |
| **Orchestration** | Temporal / Celery | Workflow management |
| **Database** | PostgreSQL + Redis | Series bible, caching |
| **File Storage** | S3 / GCS | Asset storage |
| **Queue** | Redis + Bull | Job queue |

### AI/ML Services

| Service | Provider | Use Case |
|---------|----------|----------|
| **Script Generation** | OpenAI GPT-4 | Story, dialogue, narration |
| **Image Generation** | Midjourney API / DALL-E 3 | Backgrounds, keyframes |
| **Video Generation** | Runway Gen-3 / Pika | Character shots, scenes |
| **Voice Synthesis** | ElevenLabs | Character voices |
| **Music Generation** | Suno / Udio | Background music (optional) |

### Infrastructure

| Component | Technology |
|-----------|------------|
| **Compute** | AWS EC2 / GCP Compute |
| **GPU** | NVIDIA T4 / A10G for local inference |
| **CDN** | CloudFlare / CloudFront |
| **Monitoring** | Prometheus + Grafana |

---

## Scalability Considerations

### Episode Generation Time

| Phase | Estimated Time |
|-------|----------------|
| Script Generation | 30-60 seconds |
| Visual Generation (8-12 scenes) | 8-15 minutes |
| Audio Generation | 2-4 minutes |
| Assembly & Rendering | 2-3 minutes |
| **Total** | **12-25 minutes** |

### Parallelization Opportunities

| Task | Parallelizable? | Speedup |
|------|-----------------|---------|
| Scene visual generation | ✅ Yes | 3-4x with parallel API calls |
| Voice synthesis per line | ✅ Yes | 2x with concurrent requests |
| Script generation | ❌ Sequential | - |
| Final rendering | ❌ Sequential | - |

---

## Quality Assurance

### Consistency Checks

| Check | Method | Threshold |
|-------|--------|-----------|
| **Character Visual Consistency** | CLIP embedding similarity | >0.85 |
| **Voice Consistency** | Speaker verification | >0.90 |
| **Duration Target** | Time calculation | 280-320 seconds |
| **Scene Count** | Script analysis | 8-12 scenes |
| **Relationship Rules** | Dialogue context check | No violations |

### Output Validation

```
VALIDATION_PIPELINE:
1. Duration check (target: 5 min ± 30 sec)
2. Character presence verification
3. Dialogue-personality alignment check
4. Visual consistency scoring
5. Audio quality check
6. Final render integrity check
```

---

## Implementation Roadmap

### Phase 1: Foundation (Week 1-2)
- Series Bible data model and storage
- Basic script generation with GPT-4
- Character voice setup in ElevenLabs

### Phase 2: Visual Pipeline (Week 3-4)
- Runway/Pika integration
- Character consistency testing
- Scene prompt engineering

### Phase 3: Audio & Assembly (Week 5-6)
- Voice synthesis pipeline
- Background music integration
- FFmpeg assembly automation

### Phase 4: Polish & Iteration (Week 7-8)
- Edit and regenerate features
- Quality scoring system
- User feedback integration

---

## Summary

This architecture enables consistent, repeatable generation of 5-minute character video episodes through:

1. **Series Bible** - Single source of truth for characters, relationships, and style
2. **Modular Pipeline** - Script → Scenes → Visuals → Audio → Assembly
3. **Modern AI Tools** - Runway/Pika for video, ElevenLabs for voice
4. **Consistency Mechanisms** - Reference seeding, voice locking, trait enforcement
5. **Iteration Support** - Targeted regeneration without full recomputation

The system balances quality with practicality, leveraging best-in-class AI services while maintaining the flexibility to swap components as technology evolves.
