# selfim: design

Status: draft for approval · 2026-10-24

`selfim` is a Claude Code plugin (skill + hooks + CLI) for continuous self-improvement. It captures what happens in agent sessions, distills it into facts, rules and area skills, measures whether they reduce the user's interventions at an acceptable token cost, and promotes, demotes or reverts them. Every change is a git commit with what, why and evidence.

Statements attributed to the user are theirs. Everything marked **(my call)** was chosen by Claude during design and is open to correction; all of them are collected in [08-open.md](08-open.md).

For the overview and diagrams of the workflow, see the [project README](../README.md).

## Files

| File | Covers |
|---|---|
| [01-intent.md](01-intent.md) | why selfim exists, measure of success, goals, constraints, non-goals, anti-goals |
| [02-capture.md](02-capture.md) | how sessions are captured: hooks, digest, session evidence, secrets, learning from existing history, native sources |
| [03-knowledge.md](03-knowledge.md) | what is tracked: scopes, record kinds and fields, project identity, areas |
| [04-review.md](04-review.md) | the review step: who runs it, when something new is added, commits and reverts, suggestions |
| [05-lifecycle.md](05-lifecycle.md) | stats, states and transitions, impact vs. token cost |
| [06-outputs.md](06-outputs.md) | what sessions get: ownership, rule file and `selfim-*` skills, size caps, load levels, pitfall check |
| [07-setup.md](07-setup.md) | runtime repo, layout, first-invoke setup, where the source lives |
| [10-robustness.md](10-robustness.md) | failure handling, concurrency, learning-run safety, prompt injection, pitfall overrides, secrets, two machines, hand edits, uninstall |
| [08-open.md](08-open.md) | Claude's calls to check, what's left to the builder, ideas borrowed and from where |
| [09-stack.md](09-stack.md) | tooling: plugin packaging, runtime, capture sources, storage, git, review agent, supporting pieces |
