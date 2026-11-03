---
type: task
subtype: prototype
title: Prototype the in-process plugin boundary in TypeScript
status: stable
workflow_state: ready
blocked_by: []
---

# Prototype: the plugin boundary under TypeScript

## The question

The specification requires plugins to run inside the daemon's process (`PLG-001`),
to advertise capabilities before being called (`PLG-002`), to be driven by the
host rather than keeping their own worker (`PLG-004`), to return control instead
of blocking (`PLG-005`), and to be initialised, invoked, cancelled, and released
through a versioned contract with declared extent (`PLG-010` to `PLG-028`). The
operator fixed the implementation language to TypeScript and rejected native
libraries and foreign-function bindings, so the contract must be realized in
TypeScript itself.

The decision this prototype answers is `OQ-103` to `OQ-106`: what a plugin's code
is, what the structural shape of the contract is, how pending work and
cancellation are reported, and which encoding carries payloads.

## Known choices

- A portable module, such as a WebAssembly module, instantiated in the daemon's
  runtime. Isolating, but changes the "not a sandbox" stance (`PRD-N3`) and
  constrains a plugin author's toolchain.
- A module loaded into the daemon's own TypeScript runtime. Smallest surface,
  fastest, honestly not a sandbox, but exposes the daemon's own globals.
- A separate process per plugin. Rejected by `PLG-001` unless the operator
  reopens it.

## Recommended approach

Build two throwaway harnesses in `.work/`: one instantiating a portable module,
one loading a module into the daemon's runtime. For each, exercise initialisation,
a capability call that completes later, cancellation, a payload hand-back, and
shutdown. Measure startup cost, call overhead, and what each can enforce. Do not
write production code.

## Output

A note in this task recording the measured comparison, a recommended shape for
`OQ-103` to `OQ-106`, and the consequence for `PRD-N3` and `SEC-007`.
