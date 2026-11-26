# Work plan and team split

Status: proposal for the product owner's review. Nothing in this plan starts
until the product owner says so.

This plan turns the product specification
([Smart WAF product specification](https://claude.ai/artifact/5EPsr23HyS2K8AvFR2hFKS),
accepted 2026-11-25) into phases, team work packages and an order of work.
It uses the teams and rules of [AGENTS.md](../AGENTS.md). Work package IDs
`W0`–`W16` are the spec's (section 14); `X1`–`X12` are added here for work the
spec does not list.

## 1. Summary

- Six phases to the MVP (`v0.1.0`), each ending in a gate with written exit
  criteria. Pace is set by gates, not dates.
- Three things are done early because everything else depends on them: the
  spec moves into the repo with the Shrike names (`X1`), the three core
  contracts land (`W1`, `W2`, `X2`), and the daemon link is measured (`W9`,
  gate P2) before anything heavy is built on it.
- The longest path is the ML data, not the code. To shorten it, an internal
  **capture build** is deployed to a site of the product owner's as soon as the
  daemon, nginx module and event store work (phase 3), so benign traffic
  collects while the model is trained.
- Teams 1, 2 and 5 carry the MVP. Team 3 owns the admin API contract and its
  implementation from phase 1. Team 4 owns UX, naming and the user docs in the
  MVP, then `shrikectl` and the AI skill (M2), the dashboard (M6) and the
  OpenTofu provider (M7). Team 6 (marketing) starts at the MVP release
  candidate.
- The product owner is needed at six points only (section 8), plus the
  resources in section 7.

## 2. Phases to the MVP

| Phase | Goal | Work | Exit criteria (all must hold) |
| --- | --- | --- | --- |
| **P0 Foundation** | Teams can work | X1, X3, X4, W0, X5 | PR #1 merged. Spec in `dev-docs/spec/` with Shrike names. CI green on x86-64 and ARM64 (fmt, lint, test, licence, audit). One issue per work package in this plan, labelled `team:<n>`. Dev VMs listed in `dev-docs/resources.md`. |
| **P1 Contracts** | Interfaces fixed so teams run in parallel | W1, W2, X2, X6 | Core types and engine trait compile and are documented. `openapi.yaml` complete for spec section 7 and lint-clean. Protocol codec passes golden frames; fuzz 1 h clean. Each contract approved by the lead of every team that consumes it (section 4). |
| **P2 Link proof** | Prove the daemon approach is fast enough | W3, W4, W8, W5, W9 (P2, P3), X7 | `shriked` serves the protocol; nginx module passes end-to-end and ASan. **Gates P2 and P3 pass on the reference machine**, or the product owner decides how to proceed (section 8). |
| **P3 Capture build** | Real benign traffic starts collecting | W6, W7, X8, X9 | Local API and event store done with redaction and clean sampling. Internal `.deb` installs on Debian 12. Running at the product owner's site in detect mode; fail-open verified there; first export moved to the Spark by the operator. |
| **P4 Model** | A model that passes its gates | W10, W11, W12, W13, W14 | Bundle passes every gate of spec 11.3, trained on public and synthetic data only, with evaluation report and model card. P4 to P7 and the memory gate pass. ARM64 numbers from the Spark reported. |
| **P5 MVP release** | Install with defaults and it works | W15, W16, X10, X11 | Signed packages for every supported OS, arch and nginx version. Nightly matrix green. **A team 5 tester passes the acceptance scenario (spec 14.3) on a fresh VM using only `docs/`.** Critic and reviewer sign-off from every lead. Product owner tags `v0.1.0`. |

P2 and P4 run partly in parallel: the ML side (W10, W12 with a fixture model)
starts as soon as the normaliser (W3) lands. The two sides share only the
normaliser and the bundle format.

## 3. Work packages per team

Owner is the team that delivers and answers for it; "with" teams review or
contribute a part. Every package ends in pull requests that state which
acceptance criteria they meet (spec 14.4).

### Team 1: Daemon

| ID | Package | Phase | With | Deliverable |
| --- | --- | --- | --- | --- |
| W0 | Repository bootstrap (Rust workspace, CI jobs) | P0 | 5 | Workspace layout, CI on both arches, CONTRIBUTING.md, SECURITY.md placeholder |
| W1 | Core contracts: types and engine trait (spec 3, 4) | P1 | 2, 3 | `shrike-core` crate, documented, no engine logic |
| W2 | Protocol codec (spec 5) | P1 | | `shrike-proto` (no_std), golden frames, fuzz target |
| W3 | Normaliser with Python bindings | P2 | 2 | Field extraction and decoding, golden vectors, fuzz, bindings equal to Rust output |
| W4 | Pipeline and no-op engine | P2 | | Policy, exceptions, deadlines, panic isolation, integration tests |
| W5 | Daemon | P2 | 3 | `shriked`: socket, config, atomic change sets, revisions, SIGHUP, graceful stop |
| W8 | nginx connector | P2 | 4 | Dynamic module per spec 8, e2e on one OS and nginx version, ASan run |
| W9 | `shrike-bench` and link benchmark | P2 | 5 | Method of spec 11.4, P1–P3 results committed |
| W7a | Event store (storage, redaction, retention, queue, export) | P3 | 2, 3 | Events of spec 6.7, `shriked events export`, failure case 8 |

### Team 2: ML

| ID | Package | Phase | With | Deliverable |
| --- | --- | --- | --- | --- |
| X6 | Data manifest and licence review | P1 | | `training/data/manifest.yaml` with every planned source, licence, train or eval-only flag, live-traffic flag |
| W10 | Data pipeline (ingest, extract, split, augment) | P4 | 5 | Stages runnable, attack lab traffic recorded, benign set of 1M requests or a written gap statement |
| W11 | Model training (train, calibrate, export, evaluate, bundle) | P4 | | `make model` reproduces a bundle; CPU-only tiny CI run passes |
| W12 | ML engine runtime | P4 | 1 | Spec 9 in full, tested with a fixture model |
| W13 | First model release | P4 | 1, 5 | Public bundle passing 11.3, model card, shipped as `builtin:general` |
| W14 | ML performance | P4 | 1, 5 | P4–P7 and memory gate on reference machine; ARM64 report |
| X12 | Private model line | after W13 | 5 | Same pipeline on live data, kept on the Spark and the product owner's installs only |

### Team 3: Control plane and shared API

| ID | Package | Phase | With | Deliverable |
| --- | --- | --- | --- | --- |
| X2 | Admin API contract | P1 | 1, 4 | `dev-docs/contracts/admin-api/openapi.yaml` for spec 7, versioning and error model, lint in CI |
| W6 | Local API and metrics | P3 | 1 | `shrike-api` crate mounted by `shriked`: every endpoint of 7.2 except events, token handling, metrics of 6.8, contract tests |
| W7b | Event endpoints and labelling | P3 | 1 | Event endpoints of 7.2, labelling with exception creation |
| X8 | Generated API clients and contract test suite | P3 | 4, 5 | Rust and Go clients generated from `openapi.yaml`; a contract suite any implementation (daemon now, gateway in M5) must pass |
| C10 | Daemon to control plane link (draft) | during P4 | 1 | Contract draft for spec 15.5 so M5 can start right after the MVP |

Team 3 owns the API layer as its own crate so it never edits team 1's code;
team 1 owns the daemon that mounts it.

### Team 4: Experience

| ID | Package | Phase | With | Deliverable |
| --- | --- | --- | --- | --- |
| X4 | Naming and UX conventions | P0 | all | Proposed names (section 9) and a one-page convention for CLI, config keys, API fields, metrics, messages |
| X7 | UX review of the default path | P2 | 1, 3 | Review of nginx directives, `shriked.yaml` defaults, API shapes and error messages against "install and it works" |
| X10 | Operator guide and API reference | P5 | all writers | `docs/` per spec 13.3: install, configure, events, false positives, detect to block, upgrade, troubleshoot; API reference generated from `openapi.yaml` |
| M2 prep | `shrikectl` and AI skill design | during P4 | 3 | Design notes ready so M2 starts at once |

### Team 5: DevOps and test

| ID | Package | Phase | With | Deliverable |
| --- | --- | --- | --- | --- |
| X3 | Dev infrastructure | P0 | | Dev VMs provisioned and documented in `resources.md` |
| X5 | Self-hosted runners | P0–P2 | | Runners on the reference VM pair (perf) and the Spark (ARM64, training) |
| W0 | CI support | P0 | 1 | Matrix, caching, licence and audit jobs |
| X9 | Capture deployment | P3 | 1, 2 | Capture build on the product owner's site; export procedure to the Spark per spec 10.5 |
| — | Isolated attack lab | P2 | 2 | No-egress network with vulnerable apps and attack tools, recorded through `shriked` (for W10) |
| W15 | Packaging and release pipeline | P5 | 1 | Packages of spec 13 for every OS, arch, nginx version; install, upgrade, removal tests; hardening check; signed artifacts |
| W16 | End-to-end matrix | P5 | all | Nightly matrix green |
| X11 | Docs-only acceptance | P5 | 4 | Tester runs spec 14.3 from `docs/` only; every gap becomes an issue for the owning team |

### Team 6: Marketing

Not staffed for the MVP. Starts at the P5 release candidate: website, release
announcement, community channels. Needs a lead definition in
`.claude/agents/` before it starts.

## 4. Cross-team contracts and their order

Contracts live in `dev-docs/contracts/`. A change needs approval from every
consumer's lead. Changes to the core model, engine trait, protocol and admin
API (spec 3, 4, 5, 7) also go to the product owner when they change a public
interface.

| # | Contract | Owner | Consumers | Must land by |
| --- | --- | --- | --- | --- |
| C1 | Core types and engine trait (spec 3, 4) | 1 | 2 | end of P1 (first) |
| C2 | Connector protocol (spec 5) | 1 | future connectors, 5 (tests) | end of P1 (first) |
| C3 | Admin API, `openapi.yaml` (spec 7) | 3 | 1, 4, 5 | end of P1 (first) |
| C4 | Config and resource schemas, incl. per-engine config schema (spec 6.2, 6.3, 9.5) | 1 (ML schema: 2) | 3, 4, 5 | before W5 |
| C5 | Normaliser output and Python bindings (spec 3.2, 10.1) | 1 | 2 | W3, before W10 and W12 |
| C6 | Model bundle format `.swm` (spec 9.3) | 2 | 1, 3, 5 | before W12 |
| C7 | Event record and export format (spec 6.7) | 1 | 2, 3, 4 | before W7, and before the capture build |
| C8 | Metric names and labels (spec 6.8) | 1 | 4, 5 | before W6 |
| C9 | Package layout and install behaviour (spec 13.1) | 5 | 1, 4 | before W15 |
| C10 | Daemon to control plane link (spec 15.5) | 3 | 1 | before M5 |

C1, C2 and C3 land first and together. Everything in P2 builds on them.

## 5. Critical path and dependencies

Dependencies (spec 14.1, plus the added packages):

| Package | Needs |
| --- | --- |
| W0 | X1 (names fixed before crates are created) |
| W1, W2, X2 | W0 |
| W3, W4 | W1 |
| W8 | W2 |
| W5 | W2, W4 |
| W9 | W5, W8 |
| W6, W7 | W5, X2 |
| Capture build (X9) | W6, W7, W8, internal package |
| W10 | W3, attack lab; benign data from X9 |
| W11 | W10 |
| W12 | W3, W4, C6 |
| W13 | W11, W12 |
| W14 | W9, W13 |
| W15 | W6, W7, W8 |
| W16 | W13, W15 |

- **Critical path:** X1 → W0 → W1 → W3 → W10 → W11 → W13 → W16. Data
  collection and labelling dominate it.
- **Biggest lever:** the capture build (P3). Every week it runs earlier is a
  week more of benign traffic for the 1M-request test set.
- **Early go/no-go:** W9 gate P2 at the end of P2. If it fails widely, stop
  and decide before building W6, W7 and W12 on the link.
- **Off the critical path:** W15 packaging. Team 5 builds an internal `.deb`
  for the capture build as soon as W5 exists; the full W15 follows W6 to W8.

## 6. When each team starts

| Team | Starts | First work | Load in MVP |
| --- | --- | --- | --- |
| 1 Daemon | P0 | W0, W1, W2 | High throughout |
| 5 DevOps and test | P0 | X3, X5, CI | Medium, peaks in P3 and P5 |
| 2 ML | P0 (light) | X6 data manifest, attack lab plan; heavy from W3 | High from P2 |
| 3 Control plane | P1 | X2 admin API contract | Medium in P1 and P3, then C10 |
| 4 Experience | P0 (light) | X4 names and conventions | Light until P5, then docs |
| 6 Marketing | P5 | Website and launch | — |

## 7. Resources needed from the product owner

| When | Resource | For |
| --- | --- | --- |
| P0 | Merge PR #1; enable Actions; create labels `team:1`–`team:6` (or allow agents to) | Team process |
| P0 | Branch protection on `main` requiring CI and one review | Rule "reviewed by someone other than the author" |
| P0 | 3 dev VMs (Debian 12, Ubuntu 24.04; 4 vCPU, 8 GB is enough) with SSH for agents | Teams 1, 2, 5 |
| P1 | 2 reference VMs: 8 dedicated vCPUs with AVX2, 16 GB, same network, nothing else running | W9, W14 gates |
| P1 | Permission to register self-hosted runners on those VMs and on the Spark | Perf and ARM64 CI |
| P1 | Job access to the DGX Spark, plus an encrypted volume for live data | W10–W14, spec 10.5 |
| P2 | An isolated network (or VM with no egress) for the attack lab | W10 attack traffic |
| P3 | One site where the capture build may run in detect mode, and a person to move exports to the Spark | Benign data |
| P5 | Release signing key (or approval to use keyless Sigstore signing) | W15 |
| P5 | Security contact address (spec 16.2) | SECURITY.md |
| M3 | Cloud LLM API key, anonymized data only | NL rule authoring |
| M5 | Decision on control plane roles (spec 16.2) | M5 |
| M7 | OpenTofu registry publisher account | M7 |

## 8. Product owner touchpoints

The product owner does not review every step. Leads escalate only:

1. **Start of P0.** Go for this plan.
2. **Names** (section 9). Confirm or change in P0.
3. **P2 gate fails** by a wide margin: choose between profiling, a
   shared-memory transport, or adjusting the gate (spec 16.4).
4. **Capture build on a live site** (P3). It touches production, so it needs
   an explicit go.
5. **First model misses a gate** (W13): approve shipping in detect mode with
   published numbers; gates are never lowered silently.
6. **Release `v0.1.0`.** Final go after the phase report.

At the end of each phase the leads post one short report: exit criteria met,
measured numbers, open risks, next phase.

## 9. Names to confirm

The spec predates the name Shrike. Proposed mapping, applied in `X1`:

| Spec | Proposed |
| --- | --- |
| Smart WAF | Shrike |
| `wafd` | `shriked` |
| `wafctl` | `shrikectl` |
| `wafcp` | `shrike-cp` |
| packages `smartwaf`, `smartwaf-nginx-<ver>` | `shrike`, `shrike-nginx-<ver>` |
| `/etc/smartwaf/`, `/var/lib/smartwaf/`, `/run/smartwaf/` | `/etc/shrike/`, `/var/lib/shrike/`, `/run/shrike/` |
| metric prefix `smartwaf_`, nginx `smartwaf_*` | `shrike_` |
| crates `waf-core`, `waf-proto`, `waf-bench` | `shrike-core`, `shrike-proto`, `shrike-bench` |
| OpenTofu resources `smartwaf_*` | `shrike_*` |

## 10. After the MVP

Specified in spec section 15. Each milestone gets its own package breakdown
when scheduled. Owners:

| Milestone | Owner | With | Can start |
| --- | --- | --- | --- |
| M2 learning mode (site profile) | 2 | 1 (response frame) | after `v0.1.0` |
| M2 `shrikectl`, AI skill | 4 | 3 (clients) | after `v0.1.0`, designed during P4 |
| M2 signed APT repository | 5 | | after `v0.1.0` |
| M9 deceptive responses | 1 | 2 (offender table input), 4 (decoy pack, docs) | after M2 |
| M3 ModSecurity/CRS engine, IP filter | 1 | 5 (CRS regression) | after `v0.1.0`, parallel to M2 |
| M3 NL rule authoring | 4 (loop in `shrikectl`) | 3 (validate, replay endpoints), 2 (anonymiser) | after M3 engine |
| M4 GPU tier | 2 | 1 | after M2 |
| M5 control plane `shrike-cp` | 3 | 1 | after `v0.1.0` (C10 drafted in P4) |
| M6 dashboard | 4 | 3 | after M5 API |
| M7 OpenTofu provider | 4 | 3 | after M5, or earlier against a single daemon |
| M8 HAProxy, Traefik, Apache connectors | 1 | 5 | on demand |

Teams 3 and 4 have spare capacity in the MVP. Using it to start M5 and M2
early is possible but not proposed: it would add surface before the core is
proven.

## 11. Risks

| Risk | Phase | Response |
| --- | --- | --- |
| Not enough benign traffic for the 1M test set | P3–P4 | Capture build first; state the confidence interval honestly in the model card |
| Link slower than gate P2 | P2 | Measured before anything is built on it; profile, then more connections or shared memory; protocol unchanged |
| Rust nginx bindings lack the body handling needed | P2 | Fall back to a C module (spec 8.1) |
| First model misses a gate | P4 | Ship in detect mode with published numbers; iterate |
| Contracts drift between teams | P1 onward | Contracts in repo, consumer-lead approval, contract tests in CI |
| Agent teams move fast and disagree | all | Leads settle in issues; decision records; product owner only for scope, public contracts, licensing, data |
| Live data leaks to a cloud API | all | Per-source flag in the manifest enforced by the pipeline; no automatic upload; reviewer checks every data PR |
| Resources arrive late (VMs, Spark, capture site) | P0–P3 | Requested in P0 with the phase that needs them; work reorders around them rather than stops |
| Spec still lives outside the repo | P0 | `X1` imports it; from then on the repo copy is the contract |

## Appendix: added work packages

| ID | Package | Owner |
| --- | --- | --- |
| X1 | Import the spec into `dev-docs/spec/` and apply the confirmed names | 1 (each lead reviews its sections) |
| X2 | Admin API contract | 3 |
| X3 | Dev infrastructure | 5 |
| X4 | Naming and UX conventions | 4 |
| X5 | Self-hosted runners | 5 |
| X6 | Data manifest and licence review | 2 |
| X7 | UX review of the default path | 4 |
| X8 | Generated API clients and contract test suite | 3 |
| X9 | Capture deployment | 5 |
| X10 | Operator guide and API reference | 4 |
| X11 | Docs-only acceptance | 5 |
| X12 | Private model line | 2 |
