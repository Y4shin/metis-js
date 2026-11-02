# 7. Surface identity, access control, tokens, and pairing

## 7.1 Surface identity and protocol version

- IDN-001: **Every client call MUST carry a surface identity declaration
  consisting of a protocol version, a surface name, and a client version.**
  `[current]`
- IDN-002: **The daemon MUST refuse a call whose protocol version is absent,
  malformed, or unsupported.** `[current]`
- IDN-003: **A version refusal MUST name the supported protocol versions in a form
  the client can read without parsing message text.** `[current]`
- IDN-004: **The client MUST map a version refusal to a distinct typed error.**
  `[current]`
- IDN-005: **The surface identity declaration MUST NOT authenticate a caller by
  itself and MUST NOT grant any permission.** The surface name selects which
  stored credential belongs to the caller; the credential is what authenticates.
  `[current]`
- IDN-006: **The daemon MUST be able to identify which surface made a call for
  logging and diagnostics.** `[current]`
- IDN-007: **A handler MUST NOT be exempt from checking the identity declaration
  merely because another handler checks it; each handler that depends on the
  declaration MUST check it.** `[current]`
- IDN-008: **Each surface that the application ships MUST declare one of the
  documented surface classes: `cli`, `ui`, `test`, or `unspecified`.** The daemon
  records a declared surface name as descriptive data; whether it rejects
  an unknown name is open (`OQ-099`). `[current]`

The exact metadata names used by the current realization are recorded in
`12-external-interfaces.md`.

## 7.2 Admission policies

- AUT-001: **A listener MUST mount exactly one admission policy: local access
  control for the local transport, or token required for the remote transport.**
  `[current]`
- AUT-002: **The admission decision MUST be made by the mounted policy and MUST
  NOT be inferred from the peer address.** `[current]`
- AUT-003: **Unary and streaming calls MUST pass through the same admission
  logic, so a streaming call cannot bypass a check a unary call enforces.**
  `[current]`
- AUT-004: **Under local access control, a call MUST be served without a
  credential, and the operating system's access control on the local endpoint is
  the authentication.** `[current]`
- AUT-005: **Under token-required policy, a call that presents no credential MUST
  be refused as a missing credential.** `[current]`
- AUT-006: **Under token-required policy, a call that presents a credential that
  is not a bearer credential MUST be refused as invalid, not as missing.**
  `[current]`
- AUT-007: **Under token-required policy, a call whose surface is unknown or
  revoked MUST be refused as revoked.** `[current]`
- AUT-008: **Under token-required policy, a call whose token does not verify MUST
  be refused as invalid.** `[current]`
- AUT-009: **A storage fault during credential verification MUST fail closed as
  an invalid credential.** `[current]`
- AUT-010: **The pairing call MUST be exempt from credential admission, because
  it is the call that obtains the credential.** `[current]`
- AUT-011: **The pairing exemption MUST be an explicit, enumerated exemption by
  operation; adding another pre-authentication operation MUST be a deliberate
  change.** `[current]`

## 7.3 Pairing

- AUT-012: **A surface MUST pair by presenting the preshared key, and the daemon
  MUST return one opaque token for the surface named in that caller's surface
  identity declaration.** `[current]`
- AUT-013: **The daemon MUST return the token text exactly once and MUST NOT be
  able to return it again.** `[current]`
- AUT-014: **The daemon MUST store only an irreversible hash of the token, never
  the token itself.** `[current]`
- AUT-015: **The daemon MUST compare the presented preshared key in constant time.**
  `[current]`
- AUT-043: **The daemon MUST complete the preshared-key comparison before writing
  any pairing state.** `[current]`
- AUT-016: **A wrong preshared key MUST pair nothing and MUST reveal nothing about
  existing state.** `[current]`
- AUT-017: **A surface that is already paired MUST NOT be pairable again; the
  attempt MUST be refused and the existing credential MUST remain usable.**
  `[current]`
- AUT-018: **Pairing MUST be reachable over both the local transport and the
  remote listener, gated by the same preshared key.** `[current]`
- AUT-019: **The preshared key MUST travel in the pairing request body and MUST
  NOT appear in a URL or a query string.** `[current]`
