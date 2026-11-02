# 9. Plugins: contract, bundles, manifests, host lifecycle, and authoring

The plugin system is an external compatibility contract in two directions. A
third-party author writes a plugin against the ABI and the bundle format, and an
operator installs the resulting file. The contract text exists so that a second
implementation can be written from the text alone. For that reason the ABI
operation semantics, the bundle bytes, the manifest fields, and the start-up
outcomes are requirements of the finished application, not implementation
choices.

## 9.1 Plugin model and boundary

- PLG-001: **A plugin MUST be a native shared library that the daemon loads into
  its own process.** `[current]`
- PLG-002: **A plugin MUST advertise its capabilities before the host serves any
  call to it.** `[current]`
- PLG-003: **A plugin MUST supply capabilities; it MUST NOT export workflow
  steps.** `[current]`
- PLG-004: **A plugin MUST NOT require its own asynchronous runtime, executor, or
  worker thread; the host MUST own when plugin code runs.** `[current]`
- PLG-005: **A plugin that must wait for a timer or I/O MUST request the facility
  from the host, return a pending poll, and wake the host when the answer
  arrives; it MUST NOT block its poll.** `[current]`
- PLG-006: **A plugin MUST NOT be treated as sandboxed. Running a plugin is native
  code execution in the daemon process, and a plugin that blocks during
  initialisation hangs the daemon.** `[current]`
- PLG-007: **Language-specific values, trait objects, futures, strings, vectors,
  runtime handles, and channel types MUST NOT cross the ABI boundary as such; only
  the defined primitive representations may cross.** `[current]`
- PLG-008: **A high-level authoring type in any language MUST remain internal to
  each compiled plugin; the export adapters MUST translate an implementation into
  the ABI.** `[current]`
- PLG-009: **Only pointers, integers, function pointers, byte buffers, and
  explicitly defined structures MUST cross the boundary.** `[current]`

## 9.2 ABI versioning and tables

- PLG-010: **A plugin MUST export exactly one entry symbol, named
  `metis_plugin_entry_v1`, which returns a pointer to the plugin's function
  table.** `[current]`
- PLG-011: **The ABI major version MUST be part of the entry symbol, so a library
  built for another major is refused before any table entry is read.**
  `[current]`
- PLG-012: **Both the plugin table and the host table MUST be size-prefixed, with
  the size as the first field, and presence of an entry MUST be decided by the
  reported size, never by whether a pointer looks non-null.** `[current]`
- PLG-013: **Within one ABI major, the table MUST be append-only, so a plugin
  built against a newer minor is read up to the entries this host knows, decided
  by its reported size.** `[current]`
- PLG-014: **The current ABI is major 1, minor 2; a reader MUST tolerate the minor
  it knows and any newer minor in the same major, and MUST refuse an older
  minor.** `[current]`
- PLG-015: **The manifest MUST declare the ABI major and the minimum ABI minor
  the plugin requires, with no range syntax, and the host MUST refuse a differing
  major or a host minor lower than the required minimum.** `[current]`
- PLG-016: **A capability whose descriptor version the host does not support MUST
  be refused on its own, and the rest of the plugin MUST keep serving.**
  `[current]`
- PLG-017: **The plugin's own version MUST be recorded and MUST never gate a
  load.** `[current]`
- PLG-018: **A stale pre-release plugin whose table order does not match this
  build's MUST be refused by the minor comparison rather than misread.**
  `[current]`

## 9.3 Plugin table operations

- PLG-019: **The plugin table MUST provide asynchronous initialisation, instance
  destruction, capability listing, one generic invocation, cancellation, buffer
  release, shutdown, and service completion.** `[current]`
- PLG-020: **Initialisation MUST construct and initialise an instance in one
  fallible asynchronous operation.** It receives the host table and one
  host-owned payload carrying the plugin's configuration and the host context.
  `[current]`
- PLG-076: **On success, initialisation MUST return the instance and a
  plugin-owned future, and the host MUST NOT register or route to the instance
  until that future resolves successfully.** `[current]`
- PLG-021: **There MUST NOT be a separate instance-creation entry.**
  `[current]`
