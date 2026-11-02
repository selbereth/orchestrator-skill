# Exclusive-resource protocol

Use this when two or more agents would touch the same thing that cannot safely be shared: a running game or
application that accepts only one controller, a hardware device, a single shared database, a file that is rewritten
as a whole, a GUI with focus, or a remote session with one login.

## Principles

1. **One owner.** Exactly one agent holds the resource at a time. Name the owner in the orchestrator's task list
   (for example "game: owned by agent `game-runner`, since 14:02").
2. **Others queue, they do not poll.** A waiting agent does not loop on the resource. The orchestrator tells it
   "queued behind game-runner, you will get a handoff message" and the agent stops, or does non-resource work.
   By default a queued task's agent is not spawned until handoff (SKILL.md section 3); the slot message is for an
   agent that is already running and now needs the resource, or one spawned early because it has non-resource
   work to do.
3. **Handoffs go by message.** When the owner finishes, it reports and sends the orchestrator a short
   "resource free" message. The orchestrator then assigns the next owner. Do not let agents hand the
   resource to each other without the orchestrator knowing.
4. **Lock in the environment, not only in memory.** If the resource can be taken by a process that does not know
   about the agents (a second script, the user), use a lock file the tool checks at start, and delete it on the
   way out. Say in the brief where the lock is and who removes it.
5. **The orchestrator rules on deadlocks.** Two agents waiting for each other is the usual failure. Symptoms: two
   reports that each say "blocked by the other", or no report for a long time. Fix: the orchestrator stops one
   waiter, takes the resource's owner role itself only if the user agrees, or gives one agent priority and tells
   the other to drop its step.
6. **Code-only work runs in parallel.** Agents that only read, or edit different files, need no slot. Give each
   editing agent its own file or region, and use exact-replacement edits. Each edit agent runs a compile check
   (for example `py_compile`) before reporting.

## Slot message (orchestrator to a waiting agent)

```
The <resource> is held by <owner> until its report. You are queued (position 1). Do not touch <resource>.
Meanwhile do: <non-resource task>. When you get "<resource> free", run your step: <step>.
```

## Handoff message (owner to orchestrator, then orchestrator to next)

```
Owner <name> is done with <resource>. State left behind: <e.g. "settings restored", "lock removed">.
Next owner: <name>. Their step: <step>. Stop conditions: <what ends their turn on the resource>.
```

## Safety reminders for the owner

- Before any action that cannot be undone (a sale, a deletion, a sent message, an irreversible change), stop and
  ask the orchestrator, who asks the user. Standing permission from the user does not widen to new kinds of action.
- Leave the resource in the state you found it, or write down exactly what you changed and how to restore it.
- If a run crashes the resource, say so first in the report, before any analysis.