- AUT-020: **Pairing is optional. An absent or empty preshared key MUST close
  pairing while the daemon starts and every other call keeps serving.**
  `[current]`
- AUT-021: **A configured preshared key reference that cannot be resolved MUST be
  a typed error, not a silently disabled pairing state.** `[current]`
- AUT-022: **Closing pairing MUST NOT revoke any token.** `[current]`
- AUT-044: **A surface paired before pairing was closed MUST keep serving until
  its credential is revoked.** `[current]`
- AUT-023: **A pairing refusal MUST be reported as one of: pairing disabled, wrong
  preshared key, or already paired, in a form the client maps to a typed error
  without parsing message text.** `[current]`
- AUT-024: **The client MUST surface a pairing refusal as a typed pairing error.**
  `[current]`

## 7.4 Tokens and revocation

- AUT-025: **A token MUST grant the full exposed surface and MUST NOT carry
  per-client permissions or scopes.** `[current]`
- AUT-026: **Every paired client MUST have the same access; permission decisions
  belong to agent contexts, not to devices.** `[current]`
- AUT-027: **A token MUST be presented as a bearer credential in request metadata
  on every call, including the event stream.** `[current]`
- AUT-028: **A token MUST NOT appear in a URL, a query string, or a log line.**
  `[current]`
- AUT-029: **Tokens MUST NOT expire in the first version; an expiry or refresh
  mechanism MAY be added later without invalidating the current credential
  model.** `[current]`
- AUT-030: **An operator MUST be able to revoke one surface's credential, and a
  revoked credential MUST be refused no later than the next daemon start after
  the revocation.** How revocation is performed is open (`OQ-094`). `[current]`
- AUT-031: **Revoking one surface MUST NOT affect any other surface.** `[current]`
- AUT-032: **Token verification MUST NOT return the stored hash to any caller.**
  `[current]`
- AUT-033: **A malformed token presented on the wire MUST be a typed refusal,
  never a crash or a dropped connection.** `[current]`

## 7.5 Credential storage on the client

- AUT-034: **A surface MUST keep its token in the platform keychain when the
  platform keychain's command-line tool is available, and otherwise in an
  owner-only file.** `[current]`
- AUT-035: **The fallback credential file MUST be mode 600 inside a directory at
  mode 700 on Unix-like systems.** `[current]`
- AUT-036: **The client MUST report which backend stored the credential.**
  `[current]`
- AUT-037: **A keychain that is present but refuses MUST be an error; the client
  MUST NOT silently fall back to the file when the keychain exists and fails.**
  `[current]`
- AUT-038: **A surface name that cannot be used as a single file name MUST be
  refused rather than resolved into a path that leaves the credential
  directory.** `[current]`
- AUT-039: **Credential lookup MUST consult exactly one backend, so one token
  does not exist in two places at once.** `[current]`
- AUT-041: **The preshared key MUST NOT appear in a log line.** `[current]`
- AUT-042: **The preshared key MUST NOT appear in any request body other than the
  pairing request body.** `[current]`

## 7.6 Compatibility and security notes

- AUT-040: **An implementation that adopts an existing daemon state directory
  MUST keep previously paired surfaces working, or MUST migrate their stored
  credentials deliberately.** How a credential is stored is not a product
  requirement, but discarding live credentials is not an acceptable silent
  migration. `[current]`
- Note 1: The remote transport is plaintext. The preshared key and every token can
  be observed by anyone who can read the connection. Security therefore rests on
  the proxy, the network, and the deployment, not on the daemon. This is an
  accepted risk, not a guarantee.
- Note 2: Pairing over a network-reachable listener has no rate limit and no
  lockout. Repeated guesses are slowed only by the network. This is an open risk.
- Note 3: Local access control bounds access to the daemon's operating-system
  account and configured group. Any process running as that account, in any logon
  session, can connect on Windows, exactly as mode 0700 behaves on Unix.
- Note 4: An implementation that caches credentials still makes a revocation
  take effect no later than the next start. The current implementation reads the
  store on each call, so a revocation can take effect sooner.
