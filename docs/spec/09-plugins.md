# 9. Plugins: contract, bundles, manifests, host lifecycle, and authoring

The plugin system is an external contract. A third-party author writes a plugin
against the boundary contract and installs it as a bundle, and an operator
installs that bundle by copying one file. The contract's operation semantics, the
bundle's semantics, the manifest's contents, and the start-up outcomes are
requirements of the finished application. The concrete form of each is a decision
this specification opens in `15-open-questions.md`: what code a plugin is
(`OQ-103`), the shape of the boundary contract (`OQ-104`), how pending work and
cancellation are reported (`OQ-105`), the payload encoding (`OQ-106`), the
container and archive format (`OQ-107`), the signature and trust mechanism
(`OQ-108`), the manifest format and field names (`OQ-109`), the schema dialect
(`OQ-110`), and the contract's version numbers (`OQ-111`).

## 9.1 Plugin model and boundary

- PLG-001: **A plugin MUST be code the daemon runs inside its own process.** The
  form that code takes is open (`OQ-103`). `[current]`
- PLG-002: **A plugin MUST advertise its capabilities before the host serves any
  call to it.** `[current]`
- PLG-003: **A plugin MUST supply capabilities; it MUST NOT export workflow
  steps.** `[current]`
- PLG-004: **A plugin MUST NOT run code on its own schedule or keep its own
  worker running; it MUST return control to the host, and the host MUST own when
  plugin code runs.** `[current]`
- PLG-005: **A plugin that must wait for a timer or I/O MUST request the facility
  from the host, report a pending result, and signal the host when the answer
  arrives; it MUST NOT block while it waits.** `[current]`
- PLG-006: **A plugin MUST NOT be treated as sandboxed. Running a plugin is code
  execution with the daemon's privileges in the daemon's process, and a plugin
  that blocks during initialisation hangs the daemon.** `[current]`
- PLG-007: **A value the host cannot interpret MUST NOT cross the boundary; only
  the representations the contract defines may cross it.** `[current]`
- PLG-008: **A high-level authoring type in any language MUST remain internal to
  each plugin; the export layer MUST translate an implementation into the
  boundary contract.** `[current]`
- PLG-009: **Only the representations the contract defines MAY cross the
  boundary.** `[current]`

## 9.2 Contract versioning and operations

- PLG-010: **A plugin MUST expose exactly one entry point, through which the host
  obtains every other operation.** The contract's structural shape is open
  (`OQ-104`). `[current]`
- PLG-011: **The contract's major version MUST be decided before any operation is
  used, so a plugin built for another major is refused before it serves
  anything.** `[current]`
- PLG-012: **Both directions of the contract MUST declare their extent, and an
  operation MUST be used only when the declared extent covers it, never because a
  value or a reference happens to be present.** `[current]`
- PLG-013: **Within one contract major, the operation set MUST be append-only, so
  a plugin built against a newer minor is served up to the operations this host
  knows, decided by the declared extent.** `[current]`
- PLG-014: **The host MUST accept the contract minor it knows and any newer minor
  in the same major, and MUST refuse an older minor than it requires.** The
  version numbers are open (`OQ-111`). `[current]`
- PLG-015: **The manifest MUST declare the contract major and the minimum minor
  the plugin requires, with no range syntax, and the host MUST refuse a differing
  major or a host minor lower than the required minimum.** `[current]`
- PLG-016: **A capability whose descriptor version the host does not support MUST
  be refused on its own, and the rest of the plugin MUST keep serving.**
  `[current]`
- PLG-017: **The plugin's own version MUST be recorded and MUST never gate a
  load.** `[current]`
- PLG-018: **A stale pre-release plugin whose operation order does not match this
  build's MUST be refused by the minor comparison rather than misread.**
  `[current]`

## 9.3 Plugin-side operations

- PLG-019: **The plugin side of the contract MUST provide initialisation that may
  complete later, instance destruction, capability listing, one generic
  invocation, cancellation, payload release, shutdown, and service completion.**
  `[current]`
