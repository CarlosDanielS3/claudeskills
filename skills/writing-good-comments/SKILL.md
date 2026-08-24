---
name: writing-good-comments
description: Standard for writing and reviewing code comments — when to add one, what it must never contain, and how to sweep existing comments while editing nearby code. Use when writing, editing, or reviewing any code comment, in any language.
---

# Writing Good Comments

## Default: no comment

Well-named identifiers explain WHAT. A comment only earns its place when it explains a non-obvious WHY: a hidden constraint, a subtle invariant, a workaround for a specific bug, or behavior that would surprise a reader. If removing the comment wouldn't confuse a future reader, don't write it.

## The test

A good comment survives being read with **zero conversation context**. It should make sense to someone with no memory of "the current task," "this PR," or "what we just discussed."

## Red flags — delete or rewrite on sight

| Pattern | Why it fails | Fix |
|---|---|---|
| Ticket/task ID (`PROJ-3712`, `#1234`, `JIRA-99`) | Rots the moment the ticket closes/renumbers; becomes stale noise that outlives its usefulness | Delete the reference. Keep the WHY if there is one, drop the comment entirely if the ticket ID was the only content |
| "This PR/commit adds...", "per the ticket", "for the Y flow we're building" | Same rot, plus it's restating process, not explaining code | Same as above — that context belongs in the commit message / PR description, which are versioned alongside the change |
| Restates the code (`// increment counter` above `counter++`) | Explains WHAT, which the code already says | Delete |
| References "the current fix/task" without naming a durable reason | Meaningless once the task is forgotten | Either name the durable constraint it protects, or delete |

## When editing existing code

Don't just avoid the anti-pattern in what you add — sweep the comments already touching the lines you're editing. Two things to catch:

1. **Stale task references** left over from whoever wrote it (ticket IDs, "this adds X").
2. **Comments now factually wrong** because your edit changed the behavior they describe (e.g. a comment says "out of scope" for something you just implemented).

A comment you didn't author is still your responsibility once you're editing the code next to it.

## Example

**Before:**
```ts
// PROJ-3712: allow the consumer to call the content service to dereference storageKey.
// The API's own id isn't known at synth time (it's a cross-team resource,
// so this is scoped as tightly as the available exports allow).
```

**After:**
```ts
// Allow the consumer to call the content service to dereference storageKey.
// The API's own id isn't known at synth time (it's a cross-team resource,
// so this is scoped as tightly as the available exports allow).
```

Same WHY, same durability — just no ticket ID riding along for no reason.
