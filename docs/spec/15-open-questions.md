# 15. Open questions and unresolved contradictions

Each open question records a decision this specification does not settle. No open
question is a requirement. An implementation can choose an answer and record it
as a decision; see `index.md` "Decisions not invented by this specification".

## 15.1 Product scope and vocabulary

- OQ-001: **Are conversations and agent tasks first-class persisted concepts, and
  in what form?** Product statements call the form open, while the planned
  runtime requires a reconstructable session runtime and a daemon that hosts
  conversations and tasks. The requirements in `03-agent-runtime.md` are therefore
  planned, and the product-status statements remain unresolved.
- OQ-002: **What is the final name for the declared execution boundary?** "Agent"
  is a working term. No source settles it.
- OQ-003: **What is the agent declaration format, and where is it written?**
- OQ-004: **Which permission categories exist beyond tool access?**
- OQ-005: **What is the concrete form of the chat-session harness, and does a chat
  session differ from a conversation?**
- OQ-006: **What execution limits exist, and what are their defaults?**
- OQ-007: **What lifecycle events does the agent runtime emit, and with what
  payloads?**
- OQ-008: **What is the representation of an agent task's output contract, and
  how is validation reported on failure?**

## 15.2 Agent runtime interfaces

- OQ-009: **What are the exact consumer-facing interfaces for the loop, model
  provider, tool contract, and extension hooks?**
- OQ-010: **What is the session persistence event set, and what granularity does
  a store acknowledge?**
- OQ-011: **What does "context transformation" permit, and how is it recorded so
  that session history is provably not discarded?**
- OQ-012: **Which storage backend implements session persistence, and where does
  that implementation live?**
- OQ-013: **How are persistence failures retried or escalated?**

## 15.3 Daemon, scheduling, and configuration

- OQ-014: **What are the concrete model providers, how are models selected per
  execution, and how are provider credentials stored?**
- OQ-015: **What are configuration precedence rules across files, environment
  variables, command-line flags, and platform credential stores?**
- OQ-016: **How do secrets reach plugins other than through the plugin's own
  configuration at initialisation?**
- OQ-017: **What are session and task orchestration rules: lifecycle, scheduling,
  concurrency, cancellation propagation, and what a disconnect does to running
  work?**
- OQ-018: **What is the authentication boundary between surfaces and the daemon
  beyond the per-surface token, including per-surface scopes and what local IPC
  trust means?**
- OQ-019: **What is the trust store's configuration representation, its rotation
  story, and the exact form of a revocation entry?**
- OQ-020: **What is the plugin directory's exact path, and how is it configured?**
- OQ-021: **What is the error taxonomy for plugin and daemon failures?** Three
  error types exist ahead of the decision; replacing them needs the operator's
  agreement.
- OQ-022: **What protocol and encoding does the daemon API use, and how are
  calls, streaming, and refusals expressed in it?** The requirements in
  `06-daemon-and-api.md` fix the observable behavior; this question fixes the
  wire form that realizes it.

## 15.4 Workflows

- OQ-023: **What is the workflow definition format, and how are step and port keys
  represented?**
- OQ-024: **What are mandatory-port connectivity rules, dead-path detection, and
  the effect of failed or cancelled branches on join readiness, as distinct from
  deliberate abandonment?**
- OQ-025: **What is the execution concurrency model for workflow runs?**
- OQ-026: **What are durable checkpoint and resume semantics, and what is the
  failure and retry policy, including idempotency or reconciliation for external
  writes?**
- OQ-027: **What is the script runtime and its sandbox, and what is the exact
  tool-injection interface for script steps?**
- OQ-028: **What are workflow triggers, persisted run state, progress and result
  delivery to clients, and failed or cancelled run handling?**
- OQ-029: **How does the workflow runtime interact with sessions, given that
  workflow progress and outputs are distinct from a task's session history?**

## 15.5 Workflow and task detail

- OQ-030: **How is a workflow step's output validated against its declared
  output schema, and what happens when validation fails?**
- OQ-031: **What happens when a routing function raises an error or returns a
  value that does not match the connector's declared destinations?**
- OQ-032: **Is there a bound on fan-out width, join input count, or graph size?**
- OQ-033: **Do script steps and joins declare output schemas, and how are their
  outputs typed for downstream steps?**
- OQ-034: **How does cancellation propagate through a running workflow, and what
  does a cancelled run report to its caller?**
- OQ-035: **What identifies a workflow run, and who may start, observe, or cancel
  one?**