- PLG-020: **Initialisation MUST construct and initialise an instance in one
  fallible operation that may complete later.** It receives the host side of the contract and
  one host-owned payload carrying the plugin's configuration and the host
  context. `[current]`
- PLG-076: **On success, initialisation MUST return the instance and a
  plugin-owned pending result, and the host MUST NOT register or route to the
  instance until that result completes successfully.** `[current]`
- PLG-021: **There MUST NOT be a separate instance-creation operation.**
  `[current]`
- PLG-022: **The invocation operation MUST receive the instance, the capability
  name, a host-chosen request identifier, the request payload, and the host side
  of the contract, and on success MUST return a plugin-owned pending result.** The
  request payload is valid for that call only. `[current]`
- PLG-023: **Cancellation MUST mark the call identified by the request identifier
  so the plugin resolves it as cancelled.** `[current]`
- PLG-024: **Payload release MUST be the only way the host releases a
  plugin-owned payload, and the host MUST never release one itself.**
  `[current]`
- PLG-025: **Shutdown MUST quiesce the instance; the host MUST drop every pending
  result before calling shutdown, so nothing may arrive after it returns.**
  `[current]`
- PLG-026: **Service completion MUST answer an earlier service request with the
  same request identifier, and the plugin MUST signal the waiting pending result
  from it.** `[current]`
- PLG-027: **Instance destruction MUST be called only after shutdown.**
  `[current]`
- PLG-028: **The plugin side of the contract MUST NOT expose a counters operation
  or any plugin-side metric surface.** `[current]`

## 9.4 Host-side callbacks

- PLG-029: **The host side of the contract MUST carry a host-state value and
  exactly the callbacks progress, event emission, logging, and service request.**
  `[current]`
- PLG-030: **The host MUST NOT provide a completion callback; a call completes
  when the plugin reports its result ready.** `[current]`
- PLG-031: **Progress MUST carry the request identifier and a payload.**
  `[current]`
- PLG-032: **Event emission MUST carry a topic and a payload and MUST NOT carry a
  request identifier; a plugin that needs correlation MUST put an identifier in
  its payload.** `[current]`
- PLG-033: **Logging MUST carry a severity of debug, info, warn, or error, and a
  message.** `[current]`
- PLG-034: **A service request MUST carry the facility name, the request
  identifier, and a plugin-owned payload, and the host MUST answer it later
  through service completion.** The current facility is a timer; I/O readiness is
  planned. `[current]`
- PLG-035: **Every host callback MUST contain an unexpected failure within its own
  extent, so a fault becomes a recorded internal failure instead of propagating
  into the host.** `[current]`

## 9.5 Pending results, completion signals, and status

- PLG-036: **A result query MUST answer exactly one of: success with a
  plugin-owned response payload, pending, or a terminal status.** The handshake
  that surrounds it is open (`OQ-105`). `[current]`
- PLG-037: **Only the host MUST drive a pending result, one query at a time; two
  concurrent queries on one handle are a protocol violation.** `[current]`
- PLG-038: **A query that blocks MUST be detected by the host, recorded, and MUST
  quarantine the plugin.** `[current]`
- PLG-039: **The host MUST drop a pending result exactly once, and never while a
  query is in progress.** `[current]`
- PLG-040: **The host MUST pass one owned completion handle for that query, MUST
  release its own use of it when the query returns, and the plugin MUST NOT
  release the host's use.** `[current]`
- PLG-041: **A plugin MAY take further uses of the completion handle, each is
  separately owned, and the plugin MUST release every one.** `[current]`
- PLG-042: **One use MUST be consumed by exactly one signal or release; a second
  use of a consumed handle MUST be refused rather than release state twice.**
  `[current]`
- PLG-043: **A signal MAY arrive at any time and from any execution context, MUST
  be cheap and non-blocking, and MAY arrive after the pending result was dropped;
  the host MUST detect and record such a late signal and MUST NOT touch state it
  has already released.** `[current]`
