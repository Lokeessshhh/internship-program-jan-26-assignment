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

You need to put your solution here.

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
