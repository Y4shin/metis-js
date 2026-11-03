# 2. Domain model and glossary

This section defines the vocabulary the rest of the specification uses. Terms
marked
"settled" are established project vocabulary. Terms marked "planned" belong to a
capability whose concrete form is still open; their definitions state only what
the evidence fixes.

## 2.1 Core concepts

**Metis Agent** (settled): The assistant itself, and the name of the product and
its codebase. It is minimal and extensible: the core stays small, and capability is added
rather than built in. Avoid "metis" as prose and "the bot".

**Daemon** (settled): The long-lived process at the center of Metis. It runs on
the user's own computer or on a server, stays reachable across devices, and is
not confined to a single chat session. The daemon is what makes Metis a service
the user owns rather than a window the user visits. A server is a deployment
target, not the process.

**Surface** (settled by usage): A client program through which a user or operator
reaches the daemon. The terminal CLI, the terminal TUI, the desktop application,
and the Android application are surfaces. A surface does not host the agent
runtime and does not own daemon state.

**Client**: The shared, user-interface-independent library or component a surface
uses to reach the daemon. All surfaces use the same client-facing daemon API.

**Chat session** (settled): A conversation with the assistant. It is supported
with a deliberately simple harness. A chat session is the current product form
of a conversation; the planned persisted conversation generalizes it. Whether the
two become one first-class persisted concept remains open (`OQ-001`, `OQ-005`).
This specification uses Conversation for the planned persisted concept and keeps
Chat session for the current product form. That is a deliberate distinction,
not a second name for one thing.

**Small local model** (settled): A model that runs on modest local hardware and
has a small context window relative to hosted frontier models. Metis is designed
to work well with these, which is why efficient prompt and context usage matters.

## 2.2 Agents and execution

**Agent context** (planned; working term "agent"): A declared execution boundary
consisting of a compact system prompt and permissions, initially the tools the
context may call. Every conversation, agent task, and workflow run belongs to one
agent context. The daemon owns the declarations and resolves them into prompt and
tool policy. The declaration format, the final name, and the full set of
permission categories are open.

**Consumer**: The component that embeds a reusable runtime and supplies the
interfaces that runtime requires. For the agent runtime, the consumer supplies
model provider implementations, tools, and a session persistence implementation.
For the workflow runtime, the consumer supplies storage, scheduling, transport,
and concrete integration implementations. The daemon is the consumer for the
finished application; another embedding application may be a consumer of the same
runtime.

**Conversation** (planned): An ongoing exchange that accepts successive user
messages and retains complete session history. A conversation uses its agent
context's system prompt and tool permissions.

**Agent task** (planned): A bounded, explicitly invoked execution with declared
input, instructions, selected model and tools, and execution limits. It returns
an output or a failure or cancellation outcome. A task may span multiple model
turns and tool calls. "One-shot" refers to the bounded invocation, not to a
single model request.

**Session state** (planned): The complete history and all persisted state needed
to reconstruct an idle conversation or agent task. It is owned by the agent
runtime; its storage is supplied by a consumer.

**Idle boundary** (planned): A point at which the runtime has relinquished
control and no model request or tool call is in progress. Session state is
required to be exactly reconstructable at every idle boundary once persistence
events are acknowledged.

**Execution limit** (planned): A bound on an execution, such as a turn count or a
resource budget. The exact limit categories are open.

**Lifecycle event** (planned): An event emitted by the runtime as an execution
proceeds, fails, completes, or is cancelled. The event set is open.

## 2.3 Workflows

**Workflow** (planned; definition format open): A directed acyclic graph with an
entrypoint input schema, a final output schema, named steps, and connectors. A
workflow run executes under one agent context.

**Step**: One unit that performs work in a workflow. Exactly three step types
exist:

- **Agentic task step**: A bounded model-driven task with explicit input and
  output.
- **Script step**: Runtime-provided script execution in a restricted embedded
  language, with explicitly declared tool dependencies drawn from the enclosing
  agent context. The language is open (`OQ-027`).
- **Fan-in script step** (join): A script step with multiple declared input
  ports and no tool access. It combines inputs after its ports resolve.

**Connector** (planned): A named edge from one source step that declares its
possible destination steps and input ports and carries a routing function. The
routing function receives the source step's output and returns a mapping from
selected destinations to their input values. It may omit declared destinations,
which abandons those paths for the run, and it does not target an undeclared
destination. Connectors receive no tool functions.

**Input port**: A named input of a step. One input port has at most one producing
connector destination.

**Entrypoint and final output**: Implicit special steps that carry the workflow's
input schema and final output schema. They are not members of the work-step list.

**Reachability**: For one workflow run, an input port is either resolved with a
value, resolved as unreachable because routing abandoned every path that could
produce it, or still pending. Unreachable is distinct from a null or nil value.

**Tool dependency declaration**: The list of tool functions a script step will
use. The runtime validates each declared function for existence and permission
and injects only those functions.

## 2.4 Plugins