- PLG-044: **A recorded late signal MUST name the request identifier of the call
  it belongs to.** `[current]`
- PLG-077: **An unexpected failure inside an exported result query or drop MUST
  become a terminal internal status, drop MUST remain legal afterwards, and no
  boundary path may end the daemon process.** `[current]`
- PLG-045: **The status space MUST be frozen to: ok, invalid argument, not found,
  permission denied, cancelled, internal, unsupported, and pending.** Higher
  layers add meaning rather than values. `[current]`

## 9.6 Payloads and ownership

- PLG-046: **A payload MUST declare its owner, which is one of nobody, the host,
  or the plugin.** `[current]`
- PLG-047: **Each side MUST release only what it owns; the non-owning side MUST
  never release the payload.** `[current]`
- PLG-048: **A response payload returned by a successful result query MUST be
  plugin-owned and MUST be released by the host through the plugin's payload
  release operation.** `[current]`
- PLG-049: **A request payload MUST be host-owned and valid only for the call.**
  `[current]`
- PLG-050: **A service-request payload MUST be plugin-owned and MUST be released
  by the host through the plugin's payload release operation.** `[current]`
- PLG-051: **A service-completion payload and an initialisation payload MUST be
  host-owned and borrowed for that call; a plugin that needs one afterwards MUST
  copy it.** `[current]`
- PLG-052: **Payload release MUST be reentrant; the host MAY call it from inside
  a service request.** `[current]`
- PLG-053: **Every payload MUST declare the payload format version it is written
  in.** `[current]`
- PLG-054: **An unknown payload format version MUST be refused as unsupported,
  and a malformed payload MUST be refused as invalid argument.** The transport
  encoding behind a payload format version is open (`OQ-106`). `[current]`
- PLG-055: **Payload evolution within a contract major MUST be additive: unknown
  fields are ignored and new fields are optional.** `[current]`
- PLG-056: **The boundary contract's constants and per-operation rules MUST be
  published in a form a plugin author in any language can implement from, so an
  author does not have to guess them.** `[current]`

## 9.7 Capabilities and routing

- PLG-057: **A capability MUST have a name, a kind, a version, a request schema,
  and a response schema.** `[current]`
- PLG-058: **Capability names MUST be instance-qualified at the host boundary,
  so a capability name alone does not decide which instance serves it.**
  `[current]`
- PLG-059: **An instance name that itself contains the qualification separator
  MUST route like any other instance name; the capability half MUST come from the
  routing owner rather than from a first-separator split.** `[current]`
- PLG-060: **The registry MUST refuse a duplicate qualified capability name.**
  `[current]`
- PLG-061: **The host MUST validate a request against the capability's published
  request schema and MUST answer an invalid request as an invalid argument.**
  `[current]`
- PLG-062: **The host MUST validate a plugin's response against the plugin's own
  published response schema and MUST treat a violation as that plugin's internal
  error without quarantining it.** `[current]`
- PLG-063: **The registry MUST support enable and disable of an entry.**
  `[current]`
- PLG-064: **Disabling MUST be registry state only: it refuses new calls and
  leaves the instance, its in-flight calls, and its code alone.** `[current]`
- PLG-065: **Enabling MUST accept calls again without tearing anything down.**
  `[current]`
- PLG-066: **The host MUST refuse a call to a capability the plugin did not
  declare.** `[current]`
- PLG-067: **The host MUST refuse a call to a disabled plugin with a distinct
  disabled refusal that carries the reason, so a caller can tell a disabled
  plugin from one that never loaded.** `[current]`

## 9.8 Initialisation payload and failures

- PLG-068: **The initialisation payload MUST carry two parts: the plugin's own
  resolved configuration and the host context.** `[current]`
- PLG-069: **The host context MUST contain the instance name, MAY contain a data
  directory, and MUST contain the highest capability descriptor version the host
  serves.** `[current]`
- PLG-070: **The host context MUST be permissive and forward-compatible: a plugin
  ignores context fields it does not know, and the host MAY add a field.**
  `[current]`
