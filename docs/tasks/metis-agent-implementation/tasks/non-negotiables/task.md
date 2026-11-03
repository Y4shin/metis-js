---
type: task
subtype: grilling
title: Settle the non-negotiables of implementing docs/spec/ across spawned efforts
status: stable
workflow_state: done
blocked_by: []
---

# Non-negotiables grilling: metis-agent-implementation

## The decision to settle

Settle how `docs/spec/` becomes a working application in this repository, and
what this effort refuses to do. The outcome is the map's `## Non-goals` and
`## Non-negotiable facts` sections, including the effort-level success test.

The shape this effort must serve, in the operator's own framing: this effort
carries the prototyping and grilling. It does not implement the spec itself.
When Wayfinder's planning is over, the tickets this effort produces each become
a new effort that implements its slice of the specification. The whole spec is
implemented once every spawned effort has landed.

## Raw request (verbatim)

> I want you to create an effort, that has the job of taking the spec at
> ./docs/spec/ and implementing it. That will cause there to be a lot of
> tickets, so instead I would like this effort to do most of the prototyping and
> grilling and so on and once the wayfinder phase is over, the tickets then in
> turn just create new efforts themselves which implement the spec once all are
> implemented.

## Destination, in the operator's words

A working Metis Agent, built in this repository from `docs/spec/` alone, reached
through a set of spawned implementation efforts instead of one large ticket set.

## Actor

The operator: owner of this repository, who runs and configures the daemon and
reviews the built application. `docs/spec/01-purpose-scope-and-conventions.md`
lists the further actors the stories draw on (user, surface, plugin author,
workflow author, reviewer).

## Classification

Feature. No defect is reported, so no bug ticket is created at intake.

## Stories

1. As an operator, I want a persistent daemon I own that keeps working with no
   surface attached, so that my work continues and I can reach it from several
   devices. (PRD-G1, PRD-C1)
2. As a user, I want a terminal, a desktop application, and an Android
   application served by one daemon through one client-facing API, so that I am
   not locked to a single device. (PRD-G8, scope §1.5)
3. As a plugin author, I want to ship a capability as one signed file that
   installs by file copy, so that an operator extends the daemon without a build
   step on the target machine. (PRD-G6, PRD-C5)
4. As a workflow author, I want explicit directed acyclic pipelines with three
   step types and declared tool dependencies, so that multi-step work is
   predictable. (PRD-G5, scope §1.5)
5. As a reviewer, I want every requirement traceable to an independently
   checkable acceptance criterion, so that "implemented" is measured rather than
   asserted. (`14-acceptance-criteria.md`)

## Constraints recorded at intake

- **`docs/spec/` is the sole input.** The specification's Provenance section
  points at a source repository for its evidence. That repository MUST NOT be
  consulted. The operator's reason: it contains questionable implementations
  they do not want carried over. This applies to this effort and to every
  spawned effort. `[current]` and `[implemented]` markers therefore read as
  requirement status, not as an implementation to port.
- **Start the agentic harness on `pi-durable`.** The operator wants to try that
  library for the harness. This is the one technology choice fixed before
  grilling; its fit against the spec's runtime requirements is grilling's to
  test, not to assume.
- **Stack is otherwise open.** `docs/spec/index.md` excludes language,
  framework, persistence product, internal module layout, and build system.
  Grilling settles those. Note the repository name `metis-js` and the existing
  domain language file `CONTEXT.md` as context, not as a decision.
- **This effort produces decisions and prototypes only.** No production code
  lands here. Prototypes are throwaway and live in the effort's `.work/`.
  Prototype code that survives is recorded as a decision in the ticket that
  implements it.
- **Tickets here spawn efforts.** The tickets produced from the spec do not
  carry implementation work in this effort; each one names a spawned effort
  that implements its slice. The granularity of those efforts (per spec area
  versus per ticket versus something else) is an open question for planning,
  decided with the real ticket list in hand.
- **Success is effort-level and measurable.** The effort succeeds when the
  spawned efforts' landing is checkable against `14-acceptance-criteria.md` and
  the invariant set in `11-business-rules-and-invariants.md` for every
  requirement group that is in scope.

## Sanity-check record (intake)

- Duplicate and fold-in: `docs/tasks/index.md` lists no live and no archived
  efforts; nothing to fold into.
- Links: no related effort exists. The input is the specification tree, which is
  not an effort.
- Code reach: this repository holds only `AGENTS.md`, `CONTEXT.md`, and `docs/`.
  There is no source to extend, so this is a greenfield build. The
  specification is 15 documents, about 3150 lines, with requirement groups AGT
  through OQ.

## Notes

# Settled record

## Decision, in the operator's terms

1. **The specification is the only input.** The repository the specification's
   provenance points at is never consulted, at any stage, by this effort or by
   any effort it spawns. The operator's reason: it contains implementations they
   do not want carried over.
2. **The specification must be implementation-agnostic.** It names no
   programming language, framework, library, wire protocol, transport mechanism,
   archive or container format, signature algorithm, encoding, byte layout,
   fixed field width, or operator-facing setting or path spelling. Every choice
   removed from a requirement is recorded as an open question in
   `15-open-questions.md`. The rewrite was performed in this pass and reviewed
   by an independent reviewer with no history of this session.