- PLG-022: **The invocation entry MUST receive the instance, the capability name,
  a host-chosen request identifier, the request payload, and the host table, and
  on success MUST return a plugin-owned future.** The request payload is valid
  for that call only. `[current]`
- PLG-023: **Cancellation MUST mark the cancellation token of the call identified
  by the request identifier.** `[current]`
- PLG-024: **Buffer release MUST be the only way the host frees plugin-allocated
  memory, and the host MUST never free plugin memory directly.** `[current]`
- PLG-025: **Shutdown MUST quiesce the instance; the host MUST drop every future
  before calling shutdown, so nothing may arrive after it returns.**
  `[current]`
- PLG-026: **Service completion MUST answer an earlier service request with the
  same request identifier, and the plugin MUST wake the waiting future from
  it.** `[current]`
- PLG-027: **Instance destruction MUST be called only after shutdown.**
  `[current]`
- PLG-028: **The table MUST NOT expose a counters entry or any plugin-side metric
  surface.** `[current]`

## 9.4 Host table and callbacks

- PLG-029: **The host table MUST carry a host-state pointer and exactly the
  callbacks progress, event emission, logging, and service request.**
  `[current]`
- PLG-030: **The host MUST NOT provide a completion callback; a call completes
  when a poll returns success.** `[current]`
- PLG-031: **Progress MUST carry the request identifier and a payload.**
  `[current]`
- PLG-032: **Event emission MUST carry a topic and a payload and MUST NOT carry a
  request identifier; a plugin that needs correlation MUST put an identifier in
  its payload.** `[current]`
- PLG-033: **Logging MUST carry a level in the range 0 to 3, meaning debug, info,
  warn, and error, and a message.** `[current]`
- PLG-034: **A service request MUST carry the facility name, the request
  identifier, and a plugin-owned payload, and the host MUST answer it later
  through service completion.** The current facility is a timer; I/O readiness is
  planned. `[current]`
- PLG-035: **Every host callback MUST be panic-guarded inside its own frame, so a
  panic becomes a recorded internal failure instead of unwinding across the
  boundary.** `[current]`

## 9.5 Futures, wakers, and status

- PLG-036: **A poll MUST return exactly one of: success with a plugin-allocated
  response buffer, pending, or a terminal status.** `[current]`
- PLG-037: **Only the host MUST poll a future, one poll at a time; two concurrent
  polls on one handle are a protocol violation.** `[current]`
- PLG-038: **A poll that blocks MUST be detected by the host, recorded, and MUST
  quarantine the plugin.** `[current]`
- PLG-039: **The host MUST drop a future exactly once, and never while a poll is
  in progress.** `[current]`
- PLG-040: **The host MUST pass a waker that is one owning reference for that
  poll, MUST release it when the poll returns, and the plugin MUST NOT release
  it.** `[current]`
- PLG-041: **A plugin MAY clone the waker, each clone is one owning reference, and
  the plugin MUST drop every clone.** `[current]`
- PLG-042: **One clone MUST be consumed by exactly one wake or drop; a second use
  of a consumed clone MUST be refused rather than release state twice.**
  `[current]`
- PLG-043: **A wake MAY arrive from any thread, MUST be cheap and non-blocking,
  and MAY arrive after the future was dropped; the host MUST detect and record
  such a wake and MUST NOT dereference freed state.** `[current]`
- PLG-044: **A recorded late wake MUST name the request identifier of the call it
  belongs to.** `[current]`
- PLG-077: **A panic inside an exported poll or drop MUST become a terminal
  internal status, drop MUST remain legal afterwards, and no boundary path may
  reach process abort.** `[current]`
- PLG-045: **The status space MUST be frozen to: ok, invalid argument, not found,
  permission denied, cancelled, internal, unsupported, and pending.** Higher
  layers add meaning rather than values. `[current]`

## 9.6 Buffers, payloads, and ownership

- PLG-046: **A buffer MUST carry a pointer, length, capacity, and one of the
  owners none, host, or plugin.** `[current]`
- PLG-047: **Each side MUST free only what it allocated; the non-owning side MUST
  never free the buffer.** `[current]`
