---
type: task
subtype: prototype
title: Measure pi-durable against the agent runtime requirements
status: stable
workflow_state: ready
blocked_by: []
---

# Prototype: pi-durable as the agent runtime harness

## The question

`03-agent-runtime.md` requires a re-usable runtime with a consumer-supplied model
provider and tool contract, consumer-supplied storage behind a persistence-event
interface, exact reconstruction of idle session state, lifecycle events,
cancellation, execution limits, and extension hooks (`AGT-001` to `AGT-010`,
`CNV-001` to `CNV-011`, `TSK-001` to `TSK-009`, `SES-001` to `SES-010`).
`@earendil-works/pi-durable` v1.5.0 claims durable conversations, tasks, tools,
extensions, SQLite and JSONL storage, and telemetry.

Which requirements does pi-durable already satisfy, which does it satisfy after a
thin adapter, and which would have to be built? Is the exact-reconstruction
requirement (`SES-004`) met by its commit model, and does its storage interface
carry every state change (`SES-005`, `SES-006`)?

## Known choices

- The operator wants pi-durable as the starting point, in TypeScript.
- The runtime must remain usable by an embedding application other than the
  daemon (`CNV-009`), so the adapter boundary matters.
- `pi-durable` is experimental and its API changes between releases.

## Recommended approach

Build a throwaway harness in the effort's `.work/` that runs one conversation and
one bounded task against a temporary SQLite storage, kills the process at every
commit boundary, and checks reconstruction after each restart. Record a
requirement-by-requirement verdict: met, met with an adapter, or absent. Do not
write production code.

## Output

A note in this task recording each requirement's verdict, the adapter surface the
daemon would need, and any requirement that forces a different harness decision.
