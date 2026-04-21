# Work-tracker: session bookends

Paste the block between the `BEGIN`/`END` markers below into your project's `AGENTS.md` / `CLAUDE.md`. **Keep the markers in place** — they delimit the managed block so the skill (or a human) can update the content later without guesswork. Don't put other content inside the markers.

```markdown
<!-- BEGIN work-tracker:session-bookends -->
## Session bookends
...
<!-- END work-tracker:session-bookends -->
```

---

<!-- BEGIN work-tracker:session-bookends -->
## Session bookends

This project uses the work-tracker skill for session continuity. Sessions have two bookends — a **resume** at the start and a **wrap-up** at the end — and the agent should proactively help with both.

### At the start of a session

If `work/active/` has items and the human opens with a generic greeting ("hi", "hello", "ok let's go", or jumps straight into a new topic), **proactively summarise what happened last time before anything else**:

1. Read the latest entry in `work/.sessions/$USER.md` — the "What happened" and "On my mind" sections.
2. Scan `work/active/` — Status + Tasks from each active item.
3. Tell the human: "Last time we …; what was on your mind was …; active threads are …" — concrete, not a question.
4. Then offer: "want to pick one up, review the backlog, or start something new?"

The goal is to *orient* the human, not wait for them to ask. If they've said "where were we" explicitly, you're already doing this — just do it unprompted too when the signals above are present.

### At the end of a session

Remind the human early-ish that the session has a wrap-up phase: when they wind down, they should say "let's wrap up" / "let's stop here" / "summarise" so the agent can take notes. Without this signal, the session ends and nothing gets captured.

When the human signals winding down ("ok that's enough", "I'm done", "going to stop"), prompt them:

> "Want me to wrap up the session? I'll update active tickets and append to the session log."

If they say yes, run the work-tracker summarise flow (update work items + append to `work/.sessions/$USER.md` with What happened / On my mind / Open threads).

Don't force either bookend. If the human declines or redirects, drop it — but don't silently skip the wrap-up; a brief "want me to capture this?" at the end is cheap and easy to decline.
<!-- END work-tracker:session-bookends -->
