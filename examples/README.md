# Examples

Every file in this folder is a **valid** document per
[`SPECIFICATION.md`](../SPECIFICATION.md).

This is not an exhaustive catalog: we show some valid examples per
serialization, not every shape. There are **no invalid examples** here —
deliberately. If you want to test a parser's error handling, construct
invalid documents yourself from the rules in the spec.

## The shared document

Most files carry the same content, so the mappings are comparable. The
nested XML example drops ids on purpose (see its section); the multi-root
pair adds a second tree.

- Root task: `Acme product backlog` (`id: export-root`)
- `t1` — Fix memory leak in auth service (open)
  - `t1r` — Reproduce the leak (leaf; carries only `metadata`)
  - `t1a` — Ship the fix (open; carries `linear` metadata)
- `metadata` on `t1` mixes generic keys (`priority`, `dueDate`, `tags`)
  and a tool key (`notion`)

## File by file

### `empty.json`

The minimal valid export: a root task with only `version` and `title`.
Demonstrates rule 8 — a root task with no `tasks` is a valid export
whether it stands for an empty export or a single task — and that the root
needs no `id`.

### `tree.json`

Canonical JSON, the reference form: one task object per node, children
nested in `tasks`. Demonstrates:

- root with `version` and an `id` it doesn't need (ids are optional in
  tree serializations)
- a leaf child (`t1r`) that carries no `status` and no `tasks`
- nested `metadata` (task-level `linear`, parent-level `notion`)

### `yaml.yaml`

The same document in YAML. Nesting is expressed by indentation, not
brackets; the mapping is otherwise identical to JSON. Note `dueDate` is
quoted so it stays a string instead of parsing as a YAML date.

### `jsonl.jsonl`

The tree linearized: one task per line, one line per record. The root has
no `id` and its `tasks` lists the ids of its children; every child record
carries its own `id` and lists its own children the same way. A streaming
format — a consumer processes records line by line and rebuilds the tree
by resolving the id lists, no parent references needed.

### `csv.csv`

The tree as rows. Columns: `id,title,status,notes,metadata,tasks`.
`metadata` and `tasks` are JSON-encoded inside their columns (hence the
doubled quotes). Empty cells mean the field is absent, not an empty
string. Resolution works exactly like JSONL: roots have no `id`; child
rows carry ids and list their children's ids in `tasks`.

### `markdown.md`

Frontmatter carries `version`; the `#` heading renders the root `title`
and the leading paragraph its `notes`. The tree is a nested checkbox list:
`[ ]` = `open`, `[x]` = `done` ("Ship the fix" is `[x]` here to show it).
Per-task `metadata` is preserved in hidden HTML comments so round-trips
stay lossless.

### `xml.xml`

Tree-preserving like JSON: a `<task>` element per node, children as
nested `<task>` elements. No ids anywhere — the nested form doesn't need
them. `metadata` uses attributes for simple values (`priority`, `due_date`)
and child elements for complex ones — comma-separated for lists like
`tags`, JSON text for objects like `notion` and `linear` — so nothing is
lost without a single JSON blob.

### `xml-flat.xml`

The same tree as a flat list: all tasks sit under `<tasks>`, children are
referenced by id via a `tasks` attribute (space-separated, like the
JSONL/CSV id lists), and the root carries no id. Metadata uses the same
attribute style as `xml.xml`.

### `multi-root.jsonl` / `multi-root.csv`

Two root tasks in one file — "Work" and "Personal" — a forest. Both roots
carry no `id`; each lists its children's ids in `tasks`. No task is shared
between trees: every task has exactly one parent in the whole file, and
ids are unique across it. This is how a flat export of several lists (or
several tools' data merged) can be streamed as one file.