- PLG-048: **A response buffer returned by a successful poll MUST be
  plugin-allocated and MUST be freed by the host through the plugin's buffer
  release entry.** `[current]`
- PLG-049: **A request payload MUST be host-owned and valid only for the call.**
  `[current]`
- PLG-050: **A service-request payload MUST be plugin-owned and MUST be freed by
  the host through the plugin's buffer release entry.** `[current]`
- PLG-051: **A service-completion payload and an initialisation payload MUST be
  host-owned and borrowed for that call; a plugin that needs one afterwards MUST
  copy it.** `[current]`
- PLG-052: **Buffer release MUST be reentrant; the host MAY call it from inside a
  service request.** `[current]`
- PLG-053: **Every payload MUST carry the one-byte envelope: four bits of payload
  format version, one codec bit, and three reserved bits that MUST be zero.**
  `[current]`
- PLG-054: **The current payload format version is 1 and the only supported codec
  is JSON; an unknown codec MUST be refused as unsupported, while a bad format
  version or non-zero reserved bits MUST be refused as invalid argument.**
  `[current]`
- PLG-055: **Payload evolution within an ABI major MUST be additive: unknown
  fields are ignored and new fields are optional.** `[current]`
- PLG-056: **The payload envelope constants and the per-entry slot rules MUST be
  published in a C header for plugin implementers, so an implementer in another
  language does not have to guess them.** `[current]`

## 9.7 Capabilities and routing

- PLG-057: **A capability MUST have a name, a kind, a version, a request schema,
  and a response schema.** `[current]`
- PLG-058: **Capability names MUST be instance-qualified as
  `<instance>.<tool>` at the host boundary.** `[current]`
- PLG-059: **An instance name that itself contains a dot MUST route like any
  other instance name; the tool half MUST come from the routing owner rather than
  from a first-dot split.** `[current]`
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
  leaves the instance, its in-flight calls, and its library alone.** `[current]`
- PLG-065: **Enabling MUST accept calls again without tearing anything down.**
  `[current]`
- PLG-066: **The host MUST refuse a call to a capability the plugin did not
  declare.** `[current]`
- PLG-067: **The host MUST refuse a call to a disabled plugin with a distinct
  disabled refusal that carries the reason, so a caller can tell a disabled
  plugin from one that never loaded.** `[current]`

## 9.8 Initialisation payload and failures

- PLG-068: **The initialisation payload body MUST be one JSON object with two
  parts: `config`, the plugin's own resolved configuration, and `host_context`,
  what the host discloses about itself.** `[current]`
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
- PLG-074: **On a failed initialisation, the host MUST drop the future and then
  destroy the instance, and the instance MUST tolerate destruction after a failed
  initialisation.** `[current]`
- PLG-075: **A plugin's configuration schema MUST be declared in the manifest and
  MUST be the schema of the initialisation payload's `config` part.**
  `[current]`

## 9.9 Bundles and signing

- BND-001: **A plugin MUST ship as exactly one file with the extension `.metis`,
  containing a fixed-size container header, an X.509 certificate chain over
  Ed25519 keys, a payload signature, and a ZIP archive.** `[current]`
- BND-002: **The container header MUST be exactly 40 bytes and MUST contain, in
  order and big-endian: an 8-byte magic `METISBND`, a 2-byte container format
  version equal to 1, a 2-byte signing suite equal to 1 for X.509 over Ed25519, a
  2-byte header length equal to 40, a 2-byte reserved field equal to zero, a
  4-byte chain length, an 8-byte signature offset, a 4-byte signature length, and
  an 8-byte ZIP length.** `[current]`
- BND-003: **The signature offset MUST equal the header length plus the chain
  length, and the file length MUST equal the signature offset plus the signature
  length plus the ZIP length.** `[current]`
- BND-004: **The signed message MUST be, in order, the 27-byte domain tag
  `METIS-PLUGIN-BUNDLE-PAYLOAD`, the 40 header bytes, the chain bytes, and the ZIP
  bytes, with the signature field deleted rather than zero-filled.** The domain
  tag MUST NOT carry the container version. `[current]`
