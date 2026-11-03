# 12. External interfaces and compatibility constraints

This section states what an implementation must keep stable because an operator, a
surface, or a plugin author depends on it. It names no spelling. Every name, path,
format, and encoding below is a decision the implementation makes and records; the
decision is opened in `15-open-questions.md`, and the requirement here is the
property the decision must preserve. A different implementation can choose
differently and still satisfy this specification.

Two rules govern every item in this section:

- The implementation MUST document each of these surfaces where an operator, a
  plugin author, or a surface author can read it.
- Changing a documented surface is a compatibility change. It MUST be introduced
  as a new version or through a documented migration, never by silent drift
  (`CMP-001` to `CMP-006`).

## 12.1 Operator-facing settings

The daemon MUST expose one setting for each of these meanings. The setting names
are open (`OQ-116`).

| Meaning | Required behavior |
| --- | --- |
| Local endpoint override | Wins over the platform default; absent or empty means the default. |
| Local endpoint group | Names an existing group; absent means owner-only access and no group. |
| Remote bind address | An address literal and a port; absent or empty disables the listener and there is no default. |
| Trusted proxies | Exact proxy addresses; absent or empty trusts no proxy. |
| Pairing secret | A secret reference; absent or empty closes pairing. |
| State location | A dedicated location; absent or empty means the platform's per-user data location. |

A secret value MUST be resolvable from a literal, an environment reference, a
platform credential-store reference, or a permission-checked file reference
(`CFG-006`). The syntax that expresses these forms is open (`OQ-117`). The
implementation MUST document the authoritative form; an earlier form is
superseded rather than silently accepted (`CMP-005`).

## 12.2 Locations and access rules

| Item | Rule |
| --- | --- |
| Local endpoint | A private, per-user runtime location. No shared or world-accessible location is acceptable. |
| Local endpoint access | Owner-only, or owner-and-group when a group is configured. Where the endpoint is a filesystem object, a pre-existing path that is a symbolic link, is not a directory, is owned by another user, or is wider than the rule is refused, never repaired (`NET-006`). |
| Persistent state | One durable store inside a per-user data location, restricted to the daemon's user (`CFG-011`, `CFG-012`). |
| Client credential file | Owner-only, inside an owner-only directory, where the platform expresses access as permissions (`AUT-035`). |
| Plugin directory | A directory whose regular files are bundles, one bundle per file, scanned without recursion (`HST-002`). Its path is open (`OQ-020`, `OQ-095`). |

The exact default paths and their spellings are open (`OQ-116`). A default path
that leaves the user's private area is a defect, not a choice.

## 12.3 Call identity and refusal carriers

Every client call MUST carry the surface identity declaration of `IDN-001`, and
every authenticated call MUST carry its credential with the call rather than in
its address, in anything a proxy records, or in a log line (`API-017`, `AUT-027`,
`AUT-028`, `INV-030`).

The contract MUST define, for each side of a call:

| Meaning | Requirement |
| --- | --- |
| Protocol version | Sent by the client, checked by the daemon, refused when absent, malformed, or unsupported (`IDN-002`). |
| Supported versions | Named in a version refusal so the client reads them without parsing message text (`IDN-003`). |
| Surface name and client version | Descriptive, recorded for diagnostics, never an authorization (`IDN-005`, `IDN-006`). |
| Credential | Present on every authenticated call, including the event stream (`AUT-027`). |
| Request identifier | Optional, opaque, echoed verbatim, bounded as `CAN-002` requires (`CAN-001`, `CAN-004`, `CAN-005`). |
| Refusal cause | Machine-readable, so a client maps it to a typed error without parsing message text (`API-007`, `API-008`). |

The carrier names and the encoding of these values are open (`OQ-118`).

## 12.4 Event contract

| Item | Rule |
| --- | --- |
| Channels | One covering daemon-wide events, one per session, one per task, one per workflow run. Naming is open (`OQ-119`). |
| Event fields | Identifier, kind, channel, payload (`RTS-008`). |
| Kinds | Unspecified, snapshot, heartbeat, ordinary event (`RTS-010`). |
| Identifier | Daemon-wide monotonic integer starting at 1 (`RTS-009`). |
| Cursor | Carried on the stream request; zero means no cursor (`RTS-003`). |
| Payload | Opaque to the stream machinery; its encoding is a versioned contract change (`RTS-014`). |

