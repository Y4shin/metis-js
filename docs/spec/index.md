# Metis Agent: application specification

Status: complete.

This directory specifies the intended finished application, Metis Agent. It is a
self-contained product specification. It describes observable behaviour, domain
rules, external contracts, and open questions. It does not select a programming
language, framework, persistence product, internal module layout, or build
system, except where the evidence establishes an external compatibility contract.

## Normative language

This specification applies BCP 14: [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174). The uppercase keywords
**MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** carry their BCP 14
meanings only when they appear in uppercase. Lowercase uses of these words, and
uses inside background, rationale, examples, and quotations, are not normative.

The keywords are used as follows:

- **MUST**: an absolute requirement of the finished application.
- **MUST NOT**: an absolute prohibition.
- **SHOULD**: a recommendation that can be disregarded only for a valid,
  carefully considered reason.
- **MAY**: explicitly permitted optional behavior.

This specification does not use **SHALL**, **REQUIRED**, **RECOMMENDED**, or
similar synonyms as additional requirement levels.

A requirement that names a compatibility axis (`CMP-001` to `CMP-006`) states
the rule the axis follows, not the concrete version numbers or spellings that
realize it. Those are decisions of an implementation, opened as questions in
`15-open-questions.md`.

## Document conventions

### Requirement identifiers

Each substantive normative statement has a stable identifier of the form
`<GROUP>-<three digits>`, for example `CNV-004`. Identifiers do not change when
prose is edited. A retired identifier is marked retired rather than reused.

### Status markers

Each requirement carries one status marker:

| Marker | Meaning |
| --- | --- |
| `[current]` | The behavior is part of the application and implementation evidence confirms some or all of it. |
| `[current, compile-verified only]` | The behavior is required, but the only implementation evidence is that the platform-specific code compiles for its target. It has not executed. |
| `[planned]` | The behavior is part of the intended finished application, but it has not been implemented yet. |
| `[planned, form open]` | The capability is intended, but its concrete form is undecided. The requirement states only what is fixed. |
| `[proposed]` | A proposal, not an accepted requirement. Proposed statements are not normative and carry no uppercase BCP 14 keyword. |
| `[open]` | The evidence does not settle the matter. An open statement is not a requirement and carries no uppercase BCP 14 keyword. |

The status marker does not weaken a **MUST** on a `[current]` or `[planned]`
requirement. A `[proposed]` or `[open]` statement is outside the normative set
altogether. A `[planned]` requirement is still required of the finished
application.

### Decisions not invented by this specification

An implementation is not expected to invent an answer to an open question and
present it as required. An implementation can choose an answer for its own build
and record that choice as a decision.

## Reading order

| Document | Contents |
| --- | --- |
| `01-purpose-scope-and-conventions.md` | Purpose, goals, non-goals, actors, scope, and product constraints. |
| `02-domain-model.md` | Domain concepts, relationships, lifecycle states, and glossary. |
| `03-agent-runtime.md` | Agent contexts, conversations, agent tasks, session state, and persistence. |
| `04-workflows.md` | Workflow definitions, step types, routing, joins, tool dependencies, and hosting. |
| `05-tool-disclosure.md` | Toolset, tool view, and disclosure oracle vocabulary, and the proposal's status. |
| `06-daemon-and-api.md` | Daemon responsibilities, the API contract, client, transports, configuration, and secrets. |
| `07-identity-authentication-and-pairing.md` | Surface identity, local access control, tokens, pairing, and credential storage. |
| `08-realtime-events-and-cancellation.md` | Event streams, snapshots, cursors, ordering, slow consumers, and cancellation. |
| `09-plugins.md` | Plugin contract, bundles and signing, manifests, host lifecycle, start-up policy, SDK, and CLI. |
| `10-surfaces-and-terminal.md` | Desktop and Android shells, and the terminal CLI and TUI. |
| `11-business-rules-and-invariants.md` | Cross-cutting invariants and state transitions. |
| `12-external-interfaces.md` | Externally observable configuration, transports, and compatibility contracts. |
| `13-nonfunctional-requirements.md` | Security, privacy, reliability, performance, compatibility, and legal constraints. |
| `14-acceptance-criteria.md` | Acceptance criteria and relevant edge cases. |
| `15-open-questions.md` | Unresolved questions and deferred decisions. |

## Provenance and implementation independence

This specification is self-contained. It was reviewed through four independent
rubric passes: coverage and traceability, fidelity and normative language,
implementation independence and scope, and clarity, consistency, and
verifiability. No external repository is an input to an implementation.

The specification names no programming language, framework, library, wire
protocol, archive or container format, signature algorithm, or file-name
spelling. Where a behavior needs a concrete realization, the requirement states
the observable rule and `15-open-questions.md` asks the question the
realization must answer. An implementation chooses an answer and records it as
a decision; a different implementation can choose differently provided it
preserves the stated behavior.

## Requirement index

| Group | Domain | Document |
| --- | --- | --- |
| `PRD` | Product scope, goals, non-goals, and constraints | `01-purpose-scope-and-conventions.md` |
| `AGT` | Agent contexts | `03-agent-runtime.md` |
| `CNV` | Agent loop and conversations | `03-agent-runtime.md` |
| `TSK` | Agent tasks | `03-agent-runtime.md` |
| `SES` | Session state and persistence | `03-agent-runtime.md` |
| `WFL` | Workflows | `04-workflows.md` |
| `TDS` | Tool disclosure (proposal; no normative requirements) | `05-tool-disclosure.md` |
| `DAE` | Daemon responsibilities | `06-daemon-and-api.md` |
| `API` | API contract and client | `06-daemon-and-api.md` |
| `CFG` | Configuration and secrets | `06-daemon-and-api.md` |
| `NET` | Transports | `06-daemon-and-api.md` |
| `IDN` | Surface identity and protocol version | `07-identity-authentication-and-pairing.md` |
| `AUT` | Access control, tokens, and pairing | `07-identity-authentication-and-pairing.md` |
| `RTS` | Realtime event streams | `08-realtime-events-and-cancellation.md` |
| `CAN` | Cancellation | `08-realtime-events-and-cancellation.md` |
| `PLG` | Plugin contract | `09-plugins.md` |
| `BND` | Bundles and signing | `09-plugins.md` |
| `MAN` | Manifest | `09-plugins.md` |
| `HST` | Plugin host lifecycle | `09-plugins.md` |
| `SDK` | Plugin authoring SDK and CLI | `09-plugins.md` |
| `SUR` | Desktop and Android surfaces | `10-surfaces-and-terminal.md` |
| `TRM` | Terminal CLI and TUI | `10-surfaces-and-terminal.md` |
| `INV` | Cross-cutting invariants | `11-business-rules-and-invariants.md` |
| `SEC` | Security | `13-nonfunctional-requirements.md` |
| `PRV` | Privacy | `13-nonfunctional-requirements.md` |
| `REL` | Reliability | `13-nonfunctional-requirements.md` |
| `PERF` | Performance and resource use | `13-nonfunctional-requirements.md` |
| `CMP` | Compatibility | `13-nonfunctional-requirements.md` |
| `NFR` | Dependency and footprint constraints | `13-nonfunctional-requirements.md` |
| `LEG` | Legal | `13-nonfunctional-requirements.md` |
| `AC` | Acceptance criteria | `14-acceptance-criteria.md` |
| `OQ` | Open questions | `15-open-questions.md` |
