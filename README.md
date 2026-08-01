# Open Productivity Specification (OPS)

> An open, tool-agnostic data model for exchanging tasks between
> productivity tools.

## What this is

A specification — nothing more. It defines a single **task** data structure
(an acyclic tree of tasks with metadata) that any productivity tool can
export and import. There are no epics, features, backlog items, or lists in
the model: what a platform called a task is carried in `metadata` and
preserved verbatim, so exports and imports are lossless and nothing is
locked in.

```text
[ Notion ] ──┐                                ┌──> [ Linear ]
[ Todoist ] ──┼──> [ TASK TREE (the spec) ] ──┼──> [ Obsidian ]
[ Jira ]   ──┘                                └──> [ Anything ]
```

## Repository layout

```text
todo-specs/
├── README.md           # This file — overview
├── SPECIFICATION.md    # The standard: task data model, rules, serializations
├── CHANGELOG.md        # What changed, per draft
├── LICENSE             # MIT — the spec is free to use, implement, and fork
└── examples/           # Some valid examples per serialization mapping
                        # (see examples/README.md)
```

## Read order

1. [`SPECIFICATION.md`](SPECIFICATION.md) — the standard itself.
2. [`examples/`](examples/) — the model in each serialization.
3. [`CHANGELOG.md`](CHANGELOG.md) — the draft history.

## License

[MIT](LICENSE).