- OQ-036: **What API operations expose workflow definitions, runs, progress, and
  results?**
- OQ-037: **How long are workflow definitions, run records, and outputs retained?**
- OQ-038: **How does an agent task started inside a workflow relate to the
  daemon's client-facing cancellation request identifier?**
- OQ-039: **What schema dialect represents workflow, task, and capability
  payloads?** The specification fixes that a schema exists and is validated, not
  the language it is written in (see also `OQ-110`).
- OQ-040: **Which extension hooks does the agent runtime expose, and at which
  points in the loop?**

## 15.6 Tool disclosure

The tool disclosure proposal lists these unresolved decisions. They remain open
until the proposal is accepted.

- OQ-041: What are the exact provider-neutral tool-view and disclosure types and
  events?
- OQ-042: What is the first serialized toolset representation, and what are its
  compatibility rules?
- OQ-043: Do local toolset overlays replace or extend plugin declarations?
- OQ-044: What are the default starter-selection confidence, schema budget, and
  oracle-attempt limits?
- OQ-045: Which ranking and reranking algorithms are used, and how are their
  indexes refreshed?
- OQ-046: How long does an active tool view remain sticky across topic changes?
- OQ-047: Which provider-specific equivalence tests must pass before a provider's
  own tool search can replace the generic oracle path?
- OQ-048: Do surfaces expose the active tool view or the operational oracle trace?
- OQ-049: How are toolset members resolved when a member is unavailable, and what
  is the chosen failure policy?
- OQ-050: How does a consumer define a local toolset, and is the definition part
  of daemon configuration?
- OQ-051: What is the cache key and invalidation rule for a reused disclosure
  selection?
- OQ-052: Is progressive disclosure required for the finished application at all?

## 15.7 Plugins and SDK

- OQ-053: **Which core plugins ship first, and what do they demonstrate?**
- OQ-054: **What are the CLI responsibilities: scaffold, build, package, install,
  publish, and how much cross-compilation does the build cover?**
- OQ-055: **What are the SDK's serialisation helpers, and what is their dependency
  footprint?**
- OQ-056: **How are the SDK and CLI tested against the runtime without a full
  daemon?**
- OQ-057: **What is the SDK's authoring language, how is it distributed, and does
  it live in this repository or separately?** `SDK-001` requires an SDK for the
  language the core plugins are written in; it does not fix which language.
- OQ-058: **What are the plugin version, boundary contract version, and bundle
  version policies as authors see them?**
- OQ-059: **Which parts of the revised authoring surface does a dependent effort
  re-export, and which does it hide?**
- OQ-060: **Should the manifest's inline capability schemas remain inline, or be
  replaced by an identifier fetch?** The choice is provisional and cheap to
  change because it does not move the boundary contract.
- OQ-061: **Should a bundle support assets or several files per platform, and
  what mechanism would carry them?**
- OQ-062: **What are the exact cancel-race semantics and acknowledgement
  requirements for a plugin that misses its cancellation deadline?** The host
  records a protocol violation and drops; the plugin side is documented as an
  expectation, not a hard guarantee.
- OQ-063: **Should the host add its own reclaim pass for abandoned plugin result
  queries?** The reclaim is caller-driven today.
- OQ-064: **What is the plugin host's not-found rule for an identifier held by
  another instance, and where is it recorded in the contract text?** The
  implementation records it; the contract text owes a sentence.
- OQ-065: **What representation does the trust store use, and how should a
  malformed trust configuration behave?** The daemon supplies trusted signing
  identities and their revocations; the form is open (see also `OQ-019`,
  `OQ-108`).

## 15.8 Realtime and access control

- OQ-066: **Should the event ring capacity, heartbeat interval, idle timeout, and
  client backoff be configurable, and what are their required values?** The
  current values are fixed in code and not configurable.
- OQ-067: **Are cursors per channel or daemon-wide?** They are daemon-wide today,
  so traffic on one channel can expire another channel's cursor.
- OQ-068: **Do plugin-produced events get their own channels?**
- OQ-069: **What are the snapshot contents per channel?** A channel per target
  area is required (`RTS-005`); the names and the snapshot payloads are not.
- OQ-070: **What should a caller do about a transport failure after a stream
  ends?** Deliberate drops and cancellations are typed; transport failure
  handling is not specified.
- OQ-071: **Should pairing have a rate limit or lockout?** None exists, and a
  network-reachable listener makes repeated guesses reachable.
- OQ-072: **What, if anything, should a client verify about a proxied remote
  endpoint beyond the checks the proxy itself performs?** The decision that the
  daemon terminates no encryption defers this.
