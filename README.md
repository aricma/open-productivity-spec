# Open Productivity Standard (OPS)

> An open, tool-agnostic data model for exchanging tasks between
> productivity tools.

## What is this?

A specification — nothing more. It defines a single **task** data structure
(an acyclic tree of tasks with metadata) that any productivity tool can
export and import. There are no epics, features, backlog items, or lists in
the model: what a platform called a task is carried in `metadata` and
preserved verbatim, so exports and imports are lossless and nothing is
locked in.

```text
[ Notion ]  ──┐                               ┌──> [ Linear ]
[ Todoist ] ──┼──> [ TASK TREE (the spec) ] ──┼──> [ Obsidian ]
[ Jira ]    ──┘                               └──> [ Anything ]
```

## Read order

1. [`SPECIFICATION.md`](SPECIFICATION.md) — the OPS Specifications: the
   standard itself.
2. [`examples/`](examples/) — the model in different serializations.
3. [`CHANGELOG.md`](CHANGELOG.md) — the specification history.
