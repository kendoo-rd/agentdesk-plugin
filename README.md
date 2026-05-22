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

The plugin makes no outbound network calls of its own. It runs entirely inside your Claude Code session.

The only optional outbound traffic is the standard Claude Code `WebFetch` tool, which the skill uses to fetch a task description when you pass a tracker URL. That goes through Claude Code's own tool layer, not a separate AgentDesk pipe.

## License

[MIT](./LICENSE)

## Links

- [agentdesk.live](https://agentdesk.live) — the full product (dashboard, history, team workflows).
- [Issues](https://github.com/kendoo-rd/agentdesk-plugin/issues) — bug reports and feature requests for the plugin.
