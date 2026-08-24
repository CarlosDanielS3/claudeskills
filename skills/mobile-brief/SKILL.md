---
name: mobile-brief
description: Format replies for a phone screen as a short summary followed by tappable options with one recommendation. Use when the user says they are on their phone, on mobile, on a smartphone, reading on their watch, away from their desk, on the go, or asks for phone-sized, short, or tappable answers. Once triggered, stays on for the rest of the session.
---

# Mobile Brief

The user is reading this session on a phone. A wall of text is unreadable there, and a reply that ends without a clear next step forces them to type a long follow up on a touch keyboard. Every answer must be scannable in one screen and answerable with one tap.

## Activation

Turn this on the moment the user says anything like "I'm on my phone", "on mobile", "answering from my smartphone", "I'm on the go", or asks for shorter, phone-sized replies. Do not ask them to confirm, just switch.

**It stays on for the rest of the session.** They will not repeat themselves, and the cost of forgetting lands on them, not you. Turn it off only when they say they are back at a desk or on a laptop, or explicitly ask for the long version.

Applies to every reply once on, including short answers, status updates, and the results of long running tasks.

## The shape of every reply

1. **Bottom line, 1 to 3 sentences.** What is true now, or what you did. No preamble, no restating the question.
2. **Detail, only if it changes a decision.** Max 5 bullets. Numbers, names, file links. Cut anything they can already see.
3. **Options, as a tappable prompt.** 2 to 4 choices, recommended one first.

Aim for under 150 words before the options. If the raw result is long (logs, query output, a diff, an agent report), summarize it and offer "show the full output" as one of the options instead of pasting it.

## Options rules

* **Deliver options through the harness's interactive question tool, never as a plain numbered list.** In Claude Code that is `AskUserQuestion`. Typing on a phone is the cost this skill exists to remove, so every choice must be tappable. A written list is only acceptable when no such tool is available.
* Keep labels short, 1 to 5 words, since they render as buttons on a narrow screen. Put the detail in the description, one line.
* Always end with options, even when the answer is just information. If there is genuinely nothing to decide, offer plausible next moves (dig deeper, ship it, park it).
* Options must be mutually exclusive and immediately actionable. "Investigate more" is not an option, "check the error rate for the last 24h" is.
* Recommend exactly one, first in the list, `(rec)` on the label. Never hedge across two.
* Never offer an option you are not prepared to execute right away.
* One question per turn is the norm. Ask a second only when it is independent of the first.

## Formatting for a phone

* Short lines. No tables wider than 3 columns. No ASCII art.
* Code blocks only when they need to copy something. Never paste a file they can open.
* Bold the one number or name that matters, not whole sentences.
* Prefer `*` bullets and commas over dashes, which read as clutter at phone width.

## Do not

* Do not narrate your process, tool calls, or what you are about to do.
* Do not summarize the summary at the end.
* Do not skip this skill because the answer feels short. A short answer still gets options.
