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
  variables, command-line flags, and keychain or native stores?**
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
- OQ-022: **Should the API's JSON codec be enabled for outside clients, and what
  does that mean for the authentication and error model?** This follows the
  decision to use a binary-friendly protocol contract.

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
- OQ-039: **What output schema representations are supported: a language-neutral
  subset of JSON Schema, or a project-specific schema language?**
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
- OQ-047: Which provider-specific equivalence tests must pass before native tool
  search can replace the generic oracle path?
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
- OQ-057: **Is the SDK a crate inside this repository, a separately published
  crate, or both?**
- OQ-058: **What are the plugin version, ABI version, and bundle version policies
  as authors see them?**
- OQ-059: **Which parts of the revised authoring surface does a dependent effort
  re-export, and which does it hide?**
- OQ-060: **Should the manifest's inline capability schemas remain inline, or be
  replaced by an identifier fetch?** The choice is provisional and cheap to
  change because it does not move the ABI.
- OQ-061: **Should a v1 bundle support assets or several files per target, and
  what format extension would carry them?**
- OQ-062: **What are the exact cancel-race semantics and acknowledgement
  requirements for a plugin that misses its cancellation deadline?** The host
  records a protocol violation and drops; the plugin side is documented as an
  expectation, not a hard guarantee.
- OQ-063: **Should the host add a daemon-side reaper thread for abandoned plugin
  polls?** The reclaim is caller-driven today.
- OQ-064: **What is the plugin host's `NotFound` rule for an identifier held by
  another instance, and where is it recorded in the contract text?** The
  implementation records it; the contract text owes a sentence.
- OQ-065: **Should `RootKey` and `TrustStore` carry more validation, and how should
  a malformed trust configuration behave?**

## 15.8 Realtime and access control

- OQ-066: **Should the event ring capacity, heartbeat interval, idle timeout, and
  client backoff be configurable, and what are their required values?** The
  current values are fixed in code and not configurable.
- OQ-067: **Are cursors per channel or daemon-wide?** They are daemon-wide today,
  so traffic on one channel can expire another channel's cursor.
- OQ-068: **Do plugin-produced events get their own channels?**
- OQ-069: **What are the snapshot contents per channel?** The channel names are
  fixed; their snapshot payloads are not.
- OQ-070: **What should a caller do about a transport failure after a stream
  ends?** Deliberate drops and cancellations are typed; transport failure
  handling is not specified.
- OQ-071: **Should pairing have a rate limit or lockout?** None exists, and a
  network-reachable listener makes repeated guesses reachable.
- OQ-072: **What, if anything, should a client verify about a proxied remote
  endpoint beyond the proxy's own certificate validation?** The no-native-TLS
  decision defers this.
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

- OQ-083: **Windows evidence is incomplete.** The pipe has never executed, the
  descriptor has never been read back, a remote or low-integrity client has never
  been observed, and the Windows client constructor does not exist. The checks
  that would close these are listed in `14-acceptance-criteria.md`.
- OQ-084: **The Windows continuous integration job's daemon half had never run on
  a runner with the native compiler when this specification was written.** A first
  successful run is the only proof.
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
- OQ-089: **"Extraction refuses entry permissions" is broader than the contract's
  refined rule**, which accepts an undeclared mode or `0600` and refuses anything
  else. This specification follows the contract.
- OQ-090: **The plugin ABI minor appears as both 1 and 2 in different sources.** This specification follows the approved contract and the
  implementation baseline: major 1, minor 2, with the initialisation entry at slot 0.
- OQ-091: **Whether the contract's dependency-count measurements and a few bearer
  identifier spellings should be corrected in the contract text** is an
  editorial question for the contract owner. It does not change any requirement
  in this specification.
- OQ-092: **Whether the contract text should state the host-side not-found rule
  for an identifier another instance holds** (same as OQ-064) and whether the
  staging-directory name should name all three components are contract-editorial
  items, not product decisions.
- OQ-093: **The exact criteria for declaring the Windows transport supported,
  and who owns the first Windows runner, are undecided.**

## 15.11 Additional open decisions

- OQ-094: **How is a surface credential revoked?** An operator must be able to
  revoke one surface, but no operator-facing command or API is documented.
- OQ-095: **What is the plugin directory's exact path, and what happens when it
  does not exist?** Start-up scans a directory; the path and the absent-directory
  behavior are not fixed.
- OQ-096: **Should the plugin ABI add a vtable size field, or is the residual
  accepted?** A wrong-shape vtable is not detectable at load today, because the
  vtable carries no size field.
- OQ-097: **What is the pinning and upgrade policy for the pre-1.0 protocol
  dependencies that move together?** Exact pinning and the regeneration cost of
  a bump are recorded as an open maintenance decision.
- OQ-098: **Is the daemon required to ship as a self-contained binary with no
  dynamically linked system dependency?** The source states this as a preference
  and records the static-linking posture as unverified.
- OQ-099: **Should the daemon reject a surface name outside the shipped classes?**
  It records the name as descriptive data today.
- OQ-100: **What privacy and telemetry policy applies to conversation and task
  data?** No policy is documented.
- OQ-101: **What error taxonomy should the daemon expose for plugin and routing
  failures?** Three error types exist ahead of the decision.
- OQ-102: **What confirmation step, if any, should a surface show when pairing?**
  Fingerprint confirmation belonged to the superseded client-pinning design.
