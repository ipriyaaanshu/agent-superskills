# Agent SuperSkills

Public skills repository for AI agents.

## Available Skills

### openclaw-research

Deep research skill that spawns sub-agents to gather and synthesize information on any topic.

**Triggers on:** research, look into, find out, investigate, deep dive, what's the latest on, can you find info

**Use when:**
- User asks about something you lack detailed knowledge of
- You want to verify or expand your answer with web research
- You need current/time-sensitive information
- User explicitly asks to research a topic

[Learn more →](openclaw-research/SKILL.md)

---

### meta-ads-cli

Operate Meta Ads through Meta's official Ads CLI for campaign management, asset discovery, insights extraction, and analytics workflows.

**Triggers on:** meta ads, facebook ads, ad campaigns, ad sets, creatives, insights, ROAS, ad analytics, marketing automation

**Use when:**
- You need to inspect Meta ad accounts, pages, datasets, or catalogs
- You want to create or update campaigns, ad sets, creatives, or ads from the terminal
- You need structured JSON insights for reporting or analytics workflows
- You want safer, paused-first ad operations with reproducible command recipes

[Learn more →](meta-ads-cli/SKILL.md)

---

## Installation

### Via skills

Install the full repo:

```bash
npx skills add ipriyaaanshu/agent-superskills
```

Install a specific skill only:

```bash
npx skills add ipriyaaanshu/agent-superskills --skill meta-ads-cli
```

### Manual Installation

Copy a skill folder to your agent's skills directory.

Example:

```bash
cp -r meta-ads-cli ~/.agents/skills/
```

For OpenClaw:
```bash
cp -r meta-ads-cli ~/.openclaw/workspace/skills/
```

---

## Contributing

This repo contains public skills for AI agents. To contribute:

1. Fork this repo
2. Add your skill under `<skill-name>/SKILL.md`
3. Include a clear `name` and `description` in the frontmatter
4. Test the skill works with your agent
5. Submit a PR

---

## License

MIT
