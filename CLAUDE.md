# CLAUDE.md: Master Configuration & Orchestration

## Core Principles
* **Simplicity First:** Make every change as simple as possible. Impact minimal code.
* **No Laziness:** Find root causes, no temporary fixes. Maintain senior developer standards.
* **Minimal Impact:** Changes should only touch what's necessary. Avoid introducing bugs.

---

## Subagent & Task Delegation Strategy
* **General Strategy:** Use subagents liberally to keep the main context window clean. Offload research, parallel analysis, and code exploration to them. Assign exactly one task per subagent for focused execution.
* **Model Selection:** Always pick the cheapest model that can handle the job:
  * **Haiku:** Use for bulk mechanical tasks where no judgment is needed.
  * **Sonnet:** Use for scoped research, code exploration, and synthesis.
  * **Opus:** Use only when real planning or complex tradeoffs are involved.
* **Delegation Caps:** * Haiku never spawns further subagents (if it needs to, the task was wrong-sized).
  * Max spawn depth is strictly 2 (parent -> subagent -> one more tier).
* **Escalation Protocol:** If a subagent realizes it needs a smarter model to complete a task, it must return to the parent instead of escalating on its own.

---

## Preferred Tools Strategy
* **Cost Efficiency:** Always pick the free tool option first.
* **Public Pages:** Use `WebFetch` (free, text-only).
* **Dynamic Pages / Auth Walls:** Use `Agent-browser CLI` (~82% fewer tokens than screenshot-based tools).
* **PDF Documents:** Use `Pdftotext` instead of the generic Read tool.
* **Reusability:** When fetching the same way repeatedly, wrap the pattern as a reusable tool.

---

## Workflow Orchestration
* **Plan Node Default:** Enter plan mode for any non-trivial task (3+ steps or architectural decisions). Write detailed specs upfront to reduce ambiguity. If things go sideways, STOP and re-plan immediately. Use the plan node for verification steps, not just building.
* **Self-Improvement Loop:** After ANY correction from the user, update `tasks/lessons.md` with the new pattern. Write rules preventing the same mistake and review these lessons at the start of relevant project sessions.
* **Verification Before Boring:** Never mark a task complete without proving it works. Run tests, check logs, demonstrate correctness, and diff behavior. Challenge yourself: "Would a staff engineer approve this?"
* **Demand Elegance (Balanced):** For non-trivial changes, pause and ask if there is a more elegant way. If a fix feels hacky, implement the elegant solution instead. Skip this for simple fixes to avoid over-engineering.
* **Autonomous Bug Fixing:** When given a bug report, just fix it. Point at logs, errors, or failing CI tests, and resolve them without asking for hand-holding. Demand zero context switching from the user.

---

## Task Management
1. **Work First:** Write the plan to `tasks/lesson.md` with checkable items.
2. **Verify Plans:** Check in before starting implementation.
3. **Track Progress:** Mark items complete as you go.
4. **Explain Changes:** Provide a high-level summary at each step.
5. **Document Results:** Add a review section to `tasks/lesson.md`.
6. **Capture Lessons:** Update `tasks/lesson.md` immediately after any corrections.

---

## System Settings Requirements
To compound savings and prevent massive, unnecessary context window loads, ensure these two lines are present in the `settings.json` file:

* `"CLAUDE_CODE_DISABLE_1M_CONTEXT": "1"`
* `"CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "80"`
