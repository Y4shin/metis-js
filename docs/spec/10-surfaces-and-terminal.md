# 10. Desktop, Android, and terminal surfaces

## 10.1 Shared application architecture

- SUR-001: **The application MUST provide a desktop shell and an Android shell
  that share one application frontend.** `[planned]`
- SUR-002: **The application MUST NOT expose or depend on a local network service
  to serve its own user interface.** `[planned]`
- SUR-003: Retired. This identifier is not reused.
- SUR-004: **Both shells MUST reach the daemon only through the shared
  user-interface-independent client and the daemon API.** `[planned]`
- SUR-005: **The platform application layer MUST be the only component that talks
  to the daemon; the user-interface layer MUST NOT talk to the daemon directly.**
  `[planned]`
- SUR-006: **The application MUST be a client of the persistent daemon and MUST
  NOT embed daemon orchestration in the application shell by default.**
  `[planned]`
- SUR-007: **The user interface MUST reflect daemon state without owning it.**
  Closing an application window MUST NOT stop daemon work. `[planned]`
- SUR-008: **High-frequency streamed updates MUST be batched rather than crossing
  the application bridge once per token (see `PERF-003`).** `[planned]`
- SUR-009: **Platform differences MUST be exposed as explicit platform
  capabilities, so screens can adapt between desktop and Android.** The
  documented capabilities include credential storage, notifications, file
  selection, and desktop window or tray behaviour. `[planned]`
- SUR-010: **The application MUST NOT provide an iOS implementation.** `[current]`
- SUR-011: **The application MUST NOT host a daemon locally on Android; Android
  MUST reach the daemon over the network transport.** `[planned]`

## 10.2 Desktop shell

- SUR-012: **The desktop shell MUST run on the supported desktop platforms and
  use the local transport by default.** `[planned]`
- SUR-013: **The desktop shell MUST provide the platform capabilities its screens
  require, with window and tray behaviour included where the platform supports
  it.** `[planned, form open]`
- SUR-014: **Keyboard and scrolling input MUST work on each supported desktop
  platform.** `[planned, form open]`

## 10.3 Android shell

- SUR-015: **The Android application MUST integrate with the platform's system
  assistant in the way that platform requires for assistant activation.**
  `[planned, form open]`
- SUR-016: **Where voice interaction requires it, the Android application MUST
  provide the platform's voice-interaction integration and its sessions.**
  `[planned, form open]`
- SUR-017: **The user MUST select Metis as their assistant for system-assistant
  activation to reach it; invocation gestures vary by device and configuration.**
  `[planned]`
- SUR-018: **The Android assistant service lifecycle MUST be independent of the
  user interface, and the application MUST NOT assume that the user interface is
  initialised when Android starts the service.** `[planned]`
- SUR-019: **Android cold-start activation, service-to-client communication,
  permission requests, background behaviour, and resume and reconnection are open
  (`OQ-075` to `OQ-077`).** `[planned, form open]`
- SUR-020: **Android lifecycle restrictions apply regardless of the shell
  technology; the shell MUST NOT be assumed to exempt the application from
  them.** `[planned]`
- SUR-021: **The shared client MUST support reconnection after background and
  resume on Android.** `[planned, form open]`
- SUR-022: **A release Android build MUST be signed and installable.** The
  release pipeline is open (`OQ-078`). `[planned, form open]`
- SUR-023: **The desktop and Android applications MUST let a user pair a surface by
  entering the preshared key, receiving the token, storing it through the client,
  and reporting which backend stored it.** The confirmation step is open
  (`OQ-102`). `[planned, form open]`

## 10.4 Terminal application

- TRM-001: **The application MUST provide one terminal program containing
  command-line commands and an interactive terminal user interface.**
  `[planned]`
- TRM-002: **The command-line commands MUST be usable for scripting and
  automation.** `[planned, form open]`
- TRM-003: **The interactive terminal user interface MUST use the same daemon API
  and the same shared client as the command-line commands.** `[planned]`
- TRM-004: **The terminal program MUST NOT implement its own agent runtime.**
  `[planned]`
- TRM-005: **The terminal program MUST exercise the daemon end to end, so that
  implementation work can land as vertical slices rather than as an untested
  layer.** `[planned]`
- TRM-006: **The terminal program MUST use only the shared client and MUST NOT
  depend on the desktop or Android shell.** `[planned]`
- TRM-007: **The terminal user-interface library, the command surface, and the
  first vertical slice the terminal drives are undecided.** An implementation's
  choice is a decision outside this specification. `[open]`

## 10.5 Current realization note

A particular realization may name a desktop framework, a frontend framework, and
a cross-platform application shell for Android. Those names are implementation
choices, not requirements of this specification. A different implementation can
choose different technologies provided that it
preserves the client-side user interface with no local server layer, the shared
client, the platform capability boundary, the assistant integration behavior,
and the daemon-client separation stated above.