- BND-005: **The signature MUST be raw Ed25519 of 64 octets, made by the leaf
  key, and verification MUST be strict: non-canonical key or signature encodings
  and small-order keys MUST be refused.** `[current]`
- BND-006: **The chain region MUST be a concatenation of whole DER X.509 version
  3 certificates, leaf-authorising certificate first and anchor last, MUST be
  consumed exactly with no trailing bytes, and MUST hold at most eight links.**
  `[current]`
- BND-007: **The chain region MUST be read by a strict DER decoder that refuses
  non-canonical input before interpreting it.** `[current]`
- BND-008: **Each link other than the anchor MUST be verified against the link
  above it: the child's issuer name equal to the parent's subject name and the
  child's signature verified under the parent's key.** `[current]`
- BND-009: **The anchor MUST be trusted by its recomputed key identifier being in
  the host's trust store, not by who signed it.** `[current]`
- BND-010: **A certificate that authorises another key MUST carry basic
  constraints with CA true; the leaf-authorising certificate MUST be an end
  entity.** `[current]`
- BND-011: **Basic constraints MUST be the only understood extension: any other
  critical extension MUST refuse the chain, and any other non-critical extension
  MUST be ignored.** `[current]`
- BND-012: **The leaf-authorising certificate's common name MUST equal the
  plugin's name, and an absent or empty common name MUST be refused.**
  `[current]`
- BND-013: **Each certificate's validity window MUST be checked at second
  granularity against the host clock with at most 60 seconds of skew at each
  end.** `[current]`
- BND-014: **A key identifier MUST be the first 16 bytes of the SHA-256 of the
  raw 32-byte public key, and every compared identifier MUST be recomputed from
  the public key rather than trusted from a certificate.** `[current]`
- BND-015: **The trust store's roots MUST be registered by public key, and no key
  identifier in the revocation list may appear anywhere in a chain, the anchor
  included.** `[current]`
- BND-016: **The compiled-in root set MUST be empty; a daemon starts trusting
  nothing and its roots and revocations are its own configuration.** `[current]`
- BND-017: **A bundle verification refusal MUST be classified as either a trust
  failure or a container refusal.** `[current]`
- BND-018: **A trust failure MUST stop daemon start-up; a container refusal MUST
  disable only that bundle.** `[current]`
- BND-019: **A trust failure MUST include an empty chain, an unknown root, a
  revoked key, an expired or not-yet-valid certificate, a malformed signature, a
  signature that does not verify, a broken issuer and subject link, a link
  signature that does not verify, every chain-profile violation, and a leaf with
  no usable common name.** `[current]`
- BND-020: **A container refusal MUST include a short file, bad magic, an unknown
  container version, an unknown signing suite, a bad header length, a non-zero
  reserved field, bytes before the header, a span mismatch, a malformed or
  non-canonical certificate, trailing chain bytes, and more than eight links.**
  `[current]`
- BND-021: **The container checks MUST run in this order, and each refusal MUST
  answer for itself rather than for a later consequence: container fields, chain
  region, anchor key identifier, revocation, validity windows, profile, links,
  payload signature.** `[current]`

## 9.10 The archive

- BND-022: **A bundle's archive MUST be a ZIP file whose root carries
  `manifest.toml` and whose `lib/` directory holds one directory per target named
  exactly `<os>-<arch>`.** `[current]`
- BND-023: **A target directory MUST hold exactly one file, and that file MUST be
  the target's library.** `[current]`
- BND-024: **Extraction MUST stage only the root manifest and this host's target
  directory.** `[current]`
- BND-034: **Extraction MUST validate every archive entry whether or not this
  host's target needs it.** `[current]`
- BND-025: **Extraction MUST refuse a ZIP that cannot be read, an entry name that
  is absolute, carries a `..` component, is empty after removing separators, or
  carries a null byte, a symbolic link, directory, hard link, device, socket,
  FIFO, or any other non-regular file entry, and a declared mode other than
  `0600`.** A declared `0600` with or without the regular-file type bit is
  accepted, and so are zero external attributes, which declare no mode.
  `[current]`
