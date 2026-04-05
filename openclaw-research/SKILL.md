---
name: openclaw-research
description: "Deep research on any topic using web search and synthesis. Use when: (1) user asks about something you lack detailed knowledge of, (2) you want to verify or expand your answer with web research before responding, (3) you need current/time-sensitive information, (4) user explicitly asks to research, look into, investigate, or deep dive on a topic. Triggers on: research, look into, find out, investigate, deep dive, what's the latest on, can you find info, tell me more about."
---

# OpenClaw Research Skill

Spawn a background research agent to gather and synthesize information on any topic.

## When to Spawn

- **Knowledge gap**: You don't have detailed or current knowledge on a topic
- **Verification**: You want to verify facts before answering
- **User request**: User asks you to "research X", "look into Y", "find out about Z"
- **Time sensitivity**: Information may have changed since training data
- **Synthesis needed**: Topic requires combining multiple sources

## Spawn Command

Use `sessions_spawn` in your main session:

```
sessions_spawn task: "Research [TOPIC]. Use web search to gather current, accurate information from multiple sources. Synthesize findings into a comprehensive response with: (1) brief overview, (2) key facts (3-5 bullets), (3) insights or analysis, (4) source links. Return in structured markdown format." runTimeoutSeconds: 300 cleanup: "keep"
```

## Workflow

1. **Clarify scope** (if needed) — Is it a broad overview or specific deep dive?
2. **Spawn agent** — Kick off research with clear topic and any specific aspects
3. **Wait for results** — Sub-agent delivers findings back to you
4. **Present to user** — Synthesize and share the research
5. **File if valuable** — Good research goes to wiki/ for compounding

## Output Format (For Spawned Agent)

```markdown
## [Topic] Research Findings

### Overview
[Brief 2-3 sentence summary of what you found]

### Key Facts
- [Fact 1 with source]
- [Fact 2 with source]
- [Fact 3 with source]

### Insights
[Any analysis, implications, or synthesis from the sources]

### Sources
- [Source Name](url)
- [Source Name](url)

### Relevance
[How this answers the original question]
```

## Tips for Good Research

- **Specific queries**: "LLM reasoning improvements 2025-2026" beats "AI news"
- **Multi-source**: Cross-reference at least 2-3 sources
- **Prioritize recent**: Check dates, prefer latest information
- **Ask for synthesis**: Not just links — what does it all mean?
- **Time-box**: 5 minutes usually sufficient for most topics

## Filing Research

If research is valuable, add to the wiki:
- Create a new page in `wiki/` for the topic
- Or append to an existing relevant page
- Include sources for traceability

## Related Skills

- `wiki/` — File valuable research here
- `coding-agent/` — For technical deep dives or data analysis
