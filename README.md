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

## Installation

### Via skills.sh

```bash
npx skillsadd <owner>/agent-superskills
```

### Manual Installation

Copy the skill folder to your agent's skills directory:

```bash
cp -r openclaw-research ~/.agents/skills/
```

For OpenClaw:
```bash
cp -r openclaw-research ~/.openclaw/workspace/skills/
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
