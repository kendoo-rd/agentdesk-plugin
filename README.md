# AgentDesk — Claude Code plugin

Run a software task through a five-phase loop — **intake → plan → execute → review → summary** — with each phase as an isolated Claude sub-agent.

```
INTAKE → PLAN → EXECUTION → REVIEW → SUMMARY
```

The point of the loop is the **REVIEW** phase: a separate Claude reads the diff produced by EXECUTION and flags issues before SUMMARY ships the result. That dedicated review pass is the value the plugin buys you over a single-shot agent run.

State is handed off between phases through one file: `.agentdesk/session-memory.md`.

## Install

Inside any [Claude Code](https://claude.ai/code) session:

```text
/plugin marketplace add anthropics/claude-plugins-community
/plugin install agentdesk@claude-community
```

## Usage

```text
/agentdesk:agentdesk <your task>
```

Or paste a tracker URL (Linear, Jira, or GitHub issue):

```text
/agentdesk:agentdesk https://linear.app/your-team/issue/ENG-123
```

The skill restates the task, asks for confirmation, then runs the five phases sequentially. Total wall time is typically 3–10 minutes for a small-to-medium task.

## Example session

A representative run on a small Node project (illustrative — your output will vary):

```text
> /agentdesk:agentdesk Add a greet(name) function in greet.js that returns
  'Hello, <name>!', plus a node:test covering it. Make sure npm test passes.

I'll run this through the full five-phase loop:
  INTAKE → PLAN → EXECUTION → REVIEW → SUMMARY

Proceed? [y/N] y

▸ INTAKE  · 38s · assessed scope (single session, ~2 files)
▸ PLAN    · 51s · named export, node:assert/strict, one positive test
▸ EXECUTION · 1m 12s · wrote greet.js + greet.test.js · npm test 1/1 pass
▸ REVIEW  · 58s · independent verification · verdict: APPROVE
▸ SUMMARY · 22s

✓ AgentDesk run complete

Added greet.js (ESM named-export greet(name) returning `Hello, ${name}!`)
and greet.test.js (a node:test + node:assert/strict test asserting
greet('World') === 'Hello, World!'). npm test passes 1/1, exit 0. No edits
to package.json and no new dependencies. REVIEW returned APPROVE with no
blockers or should-fix items.

PR: no PR created (no git remote configured)
Session memory: .agentdesk/session-memory.md
```

The REVIEW phase is where the loop earns its keep — it re-runs the test suite independently, reads the modified files in full (not just the diff hunks), and checks the changes against the acceptance criteria PLAN wrote down. On a different task with a subtle defect, REVIEW would flag it as `Blocker` or `Should-fix` and the run pauses for you to decide.

## What you get

- A dedicated REVIEW phase that runs as a separate Claude — independent eyes on the diff before it ships.
- Per-phase prompts focused on one job each (no role confusion, no scope creep).
- Persistent `.agentdesk/session-memory.md` so the work is auditable and resumable.
- Zero install friction: no account, no daemon, no signup. Runs entirely in-process.

## What this plugin does NOT do (v0.1)

- It does not write back to your tracker (Jira / Linear / GitHub Issues). The skill can read a tracker URL via `WebFetch` to pull the task description, but it does not post comments, status changes, or new issues.
- It does not sync to a dashboard or persist sessions beyond the local `.agentdesk/session-memory.md` file.
- It does not run multiple sessions in parallel.

For those features, see the [full AgentDesk product](https://agentdesk.live) — a separate CLI + dashboard that uses the same phased model with tracker integration, team coordination, and session history.

## Data flow

The plugin sends no telemetry. It has no AgentDesk server, account or analytics: nothing about your sessions, code or tasks is sent to AgentDesk.

It runs inside your Claude Code session, so the normal network activity of Claude Code and the tools it runs still happens. During a run, that can include:

- **Claude itself:** your prompts and the files the phases read go to your Claude provider, as in any Claude Code session.
- **`WebFetch`:** when you pass a tracker URL, the intake phase fetches the task description through Claude Code's `WebFetch` tool.
- **`gh pr view`:** the summary phase checks whether the current branch has an open pull request, if the GitHub CLI is installed. This contacts GitHub with your own `gh` login.
- **Your project's own commands:** the execute phase runs the project's tests, lint and type checks. Whatever those commands do on the network (installing packages, calling services in integration tests) happens as it would if you ran them yourself.

All of these run through Claude Code's normal tools and follow your Claude Code permission settings.

## License

[MIT](./LICENSE)

## Links

- [agentdesk.live](https://agentdesk.live) — the full product (dashboard, history, team workflows).
- [Issues](https://github.com/kendoo-rd/agentdesk-plugin/issues) — bug reports and feature requests for the plugin.
