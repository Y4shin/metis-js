# 14. Acceptance criteria and edge cases

Each acceptance criterion is independently checkable and cites the requirement
group it verifies. Criteria marked `[implemented]` have supporting evidence and test records. Criteria marked `[planned]` describe checks a
future implementation is required to pass; they are derived from the requirements, not from
an implementation that exists today.

## 14.1 API, client, and transports

- AC-001: A client opens an ordinary call to a daemon over the local transport
  and receives the response, with no transport-specific code in the caller.
  Verifies API-001, API-010, API-011, NET-001. `[implemented]`
- AC-002: The same service answers identically when mounted on the local
  transport and on the remote listener, for the same requests. Verifies DAE-010,
  NET-003, AUT-001, AUT-003. `[implemented]`
- AC-003: An unsupported protocol version is refused, and the client maps the
  refusal to a distinct typed error that names the supported versions. Verifies
  IDN-002, IDN-003, IDN-004. `[implemented]`
- AC-004: A client call that declares a request identifier receives that
  identifier back verbatim in the call's answer. Verifies CAN-004.
  `[implemented]`
- AC-005: With the remote bind setting unset or empty, the daemon opens no remote
  listener. Verifies NET-013, NET-014. `[implemented]`
- AC-006: The local endpoint and its directory are restricted to the daemon's user
  without a configured group and to that user and group with one. Verifies
  NET-004, NET-005. `[implemented]`
- AC-007: A pre-existing endpoint directory that is a symbolic link, is not a
  directory, is owned by another user, or is wider than the rule is refused and
  left unchanged. Verifies NET-006. `[implemented]`
- AC-008: A malformed remote bind value, trusted-proxy entry, secret reference, or
  endpoint override produces a typed configuration error that names the setting
  and the value. Verifies NET-014, NET-022, CFG-009, CFG-004. `[implemented]`
- AC-009: A forwarded client address is read only from a named trusted proxy, and
  information an untrusted peer forwards is ignored. Verifies NET-017 to NET-021.
  `[implemented]`
- AC-010: A listener bound outside loopback with no trusted proxy prints one
  plaintext warning before accepting. Verifies NET-023. `[implemented]`

## 14.2 Identity, pairing, and credentials

- AC-011: Pairing with the correct preshared key returns a token once, and the
  stored state contains only a hash. Verifies AUT-012 to AUT-014. `[implemented]`
- AC-012: Pairing a second time for the same surface is refused and the first
  credential is intact. Verifies AUT-017. `[implemented]`
- AC-013: A wrong preshared key writes no pairing state. Verifies AUT-015,
  AUT-016. `[implemented]`
- AC-014: Under token-required policy, a call with no credential, a wrong token,
  and an unknown or revoked surface each produce a distinct typed refusal, and
  the handler does not run. Verifies AUT-005 to AUT-009. `[implemented]`
- AC-015: Under local access control, the same call is served with no credential.
  Verifies AUT-004, NET-011. `[implemented]`
- AC-016: Pairing remains exempt from credential admission over both transports.
  Verifies AUT-010, AUT-018. `[implemented]`
- AC-017: With no preshared key configured, pairing is refused as disabled while
  previously paired surfaces keep serving. Verifies AUT-020, AUT-022.
  `[implemented]`
- AC-018: Revoking one surface leaves every other surface working. Verifies
  AUT-030, AUT-031. `[implemented]`
- AC-019: With no platform credential store available, the client stores the
  token in an owner-only file inside an owner-only directory and reports that
  backend; with a store available, the store is used. Verifies AUT-034 to AUT-036.
  `[implemented]`
- AC-020: A malformed token is refused as invalid, not as a crash or a dropped
  connection. Verifies AUT-033. `[implemented]`

## 14.3 Events and cancellation

- AC-021: A cursorless stream's first message is a snapshot; a later selected
  event arrives with a greater identifier; an unselected channel contributes
  nothing; identifiers never decrease. Verifies RTS-009, RTS-015 to RTS-017,
  RTS-031, RTS-032. `[implemented]`