**Plugin** (settled): An add-on that supplies a capability to the daemon.
Plugin-based customization is how Metis stays minimal: the core is small, and
optional complexity is added through plugins. A plugin supplies tools through
capabilities; it does not export workflow steps.

**Capability** (settled): One named thing a plugin offers, qualified by its
instance at the host boundary, carrying a kind, a version, and request and
response schemas. A plugin advertises its capabilities before it serves anything,
and the host refuses a call to one the plugin does not declare. The qualification
format is open (`OQ-104`). The documented capability kinds are tool and provider. A kind a host
build does not know refuses that one capability and leaves the rest of the plugin
usable.

**Instance**: One loaded, configured occurrence of a plugin. A plugin may have
several instances. An instance is identified by the name its configuration was
resolved under.

**Bundle** (settled): The one file a plugin is installed as. It carries the
plugin's manifest and the plugin's code for each platform the plugin supports,
inside one signed, versioned container. Signing covers everything a user
installs, so there is no second artifact to keep in step. The container and
archive formats are open (`OQ-107`).

**Manifest** (settled): The structured declaration at a bundle's root. It
declares the plugin's identity and version, the compatibility it demands, its
code for each platform, its capabilities with their schemas, and its
configuration schema. It is written by hand or generated from the same source as
the plugin's own declaration, and the host cross-checks the two. The manifest's
format and field names are open (`OQ-109`).

**Platform target** (settled): One operating-system and architecture pair a
bundle carries code for, such as a Linux x86-64 desktop. A bundle may carry
several targets, and a host loads only the one that matches it. The spelling of
a target and the file layout inside a bundle are open (`OQ-107`).

**Host context** (settled): What the host tells a plugin about itself when it
initialises it, delivered with the plugin's configuration. It names the instance
by the identity the host resolved that configuration under, and it is permissive
so a host may add a field. Its fields are the instance name, an optional data
directory, and the highest capability-descriptor version the host serves.

**Disabled plugin** (settled): A plugin the host refused to start. It is still
listed with its name, the reason, and its capabilities when a manifest was
readable. Invoking one of its capabilities answers the disabled route error with
that reason. A disabled plugin whose manifest could not be read is named by its
bundle's file stem.

**Sink** (settled): The single destination for everything a plugin says outside
its response: progress, events, logs, and service requests. It cannot fail and
does not block, and a host with no sink is legal.

**Ordering ledger** (settled): The host's ordered record of the lifecycle steps it
took for an instance, from load through call, cancellation, and teardown. It is
what makes release safe to check: plugin code is released only when no call,
pending result, or callback can still run in it.

**Service request** (settled): A plugin's request for a host facility, currently a
timer and later I/O readiness. The plugin awaits the host's answer with the same
request identifier rather than blocking.

**Pending result** (settled): A plugin's not-yet-complete answer. The host owns
the waiting side: it asks whether the answer is ready, receives either the answer,
a pending indication, or a terminal status, and it receives completion signals
that tell it to ask again. The concrete shape of this handshake is open
(`OQ-104`).

**Extension hook** (planned, undefined): A runtime extension point whose concrete
form is undecided (`OQ-040`). It is not a defined interface in this
specification.

**Request identifier**: A host-chosen identifier for one invocation, unique among
in-flight calls, and the key for cancellation, progress, and service requests.
It is distinct from the client-side request identifier used for correlation and
cancellation of daemon work.

**Payload format version**: The version every plugin payload declares. The host
refuses a version it does not know as unsupported. The encoding behind the version
is open (`OQ-106`).

**Trust store**: The set of trusted signing identities and their revocations the
host uses to decide whether a bundle is trusted. The runtime never reads daemon
configuration; the daemon supplies the trust store. Its representation is open
(`OQ-019`, `OQ-108`).

## 2.5 Tool discovery

**Toolset** (settled): A named discovery grouping of tool capabilities that are
useful together for a task or workflow. It carries a concise use case and may
name related toolsets or supporting capabilities. Membership grants no
permission; the agent context remains the authorization boundary.

**Tool view** (settled): The exact ordered set of permitted tool schemas visible
to one model request. It may contain pinned intrinsic tools, starter-selected
tools, and tools chosen through the disclosure oracle. It is reconstructable
session state rather than part of the agent context's system prompt.

**Disclosure oracle** (settled): The side-effect-free selection path an agent can
request when it needs a different tool view. It searches only permitted toolsets
and capabilities, records the resulting tool-view change, and leaves its selection
transcript out of model-visible chat history.

The vocabulary above is settled. The mechanism that uses it is a proposal and is
not a confirmed requirement; see `05-tool-disclosure.md`.

## 2.6 Access control

**Surface identity**: The declaration every client call carries: protocol
version, surface name, and client version. It supports logging and version
negotiation. It is not authentication and grants nothing.

**Local access control**: The operating system's access control on the local
transport endpoint. The daemon owns the endpoint and restricts it to its own user,
with a configured group where the platform supports one, and refuses remote and
other-account callers. A local caller needs no credential. The concrete local
transport and its access rule are open (`OQ-114`).

