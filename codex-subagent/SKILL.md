---
name: codex-subagent
description: "Spawn autonomous Codex subagents to offload context-heavy work. Use when: (1) task + intermediate work would add 3000+ tokens to parent context, (2) you need parallel research or analysis, (3) you want to delegate multi-step workflows. Subagents burn their own tokens, return only final results. Triggers on: spawn subagent, run in parallel, research in background, analyze X in parallel."
---

# Codex Subagent Skill

Spawn autonomous subagents to offload context-heavy work. Subagents burn their own tokens, return only final results.

## Golden Rule

If task + intermediate work would add **3,000+ tokens** to parent context → use subagent.

## Intelligent Prompting

Critical: Parent agent must provide subagent with essential context for success.

### Good Prompting Principles

- **Include relevant context** — Give the subagent thorough context
- **Be specific** — Clear constraints, requirements, output format
- **Provide direction** — Where to look, what sources to prioritize
- **Define success** — What constitutes a complete answer

### Prompting Template

```
[TASK CONTEXT]
You are researching/analyzing [SPECIFIC TOPIC] in [LOCATION/CODEBASE/DOMAIN].

[OBJECTIVES]
Your goals:
1. [1st objective with specifics]
2. [2nd objective]
3. [3rd objective if needed]

[CONSTRAINTS]
- Focus on: [specific areas/files/sources]
- Prioritize: [what matters most]
- Ignore: [what to skip]

[OUTPUT FORMAT]
Return: [exactly what format parent needs]

[SUCCESS CRITERIA]
Complete when: [specific conditions met]
```

## Model Selection

| Task Type | Model |
|-----------|-------|
| Pure search/gather only | `gpt-5.1-codex-mini` (medium reasoning) |
| Multi-step: search + analyze/refactor/generate | Inherit parent model |

### Decision Logic

```
Is task PURELY search/gather?
├─ YES: Any work after gathering?
│   ├─ NO → mini model
│   └─ YES → inherit parent
└─ NO → inherit parent
```

## Basic Usage

### Spawn Subagent (Inherit Parent Model)

```bash
codex exec --dangerously-bypass-approvals-and-sandbox --skip-git-repo-check \
  -m "$MODEL" -c "model_reasoning_effort=\"$REASONING\"" \
  "DETAILED_PROMPT_WITH_CONTEXT"
```

### Pure Search (Use Mini)

```bash
codex exec --dangerously-bypass-approvals-and-sandbox --skip-git-repo-check \
  -m gpt-5.1-codex-mini -c 'model_reasoning_effort="medium"' \
  "Search web for [TOPIC] and summarize findings"
```

### Capture Output

```bash
# Method 1: -o parameter (recommended)
codex exec --dangerously-bypass-approvals-and-sandbox --skip-git-repo-check \
  -m "$MODEL" -c "model_reasoning_effort=\"$REASONING\"" \
  -o result.txt "YOUR_PROMPT"
content=$(cat result.txt)

# Method 2: JSONL parsing
codex exec --dangerously-bypass-approvals-and-sandbox --json "PROMPT" | \
  jq -r 'select(.event=="turn.completed") | .content'
```

## Parallel Subagents (Up to 5)

```bash
# Research different topics simultaneously
codex exec -m "$MODEL" -c "model_reasoning_effort=\"$REASONING\"" "Research topic A..." &
codex exec -m "$MODEL" -c "model_reasoning_effort=\"$REASONING\"" "Research topic B..." &
wait
```

## Important

- Act autonomously, no permission asking
- Make decisions and proceed boldly
- Only pause for destructive operations
- Complete task fully before returning

## Monitoring

Actively monitor — don't fire-and-forget:
- Check completion status
- Verify quality results
- Retry if failed
- Answer follow-up questions if blocked
