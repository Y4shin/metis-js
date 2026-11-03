# 4. Workflows

A workflow composes bounded agentic tasks and runtime-provided script steps into
an explicit graph. The workflow runtime owns sequencing and data flow. The agent
runtime owns model and tool execution inside one agentic task. The daemon hosts
workflow runs.

## 4.1 Definition

- WFL-001: **A workflow definition MUST declare an entrypoint input schema, a
  final output schema, a list of named work steps, and connectors.**
  `[planned, form open]`
- WFL-002: **The workflow's entrypoint and final output MUST be implicit special
  steps and MUST NOT appear in the work-step list.** `[planned]`
- WFL-003: **A workflow MUST be a directed acyclic graph in the finished
  application's first version.** `[planned]`
- WFL-004: **The list of steps MUST NOT determine execution order; dependencies
  and connector routing MUST determine it.** `[planned]`
- WFL-005: **Every workflow unit that performs work MUST be exactly one of three
  step types: an agentic task, a script step, or a fan-in script step.**
  `[planned]`
- WFL-006: **An agentic task step MUST declare an explicit input and an explicit
  output and MUST execute through the agent runtime.** `[planned]`
- WFL-007: **An agentic task step MUST receive narrowly scoped instructions,
  context, tools, model selection, and execution budget instead of requiring one
  model to orchestrate the whole pipeline.** `[planned]`
- WFL-008: **A script step MUST be runtime-provided and MUST execute an
  author-supplied script in a restricted embedded language.** The language, its
  definition format, and its execution environment are open (`OQ-027`); the
  specification fixes only that the script runs in a restricted environment and
  reaches outside effects solely through declared tools.
  `[planned, form open]`
- WFL-009: **A fan-in script step MUST declare multiple input ports and MUST NOT
  have tool access.** `[planned]`
- WFL-010: **A workflow MUST pass validated step outputs into subsequent steps.**
  `[planned]`
- WFL-011: **A workflow run MUST expose progress, results, errors, and
  cancellation to its callers.** `[planned, form open]`
- WFL-012: Retired. This identifier is not reused.

## 4.2 Connectors and routing

- WFL-013: **Each connector MUST reference exactly one source step, MUST declare
  its possible destination steps and input ports, and MUST contain a routing
  function.** `[planned, form open]`
- WFL-014: **The routing function MUST receive the source step's output and MUST
  return a mapping from selected destinations to their input values.**
  `[planned, form open]`
- WFL-015: **The routing function MAY omit declared destinations, which provides
  conditional routing and fan-out.** `[planned]`
- WFL-016: **The routing function MUST NOT target an undeclared destination, and
  the runtime MUST reject a connector that targets one.** `[planned]`
- WFL-017: **A connector routing function MUST NOT call tools.** `[planned]`
- WFL-018: **Each input port MUST be referenced by at most one connector
  destination, including across different connectors, and the runtime MUST
  validate and reject multiple producers for the same port.** `[planned]`

## 4.3 Joins and reachability

- WFL-019: **A join MUST wait until every declared input port has either received
  a value or is confirmed unreachable for the current run.** `[planned]`
- WFL-020: **The runtime MUST propagate reachability through the graph, accounting
  for pending upstream routing decisions.** `[planned]`
- WFL-021: **The runtime MUST NOT mark a port unreachable merely because its value
  has not arrived yet.** `[planned]`
- WFL-022: **Once all of a join's ports are resolved, the runtime MUST execute the
  join's script with the received values and with explicit unreachable
  markers.** `[planned]`
- WFL-023: **An unreachable marker MUST be distinct from a valid null or nil
  payload.** `[planned]`
- WFL-024: **A join MUST route its result through its outgoing connectors in the
  same way as any other step.** `[planned]`
- WFL-025: **A failed or cancelled branch MUST be distinguishable from a
  deliberately abandoned branch.** How each affects join readiness and run
  outcome is open (`OQ-024`). `[planned, form open]`

## 4.4 Declared tool dependencies

- WFL-026: **A script step MUST declare the tool functions it will use.**
  `[planned]`
- WFL-027: **Before execution, the runtime MUST validate that each declared tool
  exists and is permitted by the workflow's agent context.** `[planned]`
- WFL-028: **The runtime MUST inject only the declared, permitted tool functions
  into a script step's execution.** `[planned]`
- WFL-029: **The runtime MUST NOT expose the full tool registry or an unrestricted
  dynamic tool lookup to a script step.** `[planned]`
- WFL-030: **A script step MUST NOT reach the filesystem, network, processes, or
  host interfaces through its script environment; all of its external effects
  MUST go through its declared tool calls.** `[planned]`
- WFL-031: **Tool dispatch from a script step MUST enforce the agent context's
  permissions independently of the declared dependency list.** `[planned]`
- WFL-032: **A connector or fan-in script step MUST receive no tool functions.**
  `[planned]`

## 4.5 Hosting and persistence

- WFL-033: **The daemon MUST host workflow runs.** `[planned]`
- WFL-034: **The daemon MUST supply model provider implementations and
  permission-filtered tool dispatch to workflow runs.** `[planned]`
- WFL-035: **A workflow run MUST execute under its agent context's permissions
  and MUST NOT widen them.** `[planned]`
- WFL-036: **A surface MUST NOT host workflow execution.** `[planned]`
- WFL-037: **Plugins MUST provide tools, not workflow step types; the workflow
  runtime MUST NOT require a registry of plugin-defined step executors.**
  `[planned]`
- WFL-038: **Workflow progress and outputs MUST be distinct from an agent task's
  session history.** `[planned]`
- WFL-039: **Restoring session state MUST NOT be treated as sufficient to resume a
  workflow safely.** `[planned]`
- WFL-040: **A retry of an external write MUST have explicit idempotency or
  reconciliation support; the application MUST NOT assume exactly-once delivery.**
  `[planned, form open]`
- WFL-041: **The daemon MUST persist workflow run state as needed.**
  `[planned, form open]`

## 4.6 Illustrative composition

An illustrative example is a voice recording: transcription, an agent task
that extracts structured intents, a script step that validates and routes, more
narrowly scoped agent tasks where interpretation is still needed, script steps
that call tools to create calendar entries, todo items, and knowledge-base notes,
and a final step that sends a notification summarizing actual results, including
partial failures. The integrations in this example are illustrative capabilities,
not mandatory dependencies of the workflow runtime.

Workflows exist so that smaller local models can solve focused subproblems while
explicit code controls the overall pipeline. A stronger model could perform the
same work from instructions alone, but Metis also supports explicit pipelines
without relying on that capability.
