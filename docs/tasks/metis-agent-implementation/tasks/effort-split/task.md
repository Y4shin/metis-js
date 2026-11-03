---
type: task
subtype: grilling
title: Decide the child-effort split and the ticket-to-effort handoff
status: stable
workflow_state: ready
blocked_by: []
---

# Grilling: the child-effort split and the ticket handoff

## The decision to settle

The operator wants each implementation ticket to become its own effort. Two
decisions follow, and neither can be settled from the specification:

1. **The split.** How many child efforts the specification becomes, and what each
   one owns. The candidates are one effort per specification area (runtime,
   workflows, daemon and API, plugins, surfaces, terminal), one effort per
   implementation ticket, or a hybrid in which a shared foundation is one effort
   and the rest follow areas.
2. **The handoff.** What a ticket in this umbrella effort contains, how it names
   its child effort, and what condition marks it done. The harness has no
   spawn-effort ticket type, so the ticket body and the changelog carry the
   handoff instead.

## Known choices

- The umbrella ends at `to-spec` and `to-tickets`; it writes no production code.
- The umbrella is archived only after every child effort is archived.
- Success is a complete requirement-to-effort mapping, so the split must cover
  every normative requirement exactly once.

## Recommended answer

One effort per specification area, because it keeps the count manageable while
each area still owns a coherent slice, and because the areas follow the
requirement groups. Add a separate foundation effort if the daemon and client
skeleton turns out to be a prerequisite for several areas. A ticket says which
child effort it starts, which requirement range that child owns, and what
acceptance criterion proves it; the ticket is done when the child effort is
archived.

## Downstream work this answer creates

The ticket list, the requirement-to-effort coverage matrix, and the archive
condition for the umbrella.
