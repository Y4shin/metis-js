# 3. Agent runtime: agent contexts, conversations, tasks, and session state

The agent runtime is a reusable execution component. It runs a model-driven
agent loop with model and tool contracts supplied by its consumer. It does not
depend on the network API, the plugin loader, daemon configuration, or any user
interface. Concrete model providers and tools arrive through the consumer.

## 3.1 Agent contexts

- AGT-001: **The daemon MUST declare one or more agent contexts, each with a
  compact system prompt and a set of permissions.** The declaration format and
  the permission categories, beyond tools, are open. `[planned, form open]`
- AGT-002: **Every conversation, agent task, and workflow run MUST belong to
  exactly one declared agent context.** `[planned]`
- AGT-003: **A conversation MUST use its agent context's system prompt and tool
  permissions.** `[planned]`
- AGT-004: **A workflow run MUST inherit its agent context's permissions and MUST
  NOT automatically inherit that context's system prompt.** `[planned]`
- AGT-005: **An agent task step inside a workflow MUST use its own task
  instructions within the enclosing permissions.** `[planned]`
- AGT-006: **A task or workflow step MAY narrow the available tools for its
  execution, and MUST NOT widen the enclosing agent context's permissions.**
  `[planned]`
- AGT-007: **The daemon MUST resolve each agent context's prompt and permissions
  for every execution context before that execution begins.** `[planned]`
- AGT-008: **The presence of several agent contexts MUST NOT by itself enable
  autonomous delegation or inter-agent messaging.** Those behaviors are not
  specified. `[planned]`
- AGT-009: **The agent context system prompt SHOULD stay low-level and concise to
  minimize system-prompt token overhead.** Disregarding this recommendation
  raises the token cost of every request and reduces headroom on small local
  models. `[planned]`
- AGT-010: **The agent runtime MUST accept model provider and tool
  implementations from its embedding application and MUST NOT require a specific
  model provider.** `[planned]`

## 3.2 The agent loop

- CNV-001: **The agent runtime MUST provide one agent loop that prepares a model
  request, calls a model, executes requested tools, records results, and then
  continues or finishes.** `[planned]`
- CNV-002: **The runtime MUST give its consumer explicit control over the
  instructions, messages, tool schemas, and tool results sent to the model.**
  `[planned]`
- CNV-003: **The runtime MUST expose lifecycle events, cancellation, execution
  limits, and extension hooks for an execution.** `[planned]`
- CNV-004: **The agent runtime MUST derive the context it selects for one model
  request from session state.** `[planned]`
- CNV-005: **The agent runtime MUST NOT discard session history when it prunes or
  transforms model input.** `[planned]`

## 3.3 Conversations

- CNV-006: **A conversation MUST accept successive user messages and retain its
  complete session history.** `[planned]`
- CNV-007: **A conversation MUST use the same agent loop as an agent task.**
  `[planned]`
- CNV-008: **A conversation MUST belong to an agent context and MUST use that
  context's system prompt and tool permissions.** `[planned]`

## 3.4 Agent tasks

- TSK-001: **An agent task MUST accept an explicit input, instructions, model
  selection, tool selection, and execution limits.** `[planned]`
- TSK-002: **An agent task MUST return either its output or a failure or
  cancellation outcome.** `[planned]`
- TSK-003: **An agent task MAY span multiple model turns and tool calls.** The
  bounded property of a task is its invocation, not a single model request.
  `[planned]`
- TSK-004: **An agent task MUST be executable without an existing user
  conversation.** `[planned]`
- TSK-005: **An agent task MUST maintain its own execution history separate from
  any conversation.** `[planned]`
- TSK-006: **An agent task MUST emit lifecycle events for its own execution.**
  `[planned]`
- TSK-007: **An agent task MUST support a declared output schema and MUST
  validate its output against that schema.** This requirement is a decision, so it is a MUST rather than a recommendation. How a
  validation failure is reported is open (`OQ-008`). `[planned]`
- TSK-008: **The consumer MUST choose an agent task's storage implementation
  through the runtime's storage interface (see `SES-002`).** `[planned]`
- TSK-009: **A task's declared tool selection MUST be bounded by its agent
  context's permissions.** `[planned]`

## 3.5 Session state and persistence

- SES-001: **The agent runtime MUST define and maintain session state containing
  the complete session history and all persisted state needed to reconstruct an
  idle session.** `[planned]`
- SES-002: **The runtime MUST accept a consumer-provided persistence
  implementation through a storage interface and MUST NOT require a specific
  storage product.** `[planned]`
- SES-003: **The persistence interface MUST stream ordered persistence events for
  incremental saving and MUST load a stored session.** `[planned]`
- SES-004: **The agent runtime MUST be able to reconstruct exactly the live
  session state `B` that existed after it relinquished control, given stored
  state `A` and the subsequent persistence events `E1..En`.** `[planned]`
- SES-005: **The persistence contract MUST represent every state change the
  agent runtime needs to reconstruct state `B`.** `[planned]`
- SES-006: **An idle session is safe to discard only after the store has
  acknowledged every corresponding persistence event.** `[planned]`
- SES-007: **A persistence failure MUST be surfaced to the caller and MUST NOT be
  reported as saved.** `[planned]`
- SES-008: **The exact-reconstruction invariant applies to session state only; it
  does not require replaying external tool side effects or reproducing
  process-local handles.** `[planned]`
- SES-009: **The runtime MUST own session types, persistence-event semantics, and
  reconstruction logic; the consumer MUST own the storage backend and its
  implementation.** `[planned]`
- SES-010: **The daemon MUST own its session-state policy through a persistence
  implementation supplied to the agent runtime.** `[planned]`

## 3.6 Boundaries with other components

- CNV-009: **The agent runtime MUST be usable by an embedding application other
  than the daemon.** `[planned]`
- CNV-010: Retired. This identifier is not reused.
- CNV-011: **The runtime MUST NOT assume that persisting session state makes a
  workflow safe to resume or prevents repeating external writes.**
  `[planned]`
