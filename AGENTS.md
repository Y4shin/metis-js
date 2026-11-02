# Agent conventions

Conventions for every agent that works in this repository.

## Work lives under `docs/tasks/`

An effort is one directory, `docs/tasks/<effort>/`, holding its map, its
planning tasks, its specification, its architecture document, its
implementation tickets, and its review. Archived efforts move to
`docs/tasks/archive/`. `docs/tasks/index.md` lists both.

Rejected requests live in `docs/tasks/out-of-scope/`.

## Only the named tools write `docs/tasks/`

Once `docs/tasks/state.yaml` carries `schema_version`, the built-in `write` and
`edit` tools are refused on that tree. Use the matching tool for the change:
`tw_write_ticket_note` for a ticket's notes, `tw_write_task_note` for a
planning task's record, `tw_write_section` for a map section,
`tw_write_changelog` for the changelog.

Two segments are exempt because they hold evidence rather than artifacts:
`.work/` (scratch) and `deviation-reports/`.

## Vocabulary

Use the canonical long forms when the kind could be ambiguous: a planning task
is a task, an implementation ticket is a ticket. `CONTEXT.md` carries the
project's own terms; use them and do not invent synonyms.

## Skills

A skill is invoked through the `Skill` tool. Frontmatter carries the
invocation split: `name`, `description`, and `disable-model-invocation` for a
skill a human starts. Do not add other frontmatter keys.

If this repository ever holds skills, its buckets and their promotion rules are:

- `engineering/` and `productivity/` are promoted: listed in the package's
  `pi.skills` and given a documentation page.
- `misc/`, `in-progress/`, and `deprecated/` stay in the repository and are not
  shipped.
- A retired skill moves to `deprecated/` with a note naming its replacement. It
  is not deleted.

## Writing

- No em-dashes. Use a comma, a colon, or a full stop.
- Write for an agent that reads the file without the conversation that produced
  it: define a term where it first appears.
- State what was verified and what was not.

## Commits

One commit per coherent change. The subject states what changed in the
imperative, without a ticket number when the repository has no tracker. A
schema or workflow change says so in the subject, for example
`chore: initialize task-workflow (schema_version 5)`.
