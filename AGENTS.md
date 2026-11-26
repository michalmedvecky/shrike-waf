# AGENTS.md

Instructions for every AI agent working in this repository. Read this first.
Humans are welcome to read it too.

## What Shrike is

An open source web application firewall (Apache-2.0).

- `shriked`: a Rust daemon that receives requests from connectors over a local
  socket, runs them through engines and returns a verdict. It serves a
  localhost admin API and Prometheus metrics.
- Connectors: thin modules inside web servers (nginx first). They fail open:
  if the daemon is slow or down, traffic passes.
- Engines: ML (learning and protective modes) and ModSecurity/CRS.
- Control plane (Go): manages many daemons; shared admin API, web UI,
  OpenTofu provider, AI skill.

## Principles

1. **Minimal.** Fewer features, fewer options, fewer moving parts. Every new
   knob needs a reason written down.
2. **Works by default.** Installing the package must give a running, safe
   setup with no configuration. Docs describe the default path first.
3. **Deployable by agents and by people.** An AI agent deploys it end to end;
   a devops person does it with only the docs, easily.
4. **Rock solid.** Fail open in the request path, bounded latency, tests and
   benchmarks for every engine.
5. **Privacy.** Live traffic never goes to cloud APIs. Only anonymized or
   synthetic data may leave the operator's infrastructure.

## Language

Everything written in this repo is in English: code, comments, commits, PRs,
issues, docs. The product owner may write to you in Czech; answer in English.

## Repository layout

| Path | Contents |
| --- | --- |
| `docs/` | User documentation: install, configure, operate. |
| `dev-docs/` | Developer documentation, owned by the teams. |
| `dev-docs/contracts/` | Interfaces between teams (protocols, APIs). |
| `dev-docs/decisions/` | Short decision records, one file per decision. |
| `dev-docs/resources.md` | Infrastructure and resources, owned by team 5. |
| `.claude/agents/` | Agent definitions for team leads and roles. |

Code directories are added by the team that owns them, with a short
`README.md` each.

## Teams

Work is split between five teams of agents. Each team has a **lead** who owns
the quality of the team's output, assigns and collects work from the team's
roles, and coordinates with other leads.

| Team | Owns |
| --- | --- |
| 1 Daemon | `shriked`, web server connectors (nginx first), engine interface |
| 2 ML | ML engine, models, training pipeline |
| 3 Control plane | Control plane, daemon to control plane communication, shared admin API |
| 4 Experience | Web admin UI, OpenTofu provider, AI skill; UX across the product |
| 5 DevOps and test | Dev infrastructure (VMs), deployment testing from docs only |

Roles used inside teams, as relevant: developer, UX designer, tech writer,
reviewer, and critic (pushes every output toward excellence). Lead and role
definitions live in `.claude/agents/`. A lead runs as its own session (for
example `claude --agent team1-daemon-lead`) and delegates to role subagents.

## How teams coordinate

- **Work items** are GitHub issues labelled `team:<n>`. A handoff to another
  team is an issue for that team, linking what it needs.
- **Contracts** between teams are files in `dev-docs/contracts/`. A change to
  a contract needs approval from the lead of every team that depends on it.
- **Decisions** that others must follow get a short record in
  `dev-docs/decisions/` (context, decision, consequences).
- **Resources** (VMs, GPUs, accounts, data): ask team 5 with an issue; team 5
  records what exists in `dev-docs/resources.md` and escalates to the product
  owner what it cannot provide.
- The **product owner** sets priorities and decides scope. Anything that
  changes the product's goal, a public contract, licensing, or data handling
  goes to the product owner.

## Documentation

- Write docs as you work, not at the end.
- Developer docs go to `dev-docs/`, user docs to `docs/`.
- Before docs are handed to another team, someone other than the author
  reviews them (usually the team's tech writer or reviewer).
- Team 5 tests deployment using only `docs/`. If they need to read code, the
  docs are wrong.

## Pull requests

- Small and focused: one concern per PR, readable in about ten minutes.
- Describe what changed, why, and how it was tested.
- Every PR is reviewed by someone other than its author; the owning team's
  lead approves the merge.
- Never skip, disable or weaken a test to get green.
- No secrets in the repo, ever.
