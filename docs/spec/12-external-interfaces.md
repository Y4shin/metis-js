# 12. External interfaces and compatibility constraints

This section records the names and encodings that an operator, a surface, or a
third-party plugin author depends on. A different implementation can change its
internal architecture freely. The operator settings, paths, access rules, plugin
ABI names, bundle bytes, and manifest fields are requirements: changing one is a
compatibility change and requires a deliberate versioned migration. The network
metadata spellings and the token encoding in sections 12.3 and 12.7 are the
current realization; a new implementation can change them if it preserves the
semantic requirements they carry and does not need to interoperate with an
existing client.

## 12.1 Operator-facing settings

| Setting | Meaning |
| --- | --- |
| `METIS_ENDPOINT` | Local endpoint override. An absent or empty value means the platform default. On Windows it names the pipe. |
| `METIS_SOCKET_GROUP` | Unix socket group. The named group exists and the daemon's user can use it; the socket and directory then use mode 770. Absent means mode 700 and no group. |
| `METIS_TCP_ADDR` | Remote bind address as an `IP:port` literal. Absent or empty disables the listener. No default. |
| `METIS_TRUSTED_PROXIES` | Comma-separated exact proxy IP addresses. Absent or empty trusts no proxy. |
| `METIS_PRESHARED_KEY` | Pairing secret, written as a secret reference. Absent or empty closes pairing. |
| `METIS_STATE_DB` | Persistent state location. Absent or empty falls back to the data-home path. |

Secret reference forms:

| Form | Meaning |
| --- | --- |
| `<value>` or `literal:<value>` | The value is the secret. A bare value is a literal, including a value that contains a colon. |
| `env:<NAME>` | Read environment variable `NAME`. |
| `keystore:<account>` | Read the platform keychain through its command-line tool under the service name `metis-daemon`. |
| `file:<path>` | Read a file that only its owner can read. |

These spellings are the external configuration surface. Earlier draft forms are
superseded by the forms above.

## 12.2 Paths and access rules

