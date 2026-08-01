# Open Productivity Standard (OPS)

> An open, tool-agnostic data model for exchanging tasks between
> productivity tools.

## Mission

Your tasks should live wherever you work best — and move when you do.
OPS is one open task structure that any productivity tool can export and
import, so your work travels between tools without losing title,
status, notes, or hierarchy. Any tool that reads or writes OPS can
exchange tasks with any other: unknown details ride along in
`metadata` instead of being dropped, so every export stays lossless.

OPS has three goals:

- **Interoperability** — one open task structure that any productivity
  tool can export and import. Tasks travel between tools without losing
  title, status, notes, or hierarchy.
- **Abstraction** — APIs, automations, and analysis are written once
  against the OPS shape and work across every OPS-capable tool, instead
  of being re-implemented per platform.
- **Ownership** — your productivity data stays yours. Exports are
  lossless: unknown details ride along in `metadata` instead of being
  dropped, so nothing stops you from leaving or returning to a tool
  when you want to.

That means a startup can track its work in markdown files in a repo
today and move into bigger productivity tools later — no migration
tools to write, none to buy: OPS speaks both.

OPS gives companies the same freedom at scale. Because the structure is
open and stable, tools can expose their tasks through OPS-based APIs,
and teams can build automations and analysis once — against their whole
body of productivity data, across any OPS-capable tool — instead of
rewriting them for every platform.

```text
[ Notion ]  ──┐                               ┌──> [ Linear ]
[ Todoist ] ──┼──> [ TASK TREE (the spec) ] ──┼──> [ Obsidian ]
[ Jira ]    ──┘                               └──> [ Anything ]
```

## The model in a few lines

A task needs only a **title** and a **status** — a line on paper and a
checkmark. Everything else is optional `metadata`, carried verbatim so
exports stay lossless. Subtasks nest in `tasks`.

```json
{
  "title": "Ship the OPS spec",
  "status": "open",
  "metadata": { "due_date": "2026-08-15", "priority": "high" },
  "tasks": [
    { "title": "Write the README", "status": "done" },
    { "title": "Publish it", "status": "open" }
  ]
}
```

## The streaming use case

Exports get big: 200 GB of tasks leaving Azure for company-internal
storage, piped through serverless processing units. With nested data
that pipe is expensive — a reader must hold the whole tree in memory
before it can act on a single task, so cost scales with the export, not
with the question you're asking.

The standard therefore also speaks flat, streamable formats — JSONL and
CSV. One task per line, processed one record at a time, constant
memory. Hierarchy is expressed as child id lists instead of nesting:

```jsonl
{"title": "Ship the OPS spec", "status": "open", "tasks": ["t1r", "t1a"], "metadata": {"due_date": "2026-08-15"}}
{"id": "t1r", "title": "Write the README", "status": "done"}
{"id": "t1a", "title": "Publish it", "status": "open"}
```

The root record carries no `id`; every referenced child carries one and
lists its own children the same way. A consumer rebuilds the graph by
resolving the id lists — no parent references needed.

### Flat vs. nested at a glance

| Aspect     | Nested (tree-preserving)        | Flat (streamable)                    |
|------------|---------------------------------|--------------------------------------|
| Formats    | JSON, YAML, Markdown, XML       | JSONL, CSV                           |
| Hierarchy  | `tasks` holds nested objects    | `tasks` holds child ids              |
| IDs        | optional                        | needed per referenced child; roots carry none |
| Memory     | whole tree in memory            | one record at a time                 |
| Best for   | trees traversals                | large exports, logs, pipe processing |

Both families carry the same model — see
[`examples/jsonl.jsonl`](examples/jsonl.jsonl) and
[`examples/csv.csv`](examples/csv.csv).

## Read order

1. [`SPECIFICATION.md`](SPECIFICATION.md) — the OPS Specifications: the
   standard itself.
2. [`examples/`](examples/) — the model in different serializations.
3. [`CHANGELOG.md`](CHANGELOG.md) — the specification history.

This repository holds only the spec, examples, and tests — no parsers,
no tools, no code.