- PLG-071: **Nothing secret MUST be placed in the host context; the host decides
  what it discloses.** `[current]`
- PLG-072: **A plugin's configuration MUST be resolved by the daemon and
  delivered only through initialisation; a plugin MUST NOT read daemon
  configuration.** `[current]`
- PLG-073: **A failed initialisation MUST be classified by its terminal status: an
  invalid-argument failure means the configuration does not match what the plugin
  declared, and every other terminal status means the plugin is defective.**
  `[current]`
- PLG-074: **On a failed initialisation, the host MUST drop the pending result and
  then destroy the instance, and the instance MUST tolerate destruction after a
  failed initialisation.** `[current]`
- PLG-075: **A plugin's configuration schema MUST be declared in the manifest and
  MUST be the schema of the initialisation payload's configuration part.**
  `[current]`

## 9.9 Bundles and signing

- BND-001: **A plugin MUST ship as exactly one file containing a signed container,
  the manifest, and the plugin's code.** The container format is open (`OQ-107`).
  `[current]`
- BND-002: **The container MUST declare its format version, its signing scheme,
  its signature, and the extent of the content the signature covers, so that a
  truncated file, an unknown format version, and content added after signing are
  each detectable.** `[current]`
- BND-003: **A container shorter or longer than its declared regions MUST be
  refused.** `[current]`
- BND-004: **The signature MUST cover the container's own declared fields, the
  manifest, and the plugin's code, with the signature itself excluded from the
  signed bytes rather than zero-filled.** `[current]`
- BND-005: **Signature verification MUST be strict: a non-canonical signature or
  key encoding and a weak key MUST be refused.** The scheme is open (`OQ-108`).
  `[current]`
- BND-006: **Trust MAY be delegated through a chain of authorisations, leaf first
  and anchor last, consumed exactly with no trailing bytes, and at most eight
  links long.** `[current]`
- BND-007: **Each authorisation MUST be read by a strict, canonical-input decoder
  that refuses non-canonical input before interpreting it.** `[current]`
- BND-008: **Each link other than the anchor MUST be verified against the link
  above it: the child's declared issuer MUST equal the parent's subject, and the
  child's signature MUST verify under the parent's key.** `[current]`
- BND-009: **The anchor MUST be trusted by its recomputed identifier being in the
  host's trust set, not by who signed it.** `[current]`
- BND-010: **An authorisation that delegates trust onward MUST declare that it may
  delegate; the leaf authorisation MUST NOT.** `[current]`
- BND-011: **An extension the host does not understand MUST refuse the chain when
  it is marked critical and MUST be ignored when it is not.** `[current]`
- BND-012: **The leaf authorisation's declared subject name MUST equal the
  plugin's name, and an absent or empty name MUST be refused.** `[current]`
- BND-013: **Each authorisation's validity window MUST be checked against the
  host clock with a bounded, documented tolerance for clock skew.** The tolerance
  is open (`OQ-108`). `[current]`
- BND-014: **A key identifier MUST be derived from the public key by the
  contract's defined function, and every compared identifier MUST be recomputed
  from the key rather than trusted from the artifact.** The derivation is open
  (`OQ-108`). `[current]`
- BND-015: **The trust store MUST register authorities by public key, and no
  revoked identifier may appear anywhere in a chain, the anchor included.**
  `[current]`
- BND-016: **The compiled-in trust set MUST be empty; a daemon starts trusting
  nothing and its trusted authorities and revocations are its own
  configuration.** `[current]`
- BND-017: **A bundle verification refusal MUST be classified as either a trust
  failure or a container refusal.** `[current]`
- BND-018: **A trust failure MUST stop daemon start-up; a container refusal MUST
  disable only that bundle.** `[current]`
- BND-019: **A trust failure MUST include an empty chain, an unknown anchor, a
  revoked key, an expired or not-yet-valid authorisation, a malformed signature, a
  signature that does not verify, a broken issuer and subject link, a link
  signature that does not verify, every chain-profile violation, and a leaf with
  no usable name.** `[current]`
