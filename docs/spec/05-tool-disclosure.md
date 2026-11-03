# 5. Tool disclosure

## 5.1 Status

The vocabulary of toolsets, tool views, and a disclosure oracle is settled and is
defined in `02-domain-model.md`. The mechanism is a proposal. The proposal states
that it is implementation guidance for three planned efforts. It has not been
approved as a requirement, and no acceptance evidence exists.

This specification therefore records the proposal's intent as non-normative
design context and records its unresolved decisions as open questions. A finished
application is not required by this specification to implement progressive tool
disclosure. If a future decision accepts the proposal, the statements below
describe the intended semantics and the open questions need answers first.

## 5.2 Proposal intent

The proposal separates authorization from disclosure:

- An agent context's permissions define every tool capability an execution may
  call.
- Disclosure selects a smaller visible subset of those permitted tools for one
  model request.
- Tool dispatch checks the agent context's permissions again.
- Tool visibility never grants permission, and search results never reveal a
  capability outside the agent's permission boundary.

The proposal describes two selection paths:

1. A cheap starter selection, made before the first main model request and
   optionally after a clear topic change. It may use structured resource and
   action metadata, lexical matching, embeddings, or a small selector model. It
   prefers toolsets and may add individual supporting tools. Low confidence falls
   back to pinned tools rather than exposing the complete catalog.
2. A deliberate disclosure oracle that the agent requests through a small
   intrinsic control capability. The request carries a query and a mode of
   `replace` or `add`.

The proposal gives the active tool view these contents: pinned intrinsic tools
needed for control; starter-selected tools; tools or toolsets selected by the
oracle; and a catalog revision plus stable schema identities needed for
reconstruction.

The proposal places the following rules on a disclosure request:

- A tool view changes only at an idle boundary.
- A disclosure request cannot share a provisional assistant response with
  user-visible text or ordinary tool calls.
- The assistant response that requests disclosure remains provisional and is not
  committed to the chat session.
- The oracle runs as a side branch from the last committed session state. The
  branch may use only read-only catalog operations: search toolsets, inspect a
  toolset's members and use case and approximate schema cost, search individual
  tools, and submit one structured selection decision.
- The branch cannot call ordinary plugin tools, and it performs no external
  effects.
- The provisional response and the oracle transcript are excluded from
  model-visible chat history.
- The harness records a tool-view-change persistence event, changes the view, and
  repeats the original model request from the same committed conversation state
  with the new structured tool definitions. The original system prompt is
  unchanged.
- The oracle transcript may be retained in a separate operational trace.
- Each model step has a bounded number of disclosure attempts.
- Invalid selections, unavailable members, and budget overflow are typed failures
  that leave the previous view intact unless an explicit fallback is configured.
- Repeated equivalent requests may reuse a cached selection when the permission
  and catalog revisions still match.

The proposal states that tool schemas are supplied as structured request data,
not by rewriting the system prompt or inserting schemas as system prose. A
provider adapter may use the provider's own deferred-tool search only when it
preserves the same semantics.

The proposal states that workflow script steps keep exact declared dependencies
and do not use the disclosure oracle or unrestricted dynamic lookup.

## 5.3 Ownership stated by the proposal

| Component | Proposed responsibility |
| --- | --- |
| Agent runtime | Provider-neutral tool-view and disclosure request and decision types; provisional responses; ephemeral branch; commit and rollback; idle boundaries; attempt limits; lifecycle events; exact reconstruction; consumer-supplied catalog, starter-selection, oracle-model, and persistence interfaces. |
| Daemon | Build a catalog from registered plugin capabilities and configured toolsets; filter it through agent permissions before any search; starter retrieval; the concrete oracle; ranking indexes; schema budgets; profile disclosure policy; provider adaptation; persistence of disclosure events; diagnosis of missing catalog revisions. |
| Plugin authoring SDK | Declarations for discovery metadata and reusable toolsets; build or package-time validation that every toolset member names a declared capability; generation of the agreed bundle or manifest representation; documentation of the hand-authored equivalent. |
| Plugins | Declare discovery metadata and toolsets. Membership grants no permission. |

The proposed persistence event carries at least the disclosure source (starter,
oracle, or explicit consumer policy), the `replace` or `add` mode, the selected
toolset and capability identifiers, the resulting ordered tool view, the
permission and catalog revisions, and stable schema versions or digests. The
proposal expects reload to reproduce the same ordered view. It expects a missing
or changed schema revision to fail loudly or follow an explicitly designed
compatibility path, never silently bind an old identifier to a changed schema.

## 5.4 Proposed acceptance checks

The proposal lists checks that a future implementation would need: starter
selection exposes only permitted members; the oracle combines toolsets and
individual tools in both modes; the provisional response and transcript stay out
of model-visible history; ordinary tool calls and user-visible text do not share
a disclosure request; dispatch rejects a capability removed from the permission
boundary after selection; save, drop, reload, and continue reproduce the exact
ordered view; a missing or changed schema revision follows the declared policy;
script steps still receive only their declared permitted tools; provider-specific
and generic adapters behave equivalently; and attempt and schema budgets
prevent loops and unbounded context growth.

## 5.5 Open decisions in the proposal

These are recorded as open questions `OQ-041` to `OQ-052` in
`15-open-questions.md`. They are not requirements.
