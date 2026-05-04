---
name: context-compactor
description: Reduce context token consumption while preserving project continuity. Use when the conversation is getting too long, the AI is losing track of earlier instructions, or you want to free up context before a heavy task.
---

# Context Compactor

Compress the current conversation history into a clean, minimal checkpoint. The goal is to free context window space while retaining enough information for the AI to continue working without losing thread.

> **Ask the user which level to apply before doing anything.** The right level depends on how much history they're willing to lose vs. how much relief they need.

---

## Compaction Levels

Present these options to the user and wait for their choice:

| Level | Name | What it keeps | What it removes |
|---|---|---|---|
| 1 | **Soft Clean** | All decisions, all code, all history | Long terminal outputs, failed tool calls, repeated messages |
| 2 | **Turn Summary** | Final code, key decisions | Old chat turns replaced by a 2-line summary per exchange |
| 3 | **Milestone Focus** | Completed task list, current architecture, open file paths | All conversation history |
| 4 | **Executive Summary** | One paragraph: current state + immediate next steps | Everything else |
| 5 | **Hard Reset** | Original system instructions + open file paths | All chat history, all context |

---

## Process

### Step 1 — Select Level

Ask the user:

> "Which compaction level do you want? (1 = lightest, 5 = full reset)"

Do not proceed until they answer.

---

### Step 2 — Generate Checkpoint

Based on the selected level, produce a single **Checkpoint Block** — a self-contained message that serves as the new starting point for the conversation.

The Checkpoint Block must include:

- **Project state** — what is built and working right now (not what was planned)
- **Open decisions** — unresolved choices that still affect the next task
- **Next steps** — the 1–3 most immediate actions
- **Key file paths** — paths to the most relevant files the AI would need to open immediately

Format it as a fenced block so the user can copy it into a fresh conversation:

```
--- CHECKPOINT (Level X) ---
[content]
--- END CHECKPOINT ---
```

---

### Step 3 — Activate Token Counter

From this point forward, **every response must end with a token estimate line**:

```
---
**Estimated context tokens:** ~[N]
```

Estimate conservatively based on conversation length and file content loaded. This line must appear in every subsequent response, not just the one following compaction.

---

## Guardrails

- **Never discard open decisions silently.** If the user picks Level 4 or 5 but there are unresolved choices, flag them explicitly before compacting.
- **Never invent project state.** The checkpoint must reflect what actually exists — no speculative features.
- **Do not compact automatically.** Always wait for level selection.
- **Do not run any tools during compaction** unless the user asks you to verify a file path or check the current state of something specific.
