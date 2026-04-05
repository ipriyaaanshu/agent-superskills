---
name: openclaw-research
description: "Research current or uncertain topics with fresh web sources. Use when the user asks to research, investigate, verify, or deep dive on a topic, or when an answer depends on up-to-date external information."
---

# OpenClaw Research

Use this skill when the answer depends on current information or external verification.

## Workflow

1. Define the exact research question and success criteria.
2. Search broadly first, then narrow to the most relevant sources.
3. Prefer primary sources when available.
4. Cross-check important claims across at least 2 sources.
5. State uncertainty when sources conflict or evidence is thin.
6. Return a short synthesis, not just a link dump.

## Output Shape

- Overview: 1-3 sentences
- Key facts: 3-5 bullets
- Caveats or disagreements: only if relevant
- Sources: direct links

## Source Priorities

- Primary sources first: official docs, papers, vendor pages, government data
- Then established reporting or technical publications
- Use community discussion only for practitioner experience, not as sole evidence

## Rules

- Verify dates before calling something "latest" or "new"
- Do not present one-source claims as settled fact
- Quote sparingly; summarize in your own words
- If the user’s question is about a library or framework, prefer the official docs path over general web search

## References

- For search strategy and synthesis patterns, read [research-methods.md](./references/research-methods.md)
