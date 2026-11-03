# 1. Purpose, scope, and product constraints

## 1.1 Purpose

Metis Agent is a minimal, extensible personal AI assistant that runs as a
persistent daemon on the user's own computer or on a server. The daemon is
reachable across devices and carries out work beyond an individual chat session.
The application emphasizes simplicity, plugin-based customization, and efficient
prompt and context usage, so that it works well with small local models.

The application is a general personal assistant, not exclusively a coding agent.
Capabilities are added through plugins. Chat sessions are supported with a
deliberately simple harness.

## 1.2 Goals

- PRD-G1: Run as a long-lived daemon that the user owns and reaches from several
  devices.
- PRD-G2: Keep the core small, with as little built in as possible, and add
  capability through plugins.
- PRD-G3: Use lean prompts and context so that small local models stay on task.
- PRD-G4: Support conversations and bounded agent tasks through one agent loop.
- PRD-G5: Support explicit, model-assisted pipelines through workflows.
- PRD-G6: Let a plugin author ship one signed artifact and let an operator install
  it as a file copy.
- PRD-G7: Let an operator expose the daemon to a remote device deliberately, and
  keep it closed by default.
- PRD-G8: Let the same daemon serve a terminal, a desktop application, and an
  Android application through one client-facing API.

## 1.3 Non-goals

- PRD-N1: Metis is not a chat-channel product. Chat channels are not planned.
- PRD-N2: Metis does not plan an iOS application.
- PRD-N3: Metis is not a sandbox. A plugin runs inside the daemon process
  and can crash or hang it.
- PRD-N4: Metis does not plan hot replacement, meaning replacing a running
  plugin's code while the daemon runs.
- PRD-N5: Metis does not host a daemon locally on Android. Android reaches a
  daemon over the network transport.
- PRD-N6: Metis does not plan a standalone browser client. A browser client would
  require a network endpoint or gateway, and the packaged applications use a
  platform client.
- PRD-N7: Multiple declared agents do not imply autonomous delegation or
  inter-agent messaging.
- PRD-N8: The plugin boundary is not a security boundary. It does not claim to
  make a plugin safe to run.

## 1.4 Actors

| Actor | Description |
| --- | --- |
| Operator | The person who runs and configures the daemon, names agent contexts, installs plugins, and enables remote access. |
| User | The person who talks to the assistant and starts work from a surface. |
| Surface | A client program: the terminal CLI, the terminal TUI, the desktop application, or the Android application. |
| Agent context | The declared prompt and permission boundary an execution belongs to. |
| Daemon | The long-lived process that hosts sessions, tasks, workflows, plugins, and the API. |
| Plugin author | A person who writes a plugin and packages it as a signed bundle. |
| Plugin | An add-on that supplies capabilities to the daemon. |
| Workflow author | A person who defines a workflow graph and its scripts. |
| Reviewer | A person or coding agent who builds or evaluates a Metis implementation from this specification. |

## 1.5 Scope of the finished application

The finished application comprises:

- a persistent daemon with a client-facing API, local and optional remote
  transports, configuration and secret resolution, scheduling, and agent
  declarations;
- an agent runtime with agent contexts, conversations, agent tasks, session
  state, lifecycle events, cancellation, and execution limits;
- a workflow runtime with explicit directed acyclic graphs, three step types,
  script routing, joins, and declared tool dependencies;
- a plugin system with a versioned plugin contract, signed bundles, manifests, a
  host runtime, start-up and teardown policy, and a plugin authoring SDK, core
  plugins, and a CLI;
- a terminal application with CLI commands and an interactive TUI;
- a desktop application and an Android application sharing one frontend, both
  clients of the daemon;
- an optional remote transport with per-surface pairing and credentials.

## 1.6 Product constraints

- PRD-C1: **The daemon MUST run as a long-lived process that continues its work
  when no user interface is connected.** `[current]`
- PRD-C2: **The application MUST keep capability optional and external: a plugin
  supplies capabilities, and the core MUST NOT absorb a plugin's capability as a
  built-in.** `[current]`
- PRD-C3: **Every execution that can call a tool MUST belong to a declared agent
  context.** `[planned]`
- PRD-C4: Retired. This identifier is not reused.
- PRD-C5: **A plugin MUST be installable as one signed file placed in the plugin
  directory.** Installation MUST NOT require a build step on the target machine.
  `[current]`
- PRD-C6: **Remote access MUST be off unless the operator explicitly enables
  it.** `[current]`
- PRD-C7: **The daemon MUST NOT terminate encryption for the remote listener.
  When remote access is exposed beyond a trusted network, a reverse proxy
  terminates encryption in front of the daemon.** `[current]`

## 1.7 Status and evidence caveat

The product is partially implemented. The agent runtime, workflow runtime,
daemon composition layer, plugin authoring SDK, desktop shell, and Android shell
are planned and not implemented. Their requirements are marked `[planned]` or
`[planned, form open]`. Behavior with supporting evidence is marked `[current]`
and is still subject to the rule that existing behavior is not automatically
intended behavior.

The plugin host and the network, transport, identity, pairing, and realtime
foundations are specified as current behavior. The Windows local transport
is compile-verified and has never executed; every Windows behavior is therefore
stated from design evidence and marked accordingly in the acceptance and
open-question records.
