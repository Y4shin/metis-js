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
- API-002: **The contract MUST support ordinary request-response calls and
  server-streaming calls.** `[current]`
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

The current realization uses Protobuf definitions under a versioned package and
ConnectRPC over HTTP semantics. That choice is one realization and is not a
requirement of this specification. A different realization preserves the
observable contract stated above.

## 6.3 The client

- API-010: **The application MUST provide one user-interface-independent client
  component that every surface uses to reach the daemon.** `[current]`
- API-011: **The client MUST expose one caller-facing API across all transports,
  so that a surface does not branch on the transport.** `[current]`
- API-012: **The client MUST carry the per-surface credential as request metadata
  on every call, including the event stream.** `[current]`
- API-017: **The client MUST NOT place a credential in a URL or a query string.**
  `[current]`
- API-013: **The client MUST expose a stream from a cursor and MUST report an
  expired cursor as a distinct error so that the caller can resync.** `[current]`
- API-014: **The client MUST NOT hide a retry or a resync inside the wrapper; the
  caller decides whether to resync.** `[current]`
- API-015: **The client MUST NOT depend on a specific user-interface framework or
  terminal library.** `[current]`
- API-016: **The client MUST report transport failures distinctly from deliberate
  stream drops and cancellations.** `[current]`

## 6.4 Local transports

- NET-001: **On Linux and macOS, the daemon MUST offer local access over a Unix
  domain socket, and the client MUST reach it there by default.** `[current]`
- NET-002: **On Windows, the daemon MUST offer local access over a duplex,
  byte-mode named pipe, and the client MUST reach it there by default.**
  `[current, compile-verified only]`
- NET-003: **The local transport MUST carry the same API contract as any other
  transport, with no second message set or handler.** `[current]`
- NET-004: **The daemon MUST create the local endpoint's directory and restrict
  it to the daemon's user, applying the configured group when one is set.**
  `[current]`
- NET-005: **On Unix-like systems, the daemon MUST apply mode 700 to the socket
  directory and socket when no group is configured, and mode 770 when a group is
  configured.** `[current]`
- NET-006: **On Unix-like systems, the daemon MUST refuse a pre-existing endpoint
  directory that is a symbolic link, is not a directory, is owned by another
  user, or carries a wider mode than the rule allows, and MUST NOT chmod or chown
  such a path.** `[current]`
- NET-007: **On Unix-like systems, the configured socket group MUST be an
  existing group; an unknown group name MUST be refused rather than replaced by
  the daemon's primary group.** `[current]`
- NET-008: **On Windows, every pipe instance MUST carry a security descriptor
  that grants full access to the daemon's own account and to Local System, denies
  the network logon group, and carries no inherited access.** `[current,
  compile-verified only]`
- NET-009: **On Windows, the pipe MUST reject remote clients, because the access
  control list alone does not prevent a remote caller that holds the daemon
  account's credentials.** `[current, compile-verified only]`
- NET-010: **On Windows, the first pipe instance MUST claim first-instance
  ownership so that a process already holding the name makes the daemon fail
  loudly rather than share the pipe.** `[current, compile-verified only]`
- NET-011: **The daemon MUST treat a local caller as authenticated by the
  operating system's access control and MUST NOT require a token for a local
  call.** `[current]`
- NET-012: **The daemon MUST NOT fall back from a failed local transport to the
  network transport.** `[current]`

## 6.5 Remote transport

- NET-013: **The daemon MUST expose a remote listener only when the operator
  supplies a bind address, and MUST NOT define a default bind address.**
  `[current]`
- NET-014: **The bind address setting MUST accept an IP-and-port literal and MUST
  refuse a hostname, an empty value acting as disabled, and a malformed value
  with a typed configuration error that names the setting and the value.**
  `[current]`
- NET-015: **The remote listener MUST be plaintext; the daemon MUST NOT terminate
  TLS, present a certificate, or manage certificate material.** `[current]`