- BND-031: **Extraction MUST refuse an archive with more than 256 entries, an
  entry whose declared uncompressed size exceeds 64 MiB, and entries whose
  declared sizes total more than 256 MiB.** `[current]`
- BND-032: **Extraction MUST refuse a staged entry that cannot be read, fails its
  checksum, or uses an unsupported compression method, and an archive that has no file or more than one file directly under this host's
  target directory (see `BND-023`).**
  `[current]`
- BND-026: **The total declared-size limit MUST be evaluated from declared sizes
  without decompressing an entry this host does not stage; only staged entries'
  bytes are decompressed and checksummed.** `[current]`
- BND-027: **Extraction MUST stage into one uniquely named directory, create
  staged directories at mode 700 and staged files at mode 600, and MUST NOT let a
  declared entry mode reach the filesystem.** `[current]`
- BND-028: **A refused extraction MUST remove its staging directory before
  answering, and a successful extraction MUST own its staging directory and
  remove it when released.** `[current]`
- BND-029: **Whether a manifest is present MUST be decided by the manifest step,
  not by extraction; a bundle with no root manifest still stages.** `[current]`
- BND-030: **A bundle MAY carry several targets, and the host MUST load only the
  library for its own target.** `[current]`
- BND-033: **The host MUST resolve a manifest's declared library path only inside
  the staged bundle and MUST refuse a path that is not of the form
  `lib/<target>/<file name>` for this host's target.** `[current]`

## 9.11 The manifest

- MAN-001: **Every bundle MUST carry `manifest.toml` at its archive root.**
  `[current]`
- MAN-002: **The manifest MUST declare the plugin identity, the plugin's own
  version, the ABI major, the minimum ABI minor, one library path per target, the
  capability list with each capability's name, kind, version, request schema, and
  response schema, and the configuration schema.** `[current]`
- MAN-003: **The manifest's declared name MUST equal the common name carried by
  the bundle's leaf-authorising certificate.** `[current]`
- MAN-004: **Each library path MUST be declared per target as
  `lib/<target>/<file name>`.** `[current]`
- MAN-005: **A JSON schema MUST be written as the equivalent plain-text structured
  data; the current format is TOML, whose lack of null means a schema that needs
  a null default cannot express it in that field and MUST use another
  construct.** `[current]`
- MAN-006: **A field the manifest reader does not know MUST be ignored rather than
  refused, so a manifest written by a newer producer stays readable.**
  `[current]`
- MAN-007: **The manifest MUST be validated in this order: presence, readable
  format, name equal to the leaf common name, ABI demand compatible with this
  build, and a library for this host's target.** `[current]`
- MAN-008: **The ABI demand check MUST refuse a differing major and a minimum
  minor newer than this build's minor, and MUST be decided before any library
  byte is read.** `[current]`
- MAN-009: **The library check MUST judge only this host's target; a manifest
  that declares other targets or none MUST be judged by what it declares for this
  host.** `[current]`
- MAN-010: **The host MUST cross-check the manifest's capability names against the
  names the library reports.** Only names are cross-checked at load; schemas are
  the manifest's declared text and the library's derived schemas. `[current]`
- MAN-014: **A manifest and library that disagree on a capability name MUST
  disable the plugin.** `[current]`
- MAN-011: **The manifest's capability list MUST be what the host trusts for
  discovery, and the library's own listing MUST be the cross-check.**
  `[current]`
- MAN-012: **The manifest's schemas and the library's derived schemas MUST NOT be
  assumed identical, and a load MUST NOT be refused only because the two differ.**
  `[current]`
- MAN-013: **The plugin's own version MUST be recorded and MUST never gate a
  load (see `PLG-017`).** `[current]`

## 9.12 Host lifecycle and start-up

- HST-001: **The host MUST load a bundle through one asynchronous entry that
  verifies, extracts, validates the manifest, selects the target library, checks
  the ABI, initialises, and registers; callers MUST NOT drive those steps
  separately.** `[current]`