- OQ-073: **Should tokens expire, and what refresh mechanism would replace the
  first version's non-expiring tokens?**
- OQ-074: **Should the daemon gain a Windows client credential path, and what
  would it use?**

## 15.9 Surfaces and terminal

- OQ-075: **Which Android activation path is sufficient first, and when is the
  voice-interaction service required?**
- OQ-076: **What is the Android service-to-client protocol, and how are cold
  start, permission requests, and background behaviour handled?**
- OQ-077: **How does the shared client reconnect after Android background and
  resume?**
- OQ-078: **What are the release Android signing, packaging, and continuous
  integration arrangements?**
- OQ-079: **How are credential storage, notifications, and file selection exposed
  on each platform, and what are the keyboard, scrolling, window, and tray
  behaviours?**
- OQ-080: **Which screens exist first, and how is daemon state projected into
  them?**
- OQ-081: **What terminal user-interface library, command surface, and first
  vertical slice should be used?**
- OQ-082: **Is a standalone browser client ever needed, and would it justify a
  network endpoint or gateway?**

## 15.10 Evidence gaps and recorded contradictions

- OQ-083: **Windows evidence is incomplete.** The Windows local transport has
  never executed, its access rule has never been read back, a remote or
  low-integrity client has never been observed, and the Windows client
  constructor does not exist. The checks that would close these are listed in
  `14-acceptance-criteria.md`.
- OQ-084: **The Windows continuous integration job's daemon half had never run on
  a runner carrying the toolchain the daemon needs when this specification was
  written.** A first successful run is the only proof.
- OQ-085: **The client's reconnection policy is contradicted across earlier
  sources.** An early design decision says the client reconnects with
  jittered backoff; the later realtime decisions say the client performs no
  automatic retry or resync. This specification follows the later, tested
  behavior. The question is whether any transport-level reconnect policy is
  wanted at all, and if so where it lives.
- OQ-086: **"Revocation requires a restart" and "deleting the credential revokes"
  are both stated.** This specification requires revocation to take effect no
  later than the next start after revocation, which satisfies both without
  claiming immediate effect.
- OQ-087: **An earlier secret-reference syntax and the configuration syntax
  differ.** This specification follows the operator-facing configuration forms.
- OQ-088: **The ACL wording "configured owner and group" does not match the
  implemented rule.** The rule fixes the owner as the daemon's user and makes only
  the group configurable. This specification follows that rule.
- OQ-089: **"Unpacking refuses entry permissions" is broader than the refined
  rule**, which accepts an entry that declares no mode or an owner-only mode and
  refuses anything else. This specification follows the refined rule.
- OQ-090: Retired. The removed realization named fixed contract version numbers;
  the specification now leaves them open (`OQ-111`) and requires only that the
  host refuse a differing major and an older minor (`PLG-014`).
- OQ-091: Retired. The dependency-count measurements and identifier spellings it
  referred to belonged to the removed realization and are no longer part of the
  contract text.
- OQ-092: Retired. The staging-directory name and the host-side not-found sentence
  belonged to the removed realization. The not-found rule is now stated at
  `HST-023`.
- OQ-093: **The exact criteria for declaring the Windows transport supported,
  and who owns the first Windows runner, are undecided.**

## 15.11 Additional open decisions

- OQ-094: **How is a surface credential revoked?** An operator must be able to
  revoke one surface, but no operator-facing command or API is documented.
- OQ-095: **What is the plugin directory's exact path, and what happens when it
  does not exist?** Start-up scans a directory; the path and the absent-directory
  behavior are not fixed.
- OQ-096: Retired. `PLG-012` now requires both directions of the contract to
  declare their extent, so a wrong-shape contract is refused at load rather than
  accepted.
- OQ-097: **What is the pinning and upgrade policy for third-party dependencies
  whose major versions move together?** Exact pinning and the regeneration cost of
  a bump are an open maintenance decision.
- OQ-098: **Is the daemon required to ship as one self-contained artifact with no
  dependency on a separately installed runtime?** The preference is recorded; the
  distribution posture is unverified.
- OQ-099: **Should the daemon reject a surface name outside the shipped classes?**
  It records the name as descriptive data today.
- OQ-100: **What privacy and telemetry policy applies to conversation and task
  data?** No policy is documented.
- OQ-101: **What error taxonomy should the daemon expose for plugin and routing
  failures?** Three error types exist ahead of the decision.
