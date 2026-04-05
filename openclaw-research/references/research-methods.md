# Research Methods Reference

## Web Search Tools

| Tool | Use For | Command |
|------|---------|---------|
| `web_fetch` | Extract content from specific URLs | `web_fetch url:...` |
| Browser | Interactive browsing | Chrome DevTools MCP |

## Search Strategy

### 1. Query Formulation

**Broad first**: Start with a general search to understand scope
```
"topic name" OR "related topic" -exclude
```

**Specific**: Narrow down once you understand the landscape
```
site:github.com "topic" since:2025
site:arxiv.org "topic" after:2025-01-01
```

### 2. Source Prioritization

| Priority | Source Type | When to Use |
|----------|-------------|-------------|
| 1 | Primary sources (docs, papers, official blogs) | Technical accuracy required |
| 2 | Established publications (HN, TechCrunch, Ars) | News, trends |
| 3 | Community discussions (Reddit, StackOverflow) | Practical user experiences |
| 4 | Social (X, LinkedIn) | Latest developments, opinions |

### 3. Fact-Checking

- Cross-reference at least 2-3 sources before stating as fact
- Check publication dates — prioritize recent
- Look for consensus vs disagreements
- Note confidence levels: "reportedly" vs "confirmed"

## Topic-Specific Sources

| Topic | Recommended Sources |
|-------|---------------------|
| AI/ML | arxiv.org, huggingface.co, papers.nips.cc |
| Tech News | news.ycombinator.com, techcrunch.com, theverge.com |
| Code/OSS | github.com/trending, stackoverflow.com |
| Research | google-scholar.com, semantic scholar |
| News | news.google.com, reuters.com, apnews.com |
| General | wikipedia.org, wolframalpha.com |

## Synthesis Framework

When combining multiple sources:

1. **Identify consensus** — What do most sources agree on?
2. **Note conflicts** — Where do sources disagree? Why?
3. **Assess quality** — Peer-reviewed > edited > unedited
4. **Acknowledge gaps** — "Evidence suggests X, but Y is unclear"
5. **Connect dots** — What does it all mean together?

## Quick Research Checklist

- [ ] Defined clear research question
- [ ] Searched at least 2-3 sources
- [ ] Verified dates are recent enough
- [ ] Cross-referenced key facts
- [ ] Noted any disagreements or uncertainties
- [ ] Synthesized into coherent findings
- [ ] Included source links
