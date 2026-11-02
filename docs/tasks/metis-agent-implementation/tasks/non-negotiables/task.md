---
type: task
subtype: grilling
title: Settle the non-negotiables of implementing docs/spec/ across spawned efforts
status: stable
workflow_state: ready
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