**Pairing**: The one-time process by which a surface obtains its own credential
from the daemon. Pairing presents a preshared key in the request body and returns
an opaque per-surface token once.

**Token**: An opaque per-surface credential minted at pairing. The daemon stores
only an irreversible hash. A token grants the full exposed surface, with no
per-client permissions or scopes in the first version. Tokens do not expire in
the first version.

**Preshared key**: The operator-supplied secret that gates pairing. It is a
literal, an environment reference, a keystore reference, or a permission-checked
file reference. Removing it closes pairing, which is a state and not a
revocation.

**Revocation**: The act of stopping one surface's credential. An operator can
revoke one surface without affecting another, with effect no later than
the next daemon start (`AUT-030`). How revocation is performed is not fixed; the
current implementation deletes the stored credential record.

## 2.7 Realtime and cancellation

**Event stream**: A long-lived API operation that delivers events to a
client. A stream selects one or more channels and may carry a cursor.

**Channel**: An event topic selected by a stream. One channel carries
 daemon-wide events, one carries a session's events, one a task's events, and one
a workflow run's events. The channel names and their spelling are open
(`OQ-119`).

**Event**: One delivery carrying a daemon-wide monotonic identifier, a kind, a
channel, and a payload. Kinds are unspecified, snapshot, heartbeat, and ordinary
event. Unspecified is the contract's "cannot act on this" value; a receiver does
not act on a kind it does not know.

**Snapshot**: The opening set of messages of a stream that carries no cursor.
Each snapshot message carries the head of one selected channel; when no selected
channel holds an event, one empty snapshot message opens the stream.

**Heartbeat**: An ordinary stream message that keeps an idle stream alive. It
carries a kind of its own and the daemon's latest identifier.

**Cursor**: The highest event identifier a client has processed, sent with a new
stream so that the stream resumes after it. Zero means no cursor. The cursor is
not a durable position: the daemon holds events only in a bounded in-memory
window. The request field that carries it is open (`OQ-119`).

**Resync**: A fresh stream opened with no cursor, which begins with a snapshot.
A client resyncs when it has no cursor or when the daemon refuses the cursor it
presented.

**Slow consumer**: A stream client that stops reading long enough to fall behind
the daemon's bounded delivery window. The daemon ends that stream with a typed
refusal instead of silently skipping events.

**Request identifier (client-facing)**: An optional client-generated opaque
identifier carried with the call, at most 64 printable characters with no
space, whose exact form is open (`OQ-123`), used to correlate a call with its
answer and to name the work in a cancellation request. A call that declares none is not cancellable. The daemon
echoes the identifier verbatim in the call's answer.

**Cancellation**: An explicit idempotent request that stops the work one client
request started. A dropped stream cancels nothing by itself.

## 2.8 Concept relationships

```text
Operator
  declares .......... Agent contexts
  configures ........ daemon configuration, secrets, trust store, listeners
  installs .......... bundles

Agent context (prompt + permissions)
  owns .............. conversations, agent tasks, workflow runs
  bounds ............ every tool call in those executions

Conversation / agent task
  produces .......... session state
  persisted through . consumer-supplied store
  emits ............. lifecycle events

Workflow (DAG)
  contains .......... agentic task steps, script steps, fan-in script steps
  connects .......... connectors with routing functions
  calls ............. tools permitted by its agent context

Plugin
  ships as .......... one signed bundle
  contains .......... manifest + code for each supported platform
  advertises ........ capabilities
  adapted into ...... tools, model providers, hooks

Surface -> shared client -> daemon API -> daemon -> agent runtime, workflows, plugins
```

## 2.9 Lifecycles

### Plugin instance lifecycle

```text
bundle on disk
  -> verified (signature trusted)
  -> unpacked (manifest and this host's code made available)
  -> manifest validated (identity, compatibility demand, code for this platform)
  -> plugin code loaded and boundary contract checked
  -> initialised with configuration and host context
  -> registered (capabilities routable)
  -> serving calls
  -> disabled (registry state only; in-flight calls continue)
  -> enabled (accepts calls again)
  -> shutdown (pending results dropped, plugin quiesced)
  -> instance destroyed
  -> plugin code released (only when nothing is inside it)
```

Failure outcomes are defined in `09-plugins.md` and `11-business-rules-and-invariants.md`.

### Surface access lifecycle

```text
local surface: OS access control -> calls served without a credential
remote surface: no token -> Pair(preshared key) -> token stored -> calls served
credential revoked -> refused as revoked (no later than next start)
preshared key removed -> pairing disabled; existing tokens keep serving
```

### Stream lifecycle

```text
open with cursor 0     -> snapshot -> live events -> heartbeats
open with live cursor  -> replay after cursor -> live events -> heartbeats
cursor outside window  -> refused before the stream opens
consumer falls behind  -> terminal slow-consumer refusal
client cancels request -> stream ends with a cancelled refusal
daemon restarts        -> every cursor expires
```