| Item | Rule |
| --- | --- |
| Local Unix endpoint default | `$XDG_RUNTIME_DIR/metis/daemon.sock`, else `$TMPDIR/metis/daemon.sock`, else a typed refusal. |
| Local Windows endpoint default | `\\.\pipe\metis-daemon`, prefix `\\.\pipe\` required, at most 256 characters, no interior NUL. |
| Socket directory and socket | Mode 700, or 770 with a configured group, owned by the daemon's user. |
| Persistent state | `$XDG_DATA_HOME/metis/metis.db`, else `$HOME/.local/share/metis/metis.db`, directory mode 700, file mode 600. |
| Client credential file | `$XDG_DATA_HOME/metis/client/<surface>.token`, else under `$HOME/.local/share/metis/client/`, file mode 600 inside a directory mode 700. |
| Plugin directory | A directory of bundle files scanned by the daemon. The exact path is not fixed by this specification and is open. |

## 12.3 Wire metadata and refusal channels

The current contract uses these metadata names. Their meanings are requirements
(`API-007`, `API-008`, `IDN-001` to `IDN-004`); the spellings are the current
realization. A new implementation can choose different metadata spellings if it
preserves those meanings and does not need to interoperate with an existing
client.

| Name | Direction | Meaning |
| --- | --- |
| `metis-protocol-version` | request | Protocol version the client speaks. Current supported version is 1. |
| `metis-surface` | request | Surface class: the current clients send `cli`, `ui`, `test`, or `unspecified`; the daemon records any declared name as descriptive data. |
| `metis-client-version` | request | Client version string for diagnostics. |
| `metis-supported-versions` | response | Comma-separated supported protocol versions on a version refusal. |
| `authorization` | request | `Bearer <token>`. |
| `metis-auth-error` | response | Credential refusal cause: missing, invalid, or revoked. |
| `metis-pair-error` | response | Pairing refusal: `disabled`, `wrong-key`, or `already-paired`. |
| `metis-cursor-error` | response | Cursor refusal: `expired`. |
| `metis-stream-error` | stream trailer | Terminal stream refusal: `slow-consumer` or `cancelled`. |
| `metis-request-id` | request and response | Client-generated opaque correlation identifier, at most 64 printable ASCII characters with no space, echoed verbatim. |

`API-007` requires a client to interpret a refusal without parsing human-readable
message text. The table above is one realization of that requirement.

## 12.4 Event contract

| Item | Rule |
| --- | --- |
| Channels | `daemon`, `session:<id>`, `task:<id>`, `workflow_run:<id>`. |
| Event fields | Identifier, kind, channel, payload. |
| Kinds | Unspecified, snapshot, heartbeat, ordinary event. |
| Identifier | Daemon-wide monotonic integer starting at 1. |
| Cursor | Request field `last_event_id`; 0 means no cursor. |
| Payload encoding (current) | JSON text in a text field. |

Changing the payload encoding is a breaking contract change governed by `CMP-004`
and section 12.8.

## 12.5 Plugin ABI names and encodings

These are external because a plugin in any language can implement them.

| Item | Rule |
| --- | --- |
| Entry symbol | `metis_plugin_entry_v1`; exactly one exported `metis_*` symbol. |
| ABI major / minor | 1 / 2. Minor is additive: a host accepts a plugin minor at or above its own within major 1. |
| Plugin table | Size, ABI major, ABI minor, initialise, destroy, list capabilities, invoke, cancel, release buffer, shutdown, service complete. |
| Host table | Size, host state, progress, emit event, log, service request. |
| Statuses | Ok = 0, InvalidArgument = 1, NotFound = 2, PermissionDenied = 3, Cancelled = 4, Internal = 5, Unsupported = 6, Pending = 7. |
| Buffer | Pointer, length, capacity, owner (none, host, plugin), reserved. |
| Payload envelope | Bits 0 to 3: format version 1. Bit 4: codec, 0 = JSON. Bits 5 to 7: reserved, zero. |
| Log levels | 0 debug, 1 info, 2 warn, 3 error. |

## 12.6 Bundle and manifest format

| Item | Rule |
| --- | --- |
| File extension | `.metis`. |
| Header | 40 bytes, big-endian: magic `METISBND`, container version 1, signing suite 1, header length 40, reserved 0, chain length, signature offset, signature length, ZIP length. |
| Domain tag | `METIS-PLUGIN-BUNDLE-PAYLOAD` (27 ASCII bytes). |
| Signature | Raw Ed25519, 64 octets. |
| Chain | DER X.509 v3; leaf first, anchor last; at most 8 links; canonical DER only. |
| Key identifier | First 16 bytes of SHA-256 over the raw 32-byte public key. |
| Archive | ZIP; `manifest.toml` at the root; `lib/<target>/<file>`; one file per target directory. |
| Limits | 256 entries; 64 MiB per entry; 256 MiB total declared uncompressed size. |
| Declared entry mode | `0600`, or no declared mode; anything else refused. |
| Manifest fields | `name`, `version`, `abi_major`, `min_abi_minor`, `libraries`, `capabilities[]` with `name`, `kind`, `version`, `request_schema`, `response_schema`, and `config_schema`. |
| Capability kinds | `tool`, `provider`. |

## 12.7 Credential encoding

- A token is opaque text. The current implementation mints 32 random bytes as 64
  lowercase hexadecimal characters.
- The current implementation stores the SHA-256 of the token's canonical
  lowercase hexadecimal text, not of the decoded bytes. This is a realization
  detail, not a requirement. `AUT-040` requires an implementation that adopts an
  existing state directory to keep paired surfaces working or migrate them
  deliberately.
- A request identifier generated by the client is random and unique per stream.
  The current implementation uses 16 random bytes as 32 lowercase hexadecimal
  characters. Only the 64-character printable-ASCII bound with no space is
  normative (`CAN-002`).

## 12.8 Protocol compatibility

- The protocol major version is part of the contract's versioned identity
  (`API-005`).
- `API-004` requires an automated compatibility check to reject an unintentional
  breaking contract change. A deliberate breaking change is introduced as a new
  major version or as a documented migration.
- `CMP-005` and `CMP-006` require the plugin ABI major, the bundle container
  version, the plugin's own version, and the network protocol version to remain
  independent compatibility axes, and require existing paired surfaces to keep
  working or be migrated.