- NET-016: **The remote listener MUST require a per-surface credential on every
  call, including the event stream.** `[current]`
- NET-017: **The daemon MUST read forwarded information only from proxy addresses
  the operator names explicitly.** `[current]`
- NET-018: **The daemon MUST trust no proxy implicitly, including loopback, and an
  absent or empty trusted-proxy setting MUST name no proxy.** `[current]`
- NET-019: **When the peer is a trusted proxy, the daemon MUST read the left-most
  entry of the forwarded-for header as the client address and MUST treat a
  forwarded protocol value of `https` as an encrypted client leg; the daemon MUST
  NOT walk a multi-hop forwarded chain.** `[current]`
- NET-020: **Forwarded headers from an untrusted peer MUST be ignored.** `[current]`
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

- CFG-001: **The local endpoint MUST resolve to `$XDG_RUNTIME_DIR/metis/daemon.sock`
  when that variable is set and non-empty, otherwise to
  `$TMPDIR/metis/daemon.sock` when that variable is set and non-empty, otherwise
  to a typed "no private base" error.** `[current]`
- CFG-002: **An endpoint override setting MUST win over the platform default, and
  an absent or empty override MUST be treated as unset.** `[current]`
- CFG-003: **On Windows, the local endpoint MUST default to
  `\\.\pipe\metis-daemon`, and an override MUST start with the local pipe prefix
  compared case-insensitively, MUST fit 256 characters, and MUST carry no
  interior null character.** `[current, compile-verified only]`
- CFG-004: **A malformed Windows pipe name MUST be a typed configuration error
  naming the setting and the value, and MUST NOT fall back to the default.**
  `[current, compile-verified only]`
- CFG-005: **Configuration MUST be read once at start-up, and changing it MUST
  require a restart (see `DAE-015`).** `[current]`
- CFG-006: **A secret configuration value MUST be resolvable from a literal, an
  environment variable reference, a platform keystore reference, or a secret file
  reference.** `[current]`
- CFG-007: **A secret file reference MUST be refused unless the file is readable
  by its owner and grants no access to group or others.** `[current]`
- CFG-008: **A keystore reference MUST use the platform's keychain through its
  command-line tool when that tool is available; a keychain that is present but
  refuses MUST be an error, not a silent fallback to another source.**
  `[current]`
- CFG-009: **A configured secret reference that cannot be resolved MUST fail the
  daemon start-up that needs it with a typed error, and MUST NOT be treated as an
  absent secret.** `[current]`
- CFG-010: **An empty secret MUST be treated as unresolved, not as a secret that
  an empty presentation can match.** `[current]`
- CFG-011: **The daemon's persistent state MUST live in one durable store whose
  directory is created at mode 700 and whose file is created at mode 600 on
  Unix-like systems.** `[current]`
- CFG-012: **The state location MUST resolve from a dedicated setting when set
  and non-empty, otherwise `$XDG_DATA_HOME/metis/metis.db`, otherwise
  `$HOME/.local/share/metis/metis.db`, otherwise a typed "no state directory"
  error.** `[current]`
- CFG-013: **A fresh installation MUST require no preparation step, and stored
  state from an earlier version MUST be upgraded automatically when the daemon
  opens it.** `[current]`
- CFG-014: **The state store MUST NOT contain a token, a preshared key, or any
  other plaintext secret (see `AUT-014`).** `[current]`
- CFG-015: **Configuration precedence across files, environment variables,
  command-line flags, and native stores is unresolved.** An implementation's
  choice is a decision outside this specification. `[open]`
- CFG-016: **The daemon's trust store MUST contain the root keys and key
  revocations used to verify bundles, and the compiled-in default root set MUST
  be empty so that a fresh installation trusts no third-party key.**
  `[current]`

The operator-facing setting names are part of the external interface and are
recorded in `12-external-interfaces.md`. Their meanings above are requirements;
renaming one is a breaking change for operators.
