# 6. Daemon, API, client, transports, configuration, and secrets

## 6.1 Daemon responsibilities

- DAE-001: **The daemon MUST serve the client-facing API.** `[current]`
- DAE-002: **The daemon MUST host conversations and agent tasks on the agent
  runtime with its own session-state persistence implementation.** `[planned]`
- DAE-003: **The daemon MUST own the scheduling of conversations, agent tasks,
  and workflow runs.** The scheduling rules are open (`OQ-017`). `[planned]`
- DAE-004: **The daemon MUST resolve configuration and secrets.** `[current]`
- DAE-005: **The daemon MUST declare agent contexts and resolve their prompts and
  permissions for every execution context.** `[planned]`
- DAE-006: **The daemon MUST adapt plugin capabilities into the tools, model
  providers, and hooks the agent runtime consumes.** `[planned]`
- DAE-007: **The daemon MUST implement the model providers the runtime
  consumes.** `[planned, form open]`
- DAE-008: **The daemon MUST host workflow runs with permission-filtered tool
  dispatch.** `[planned]`
- DAE-009: **The daemon MUST keep running and keep executing work independently
  of any connected surface.** `[current]`
- DAE-010: **All exposed transports MUST serve the same API behavior and the
  same schema.** `[current]`
- DAE-011: Retired. This identifier is not reused.
- DAE-012: **The daemon MUST own plugin discovery, plugin configuration delivery,
  and the trust policy it hands to the plugin host.** `[current]`
- DAE-013: **The plugin host MUST NOT read the daemon's configuration files.**
  `[current]`
- DAE-014: **Plugin installation MUST NOT be a daemon-internal concern.** The
  operator installs a bundle by placing one file in the plugin directory.
  `[current]`
- DAE-015: **The daemon MUST read its configuration once at start-up, and a
  configuration change MUST require a restart.** `[current]`
- DAE-016: **The daemon MUST start with local access control as its default
  posture and MUST NOT open a remote listener unless the operator names a bind
  address.** `[current]`

## 6.2 The API contract

- API-001: **The daemon's API MUST be defined by a versioned, language-neutral,
  schema-first contract that is the single source of truth for both client and
  server.** `[current]`
- API-002: **The contract MUST support calls that return one answer and calls
  that keep returning answers until they are closed.** `[current]`
- API-003: **The contract MUST be free of concepts that only one implementation
  language can consume.** `[current]`
- API-004: **The contract MUST have a compatibility guarantee: an unintentional
  change that breaks an existing client MUST be detectable and treated as a
  defect, and a deliberate breaking change MUST be a versioned change.**
  `[current]`
- API-005: **The protocol major version MUST be part of the contract's versioned
  identity.** `[current]`
- API-006: **The API MUST expose pairing, cancellation, and the event stream as
  specified in this document.** `[current]`
- API-007: **The API MUST report a refusal in a form a client can map to a typed
  error without parsing human-readable message text.** `[current]`
- API-008: **A client MUST be able to distinguish, at minimum: a missing
  credential, an invalid credential, a revoked credential, pairing disabled, a
  wrong preshared key, an already-paired surface, an expired cursor, a
  slow-consumer drop, a cancelled stream, and an unsupported protocol version.**
  `[current]`
- API-009: **The contract MUST carry the realtime semantics of channel selection,
  cursor, ordering, cancellation, resume, expiry, and slow-consumer handling; a
  schema alone is not sufficient.** `[current]`

The protocol and its encoding are not fixed by this specification. The
requirements above state the observable behavior; the wire protocol, its schema
language, and its encoding are open (`OQ-022`).

## 6.3 The client

- API-010: **The application MUST provide one user-interface-independent client
  component that every surface uses to reach the daemon.** `[current]`
- API-011: **The client MUST expose one caller-facing API across all transports,
  so that a surface does not branch on the transport.** `[current]`
- API-012: **The client MUST carry the per-surface credential with every call,
  including the event stream.** `[current]`