- BND-020: **A container refusal MUST include a short container, an unknown
  container version, an unknown signing scheme, a declared region length that
  contradicts the file, bytes before the first declared region, a malformed or
  non-canonical authorisation, trailing chain bytes, and more than eight
  links.** `[current]`
- BND-021: **The container checks MUST run in this order, and each refusal MUST
  answer for itself rather than for a later consequence: container fields, chain
  region, anchor key identifier, revocation, validity windows, profile, links,
  payload signature.** `[current]`

## 9.10 The archive

- BND-022: **A bundle's payload MUST be one archive whose root carries the
  manifest and whose declared layout carries one code entry directory per
  supported platform.** The archive format is open (`OQ-107`). `[current]`
- BND-023: **Each platform's entry MUST hold exactly one file, and that file MUST
  be this platform's plugin code.** `[current]`
- BND-024: **Unpacking MUST make available only the root manifest and this
  platform's code entry.** `[current]`
- BND-025: **Unpacking MUST refuse a bundle that cannot be read, an entry name
  that is absolute, carries a parent-directory component, is empty after removing
  separators, or carries a null character, a symbolic link, a directory, or any
  entry that is not a regular file, and any entry whose declared access mode
  grants access beyond the owner.** A declared owner-only mode is accepted, and
  so are entries that declare no mode. `[current]`
- BND-026: **The total declared-size limit MUST be evaluated from declared sizes
  without decompressing an entry this host does not make available; only the
  entries it makes available are decompressed and checked.** `[current]`
- BND-027: **Unpacking MUST use one uniquely named private location, create what
  it writes owner-only, and MUST NOT let a declared entry mode reach the
  filesystem.** `[current]`
- BND-028: **A refused unpacking MUST remove what it created before answering,
  and a successful unpacking MUST own that location and remove it when released.**
  `[current]`
- BND-029: **Whether a manifest is present MUST be decided by the manifest step,
  not by unpacking; a bundle with no root manifest is still unpacked.** `[current]`
- BND-030: **A bundle MAY carry several platform entries, and the host MUST load
  only the code for its own platform.** `[current]`
- BND-031: **Unpacking MUST refuse a bundle with more than 256 entries, an
  entry whose declared uncompressed size exceeds 64 MiB, and entries whose
  declared sizes total more than 256 MiB.** `[current]`
- BND-032: **Unpacking MUST refuse an entry that cannot be read, fails its
  checksum, or uses an unsupported compression method, and a bundle that has no
  file or more than one file directly under this host's platform entry.**
  `[current]`
- BND-033: **The host MUST resolve the manifest's declared code path only inside
  the staged bundle and MUST refuse a path outside this host's platform entry.**
  `[current]`
- BND-034: **Unpacking MUST validate every entry whether or not this
  host's platform needs it.** `[current]`

## 9.11 The manifest

- MAN-001: **Every bundle MUST carry the manifest at its archive root.** The
  manifest's format is open (`OQ-109`). `[current]`
- MAN-002: **The manifest MUST declare the plugin identity, the plugin's own
  version, the contract major, the minimum contract minor, one code path per
  platform, the capability list with each capability's name, kind, version,
  request schema, and response schema, and the configuration schema.**
  `[current]`
- MAN-003: **The manifest's declared name MUST equal the name carried by the
  bundle's leaf authorisation.** `[current]`
- MAN-004: **Each code path MUST be declared per platform in the layout the
  bundle format defines.** `[current]`
- MAN-005: **Every schema the manifest carries MUST be expressible as structured
  data, including a schema whose default value is null.** The schema dialect is
  open (`OQ-110`). `[current]`
- MAN-006: **A field the manifest reader does not know MUST be ignored rather than
  refused, so a manifest written by a newer producer stays readable.**
  `[current]`
- MAN-007: **The manifest MUST be validated in this order: presence, readable
  format, name equal to the leaf name, contract demand compatible with this
  build, and code for this host's platform.** `[current]`
