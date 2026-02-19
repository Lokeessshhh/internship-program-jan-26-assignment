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

You need to put your solution here.

## Problem 4: Architecture Proposal for 5-Min Character Video Series Generator

We want to build a system that helps a user create a short video series (around **5 minutes per episode**) using predefined characters. The user defines characters (image + personality) and relationships once, then provides a short story/situation for an episode. The system outputs a complete episode package and optionally a final video. [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

### **Task**

Create a **small, clear architecture proposal** (no code, no prompts) describing how you would design and build this system.

### Your Solution for problem 4:

You need to put your solution here.
