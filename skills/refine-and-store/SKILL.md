---
name: refine-and-store
description: Grill the user on each ticket/task using the grill-me technique, then store the final refined version to memory partitioned by year, month, and day. Use when the user wants to refine tickets, plans, or tasks and persist the outcomes for daily summaries.
---

## Purpose

Dual-purpose system: **agent memory** (recall what was done, pick up context across sessions) and **brag document** (track accomplishments over time).

## Storage Location

```
{workspace}/.claude/memory/{YYYY}/{MM}/{DD}/{NN}-{slug}.md
```

- Workspace-local (co-located with the code)
- Date-partitioned for daily summaries
- Numbered (`01-`, `02-`, ...) for ordering within a day
- Slugged for readability
- **Gitignored** — personal working memory, not team docs

## What Gets Stored

Any meaningful output where a **decision was made** or an **artifact was produced**:
- Refined tickets or plans
- Architecture/design decisions
- Bug diagnoses and fixes
- Implementation strategies
- Anything someone would want to reference later

**NOT stored:** quick lookups, file exploration, mid-conversation questions.

## Trigger

When you recognize a **completed output** (a plan reached final form, a ticket was refined, a decision was made), offer:

> "Want me to store this to memory?"

Store only after explicit user confirmation. Do NOT auto-detect from words like "good" or "yes" mid-conversation — only offer when a distinct deliverable is complete.

## Entry Format

```markdown
# {Descriptive Title}
**Type:** {Decision | Plan | Ticket | Fix | Discovery}
**Date:** {ISO timestamp}
**Context:** {1-2 sentence summary of what prompted this}

## Outcome
{The actual result — could be a refined ticket, a decision, a plan, a diagnosis}

## Key Decisions
- {Decision and rationale}

## Next Steps
- {What follows from this, if anything}
```

Lean entries omit sections that have nothing to say (e.g., skip "Key Decisions" if it was straightforward).

## Read Behavior

### On session start
Check `{workspace}/.claude/memory/{YYYY}/{MM}/{DD}/` for today's entries. If they exist, read them for continuity with prior conversations.

### On demand (rollup)
When asked "what did I do this week/month?", read across date folders and produce a summary grouped by type or theme.

## Grilling Workflow (when refining)

1. Ask one probing question per turn
2. Provide your recommended answer based on codebase exploration
3. Resolve each decision branch before moving to the next
4. Continue until shared understanding is reached
5. Present the final version and offer to store

## Rules
- One question at a time during grilling
- Explore the codebase to answer questions yourself when possible
- Never write to memory without explicit user confirmation
- Each output gets its own file — don't bundle unrelated work in one file
- Number files sequentially within a day
