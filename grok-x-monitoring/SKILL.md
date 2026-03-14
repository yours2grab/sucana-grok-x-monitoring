---
name: grok-x-monitoring
description: "Weekly X/Twitter monitoring via Grok API. Scans for AI marketer pain points and trends, outputs structured analysis + content hooks. MANDATORY triggers on: x monitoring, grok scan, x scan, twitter scan, x analysis, weekly research, scan x, grok monitoring."
---

# Grok API X Monitoring Skill

Weekly automated scan of X/Twitter via Grok API (xAI) for AI marketer pain points, trends, and content hooks. Complements Reddit monitoring (PPC audience) by covering Audience 2 (broad AI marketers).

## Requirements

- `XAI_API_KEY` in `.env` (xAI account — $25 free credits on signup)
- Model: `grok-4-fast` (only grok-4 family supports x_search tools)
- Endpoint: `/v1/responses` (NOT `/v1/chat/completions` — that's deprecated for search)
- Tools: `[{"type": "x_search"}]`

## Step 1: Load Config

Read these before running:

1. **Pain clusters** (Section: Pain Clusters below)
2. **Search queries** (Section: Search Queries below)
3. **Previous analysis** (if exists): `Marketing/Research/latest-x-analysis.md`
4. **Content ideas file** (grep only, never full read): `Marketing/Research/content-ideas.md`

## Step 2: Check API Key

```bash
# Verify XAI_API_KEY exists
grep -q "XAI_API_KEY" .env 2>/dev/null
```

If missing, ask Virgil to add it:
```
Add this to your .env file:
XAI_API_KEY=your-key-from-console.x.ai
```

Stop here until key is confirmed.

## Step 3: Call Grok API

Make one API call per pain cluster (6 total). Use Grok 4.1 Fast with web search enabled.

```python
import requests
import os
import json
from datetime import datetime

XAI_API_KEY = os.getenv("XAI_API_KEY")
BASE_URL = "https://api.x.ai/v1/responses"

def scan_pain_cluster(cluster_name, search_query):
    """Call Grok API with x_search tool to scan X for a pain cluster."""
    response = requests.post(
        BASE_URL,
        headers={
            "Authorization": f"Bearer {XAI_API_KEY}",
            "Content-Type": "application/json"
        },
        json={
            "model": "grok-4-fast",
            "stream": False,
            "input": [
                {
                    "role": "system",
                    "content": "You are a social media research analyst. Search X/Twitter for real conversations. Return only real posts you find, not hypothetical examples."
                },
                {
                    "role": "user",
                    "content": f"""Search X/Twitter for the most engaged conversations from the past 7 days about: {search_query}

For each conversation found, return:
1. Direct quote (exact text from the post)
2. Author handle (if visible)
3. Engagement (approximate likes, replies, reposts)
4. Thread URL (if available)
5. Pain category: {cluster_name}
6. Hook potential (one sentence — what content angle does this suggest?)

Return 3-5 of the most engaged/relevant posts. If you can't find recent posts, say so — don't make them up."""
                }
            ],
            "tools": [
                {"type": "x_search"}
            ]
        },
        timeout=90
    )
    result = response.json()
    # Extract text from responses API output format
    for item in result.get("output", []):
        if item.get("type") == "message":
            for content in item.get("content", []):
                if content.get("type") == "output_text":
                    return content["text"]
    return None
```

## Step 4: Parse Results

For each Grok response, extract into this structure:

```
PAIN CLUSTER: [name]
---
POST 1:
- Quote: "[exact text]"
- Author: @handle
- Engagement: ~X likes, Y replies
- URL: [link if available]
- Hook potential: [content angle]

POST 2:
...
```

Flag any cluster that returned zero results — it may be losing relevance.

## Step 5: Generate Outputs

### 5a. Analysis Report

Save full analysis to: `Marketing/Research/latest-x-analysis.md` (overwrite)

Also archive to: `Marketing/Research/x-analysis-history/YYYY-MM-DD-x-analysis.md`

Format:

```markdown
# X/Twitter Analysis — [date]

**Scan date:** [date]
**Model:** Grok 4 Fast
**Clusters scanned:** 6
**Total posts found:** [count]

## Top Findings

[3-5 bullet summary of the most interesting pain points this week]

## Cluster: AI Tool Overwhelm

[posts]

## Cluster: AI Content Sameness

[posts]

## Cluster: AI Time Waste

[posts]

## Cluster: Enterprise-Only Cases

[posts]

## Cluster: AI Hype Fatigue

[posts]

## Cluster: Tool ROI Questions

[posts]

## Ad Angle Validation

Which of our 10 ad angles (from ad-angles-the-lead.md) showed up in this week's conversations:
- [angle]: [still resonating / fading / new twist: ...]

## Suggested Newsletter Topics

1. [topic + why it's timely]
2. [topic + why it's timely]
3. [topic + why it's timely]
4. [topic + why it's timely]
5. [topic + why it's timely]
```

### 5b. Content Hooks

Extract 5-10 content hooks from the analysis. Format each as:

```
HOOK: [hook text — in practitioner voice, sourced from real quotes]
SOURCE: X/Twitter — @handle, [date]
ANGLE: [which ad angle or content pillar this maps to]
AUDIENCE: [1 = PPC, 2 = Broad AI, or Both]
```

### 5c. Show Summary to Virgil

Display in chat:
```
X MONITORING — [date]

Posts scanned: [count across 6 clusters]
Hottest cluster: [which pain cluster had most engagement]

TOP 3 FINDINGS:
1. [finding]
2. [finding]
3. [finding]

NEWSLETTER TOPIC SUGGESTIONS:
1. [topic]
2. [topic]
3. [topic]
4. [topic]
5. [topic]

NEW HOOKS: [count] ready to append to content-ideas.md

Approve hooks to append? (yes / edit first / skip)
```

## Step 6: Append Hooks (on approval)

**HUMAN LOOP:** Wait for Virgil's approval before appending.

On approval, append hooks to `Marketing/Research/content-ideas.md`:

```markdown

## X/Twitter — [date]

- [hook 1]
- [hook 2]
- ...
```

Confirm: "Appended [count] hooks to content-ideas.md"

## Pain Clusters

Monitor these 6 clusters weekly. Each maps to a real audience frustration validated through prior research.

| # | Cluster | What to look for |
|---|---------|-----------------|
| 1 | AI Tool Overwhelm | "tried 11 tools, nothing moved" — marketers drowning in options |
| 2 | AI Content Sameness | "every brand sounds the same" — AI-generated content all looks alike |
| 3 | AI Time Waste | "45 min prompting for 1 paragraph" — AI taking more time, not less |
| 4 | Enterprise-Only Cases | "works for Nike, not me" — case studies that don't apply to SMBs/agencies |
| 5 | AI Hype Fatigue | "stop telling me AI will change everything" — backlash against AI marketing |
| 6 | Tool ROI Questions | "which AI tools actually work?" — demand for honest reviews and proof |

## Search Queries

Rotate these weekly. Use 2-3 per scan (don't repeat the same set two weeks in a row).

**Set A (frustration-focused):**
- `"AI marketing" frustration OR complaint OR problem`
- `"AI content" generic OR "all sounds the same" OR "every brand" OR "same output"`
- `"AI tools" overwhelmed OR confused OR "too many"`

**Set B (practitioner-focused):**
- `"AI tools for agencies" honest OR real OR actually`
- `"marketing AI" results OR ROI OR "actually works"`
- `agency owner "AI" challenge OR struggle`

**Set C (trend-focused):**
- `"AI content" "all looks the same" OR generic OR boring`
- `"AI marketing tools" review OR comparison OR "which one"`
- `marketer "stopped using" AI OR "went back to"`

## Complements Reddit

| Source | Audience | Pain type |
|--------|----------|-----------|
| Reddit monitoring | Audience 1 (PPC practitioners) | Platform-specific: attribution, tracking, bid strategy |
| X/Grok monitoring | Audience 2 (Broad AI marketers) | Tool-level: overwhelm, sameness, ROI, hype fatigue |
| Both | Feed → content-ideas.md → Blog → Newsletter | |

## What NOT to Do

- Don't fabricate posts. If Grok returns nothing for a cluster, report it empty.
- Don't run more than 6 API calls per scan (cost control).
- Don't append to content-ideas.md without Virgil's approval.
- Don't read content-ideas.md in full (50K+ tokens). Grep only.
- Don't skip the archive — always save to x-analysis-history/ with dated filename.
