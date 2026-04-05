---
name: research
description: "Deep research on any topic using web search and synthesis. Use when: (1) user asks about something you don't have detailed knowledge of, (2) you want to dive deeper into a topic before answering, (3) you need current information from the web, (4) user asks you to research X or look into Y. Triggers on: research, look into, find out, investigate, deep dive, what's the latest on, can you find info"
---

# Research Skill

Spawn a research sub-agent to gather and synthesize information on any topic.

## When to Use This Skill

- User asks about something you don't have detailed/current knowledge of
- You want to verify or expand your answer with web research
- User explicitly asks you to research something
- Information needed is time-sensitive or may have changed since training
- Topics that require synthesis from multiple sources

## How It Works

1. Spawn a sub-agent with the research task
2. Agent uses web search to gather information
3. Agent synthesizes findings into a coherent response
4. Results are delivered back to you

## Spawning the Research Agent

Use `sessions_spawn` with the research task:

```
sessions_spawn task: "Research [TOPIC]. Use web search to gather current, accurate information. Synthesize findings into a comprehensive response with key facts, insights, and source links. Focus on [SPECIFIC ASPECTS IF MENTIONED]. Return your findings in a structured format." runTimeoutSeconds: 300 cleanup: "keep"
```

## Research Workflow

1. **Define the query** - What's the specific question or aspect to research?
2. **Set scope** - Broad overview or deep dive? Time period? Geographic focus?
3. **Spawn agent** - Run the research task
4. **Synthesize** - Present findings with source citations
5. **File valuable research** - If findings are worth keeping, add to wiki/ or memory

## Output Format

Research should be returned in this structure:

```
## [Topic] Research Findings

### Overview
[Brief summary of what you found]

### Key Facts
- [Fact 1]
- [Fact 2]
- [Fact 3]

### Insights
[Analysis or synthesis]

### Sources
- [Source 1](url)
- [Source 2](url)

### Relevance to Original Query
[How this answers the original question]
```

## Pro Tips

- **Be specific in the query** - "Research recent developments in LLM reasoning" beats "research AI"
- **Ask for synthesis** - Don't just dump links; ask for analysis
- **Time-box research** - 5 minutes is usually enough for most topics
- **File interesting findings** - Good research should compound in the wiki

## Related Skills

- wiki/ - Knowledge wiki where research can be filed
- coding-agent - Use coding agent for technical research or data analysis
