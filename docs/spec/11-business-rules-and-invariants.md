# 11. Business rules, invariants, and state transitions

This section states the cross-cutting rules that hold across components.
Component requirements define their own behavior; the rules here are the
obligations that no component violates.

## 11.1 Permission invariants

- INV-001: **Every execution that can call a tool MUST have exactly one enclosing
  agent context, and every tool call MUST be checked against that context's
  permissions at dispatch time.** `[planned]`
- INV-002: **A task, step, workflow, or disclosure selection MAY narrow the
  permissions of its enclosing context and MUST NOT widen them.** `[planned]`
- INV-003: **A plugin declaration or a model selection MUST NOT grant
  permission.** Tool visibility and toolset membership are proposal material and
  are covered by `INV-005`. `[current]`
- INV-004: **A hidden tool MAY remain permitted; a visible tool MUST still pass
  the dispatch permission check.** `[current]`
- INV-005: **Under the tool disclosure proposal, search results and disclosure
  decisions would not reveal a capability outside the agent context's permission
  boundary.** This is proposal material and not an accepted requirement.
  `[proposed]`
- INV-006: **A surface credential MUST NOT carry permissions; permissions belong
  to agent contexts, not devices.** `[current]`

## 11.2 Persistence invariants

- INV-007: **The agent runtime MUST be able to reconstruct the live session state
  exactly at every idle boundary, from the persisted state plus the acknowledged
  persistence events.** `[planned]`
- INV-008: **An idle session MUST NOT be discarded before its persistence events
  are acknowledged.** `[planned]`
- INV-009: **A persistence failure MUST be reported and MUST NOT be represented as
  a successful save.** `[planned]`
- INV-010: **External side effects and process-local handles are outside the
  reconstruction invariant; they MUST NOT be claimed as restorable.**
  `[planned]`
- INV-011: **A daemon restart MUST NOT be assumed to preserve the event window,
  the in-flight cancellation registry, or any cursor.** `[current]`
- INV-012: **A stored configuration change MUST NOT take effect in a running
  daemon (see `DAE-015`).** `[current]`

## 11.3 Plugin and bundle invariants

- INV-013: **Each side of the plugin boundary MUST free only memory it
  allocated.** `[current]`
- INV-014: **Plugin code MUST run only while the host knows it is running; a
  library MUST NOT be unmapped while a future, waker clone, or callback can still
  reference it.** `[current]`
- INV-015: **The plugin host MUST refuse a capability call when the capability is
  undeclared, disabled, or version-incompatible.** `[current]`
- INV-036: **The daemon MUST refuse a tool call that is outside the caller's
  agent-context permissions.** `[planned]`
- INV-016: **A trust failure MUST stop daemon start-up; a container refusal MUST
  disable only that bundle.** `[current]`
- INV-017: **A plugin's identity for configuration MUST be the name under which
  its bundle was found until its manifest is read; after that, the manifest's
  name is the registration and routing identity.** `[current]`
- INV-018: **A disabled plugin MUST remain visible and identifiable, and invoking
  one of its capabilities MUST answer with the disabled refusal and its reason.**
  `[current]`
- INV-019: **A plugin MUST NOT observe two configurations in one process
  lifetime.** `[current]`
- INV-020: **A plugin's own version MUST NOT change whether it loads (see
  `PLG-017`).** `[current]`
- INV-021: **A bundle MUST be signed over every byte a user installs, so no
  installed artifact escapes the signature.** `[current]`
- INV-022: **A carried key identifier MUST decide nothing; the daemon compares only a
  recomputed key identifier.** `[current]`
- INV-023: **A plugin MUST NOT be unloaded to disable it.** `[current]`

## 11.4 Event and stream invariants

- INV-024: **The daemon MUST assign event identifiers monotonically and daemon-wide,
  and one stream's observed order MUST equal event-identifier order.** `[current]`
- INV-025: **The daemon MUST deliver an event at most once on a stream and MUST NOT
  silently skip an event.** `[current]`
- INV-026: **The daemon MUST NOT treat a cursor as a durable position; only the
  non-durable event window bounds resume.** `[current]`
- INV-027: **The daemon MUST NOT replay an event already covered by a cursor and
  MUST NOT open a resumed stream with a snapshot.** `[current]`
- INV-028: **The daemon MUST end delivery at a terminal stream refusal, and no
  event follows it.** `[current]`
- INV-029: **A disconnect or a closed user interface MUST NOT cancel work (see
  `PRD-C1`).** `[current]`

## 11.5 Security invariants

- INV-030: **A token, a preshared key, or any other secret MUST NOT appear in a
  URL, a query string, or a log line.** `[current]`
- INV-031: **The daemon MUST store only an irreversible hash of a token (see
  `AUT-014`).** `[current]`
- INV-032: **A network-reachable listener MUST require a credential on every
  call, and the pairing operation is the only pre-authentication exemption.**
  `[current]`
- INV-033: **A local endpoint's access control MUST be established before the
  daemon accepts a connection, and a pre-existing path that violates the rule
  MUST be refused rather than repaired.** `[current]`
- INV-034: **A forwarded header MUST NOT influence authorization; it informs
  only exposure decisions and diagnostics.** `[current]`
- INV-035: **A component MUST NOT claim encryption, authentication, or isolation it
  cannot observe or enforce.** `[current]`

## 11.6 State transitions

### Daemon start-up

```text
read configuration once
  -> resolve secrets
  -> open and migrate persistent state
  -> prepare local endpoint with its access rule
  -> if a bind address is configured: prepare remote listener and policy
  -> start plugin set (see the plugin start-up table)
  -> serve
```

A trust failure, a plugin directory that cannot be listed, a bundle file that
cannot be read from the filesystem, an invalid or name-mismatched manifest, or a
refused unload stops start-up. A disable-class failure, including a container
this build cannot read, lets start-up finish with that plugin listed as disabled.

### Surface credential

```text
unpaired --Pair(correct preshared key)--> paired (token stored by surface)
unpaired --Pair(wrong key)--> unpaired (no state written)
paired   --Pair(any key)--> paired (already-paired refusal)
paired   --revoke credential--> revoked (refused as revoked; no later than next
                               start, or immediately if credentials are read per call)
paired   --remove preshared key--> paired (keeps serving; new pairing disabled)
```

### Event stream

```text
open(cursor = 0)          -> snapshot -> events -> heartbeat/terminal
open(servable cursor)     -> replay -> events -> heartbeat/terminal
open(expired cursor)      -> refused before the stream opens
consumer falls behind     -> terminal slow-consumer refusal, then end
Cancel(request id)        -> terminal cancelled refusal, then end
transport failure         -> transport error, no claim about the work
```

### Plugin instance

The lifecycle is defined in `02-domain-model.md` section 2.9. The outcomes that
govern each transition are defined in `09-plugins.md` sections 9.12 and 9.13.

### Cancellation

```text
call in flight --Cancel--> cancelled outcome; work stops; stream ends with a
                           cancelled refusal if the call was a stream
call finished  --Cancel--> already-finished outcome
call unknown   --Cancel--> already-finished outcome
call dropped   --(no Cancel)--> buffers released, service requests cancelled,
                                nothing delivered to the caller
```