- AC-022: A stream opened with a servable cursor replays exactly the held events
  that follow it and carries no snapshot. Verifies RTS-018, RTS-024, RTS-027.
  `[implemented]`
- AC-023: The two edges of the window are servable: a cursor one below the oldest
  held identifier and the newest identifier. Verifies RTS-024.
  `[implemented]`
- AC-024: A cursor below the window and a cursor above the newest identifier are
  both refused as expired before any message is delivered, and a cursorless
  stream then opens with a snapshot. Verifies RTS-007, RTS-025, RTS-026,
  RTS-027. `[implemented]`
- AC-025: A disconnect and a reconnect with the last seen identifier deliver
  exactly the missed events, with no gap and no duplicate. Verifies RTS-024 to
  RTS-026. `[implemented]`
- AC-026: A restarted daemon expires every cursor from the earlier lifetime.
  Verifies RTS-012, RTS-026. `[implemented]`
- AC-027: A reader that stops reading receives a terminal slow-consumer refusal
  on its next read, no event follows it, and the client reports a distinct typed
  error. Verifies RTS-034 to RTS-037. `[implemented]`
- AC-028: A first cancellation of an in-flight request reports cancelled and the
  stream stops; a second cancellation reports already finished. Verifies CAN-008
  to CAN-011, CAN-016. `[implemented]`
- AC-029: A cancellation under another surface's identifier reports already
  finished and does not touch that work. Verifies CAN-012, CAN-013.
  `[implemented]`
- AC-030: Disconnecting a client mid-stream cancels nothing; a second stream and
  its cancellation still work. Verifies CAN-014, CAN-015. `[implemented]`
- AC-031: A call that declares no request identifier behaves exactly as before.
  Verifies CAN-001. `[implemented]`
- AC-032: A transport failure is reported as a transport error, not as a
  deliberate drop or a cancellation. Verifies RTS-038. `[implemented]`

## 14.4 Plugin contract, bundles, and host

- AC-033: A real plugin loads through the versioned entry point, is served the
  operations it declares, dispatches a ready result and a pending result,
  cancels, and releases payloads with ownership balanced. Verifies PLG-010 to
  PLG-026, PLG-036 to PLG-052. `[implemented]`
- AC-034: An unexpected failure inside a plugin's result query and one inside a
  host callback each become an internal status, and the instance still serves the
  next call. Verifies PLG-077, PLG-035. `[implemented]`
- AC-035: The published boundary contract and the host's declared contract extent
  agree, and a deliberate mismatch is refused rather than misread. Verifies
  PLG-012, PLG-056. `[implemented]`
- AC-036: A signed bundle verifies and loads; a one-byte alteration is refused; a
  chain signed by a key that is not a configured root is refused; the test key is
  not a compiled-in root. Verifies BND-001 to BND-021, SEC-002, SEC-006,
  HST-042. `[implemented]`
- AC-037: A bundle entry that escapes the private location, is a symbolic link,
  is a directory or not a regular file, declares a mode that grants access beyond
  the owner, or exceeds a size limit is refused, and what was created is removed.
  Verifies BND-024 to BND-032, SEC-003. `[implemented]`
- AC-038: A manifest whose name disagrees with the leaf authorisation's name,
  whose contract demand is incompatible, or whose declared code is absent for this
  platform is refused with a reason that names the failure. Verifies MAN-003,
  MAN-007 to MAN-010. `[implemented]`
- AC-039: A trust failure stops start-up; a container this build cannot read
  disables only that plugin; a failed initialisation becomes a listed disabled
  placeholder; a refused unload stops start-up. Verifies HST-005 to HST-008,
  INV-016. `[implemented]`
- AC-040: A disabled plugin's capabilities are excluded from what an agent is
  offered, and invoking one answers the disabled refusal with its reason.
  Verifies HST-008, HST-009, PLG-067. `[implemented]`
- AC-041: Releasing an instance unregisters, quiesces, destroys, then releases
  the plugin's code, in that order, and reports the lifecycle entries. Verifies
  HST-012, HST-019. `[implemented]`
- AC-042: An instance that cannot be released stays resident and is reported.
  Verifies HST-014, HST-015, HST-020. `[implemented]`