- HST-002: **Start-up MUST scan a plugin directory: every regular file directly
  in the directory is one bundle, named by its file stem, with no recursion and
  no discovery file.** A directory entry that is not a regular file is skipped.
  `[current]`
- HST-003: **A bundle's identity before its manifest is read MUST be its file's
  stem, and the daemon MUST resolve that bundle's configuration under that
  identity.** `[current]`
- HST-004: **Once the manifest is readable, the manifest's name MUST become the
  registration and routing identity.** `[current]`
- HST-005: **A trust failure, a bundle file that cannot be read from the
  filesystem, a plugin directory that cannot be listed, an absent, unreadable,
  invalid, or name-mismatched manifest, and a refused unload MUST stop
  start-up.** `[current]`
- HST-006: **A container this build cannot read, an extraction refusal, a missing
  or ambiguous target library, a manifest that declares no library for this host
  or one the bundle does not carry, an ABI mismatch, a capability-name
  disagreement, and a refused registration MUST disable only that plugin while
  the daemon runs.** `[current]`
- HST-007: **A failed initialisation MUST unload the instance and replace it with
  a disabled placeholder; a refused unload during that recovery MUST stop
  start-up.** `[current]`
- HST-008: **A disabled plugin MUST be listed with its name, the reason, and its
  capabilities when a manifest was readable, or with the bundle file stem and no
  capabilities when one was not.** `[current]`
- HST-009: **A disabled plugin's capabilities MUST be excluded from what an agent
  context is offered.** `[current]`
- HST-010: **Plugins MUST load and initialise at daemon start-up and MUST be
  released at daemon shutdown; there MUST be no live teardown and no voluntary
  deinitialisation of a running plugin.** `[current]`
- HST-011: **There MUST be no live reconfiguration: a plugin MUST NOT observe two
  different configurations in one lifetime (see `DAE-015` and `CFG-005`).**
  `[current]`
- HST-012: **Releasing an instance MUST be one awaited entry that unregisters the
  instance, cancels or drains in-flight calls to the caller's deadline, calls the
  plugin's shutdown entry, drops every future, destroys the instance, unloads the
  library, and reports the lifecycle entries it added.** `[current]`
- HST-013: **A library MUST be unloaded only when nothing is inside plugin code
  and every future and waker clone has been dropped.** `[current]`
- HST-014: **An instance that cannot be released MUST stay mapped and MUST be
  reported; the host MUST NOT unmap a library that may still be running.**
  `[current]`
- HST-015: **The daemon's shutdown MUST be bounded by a deadline; on expiry it
  MUST report which instances could not be released and exit, leaving those
  libraries mapped.** `[current]`
- HST-016: **A plugin whose abandoned poll never returns MUST be quarantined: it
  accepts no new futures, its library stays resident, and its recorded lifecycle
  state stays accurate.** `[current]`
- HST-017: **There MUST be no fixed waiting period for unloading; quarantine plus
  a reclaim on the abandoned poll's return MUST be the mechanism.**
  `[current]`
- HST-018: **Hot unloading, meaning replacing a mapped library while the daemon
  runs, MUST NOT be provided.** `[current]`
- HST-019: **The host MUST record an ordered lifecycle ledger per instance, from
  load through call, cancellation, and teardown.** `[current]`
- HST-020: **The host MUST report which plugins it could not release at shutdown,
  so the operator can decide whether to remove one.** `[current]`

## 9.13 Cancellation in the plugin host

- HST-021: **The caller that starts a plugin call MUST own its deadline; the
  runtime MUST hold no timer and no default deadline.** `[current]`
- HST-022: **Cancellation MUST mark the call's token through the frozen entry, and
  the plugin MUST be expected to resolve the call as cancelled on its next
  poll.** `[current]`
- HST-023: **Cancelling a request identifier the host does not hold MUST answer
  not found without calling the plugin.** `[current]`
- HST-024: **An unacknowledged cancellation MUST be recorded on the ordering
  ledger, and the instance MUST keep serving.** `[current]`
- HST-025: **Dropping a call MUST release every plugin buffer the host still holds
  for that call and MUST cancel that call's outstanding service requests before
  the context is reclaimed, so no completion is delivered into a released
  context.** `[current]`
