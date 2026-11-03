---
type: map
title: Implement Metis Agent from the specification, split across spawned efforts
status: stable
---
# Implement Metis Agent from the specification, split across spawned efforts

## Destination

A working Metis Agent, built from `docs/spec/` alone, arrived at through a set of
spawned implementation efforts rather than one large ticket set. The program is
done when every normative requirement in `docs/spec/` is owned by a spawned
effort and that effort has reached an implemented state whose acceptance
criterion passes, so a complete requirement-to-effort map exists with nothing
normative unmapped.

## Constraints

- The implementation is TypeScript, including anything an addon would have been.
  No native shared library, no foreign-function binding, no compiled addon.
- The agent harness starts on `@earendil-works/pi-durable`. Its fit against
  `03-agent-runtime.md` is tested, not assumed.
- The specification stays implementation-agnostic. A concrete choice belongs in a
  decision record, never back in the specification text.
- This effort writes no production code. Prototypes are throwaway and live in the
  effort's `.work/`.
- Each implementation ticket becomes one spawned effort. The umbrella ends at
  `to-spec` and `to-tickets` and is archived only after every child is archived.
- The platform set is `CMP-007`: Linux, macOS, Windows desktop, and Android; no
  iOS. The license is AGPL-3.0 (`LEG-001`).

## Non-goals

- Porting or consulting the repository the specification's provenance points at.
  It is not an input at any stage, for this effort or any effort it spawns.
- Implementing the specification in this effort. Decisions, prototypes, and the
  specification change only.
- Shipping production code from a prototype. Prototype code that survives is
  recorded as a decision and re-built by the child effort that owns it.
- One effort with a large ticket set. Every implementation ticket becomes a
  spawned effort.
- Reintroducing a concrete choice into `docs/spec/`. A decided choice is a
  decision record, and the specification keeps asking the question.

## Non-negotiable facts

**Success test:** every normative requirement in `docs/spec/` is owned by exactly
one spawned effort, and each child effort reaches an implemented state whose
acceptance criterion for that requirement passes, so a complete
requirement-to-effort map exists with nothing normative unmapped.

- `docs/spec/` is the only input. The repository its provenance points at is
  never consulted.
- The specification names no implementation choice. Every concrete question is in
  `15-open-questions.md` and is answered by a decision record in this effort.
- The implementation is TypeScript, including addons, and the plugin contract is
  an abstract shape realized by the implementation.
- The agent harness uses `pi-durable` unless a prototype shows it cannot meet
  `03-agent-runtime.md`.
- Every implementation ticket becomes one spawned effort, and the umbrella is
  archived only after every child effort is archived.
- The platforms are the ones `CMP-007` fixes, and no iOS implementation exists.
- The application is distributed under AGPL-3.0 (`LEG-001`), and the compiled-in
  trust set is empty (`SEC-006`).

## Decisions so far

- `docs/spec/` is the only input, and the repository its provenance points at is
  never consulted. Recorded by the `non-negotiables` grilling task.
- `docs/spec/` was rewritten in this pass to be implementation-agnostic: no
  language, framework, library, wire protocol, transport mechanism, container
  format, signature algorithm, encoding, byte layout, fixed field width, or
  operator-facing name remains. Every removed choice became an open question
  (`OQ-103` to `OQ-123`), and seven retired requirement identifiers were marked.
  An independent reviewer with no history of the session verified the result.
- The implementation language is TypeScript, including addons; the plugin
  contract is an abstract shape realized later by the implementation.
- The agent harness starts on `pi-durable`, subject to a prototype.
- Success is measured by requirement ownership: every normative requirement is
  mapped to exactly one spawned effort whose acceptance criterion passes.
- Open questions are decided in this effort before any child effort starts.
- Build-time proofs remain a live question (`OQ-112`): the specification keeps
  `HST-043` as a general requirement to assert boundary hand-offs at build time
  where the toolchain can prove the property.

## Fog

- What form a plugin's code takes and how the daemon runs it in its own process:
  a portable module instantiated by a runtime, or a module loaded into the
  daemon's own runtime (`OQ-103`).
- How pi-durable's durable sessions, tasks, and events line up against
  `03-agent-runtime.md`, and where the gaps are. No prototype has run.
- What the requirement-to-effort coverage matrix looks like, and therefore how
  the success test is checked.
- How many child efforts the specification splits into, and what each one owns.
  Deferred by the operator until the ticket list exists.
- What the plugin manifest, bundle, and payload formats should be
  (`OQ-106` to `OQ-110`), and which of them the child efforts may choose
  independently.