- AC-043: A capability that blocks its result query is detected and quarantined,
  and its code is not released while the abandoned query may still run. Verifies
  PLG-038, HST-013, HST-016, HST-017. `[implemented]`
- AC-044: A capability whose instance name itself carries the qualification
  separator routes to the correct owner. Verifies PLG-058, PLG-059.
  `[implemented]`
- AC-045: A host with no sink records deliveries in its accessors; a sink receives
  owned copies, per-instance emission order, and an unexpected failure in the sink
  is recorded without failing the plugin call. Verifies HST-028 to HST-038.
  `[implemented]`
- AC-046: A sample plugin is packaged as a signed bundle, loaded end to end,
  dispatches a capability, emits an observable event, and releases cleanly in the
  continuous integration gate. Verifies HST-041. `[implemented]`
- AC-047: A plugin authored in a second language against the published boundary
  contract participates in the same end-to-end gate. Verifies PLG-056, SDK-013.
  `[implemented]`

## 14.5 Planned runtime and workflow criteria

- AC-048: A conversation retains all prior messages after a request context is
  pruned or transformed. Verifies CNV-005, CNV-006. `[planned]`
- AC-049: Dropping an in-memory session and reloading it from stored state plus
  acknowledged persistence events reproduces the same state at an idle boundary.
  Verifies SES-004 to SES-006. `[planned]`
- AC-050: An agent task with a declared output schema rejects an output that does
  not validate and does not report success. Verifies TSK-007. `[planned]`
- AC-051: A task or step cannot call a tool outside its enclosing agent context's
  permissions. Verifies AGT-006, INV-001, INV-002. `[planned]`
- AC-052: A workflow step list that omits a required step still executes according
  to dependencies and connector routing. Verifies WFL-004. `[planned]`
- AC-053: Two connectors that target the same input port are rejected. Verifies
  WFL-018. `[planned]`
- AC-054: A join waits for a value or a confirmed unreachable port and does not
  treat an unresolved port as unreachable. Verifies WFL-019 to WFL-022.
  `[planned]`
- AC-055: An unreachable marker is distinguishable from a null payload at a join.
  Verifies WFL-023. `[planned]`
- AC-056: A script step that declares a tool it is not permitted to use is
  rejected before execution. Verifies WFL-026, WFL-027, WFL-031. `[planned]`
- AC-057: A script step cannot read the filesystem, open a network connection,
  start a process, or reach host interfaces through its script environment.
  Verifies WFL-029, WFL-030. `[planned]`
- AC-058: A workflow run executes with its agent context's permissions and cannot
  widen them through a step. Verifies WFL-035, AGT-006. `[planned]`

## 14.6 Surface criteria

- AC-059: The application does not expose a local network service to serve its own
  interface, and the user-interface layer does not talk to the daemon directly.
  Verifies SUR-002, SUR-005. `[planned]`
- AC-060: Closing the desktop or Android user interface does not stop daemon
  work. Verifies SUR-007, PRD-C1. `[planned]`
- AC-061: Android system-assistant activation reaches Metis from a cold start
  without assuming the user interface is initialised. Verifies SUR-015, SUR-018.
  `[planned]`
- AC-062: No iOS project is present or produced. Verifies SUR-010.
  `[current]`
- AC-068: The desktop and Android applications let a user pair by entering the
  preshared key, store the returned credential through the client, and report
  which backend stored it. Verifies SUR-023, AUT-034 to AUT-036. `[planned]`
- AC-069: The implementation's own documentation does not claim encryption,
  client authentication, remote attestation, or measured Windows behavior that
  the daemon does not provide. Verifies SEC-007, SEC-014, CMP-008, INV-035.
  This criterion is checked by document review. `[current]`

## 14.7 Edge cases