- MAN-008: **The contract demand check MUST refuse a differing major and a minimum
  minor newer than this build's minor, and MUST be decided before any plugin code
  is read.** `[current]`
- MAN-009: **The code check MUST judge only this host's platform; a manifest that
  declares other platforms or none MUST be judged by what it declares for this
  host.** `[current]`
- MAN-010: **The host MUST cross-check the manifest's capability names against the
  names the plugin reports.** Only names are cross-checked at load; schemas are
  the manifest's declared text and the plugin's derived schemas. `[current]`
- MAN-011: **The manifest's capability list MUST be what the host trusts for
  discovery, and the plugin's own listing MUST be the cross-check.**
  `[current]`
- MAN-012: **The manifest's schemas and the plugin's derived schemas MUST NOT be
  assumed identical, and a load MUST NOT be refused only because the two differ.**
  `[current]`
- MAN-013: **The plugin's own version MUST be recorded and MUST never gate a
  load (see `PLG-017`).** `[current]`
- MAN-014: **A manifest and plugin that disagree on a capability name MUST
  disable the plugin.** `[current]`

## 9.12 Host lifecycle and start-up

- HST-001: **The host MUST load a bundle through one entry that may complete later and
  that verifies, unpacks, validates the manifest, selects this platform's code, checks
  the contract, initialises, and registers; callers MUST NOT drive those steps
  separately.** `[current]`
- HST-002: **Start-up MUST scan a plugin directory: every regular file directly
  in the directory is one bundle, with no recursion and no discovery file.** A
  directory entry that is not a regular file is skipped. `[current]`
- HST-003: **A bundle's identity before its manifest is read MUST be its file
  name, and the daemon MUST resolve that bundle's configuration under that
  identity.** `[current]`
- HST-004: **Once the manifest is readable, the manifest's name MUST become the
  registration and routing identity.** `[current]`
- HST-005: **A trust failure, a bundle file that cannot be read from the
  filesystem, a plugin directory that cannot be listed, an absent, unreadable,
  invalid, or name-mismatched manifest, and a refused release MUST stop
  start-up.** `[current]`
- HST-006: **A container this build cannot read, an unpacking refusal, a missing
  or ambiguous code entry for this platform, a manifest that declares no code for
  this platform or one the bundle does not carry, a contract mismatch, a
  capability-name disagreement, and a refused registration MUST disable only that
  plugin while the daemon runs.** `[current]`
- HST-007: **A failed initialisation MUST release the instance and replace it with
  a disabled placeholder; a refused release during that recovery MUST stop
  start-up.** `[current]`
- HST-008: **A disabled plugin MUST be listed with its name, the reason, and its
  capabilities when a manifest was readable, or with the bundle's file name and no
  capabilities when one was not.** `[current]`
- HST-009: **A disabled plugin's capabilities MUST be excluded from what an agent
  context is offered.** `[current]`
- HST-010: **Plugins MUST load and initialise at daemon start-up and MUST be
  released at daemon shutdown; there MUST be no live teardown and no voluntary
  deinitialisation of a running plugin.** `[current]`
- HST-011: **There MUST be no live reconfiguration: a plugin MUST NOT observe two
  different configurations in one lifetime (see `DAE-015` and `CFG-005`).**
  `[current]`
- HST-012: **Releasing an instance MUST be one awaited operation that unregisters
  the instance, cancels or drains in-flight calls to the caller's deadline, calls
  the plugin's shutdown operation, drops every pending result, destroys the
  instance, releases the plugin's code, and reports the lifecycle entries it
  added.** `[current]`
- HST-013: **Plugin code MUST be released only when no call, pending result,
  completion handle, or callback can still run in it.** `[current]`
- HST-014: **An instance that cannot be released MUST stay resident and MUST be
  reported; the host MUST NOT release code that may still be running.**
  `[current]`
- HST-015: **The daemon's shutdown MUST be bounded by a deadline; on expiry it
  MUST report which instances could not be released and exit, leaving that code
  resident.** `[current]`