- API-017: **The client MUST NOT place a credential in the call's address or in
  any part of it that a proxy or a log records.** `[current]`
- API-013: **The client MUST expose a stream from a cursor and MUST report an
  expired cursor as a distinct error so that the caller can resync.** `[current]`
- API-014: **The client MUST NOT hide a retry or a resync inside the wrapper; the
  caller decides whether to resync.** `[current]`
- API-015: **The client MUST NOT depend on a specific user-interface framework or
  terminal library.** `[current]`
- API-016: **The client MUST report transport failures distinctly from deliberate
  stream drops and cancellations.** `[current]`

## 6.4 Local transports

- NET-001: **On Linux and macOS the daemon MUST offer local access without
  opening a network port, and the client MUST reach it there by default.** The
  concrete local transport is open (`OQ-114`). `[current]`
- NET-002: **On Windows the daemon MUST offer local access that rejects remote
  callers, and the client MUST reach it there by default.** The concrete local
  transport is open (`OQ-114`). `[current, compile-verified only]`
- NET-003: **The local transport MUST carry the same API contract as any other
  transport, with no second message set or handler.** `[current]`
- NET-004: **The daemon MUST create the local endpoint and restrict it to the
  daemon's own user before it accepts a call, applying the configured group where
  the platform supports one.** `[current]`
- NET-005: **Where the local endpoint is a filesystem object and the platform
  expresses access as permissions, the endpoint and its directory MUST use
  owner-only permissions when no group is configured, and owner-and-group
  permissions when one is configured.** `[current]`
- NET-006: **Where the local endpoint is a filesystem object, the daemon MUST
  refuse a pre-existing path that is a symbolic link, is not a directory, is
  owned by another user, or is wider than the rule allows, and MUST NOT repair
  such a path.** `[current]`
- NET-007: **A configured group MUST name an existing group; an unknown group
  MUST be refused rather than replaced by the daemon's primary group.**
  `[current]`
- NET-008: **On Windows the local endpoint MUST grant access to the daemon's own
  account and to the system account only, and MUST carry no inherited access.**
  `[current, compile-verified only]`
- NET-009: **On Windows the local endpoint MUST reject remote clients, because the
  access rule alone does not prevent a remote caller that holds the daemon
  account's credentials.** `[current, compile-verified only]`
- NET-010: **On Windows the daemon MUST fail loudly rather than share its local
  endpoint with a process that already holds it.** `[current, compile-verified only]`
- NET-011: **The daemon MUST treat a local caller as authenticated by the
  operating system's access control and MUST NOT require a token for a local
  call.** `[current]`
- NET-012: **The daemon MUST NOT fall back from a failed local transport to the
  network transport.** `[current]`

## 6.5 Remote transport

- NET-013: **The daemon MUST expose a remote listener only when the operator
  supplies a bind address, and MUST NOT define a default bind address.** The
  listener's form is open (`OQ-115`). `[current]`
- NET-014: **The bind address setting MUST name an address literal and a port,
  and MUST refuse a hostname, an empty value acting as disabled, and a malformed
  value with a typed configuration error that names the setting and the value.**
  `[current]`
- NET-015: **The remote listener MUST be plaintext; the daemon MUST NOT terminate
  encryption or manage key material for it.** `[current]`
- NET-016: **The remote listener MUST require a per-surface credential on every
  call, including the event stream.** `[current]`
- NET-017: **The daemon MUST read forwarded information only from proxy addresses
  the operator names explicitly.** `[current]`
- NET-018: **The daemon MUST trust no proxy implicitly, including loopback, and an
  absent or empty trusted-proxy setting MUST name no proxy.** `[current]`
- NET-019: **When the peer is a trusted proxy, the daemon MUST read the left-most
  entry of the forwarded client address as the client address and MUST treat a
  forwarded indication of an encrypted client leg as encrypted; the daemon MUST
  NOT walk a multi-hop forwarded chain.** `[current]`