- HST-026: **A call the plugin resolves as cancelled MUST reach its caller as the
  cancelled status; a call the caller dropped MUST yield nothing to that
  caller.** `[current]`
- HST-027: **Instance-wide cancellation MUST be a host-side pass over the
  instance's own in-flight set, and MUST NOT add a plugin table entry.**
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
  plugin's buffer is released, so the sink never holds a reference into plugin
  memory.** `[current]`
- HST-032: **A sink method MUST be infallible and MUST NOT block, because it is
  called inside the plugin's poll frame.** `[current]`
- HST-033: **A panic inside a sink MUST be caught in its own frame and recorded on
  the ordering ledger.** `[current]`
- HST-034: **Delivery MUST be ordered per instance and in emission order; there
  MUST be no ordering relationship between callbacks from different instances,
  and no promised position relative to a call's completion.** `[current]`
- HST-035: **The runtime MUST NOT keep a queue on a sink's behalf; a host that
  wants backpressure MUST put its own queue behind the sink.** `[current]`
- HST-036: **A sink MUST NOT call back into the runtime or the plugin.**
  `[current]`
- HST-037: **A service request MUST be accepted or refused synchronously by the
  sink and answered later through the runtime's service-completion entry.**
  `[current]`
- HST-038: **The host MUST strip the payload envelope before delivering a body to
  the sink.** `[current]`

## 9.15 Error surface and evidence obligations

- HST-039: **Load and dispatch failures, routing and validation failures, and
  registration failures MUST be distinct typed errors, and staleness, ABI
  mismatch, and trust failure MUST be three distinct typed refusals.**
  `[current]`
- HST-040: **A bundle's bytes MUST be checked against committed test vectors so a
  second implementation can be validated against the same rules.**
  `[current]`
- HST-041: **A reference plugin MUST be packaged into a signed bundle and loaded
  end to end through the same host entry as any other bundle, so the format is
  proved by a real load rather than by a library seam alone.** `[current]`
- HST-042: **The test signing key MUST NOT be a compiled-in trust root, and the
  tests that use it MUST install it explicitly.** `[current]`
- HST-043: **The host-facing hand-off frames SHOULD be asserted panic-free at
  build time where the toolchain can prove it.** Disregarding this recommendation
  leaves unwinding across the boundary checked only by per-frame catches. This is
  a verification obligation, not a substitute for the per-frame catch.
  `[current]`

## 9.16 Plugin authoring SDK, core plugins, and CLI

- SDK-001: **The application MUST ship a first-class authoring SDK for the
  language its own core plugins are written in, because those plugins exercise
  the SDK rather than reaching past it.** The current language is Rust. The C ABI
  remains the language-neutral integration surface. `[planned]`
- SDK-002: **The SDK MUST add the serialisation and asynchronous scheduling
  helpers a plugin needs, and MUST keep them out of the host runtime's dependency
  footprint.** `[planned]`
- SDK-003: **SDK functionality MUST be separated so that export helpers,
  serialisation, and async scheduling do not weigh down the host runtime.**
  `[planned]`
- SDK-004: **The SDK MUST NOT change the ABI, and the host runtime MUST be usable
  without the SDK.** `[planned]`
- SDK-005: **The authored plugin surface MUST offer one asynchronous, fallible
  initialisation that constructs and initialises the plugin, so a plugin value
  that has not completed initialisation is not representable.** `[planned]`
- SDK-006: **The authored plugin surface MUST offer one capability-name-routed
  call, so the number of capabilities a plugin can declare is not bounded by the
  authoring surface.** `[planned]`
- SDK-007: **The SDK MUST own the waker bridge between the host waker and the
  language's future abstraction, the boxed-future helper, panic containment at
  poll and drop, consuming the ready buffer on the poll return, cancellation
  mechanics, schema derivation, typed facades, and serialisation helpers.**
  `[planned]`
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
- SDK-013: **Other languages MUST be able to implement the C ABI directly; the SDK
  is one authoring path, not a requirement.** `[current]`
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
  of scope. The ABI is not a sandbox.
- Hot unloading is out of scope.