- HST-016: **A plugin whose abandoned query never returns MUST be quarantined: it
  accepts no new pending results, its code stays resident, and its recorded
  lifecycle state stays accurate.** `[current]`
- HST-017: **There MUST be no fixed waiting period for release; quarantine plus a
  reclaim on the abandoned query's return MUST be the mechanism.** `[current]`
- HST-018: **Hot replacement of plugin code while the daemon runs MUST NOT be
  provided.** `[current]`
- HST-019: **The host MUST record an ordered lifecycle ledger per instance, from
  load through call, cancellation, and teardown.** `[current]`
- HST-020: **The host MUST report which plugins it could not release at shutdown,
  so the operator can decide whether to remove one.** `[current]`

## 9.13 Cancellation in the plugin host

- HST-021: **The caller that starts a plugin call MUST own its deadline; the
  runtime MUST hold no timer and no default deadline.** `[current]`
- HST-022: **Cancellation MUST mark the call's token, and the plugin MUST be
  expected to resolve the call as cancelled on its next result query.**
  `[current]`
- HST-023: **Cancelling a request identifier the host does not hold MUST answer
  not found without calling the plugin.** `[current]`
- HST-024: **An unacknowledged cancellation MUST be recorded on the ordering
  ledger, and the instance MUST keep serving.** `[current]`
- HST-025: **Dropping a call MUST release every plugin payload the host still
  holds for that call and MUST cancel that call's outstanding service requests
  before the context is reclaimed, so no completion is delivered into a released
  context.** `[current]`
- HST-026: **A call the plugin resolves as cancelled MUST reach its caller as the
  cancelled status; a call the caller dropped MUST yield nothing to that
  caller.** `[current]`
- HST-027: **Instance-wide cancellation MUST be a host-side pass over the
  instance's own in-flight set, and MUST NOT add a plugin-side operation.**
  `[current]`

## 9.14 The sink

- HST-028: **The host MUST expose one sink destination for progress, events,
  logs, and service requests, set when the host is built.** `[current]`
- HST-029: **A host with no sink MUST be legal, and the default MUST record into
  the host's readable accessors.** `[current]`
- HST-030: **Every sink delivery MUST name the instance; progress and service
  requests MUST also name the request identifier, an event MUST carry its topic
  and payload, and a log MUST carry its level and message.** `[current]`
- HST-031: **A payload MUST reach the sink as an owned copy taken before the
  plugin's payload is released, so the sink never holds plugin-owned state.**
  `[current]`
- HST-032: **A sink method MUST be infallible and MUST NOT block, because it is
  called from inside the plugin's result query.** `[current]`
- HST-033: **An unexpected failure inside a sink MUST be contained at that
  boundary entry and recorded on the ordering ledger.** `[current]`
- HST-034: **Delivery MUST be ordered per instance and in emission order; there
  MUST be no ordering relationship between callbacks from different instances,
  and no promised position relative to a call's completion.** `[current]`
- HST-035: **The runtime MUST NOT keep a queue on a sink's behalf; a host that
  wants backpressure MUST put its own queue behind the sink.** `[current]`
- HST-036: **A sink MUST NOT call back into the runtime or the plugin.**
  `[current]`
- HST-037: **A service request MUST be accepted or refused synchronously by the
  sink and answered later through the runtime's service-completion operation.**
  `[current]`
- HST-038: **The host MUST strip the payload's declared format version before
  delivering a body to the sink.** `[current]`

## 9.15 Error surface and evidence obligations

- HST-039: **Load and dispatch failures, routing and validation failures, and
  registration failures MUST be distinct typed errors, and staleness, contract
  mismatch, and trust failure MUST be three distinct typed refusals.**
  `[current]`
- HST-040: **The host's bundle reader MUST be exercised against committed fixtures
  that cover every refusal class, so a change to the reader is checked against
  them.** `[current]`