| Edge case | Required behavior |
| --- | --- |
| Endpoint override is set to an empty string | Treated as unset; the platform default applies. |
| No private runtime or temp directory exists | The daemon refuses with a typed "no private base" error rather than using a shared location. |
| An endpoint override names a remote endpoint or contains a null | Refused with a typed configuration error. |
| A secret file is group- or world-readable | Refused; the secret is not read. |
| A secret reference points at a missing file or unset variable | Daemon start-up fails with a typed unresolved error, not a silent disable. |
| A credential store exists but exits non-zero | Error; no fallback. |
| A token is malformed or its presentation with a call is malformed | Invalid credential, not missing and not a crash. |
| A surface paired before pairing was closed | Keeps serving until its credential is revoked. |
| A cursor is zero | Never expired. |
| A cursor equals the newest identifier | Servable; replays nothing. |
| A cursor is one below the oldest held identifier | Servable; the next event is the one that opened the window. |
| A cursor is above the newest identifier | Expired (daemon restart case). |
| A cursor is presented on a changed channel selection | The caller resyncs; cursor reuse across selections is undefined. |
| A stream selects the same channel twice | The channel contributes once. |
| A stream selects no channel | No channel events; a cursorless stream still opens with an empty snapshot message. |
| A slow consumer reconnects with its last identifier | It may resume if the window still holds the position, otherwise it resyncs. |
| A request identifier is malformed | Echoed verbatim for correlation; cancellation and stream registration refuse it as invalid argument, and other calls serve it. |
| A cancellation names work that already finished | Already-finished outcome. |
| A cancellation names work from another surface | Already-finished outcome; the work is untouched. |
| An event is published between opening the subscription and reading the window | It is not lost. |
| An event carries a kind the receiver does not know | The receiver treats it as unspecified and does not act on it. |
| A pair request declares a surface name outside the shipped classes | Open (`OQ-099`); the daemon records the name as descriptive data today. |
| A plugin manifest is absent or invalid | Start-up stops; the plugin is not half-described. |
| A bundle file cannot be read from the filesystem | Start-up stops. |
| The plugin directory cannot be listed | Start-up stops. |
| A bundle carries no code for this platform | That plugin is disabled with a reason; the daemon runs. |
| A bundle declares a code path outside the private location | Refused; the path matches the manifest form (`BND-033`). |
| A manifest declares a schema whose default is null | The schema must express it; the manifest format is chosen so that it can. |
| A plugin initialisation refuses its configuration | Invalid-argument failure; the instance is destroyed and a disabled placeholder is listed. |
| A plugin cannot be destroyed after a failed initialisation | Start-up stops. |
| A plugin blocks during initialisation | The daemon hangs; this is accepted and documented, not mitigated. |
| A plugin signals a completion handle twice | The second use is refused; state is not released twice. |
| A plugin signals after its pending result was dropped | Recorded as a late signal naming the request identifier; no dangling access. |
| A plugin capability is disabled while a call is in flight | The in-flight call continues; new calls are refused. |
| A plugin is re-enabled | New calls are accepted without a reload. |
| A plugin returns a response that violates its own schema | Internal error attributed to the plugin; no quarantine. |
| A plugin declares a capability kind the host does not know | That capability is refused; the rest of the plugin serves. |
| Plugin code cannot be released | It stays resident and is reported; the daemon does not force it. |
| A workflow port has no value and no confirmed unreachable path | The join keeps waiting; value absence is not unreachability. |
| A workflow branch fails or is cancelled | The effect on join readiness and run outcome is open (`OQ-024`); the branch is not treated as deliberately abandoned. |
| A workflow retry would repeat an external write | Requires explicit idempotency or reconciliation; exactly-once is not assumed. |

## 14.8 Checks that cannot currently be claimed

- AC-063: The Windows local transport executes on a Windows host and passes a
  serve-and-connect test. **Not met.** The path compiles for the Windows target
  and its platform-independent rules are tested on Linux; no Windows execution
  has occurred.
- AC-064: The Windows endpoint's access rule is read back and asserted to contain
  exactly the intended access entries. **Not met.**
- AC-065: A remote client and a low-integrity same-user process are both denied by
  the Windows local transport. **Not met.**
- AC-066: The Windows client wrapper has a constructor and a credential path.
  **Not met.** The connector exists; the client constructor is missing.
- AC-067: The agent runtime, workflow runtime, daemon composition layer, plugin
  SDK, desktop shell, and Android shell pass their acceptance tests. **Not met.**
  These components are planned and not implemented.