3. **The implementation is TypeScript, including addons.** No native shared
   library, no foreign-function binding, no compiled addon. The plugin contract
   is an abstract shape that the implementation later realizes in its own
   language (`OQ-103`, `OQ-104`, `OQ-105`).
4. **The stack is otherwise decided in this effort**, not in a child effort.
5. **The agent harness starts on `@earendil-works/pi-durable`** (TypeScript,
   v1.5.0: durable conversations, tasks, tools, extensions, SQLite or JSONL
   storage, telemetry). Its fit against `03-agent-runtime.md` is tested by a
   prototype, not assumed.
6. **This effort produces decisions, prototypes, and the specification change
   only.** No production code lands here. Prototypes are throwaway.
7. **Success is a complete mapping.** The program is done when every normative
   requirement in `docs/spec/` is owned by a spawned effort and that effort has
   reached an implemented state whose acceptance criterion passes. "Every
   requirement implemented by this effort" is explicitly not the test.
8. **Tickets become efforts.** Each implementation ticket names a child effort
   and the specification slice it owns. The umbrella ends at `to-spec` and
   `to-tickets`; it is archived only after every child effort is archived.
9. **Open questions are decided here, before children start.** Cross-cutting and
   per-area questions alike, by grilling and prototyping in this effort, so each
   child builds from a decided slice.
10. **Build-time proofs are worth investigating.** The specification keeps a
   general requirement that boundary hand-offs are asserted at build time where
   the toolchain can prove the property (`HST-043`), and `OQ-112` asks which
   properties are worth proving and with what.

## Alternatives considered and rejected

- **A native daemon for the plugin ABI.** Rejected: the operator wants one
  TypeScript implementation, and the specification now leaves the plugin
  mechanism open.
- **Keeping the plugin ABI, bundle bytes, and metadata names concrete**, since
  they are an author-facing contract. Rejected: the operator wants the
  specification independent of the existing realization.
- **Relocating the concrete formats into this program's architecture document.**
  Rejected in favour of dropping them and posing each as an open question.
- **One effort with many tickets.** Rejected: the ticket count would be large,
  and the operator wants each slice to get its own full planning and
  implementation arc.
- **Core-first scope**, deferring the desktop, Android, and Windows surfaces.
  Rejected: the success test is the whole specification.
- **Deciding the spawned-effort granularity now.** Deferred to a planning task,
  because it depends on the real ticket list.

## Constraints and rationale

- The specification's `[current]` and `[implemented]` markers record the status
  of the behavior, not the format it was evidenced in. The rewritten text keeps
  every requirement identifier stable and marks seven retired identifiers where
  sequences had gaps.
- `CONTEXT.md` holds no settled terms yet. The specification's own glossary in
  `02-domain-model.md` is the vocabulary source for the child efforts.
- The implementation must satisfy `LEG-001`: AGPL-3.0, and a modified version run
  as a network service must offer its corresponding source.
- `CMP-007` fixes the platform set: Linux, macOS, Windows desktop, and Android;
  no iOS.
- The test signing key must never become a compiled-in trust anchor
  (`HST-042`), and the compiled-in trust set stays empty (`SEC-006`).

## Consequences for dependent work

- `to-spec` receives a map whose decisions are closed and whose non-goals rule
  out porting any existing implementation.
- `to-tickets` must produce tickets whose target is a child effort, not a code
  change in this effort. The harness has no spawn-effort ticket type, so the
  ticket body carries the handoff and the changelog records the child effort.
- Every child effort inherits the non-goal about the provenance repository and
  the success test about requirement ownership.
- A requirement-to-effort coverage matrix is needed to prove the success test.
  It is not yet designed; that is fog for the next pass.

## Remaining fog and newly discovered work

- **Plugin runtime mechanism.** `OQ-103` to `OQ-105` ask what a plugin is and how
  the host runs it. Under "everything in TypeScript" the realistic candidates
  are a WebAssembly module instantiated by the daemon, or a module loaded into
  the daemon's own runtime. Neither has been tested.
- **pi-durable fit.** `03-agent-runtime.md` requires a consumer-supplied storage
  interface, exact session reconstruction, lifecycle events, cancellation,
  execution limits, and extension hooks. pi-durable claims durable sessions,
  storage backends, tasks, and events. The overlap and the gaps are unmeasured.
- **Requirement-to-effort coverage.** The success test needs a matrix that maps
  every normative requirement to exactly one child effort. Its form is undecided.
- **Child-effort count and boundaries.** Deferred by the operator; depends on the
  ticket list.
- **Specification integrity after the rewrite.** An independent reviewer
  confirmed that no requirement names a language, library, protocol, format,
  algorithm, layout, or spelling, that every referenced identifier resolves, and
  that every open question `OQ-103` to `OQ-123` is cited by a numbered
  requirement. Two blocking findings from its first round (a duplicated
  `AUT-039`, and a version number left in `OQ-061`) were fixed, and a second
  review round was not run: that is a known, accepted gap in verification.