Changing the payload encoding or a channel name is a breaking contract change
governed by `CMP-004`. The payload encoding is open (`OQ-119`).

## 12.5 Plugin boundary contract

The boundary is external because a plugin author in any language can implement it
(`SDK-013`). Its concrete shape is open (`OQ-104`); its observable rules are:

| Item | Rule |
| --- | --- |
| Entry | Exactly one entry point, through which the host obtains every other operation (`PLG-010`). |
| Compatibility | The major is decided before anything else is used; within a major the operation set is append-only and a newer minor is tolerated while an older one is refused (`PLG-011` to `PLG-014`). |
| Extent | Both directions declare their extent, and an operation is used only when the declared extent covers it (`PLG-012`). |
| Operations | Initialisation that may complete later, destruction, capability listing, invocation, cancellation, payload release, shutdown, and service completion (`PLG-019`). |
| Status space | The frozen status set of `PLG-045`; higher layers add meaning, not values. |
| Ownership | Each side releases only what it owns; plugin-owned payloads are released only through the plugin's release operation (`PLG-046` to `PLG-052`). |
| Completion | A pending result is driven by the host and completed through a completion handle the plugin may take further uses of and must release (`PLG-036` to `PLG-044`). |
| Payloads | Every payload declares its format version; an unknown version is unsupported and a malformed payload is invalid (`PLG-053`, `PLG-054`). |
| Publication | The contract's constants and per-operation rules are published in a form any language can implement from (`PLG-056`). |

The contract's version numbers are open (`OQ-111`). The completion handshake and
cancellation are open (`OQ-105`), as is the payload encoding (`OQ-106`).

## 12.6 Bundle and manifest

| Item | Rule |
| --- | --- |
| Artifact | Exactly one file per plugin, carrying a signed container, the manifest, and the code (`BND-001`). |
| Signature | Covers every byte a user installs, with the signature itself excluded (`BND-004`, `INV-021`). |
| Trust | Anchored in a configured trust store and its revocations; the compiled-in set is empty (`BND-009` to `BND-016`, `SEC-006`). |
| Refusal classes | A trust failure stops start-up; a container refusal disables one bundle (`BND-017`, `BND-018`). |
| Unpacking | Validates every entry, makes available only this platform's code, refuses path escape and non-regular entries, and applies the declared limits (`BND-024` to `BND-034`). |
| Manifest | Declares identity, version, contract demand, code per platform, capabilities with schemas, and the configuration schema (`MAN-002`). |
| Validation order | Presence, readable format, name, contract demand, then code for this platform (`MAN-007`). |

The container and archive format are open (`OQ-107`); the signature and trust
mechanism are open (`OQ-108`); the manifest's format and field names are open
(`OQ-109`); the schema dialect is open (`OQ-110`).

## 12.7 Credential encoding

- A token is opaque text. Its length and alphabet are open (`OQ-120`).
- The daemon stores only an irreversible function of the token, never the token
  itself, and MUST NOT be able to return it again (`AUT-013`, `AUT-014`). The
  function is open (`OQ-120`).
- An implementation that adopts an existing state directory MUST keep previously
  paired surfaces working or migrate their credentials deliberately (`AUT-040`).
- A client-generated request identifier is opaque, at most 64 printable
  characters with no space, and unique per stream (`CAN-002`, `CAN-003`). Its
  form is open (`OQ-123`).

## 12.8 Protocol compatibility

- The protocol major version MUST be part of the contract's versioned identity
  (`API-005`).
- `API-004` requires an automated compatibility check to reject an unintentional
  breaking contract change. A deliberate breaking change is introduced as a new
  major version or as a documented migration.
- `CMP-005` and `CMP-006` require the plugin contract's version, the bundle
  container version, the plugin's own version, and the network protocol version to
  remain independent compatibility axes, and require existing paired surfaces to
  keep working or be migrated.

The protocol the implementation uses is open (`OQ-022`), and so is the
remote listener and the way forwarded client information arrives (`OQ-115`).