- OQ-102: **What confirmation step, if any, should a surface show when pairing?**
  Fingerprint confirmation belonged to the superseded client-pinning design.

## 15.12 Choices opened by implementation independence

The specification states behavior without naming a programming language, a
library, a wire protocol, an archive or container format, a signature algorithm,
or a file-name spelling. Each question below is a choice an implementation makes
and records as a decision (see `index.md`, "Decisions not invented by this
specification"). None of them weakens a requirement that references it.

- OQ-103: **What form does a plugin's code take, and how does the daemon run it
  inside its own process?** Candidates include code loaded from a shared library,
  a portable module instantiated by the daemon's runtime, or code written in the
  daemon's own language. Each differs in isolation, performance, and the
  toolchain a plugin author needs. See `PLG-001`.
- OQ-104: **What is the structural shape of the plugin boundary contract?** The
  entry point, the operation set, how each side declares its extent, the handshake
  by which a call starts and completes, and the ownership rule for each payload.
  See `PLG-010` to `PLG-019`, `PLG-046` to `PLG-052`.
- OQ-105: **How does a plugin report pending work, receive cancellation, and ask
  the host for a facility?** See `PLG-005`, `PLG-022` to `PLG-026`, `PLG-034`,
  `PLG-036` to `PLG-044`.
- OQ-106: **What encoding carries plugin payloads, and how is a payload's format
  version declared and negotiated?** See `PLG-053` to `PLG-055`.
- OQ-107: **What container and archive format carries an installed plugin?** The
  file layout, how a platform target is spelled, where the manifest and the code
  for a platform are found, and which compression methods are accepted. See
  `BND-001` to `BND-034`.
- OQ-108: **What signature scheme and trust mechanism decide that a bundle is
  trusted?** The signature algorithm, the form of an authorisation chain, how a
  key identifier is derived, and the trust store's representation. See `BND-005`
  to `BND-016`, and `OQ-019`, `OQ-065`.
- OQ-109: **What format does the plugin manifest take, and what are its field
  names?** See `MAN-001` to `MAN-014`.
- OQ-110: **What schema dialect describes capability request and response
  schemas, configuration schemas, and workflow and task payload schemas?** See
  `PLG-061`, `PLG-062`, `PLG-075`, `MAN-005`, and `OQ-039`.
- OQ-111: **What version numbers does the plugin boundary contract carry, and how
  do the contract version, the container version, and a plugin's own version
  relate?** See `PLG-011` to `PLG-015`, `OQ-058`.
- OQ-112: **Which correctness properties are worth proving at build time, and with
  what tooling?** The boundary hand-offs are the first candidate, but the question
  covers any property a compiler or a proof tool could establish instead of a
  runtime check. See `HST-043`.
- OQ-113: Retired. It duplicated `OQ-022`; one decision is opened once.
- OQ-114: **What local transport reaches the daemon on each supported platform,
  and what access rule protects it?** See `NET-001`, `NET-002`, `NET-004` to
  `NET-010`, and `CFG-001` to `CFG-004`.
- OQ-115: **What remote listener exposes the daemon, and how does forwarded
  client information arrive at it?** See `NET-013` to `NET-025`.
- OQ-116: **What names and default locations do the operator-facing settings, the
  local endpoint, and the state store use?** See `CFG-001` to `CFG-015` and
  section 12.1.
- OQ-117: **What syntax expresses a secret reference from a literal, an
  environment variable, a platform credential store, or a file?** See `CFG-006` to
  `CFG-010`.
- OQ-118: **What carrier names and value encodings carry surface identity,
  credentials, request identifiers, and refusal causes?** See `IDN-001` to
  `IDN-008`, `API-007`, and `CAN-002` to `CAN-006`.
- OQ-119: **What names do event channels use, and what encoding carries an event
  payload?** See `RTS-005`, `RTS-008`, `RTS-014`.
- OQ-120: **What form does a minted credential take, and which irreversible
  function stores it?** See `AUT-013`, `AUT-014`, `AUT-032`.
- OQ-121: **Which programming language, runtime, and third-party dependency set
  does the implementation use, and what footprint must they respect?** See
  `NFR-001` to `NFR-003`, `CMP-007`, `OQ-098`.
- OQ-122: **What product stores the daemon's persistent state and the agent
  runtime's session state?** See `CFG-011`, `CFG-012`, `SES-002`, `OQ-012`.
- OQ-123: **What form does a client request identifier take, within the length
  and character bound `CAN-002` fixes?** See `CAN-002`, `CAN-004`, `CAN-005`.
