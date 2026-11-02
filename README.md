# Orchestrator skill

A skill for Claude Code that turns the main thread into an orchestrator. That thread stays free to talk to you. It
keeps a task list and hands the real work (code edits, long runs, live tests, research, docs) to sub-agents it
spawns. It steers them with messages and reports back in plain language.

You can interrupt at any time and get an answer straight away, because the main thread is never stuck in a long run.

## Install

**Whole account (claude.ai upload).** Zip this folder, then upload it in claude.ai under **Settings → Capabilities →
Skills**. Claude Code installs signed in with that account pick it up automatically, as
`anthropic-skills:orchestrator`.

**One machine (Claude Code, all projects):**

```bash
git clone https://github.com/selbereth/orchestrator-skill ~/.claude/skills/orchestrator
```

**One project:** clone it into `<project>/.claude/skills/orchestrator` instead.

## Use

- Type `/orchestrator`, or say something like "keep this thread open for me and spawn agents to do the work".
- Give it work as it comes up. Each item becomes a task with an owner and a state.
- Ask "what's outstanding?" at any time for an answer taken from the task list.
- Say "loop", "keep going" or "work through the backlog" to start continuous mode: it works through the task list
  on its own, adds whatever agents find, and runs discovery rounds when the list is empty. It pauses to check in
  when two discovery rounds in a row find nothing new, or when what is left is minor (the long tail).
- Say "stop looping" (or just "stop" while looping) to end the loop but keep the orchestrator.
- Say "stop orchestrating" to end it. Running agents stop at a safe point and you get a summary of what is still open.

## What it does

- Sorts every message into one of these: new work, a correction to running work, a question, a status request, or a
  change of priority. Each kind is handled its own way.
- Spawns every sub-agent on Haiku, to keep the work cheap. When Haiku fails a task once, misreads it, or the task
  needs reasoning across many files, it re-spawns that task on Sonnet and tells you. Two failed attempts mark the
  task blocked.
- Spawns agents in the background and keeps each one's ID, so it stays free to talk and can message them later.
- Writes a self-contained brief for each agent: the goal, paths, rules, deliverable, checks, and a capped report.
- Runs code-only agents in parallel. Runs agents that share one exclusive resource (a game, a device, a database)
  one at a time, with handoffs.
- Checks what agents claim before passing it on, and keeps "verified live" separate from "compile only".

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The skill itself: role, message routing, task-list rules, briefs, models, reporting, anti-patterns, continuous mode. |
| `references/agent-prompt-template.md` | Fill-in template for agent briefs. |
| `references/exclusive-resource-protocol.md` | One-owner rules, queueing, handoff messages, deadlock handling. |

## Requirements and limits

- It needs Claude Code, or another client that has the `Agent` and `SendMessage` tools. In a plain claude.ai chat
  the skill loads, but it can't spawn agents.
- If the client has no task tools (TaskCreate, TaskUpdate, TaskList), the task list is kept in `BACKLOG.md` in the
  working directory instead, along with the loop state, so a loop survives context compaction.
- It is a prompt, not enforcement. In a long session the context can be compacted. If the orchestrator starts doing
  work itself, ask it to re-read the skill.
- Agent results arrive when each agent stops. The orchestrator does not poll them.
- Agents start cold, so the brief is the only context they get.
- Sub-agents run on Haiku (`model: "haiku"` on every spawn) unless you ask for another model, or a task is bumped
  to Sonnet as above. The orchestrator thread keeps the session's model.
- Agents are told to stop and report before anything that cannot be undone (a deletion, a push, a sent message).
- Other standing preferences belong in your memory or CLAUDE.md. The skill checks them before each brief.
