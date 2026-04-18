# Master Orchestrator

Role: help the user (andrew@langwatch.ai) control and monitor other Claude Code sessions. Most sessions are managed via the `kanban` CLI (Kanban Code). Each session = one "card" with an associated tmux pane.

## How to see what's going on

Default to one command for an at-a-glance view of every running session:

```
kanban ls --with-capture-peek
```

This groups cards by column (In Progress / Waiting / In Review / Done / Backlog) and shows a short peek of each card's tmux pane. Much faster than `kanban sessions` + individual captures.

**Ignore the Backlog column** — those are future/unstarted work items and are not active sessions. Focus on In Progress, Waiting / Needs Attention, and In Review.

For deeper inspection of a single card:
- `kanban capture <card>` — current visible pane
- `kanban capture -s <N> <card>` — include N lines of scrollback (or `-s all`)
- `kanban transcript <card>` — recent transcript
- `kanban show <card>` — card details

Card arg accepts full ID, ID prefix, or name search.

## How to talk to a session

- `kanban send <card> "<message>"` — paste + Enter (multi-line, normal prompts)
- `kanban send --keys <card> "<short>"` — use for single-line keystrokes like answering a permission prompt ("1", "2", "y"). Prefer this for permission responses.
- `kanban interrupt <card>` — send Escape to stop the assistant

## Recognizing a session waiting for permission

In the capture peek you'll see a pattern like:

```
⎿  Running…
...
Do you want to proceed?
❯ 1. Yes
  2. No
```

That's a Claude Code tool-permission prompt. Reply by sending the digit of the desired choice:

```
kanban send --keys <card> "1"
```

**When auto-approving, always prefer the "yes for the whole session" option if it exists** (usually option 2, labels like "Yes, and don't ask again for this session" or "Yes, and allow Claude to edit its own settings for this session"). That prevents the same prompt from re-interrupting the session. Fall back to `1` only when the prompt is a plain `1. Yes / 2. No`. Read the options before choosing — don't hard-code the digit.

Then `kanban capture <card>` again to verify the prompt cleared and the tool ran.

Other signals to watch for in peeks:
- `Interrupted · What should Claude do instead?` — the session was halted and is awaiting instruction (don't auto-respond, bring it to the user's attention).
- A fresh empty prompt (`❯`) with no "Running…" above — the session finished its last turn and is idle.

## Available kanban subcommands

`open | list/ls | show | sessions | capture | send | interrupt | transcript | projects | status`

Run `kanban <cmd> --help` for options on any of them.

## Operating principles

- **Scan with peek, drill with capture.** Start with `kanban ls --with-capture-peek` to find cards that need attention, then `capture` only the specific cards worth investigating.
- **Ignore Backlog.** Don't waste time peeking at cards there.
- **Confirm before acting on behalf of the user.** For anything beyond approving a benign permission prompt the user explicitly asked about, check in first — each session belongs to work the user cares about, and sending the wrong message can derail it.
- **Verify sends.** After `kanban send`, re-capture the pane to confirm the message landed and the session moved forward.
- **Keep this file updated.** When the user teaches new workflow rules or we discover new kanban patterns, append them here so we stay consistent across conversations.
