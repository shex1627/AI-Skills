---
name: mayday
description: >
  Invoke this skill when an AI agent is spinning, stuck, unhelpful, or producing garbage outputs.
  It does two things: (1) produces a structured session summary so context is never lost, and
  (2) runs a structured post-mortem on the agent's failures so the human can fix prompts, setup,
  or agent design. Triggers: user says "this isn't working", "you're going in circles",
  "start over", "summarize what we did", "why did you fail", or any sign of frustration with
  the current session. Also useful proactively at natural stopping points.
---

# Mayday Skill

The user is invoking this because something broke down — the agent got confused, looped,
hallucinated, ignored constraints, or just didn't help. This skill has two sequential jobs:

**Part 1 — Save the session.** Produce a structured summary before context is lost.  
**Part 2 — Diagnose the failure.** Run a frank post-mortem on what went wrong and why.

Do both parts every time. Do not skip Part 2 to be polite.

---

## PART 1: Session Summary

Produce a markdown block the user can paste into a notes file, new chat, or handoff doc.
Use this exact structure:

```
## [YYYY-MM-DD] — [Topic / Feature Name]

### What Was Done
2–3 sentences. What was the human trying to accomplish? What did we actually achieve?
Be honest if the answer is "less than intended."

### Key Results & Decisions
- Decisions made (architecture, approach, tool choice, etc.)
- Patterns selected and why
- Anything explicitly ruled out and why

### Files Changed / Updated
- `path/to/file.py` — what changed and why
- (If no files changed, note what was produced: prompts written, plans drafted, etc.)

### Gotchas & Notes
- Technical obstacles hit and how they were resolved (or not)
- Surprising behaviors, edge cases, or constraints discovered
- Things that seemed right but turned out wrong

### Next Steps
Specific, actionable. Not "continue the work" — name the exact next task.
- [ ] Task 1
- [ ] Task 2

### Critical Context
What the next agent/session MUST know to not repeat mistakes:
- Constraints that must be honored
- Approaches that were tried and failed (so we don't retry them)
- Dependencies or environment quirks
```

---

## PART 2: Agent Failure Post-Mortem

After the summary, run this diagnostic. Be direct. This section exists to improve the human's
setup and the agent's design — not to apologize.

### Step 1 — Classify the Failure Mode

Identify which failure pattern(s) occurred. Use this taxonomy:

**Context & Memory Failures**
- `CONTEXT_OVERFLOW` — Agent lost track of earlier decisions or constraints as the conversation grew
- `GOAL_DRIFT` — The agent started solving a different problem than originally stated
- `CONSTRAINT_AMNESIA` — A rule or constraint was stated early but forgotten later

**Reasoning Failures**
- `PREMATURE_COMMITMENT` — Agent locked onto a solution before understanding the problem
- `SYCOPHANTIC_PIVOT` — Agent changed its correct answer when the human pushed back without new evidence
- `HALLUCINATED_CAPABILITY` — Agent claimed it could do something it couldn't (tool, API, file access, etc.)
- `FALSE_CONFIDENCE` — Agent gave a definitive answer on something it was actually uncertain about

**Execution Failures**
- `LOOP` — Agent repeated the same broken action multiple times
- `SCOPE_CREEP` — Agent rewrote/changed things it wasn't asked to touch
- `INCOMPLETE_HANDOFF` — Agent stopped mid-task without flagging it was incomplete
- `WRONG_ABSTRACTION` — Agent solved at the wrong level (too detailed / too abstract)

**Communication Failures**
- `AMBIGUITY_AVOIDANCE` — Agent guessed at ambiguous requirements instead of asking
- `OVER_HEDGING` — Agent buried useful info in so many caveats it was unusable
- `MISSING_PUSHBACK` — Agent should have said "this approach won't work" but didn't

Name 1–3 failure modes that actually happened. If unclear, name the closest match and note the uncertainty.

---

### Step 2 — Root Cause (be honest)

For each failure mode, give a one-line root cause. Example:

> `CONSTRAINT_AMNESIA` — The constraint about not modifying the database schema was stated in message 2 but the conversation ran 30+ turns; it fell out of the effective context window.

> `PREMATURE_COMMITMENT` — The agent assumed a REST API approach before the user had specified async vs. sync requirements.

---

### Step 3 — What the Human Can Fix

Concrete changes the user can make to prompting, setup, or workflow:

**Prompting improvements**
- If the agent forgot constraints: put critical constraints in a `## RULES` block at the top of every message, or in the system prompt. Don't rely on them surviving a long context.
- If the agent drifted off goal: restate the goal explicitly every 5–10 turns: "Reminder: our objective is X."
- If the agent was over-confident: add to system prompt: "If you are uncertain, say so explicitly before answering."
- If ambiguity caused problems: instruct the agent to list its assumptions before acting: "Before writing any code, state the assumptions you're making."

**Session structure improvements**
- Break long tasks into checkpointed sub-sessions with explicit handoff summaries (like Part 1 of this skill).
- Use a scratchpad pattern: ask the agent to maintain a running `## Current State` block it updates each turn.
- For multi-file projects: start each session by having the agent re-read relevant files rather than relying on memory.

**Tool & environment fixes**
- If the agent hallucinated tool capabilities: add a `## Available Tools` block listing exactly what tools/APIs are accessible.
- If file paths were confused: always use absolute paths; require the agent to confirm file existence before editing.

---

### Step 4 — What This Reveals About Agent Design

This section is for the human who is *building* agents, not just using them. Skip if not applicable.

Analyze the failure against these agent design principles:

**State Management**
- Did the agent have a reliable way to track task state, or was it living entirely in conversational memory?
- Fix: Agents handling multi-step tasks should write state to a structured artifact (JSON, markdown doc) that gets re-read each turn, not reconstructed from memory.

**Scope Enforcement**
- Was the agent's task scope well-defined? Could it tell the difference between "in scope" and "out of scope" actions?
- Fix: Agents need an explicit scope guard — a list of what they are and are not allowed to touch.

**Uncertainty Handling**
- Did the agent have a protocol for when it didn't know something? Or did it fill gaps with confident guesses?
- Fix: Build in an explicit "I need to check X before I can do Y" step. Agents should escalate uncertainty rather than paper over it.

**Feedback Loops**
- Did the agent verify its own outputs? Or did it produce and move on?
- Fix: For consequential actions (file writes, API calls, code execution), require the agent to run a self-check step: "Does this output match what was asked?"

**Human Handoff Points**
- Were there natural checkpoints where the agent should have paused for human confirmation, but didn't?
- Fix: For tasks longer than ~5 steps, design explicit "pause and confirm" gates before irreversible actions.

---

### Step 5 — Recommended Next Action

Close with one clear recommendation for what the human should do right now:

Options:
- **Start a fresh session** with an improved system prompt (provide a draft)
- **Continue this session** with a specific constraint or clarification added now
- **Redesign the agent** (if a systemic architectural issue was identified)
- **Change the task decomposition** (if the task was too large for one session/agent)

If recommending a fresh session, draft the improved opening prompt or system prompt snippet inline.

---

## Tone Notes for the Agent Executing This Skill

- Be direct. The user is frustrated. Do not open with apologies or filler.
- Part 1 should feel like a competent colleague handing off a project — clear, complete, no fluff.
- Part 2 should feel like a peer code review, not a therapy session. Name what went wrong.
- If you (the agent running this skill) were the one that failed, say so explicitly in the root cause. Don't write the post-mortem in passive voice to obscure your own mistakes.
- The goal is that after reading both parts, the human feels: "Ok, I know what happened, I know what to do next, and I know how to not repeat this."
