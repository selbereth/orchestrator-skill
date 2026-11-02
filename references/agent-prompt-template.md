# Agent prompt template

A sub-agent starts cold. It knows nothing of this conversation, so the brief must stand alone. Copy the block
below, fill every field, delete the lines that do not apply. A brief that fits on one screen is usually enough;
if it needs more, put the detail in a file and point the agent at it.

```
GOAL
<The user's goal, in the user's words where possible. One or two sentences. Say what "done" looks like.>

CONTEXT
- Repo / working directory: <absolute path>
- Read first (only these): <2-4 absolute paths, e.g. the progress note, the design note>
- Current state you must respect: <what other agents or the user changed recently; what is known to be broken>

HOW TO RUN THINGS
- Interpreter / command conventions: <e.g. "python -I, with tools/ on sys.path">
- Long jobs: start in the background, then read the log at milestones. Do not use streaming monitors.
- Each tool call must finish in well under the time limit; use bounded waits only.

RULES
- <Safety rules from the user, verbatim where they are strict.>
- <Exclusive resources: "you own X" or "you must not touch X until ...". See exclusive-resource-protocol.md.>
- Do not edit: <files or regions owned by other agents>.
- Do not widen scope. If you find something outside the task, write it in the report; do not fix it.
- Do not claim a fix works unless you ran the check named below. Say which checks you did.
- Before any action that cannot be undone (a deletion, a push, a sent message, a sale), stop and report; do not do it.

DELIVERABLE
- Change(s): <files and what changes, or "no code changes">
- Findings file to append (dated section, do not overwrite others): <absolute path, or "none">
- Checks: <the exact commands or live steps; state which ones are verified live and which only compile or read>

REPORT (your final message, at most N words)
Use these headings:
- Done: what changed, with paths
- Verified: what you ran and what it showed (live vs compile-only)
- Observations: facts you saw directly
- Inferences: conclusions you drew, marked as such
- Unknowns / blocked: what you could not establish, and what would settle it
- Suggested next step: one line
```

## Writing notes

- Quote the user's exact words for any rule they set. Paraphrases drift.
- Name the lock or handoff signal when the work shares a resource (a file, a process, a device).
- Give acceptance checks as observable facts ("the log line `destroyed` appears with N swings"), not as "make it
  work".
- The model is set on the `Agent` call (SKILL.md section 5), not in the brief: `model: "haiku"` by default,
  `"sonnet"` for a bumped task, or whatever the user asked for. Haiku follows a brief literally, so state the goal,
  paths and checks exactly.
- Keep the report cap short (about 150-250 words). Long reports cost the orchestrator's context on every turn.