- NET-020: **Information a proxy forwards from an untrusted peer MUST be
  ignored.** `[current]`
- NET-021: **A forwarded client address that is not an IP literal MUST fall back
  to the peer address rather than be invented.** `[current]`
- NET-022: **A malformed trusted-proxy entry MUST be a typed configuration error
  and MUST refuse the listener rather than silently produce an empty trust set.**
  `[current]`
- NET-023: **A listener bound to an address that is not loopback with no trusted
  proxy named MUST print a plaintext warning before accepting connections.**
  `[current]`
- NET-024: **A pairing call whose client leg is plaintext MUST print a warning at
  most once per service instance rather than once per call.** `[current]`
- NET-025: **The daemon MUST NOT claim an encrypted client leg it cannot observe,
  and MUST NOT detect or prevent an operator who exposes the listener without a
  proxy.** `[current]`

## 6.6 Endpoint and configuration resolution

- CFG-001: **The local endpoint MUST resolve to a private, per-user runtime
  location when the platform provides one, and otherwise MUST fail with a typed
  "no private base" error rather than use a shared location.** `[current]`
- CFG-002: **An endpoint override setting MUST win over the platform default, and
  an absent or empty override MUST be treated as unset.** The setting names and
  default locations are open (`OQ-116`). `[current]`
- CFG-003: **An endpoint override MUST be validated against the platform's
  endpoint rules, and a value that names a non-local endpoint, or that the
  platform cannot represent as a local endpoint, MUST be refused.**
  `[current, compile-verified only]`
- CFG-004: **An invalid endpoint override MUST be a typed configuration error
  naming the setting and the value, and MUST NOT fall back to the default.**
  `[current, compile-verified only]`
- CFG-005: **Configuration MUST be read once at start-up, and changing it MUST
  require a restart (see `DAE-015`).** `[current]`
- CFG-006: **A secret configuration value MUST be resolvable from a literal, an
  environment variable reference, a platform credential-store reference, or a
  secret file reference.** The reference syntax is open (`OQ-117`). `[current]`
- CFG-007: **A secret file reference MUST be refused unless the file is readable
  by its owner and grants no access to group or others.** `[current]`
- CFG-008: **A credential-store reference MUST use the platform's credential store
  when one is available; a store that is present but refuses MUST be an error,
  not a silent fallback to another source.** `[current]`
- CFG-009: **A configured secret reference that cannot be resolved MUST fail the
  daemon start-up that needs it with a typed error, and MUST NOT be treated as an
  absent secret.** `[current]`
- CFG-010: **An empty secret MUST be treated as unresolved, not as a secret that
  an empty presentation can match.** `[current]`
- CFG-011: **The daemon's persistent state MUST live in one durable store whose
  state is restricted to the daemon's user, using owner-only access where the
  platform expresses access that way.** The store product is open (`OQ-122`).
  `[current]`
- CFG-012: **The state location MUST resolve from a dedicated setting when set
  and non-empty, otherwise from the platform's per-user data location, and
  otherwise MUST fail with a typed "no state directory" error.** `[current]`
- CFG-013: **A fresh installation MUST require no preparation step, and stored
  state from an earlier version MUST be upgraded automatically when the daemon
  opens it.** `[current]`
- CFG-014: **The state store MUST NOT contain a token, a preshared key, or any
  other plaintext secret (see `AUT-014`).** `[current]`
- CFG-015: **Configuration precedence across files, environment variables,
  command-line flags, and platform stores is unresolved.** An implementation's
  choice is a decision outside this specification. `[open]`
- CFG-016: **The daemon's trust store MUST contain the trusted signing identities
  and their revocations, and the compiled-in default set MUST be empty so that a
  fresh installation trusts no third-party signer.** `[current]`

The operator-facing setting names and default locations are an implementation's
choice. They are a compatibility surface once shipped: renaming one, or moving a
default location, is a breaking change for operators and needs a deliberate
migration. `12-external-interfaces.md` states the rule.
