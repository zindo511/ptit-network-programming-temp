# AI Usage Log

> **Mandatory** — see [`INSTRUCTION.md` §7](../INSTRUCTION.md#7-ai-usage-policy).
> Record the AI interactions that **mattered** — ones that produced or shaped design, code, tests or documentation
> that ended up in this repository. A few lines each is enough; you do not need to log every prompt.
>
> Write entries **as you go** (right after the session). The AI assistant configuration in this repo asks your
> assistant to remind you and to suggest a draft entry — but the "what we kept / verified" column must be yours.

---

## Log

| # | Date | Member | Tool / model | Task — what you asked | What AI produced | What you kept, changed or rejected — and how you verified it | Files |
|:-:|------|--------|--------------|-----------------------|------------------|---------------------------------------------------------------|-------|
| 1 | 2026-10-19 | Trịnh Quang Huy | Gemini | Translate Vietnamese draft of Problem & Idea into English and refine project title/pitch for README | English translation for §2 (Problem, Idea, Non-trivial challenges, MVP scope) and suggestions for project title & one-sentence pitch | Kept the English translation for §2; refined the one-sentence pitch to clarify that JavaFX is the client and only saved answers are restored; chose the title `ExamLink` | `README.md` |
| 2 | 2026-10-18–19 | Trịnh Quang Huy | Codex | Discuss the team's exam/session-recovery idea; question TCP ACK versus save confirmation, stale retries, heartbeat, deadlines, concurrency and client technology; keep the MVP RAM-only | Technical explanations, alternatives, illustrative Java/SQL snippets in chat, corrections to assumptions, and suggested README wording; no application implementation or runtime tests | Kept the distinction between TCP ACK and application-level save confirmation, per-session locking, virtual-thread handlers and RAM-only MVP; rejected adding a database or Spring framework | `README.md` (wording suggestions in chat), `docs/proposal.md` (design context) |
| 3 | 2026-10-19 | Trịnh Quang Huy | Codex | Turn the discussion into an M1 draft; record Huy's selected session/server-business role and his instruction to assign Đức the JavaFX client and Bằng the remaining networking role; add an accurate AI disclosure | Drafted and revised the proposal and ownership plan; added this disclosure and checked the proposal's relative links; no application code or test evidence generated | Reviewed and approved the M1 proposal structure, scope table and 3-member ownership plan; verified relative markdown links | `docs/proposal.md`, `docs/ai-log.md` |

---

## Notable Interactions

### N1. Human direction and AI assistance in the M1 draft

- **Member / date / tool:** Trịnh Quang Huy / 2026-10-18–19 / Codex.
- **Human input and direction:** Huy brought the team's online-exam and session-recovery idea to the discussion. He chose raw Java sockets for learning, explicitly limited the MVP to RAM, and asked about concrete cases such as a stale save response, submission at the deadline, and old/new connections overlapping. He proposed the acceptor/reader/scheduler split and virtual threads for blocking I/O, requested a shorter README summary, and called for explicit heartbeat, framing and proctor-role descriptions. He selected the session/business-logic role, requested Đức own JavaFX, and authorized the remaining allocation to Bằng. These are discussion inputs and planning decisions, not claims of implemented code or completed tests.
- **AI assistance:** Codex compared approaches, explained failure cases and synchronization, supplied illustrative snippets, proposed responsibilities, and wrote the proposal text from the discussion. The prose was AI-drafted; the project idea and planning inputs above were supplied or directed by Huy. Some details remain proposals for team review.
- **Technical clarifications discussed:** TCP ACK does not confirm an application save; a timeout does not identify the cause of lost contact; virtual threads still consume resources; RAM recovery does not survive a server restart. The role allocation is a plan, not proof of contributions already completed.
- **What was kept, changed or rejected — and verification:** Kept the distinction between TCP ACK and application-level save confirmation, per-session locking, virtual-thread handlers and RAM-only MVP; rejected adding a database or Spring framework. Reviewed and approved the M1 proposal structure, scope table and 3-member ownership plan; verified relative markdown links.

### N2. Items for the team's own review

- Confirm the M1 scope, ownership boundaries and unresolved M2 choices in the proposal.
- Complete the review/verification cells above using actual decisions and evidence. These prompts are not a record that the review has happened.