- HST-041: **A reference plugin MUST be packaged into a signed bundle and loaded
  end to end through the same host entry as any other bundle, so the contract is
  proved by a real load rather than by a seam alone.** `[current]`
- HST-042: **The test signing key MUST NOT be a compiled-in trust anchor, and the
  tests that use it MUST install it explicitly.** `[current]`
- HST-043: **The boundary hand-offs SHOULD be asserted correct at build time
  where the toolchain can prove the property; where it cannot, a runtime check
  MUST cover the same property.** Which properties are worth proving, and with
  what, is open (`OQ-112`). Disregarding this recommendation leaves the boundary
  checked only at each boundary entry. This is a verification obligation, not a
  substitute for the handling at each entry. `[current]`

## 9.16 Plugin authoring SDK, core plugins, and CLI

- SDK-001: **The application MUST ship a first-class authoring SDK for the
  language its own core plugins are written in, because those plugins exercise
  the SDK rather than reaching past it.** The authoring language and the SDK's
  distribution are open (`OQ-057`). `[planned]`
- SDK-002: **The SDK MUST add the serialisation and deferred-completion helpers a
  plugin needs, and MUST keep them out of the host runtime's dependency
  footprint.** `[planned]`
- SDK-003: **SDK functionality MUST be separated so that export helpers,
  serialisation, and deferred-completion helpers do not weigh down the host
  runtime.** `[planned]`
- SDK-004: **The SDK MUST NOT change the boundary contract, and the host runtime
  MUST be usable without the SDK.** `[planned]`
- SDK-005: **The authored plugin surface MUST offer one fallible initialisation
  that may complete later and that constructs and initialises the plugin, so a
  plugin value that has not completed initialisation is not representable.**
  `[planned]`
- SDK-006: **The authored plugin surface MUST offer one capability-name-routed
  call, so the number of capabilities a plugin can declare is not bounded by the
  authoring surface.** `[planned]`
- SDK-007: **The SDK MUST own the completion-handle bridge between the host and
  the language's own concurrency abstraction, the pending-result helper, unexpected
  failure containment at query and drop, consuming the ready payload on the query
  return, cancellation mechanics, schema derivation, typed facades, and
  serialisation helpers.** `[planned]`
- SDK-008: **Per-capability typed facades MAY be generated from schemas; they MUST
  NOT become a constraint on the core authoring surface.** `[planned, form open]`
- SDK-009: **A set of core plugins MUST ship with Metis and MUST exercise the SDK
  rather than reaching past it.** `[planned, form open]`
- SDK-010: **Core plugins MUST use only the SDK and the host context they are
  given; they MUST NOT read daemon configuration or reach around the plugin
  interface.** `[planned]`
- SDK-011: **A plugin CLI MUST scaffold a new plugin, build it for the supported
  platforms, package it into a bundle, and install it into the daemon.**
  `[planned, form open]`
- SDK-012: **The CLI MUST produce only the bundle format the host accepts; it MUST
  NOT invent a second format.** `[planned]`
- SDK-013: **Other languages MUST be able to implement the boundary contract
  directly; the SDK is one authoring path, not a requirement.** `[current]`
- SDK-014: **The manifest MUST be generated from the same source as the plugin's
  own capability declaration where the SDK is used, and the hand-authored
  equivalent MUST stay documented and practical.** `[current]`
- SDK-015: **The proposal expects the SDK to validate at build or package time
  that every declared toolset member names a capability the plugin declares.**
  This is proposal material and not an accepted requirement until the tool
  disclosure proposal is accepted. `[proposed]`
- SDK-016: **Under the proposal, a declared toolset grants no permission and
  changes no dispatch behavior.** This is proposal material and not an accepted
  requirement until the tool disclosure proposal is accepted. `[proposed]`

## 9.17 Out of scope for the plugin system

- Plugins provide tools, model providers, and hooks. They do not define workflow
  step types.
- Sandboxing, resource limits, and any claim that a plugin is safe to run are out
  of scope. The boundary contract is not a sandbox.
- Hot replacement of plugin code is out of scope.
