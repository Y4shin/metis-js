# 13. Nonfunctional requirements

## 13.1 Security

- SEC-001: **A caller's access MUST be enforced at the transport or admission
  layer before a handler runs: local access control on the local transport and a
  per-surface credential on the remote transport.** `[current]`
- SEC-002: **The daemon MUST refuse a bundle whose signature chain it does not
  trust, and MUST treat the refusal as a start-up failure rather than a
  workaround.** `[current]`
- SEC-003: **The daemon MUST validate a bundle's contents before making any of
  them available, under the rules in `BND-024` to `BND-033`.** `[current]`
- SEC-004: **Untrusted bundle bytes MUST be parsed with strict, canonical-input
  decoders, so a second encoding of the same value cannot be used to make one
  signature verify under two keys.** `[current]`
- SEC-005: **The daemon MUST recompute every identifier it compares and MUST NOT
  trust a carried one.** `[current]`
- SEC-006: **The compiled-in trust store MUST be empty; no shipped build may
  trust a test key or an arbitrary third-party signer.** `[current]`
- SEC-007: **A plugin MUST be treated as code with the daemon's privileges; the
  application MUST NOT claim that the boundary isolates or sandboxes it.**
  `[current]`
- SEC-008: **The daemon MUST NOT write a token, a preshared key, or any other
  plaintext secret to persistent state, logs, or a call's address (see `AUT-014`).**
  `[current]`
- SEC-009: **A secret file MUST be refused unless only its owner can read it.**
  `[current]`
- SEC-010: **An available credential store that refuses MUST NOT cause a silent fallback
  to a less protected storage backend.** `[current]`
- SEC-011: **The host context delivered to a plugin MUST NOT contain secrets.**
  `[current]`
- SEC-012: **Information a proxy forwards MUST NOT be used for authorization.**
  `[current]`
- SEC-013: **The daemon MUST warn when it exposes a plaintext remote listener
  outside loopback without a named trusted proxy, and when a secret-bearing
  pairing call travels over a plaintext client leg.** `[current]`
- SEC-014: **The application MUST NOT claim encryption, client authentication, or
  remote attestation that the daemon does not perform.** `[current]`

## 13.2 Privacy

- PRV-001: **Conversation and task data MUST be stored under the operator's
  control, in the daemon's state store.** `[planned, form open]`
- PRV-002: Retired. This identifier is not reused.
- PRV-003: **Under the tool disclosure proposal, a model provider's own transport
  history would not become the canonical conversation history.** This is proposal
  material and not an accepted requirement. `[proposed]`
- PRV-004: **Under the tool disclosure proposal, the disclosure oracle's selection
  transcript would be excluded from model-visible chat history and could be
  retained only in a separate operational trace.** This is proposal material and
  not an accepted requirement. `[proposed]`
- Note: This specification states no privacy or telemetry policy. Whether the
  daemon may collect telemetry, and what may leave the machine beyond a configured
  model provider, is open (`OQ-100`).

## 13.3 Reliability

- REL-001: **A failure to persist session state MUST be surfaced and MUST NOT be
  reported as saved.** `[planned]`
- REL-002: **A disable-class plugin failure MUST leave the daemon running, and a
  trust failure MUST stop start-up rather than allow untrusted code to run.**
  `[current]`
- REL-003: **Plugin code MUST NOT be released while it, a pending result, a
  completion signal, or a callback can still reference it.** `[current]`
- REL-004: **Shutdown MUST be bounded by a deadline, MUST report instances it
  could not release, and MUST leave that plugin code resident rather than force
  an unsafe release.** `[current]`
- REL-005: **A consumer that falls behind the event window MUST receive a
  terminal refusal rather than a silent gap.** `[current]`
- REL-006: **A daemon restart MUST expire cursors and in-flight cancellation
  state rather than pretend they survived.** `[current]`
- REL-007: **A transport failure and a deliberate stream termination MUST be
  distinguishable.** `[current]`
- REL-008: **The daemon MUST continue executing work when every surface
  disconnects (see `PRD-C1`).** `[current]`
- REL-009: **The application MUST NOT assume exactly-once delivery of external
  writes; retries require idempotency or reconciliation.** `[planned]`

## 13.4 Performance and resource use

- PERF-001: **The agent context system prompt and the tool schemas sent with a
  model request SHOULD be kept lean, because the target models have small context
  windows.** Disregarding this recommendation reduces the work a small local
  model can complete and increases cost on every request. `[planned]`
- PERF-002: **The daemon MUST bound the set of events from which a cursor can
  resume, so memory use does not grow without bound.** `[current]`
- PERF-003: **High-frequency streamed updates to a surface MUST be batched across
  the application bridge rather than delivered once per token.** `[planned]`
- PERF-004: **Under the tool disclosure proposal, a model request's visible tool
  set would be bounded by a configured schema budget.** This is proposal material
  and not an accepted requirement. `[proposed]`
- PERF-005: **The documented performance thresholds for request latency, startup
  time, memory use, and model token budgets are not established.** A numeric
  target is a decision outside this specification. `[open]`

## 13.5 Compatibility

- CMP-001: **The plugin contract MUST stay compatible within a major version by
  adding operations only, and an older minor MUST be refused while a newer minor
  is tolerated.** `[current]`
- CMP-002: **The bundle container version MUST match the host's exactly; an
  unknown version MUST be refused and degrade that bundle.** `[current]`
- CMP-003: **The plugin's own version MUST NOT gate loading (see `PLG-017`).**
  `[current]`
- CMP-004: **An unintentional breaking contract change MUST be detectable and
  treated as a defect; a deliberate breaking change is introduced as a new version
  or a documented migration.** `[current]`
- CMP-005: **The externally documented settings, default locations, contract
  version, bundle and manifest declarations, and network protocol version MUST be
  preserved or migrated deliberately.** `[current]`
- CMP-006: **Existing paired surfaces MUST keep working across an upgrade, or
  their stored credentials MUST be migrated deliberately (see `AUT-040`).**
  `[current]`
- CMP-007: **The application MUST support Linux, macOS, and Windows desktop
  platforms and Android; it MUST NOT provide an iOS implementation.**
  `[current]`
- CMP-008: **The Windows local transport MUST be treated as compile-verified and
  design-verified only until it has executed on a Windows host; an implementation
  MUST NOT claim measured Windows behavior beyond that.** `[current]`

## 13.6 Legal

- LEG-001: **Metis Agent MUST be distributed under the GNU Affero General Public
  License version 3.0, and a modified version run as a network service MUST make
  its corresponding source available to that service's users as the license
  requires.** `[current]`

## 13.7 Dependency and footprint constraints

- NFR-001: **The plugin runtime SHOULD keep its dependency footprint small,
  because it needs only the contract plus the readers for the bundle, the
  manifest, and the signature.** Disregarding this recommendation increases
  attack surface and build cost in the process that parses untrusted bytes. The
  dependency set is open (`OQ-121`). `[current]`
- NFR-002: **The code path that reads untrusted bundle bytes SHOULD avoid adding
  a dependency that must be compiled for the target machine, so reading untrusted
  input needs no build step on that machine.** Disregarding this recommendation
  adds a build requirement to the process that parses untrusted bytes.
  `[current]`
- NFR-003: **The SDK's serialisation, scheduling, and export helpers MUST stay out
  of the runtime's dependency footprint (see `SDK-002`).** `[planned]`
