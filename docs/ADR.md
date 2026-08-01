# Architectural Decision Records (ADR)

This file records the decisions that shape the OPS model and the
reasoning behind them. A proposal to change a decision must overturn
this reasoning, not just edit the specification.

## ADR-001: `id` is optional — required only where a format references by id

**Status:** Accepted.

**Context.** Trees serialize two ways: nested objects, or flat lists
that map edges by id. Nested formats need no ids; flat formats need
ids to reference nodes.

**Decision.** `id` is optional on every task. Flat formats require ids
on non-root tasks. An id, when present, is unique across the whole
graph.

**Consequences.** Nested documents stay minimal; flat documents get a
stable reference key; roots never need an id but may carry one.

## ADR-002: Only `title` and `status` are required

**Status:** Accepted.

**Context.** The model's core is a line on paper with a checkmark.
Requiring more — an id, timestamps, metadata, a version — would force
tools without such data to invent placeholder values on export, and
every other tool to guess whether the value is real.

**Decision.** Every task carries exactly `title` (string) and
`status` (enum); all other fields are optional.

**Consequences.** Any tool can always write and read a valid document
without inventing data.

## ADR-003: Binary status (`open` | `done`)

**Status:** Accepted.

**Context.** Every task manager distinguishes "not done" from "done",
but agrees on nothing else. Requiring `in_progress`, `blocked`,
`in_review`, or `backlog` would exclude tools that don't model them or
force lossy guessing on import. The two values must be short and
self-explanatory: `open` and `done` are both four characters, a simple
balanced pair — `pending`/`done` are not. This is the same
minimum-intersection argument as ADR-002, applied to statuses instead
of fields.

**Decision.** The only status values are `open` and `done`. Anything
richer is platform metadata, under a `metadata` key like `status`.

**Consequences.** Every document's core is readable by every tool
without guessing. Richer states live in `metadata`, so round-trips stay
lossless.

## ADR-004: `version` is optional and lives on the root

**Status:** Accepted.

**Context.** A version names the spec a document is modeled after.
Without one, a document follows the latest released version, so
writing task data and building tools stays easy while the specs
develop; a document can still pin the exact version.

**Decision.** `version` is optional, must be a released OPS
Specification version, and can only be set on root tasks.

**Consequences.** If no version is set and the latest version has
deprecated features the dataset uses, tools must migrate or fail. A
document can hold multiple roots with different versions.

## ADR-005: `notes` is a model field, not metadata

**Status:** Accepted.

**Context.** After the title, long-form text is the most universal
content: every tool has a description, body, or comment field. It is
the task's content, not data about the task. It can be long,
multi-line, and Markdown-formatted — awkward as an XML attribute or a
CSV cell.

**Decision.** `notes` is an optional top-level string field (plain or
Markdown).

**Consequences.** Long-form content gets an explicit, uniform mapping
in every serialization; a tool without notes omits the field.

## ADR-006: `subtasks` instead of `children`

**Status:** Accepted.

**Context.** Tools name hierarchy levels differently: epics, features,
stories, subtasks. Modeling named levels would make the standard a
union of every tool's taxonomy, so the hierarchy field must be a
neutral, self-explanatory word — candidates: `children`, `nodes`,
`tasks`, `subtasks`. `nodes` is too abstract; `children` evokes DOM or
XML parsing. The field names the objects below a task, so it
should say what those objects are and nothing about the task itself.
`tasks` fails: a task with a full `tasks` field reads as if it isn't
itself a task. Every node is a task; `subtasks` says what the field
holds without implying a separate type.

**Decision.** Hierarchy is expressed by the `subtasks` field on every
task. A task is a parent exactly when it has `subtasks`; an epic, a
feature, and a subtask are all the same node type. A root is just a
task nothing references.

**Consequences.** One recursive rule covers the whole forest — no
special types, no level names, no rules about who may be a parent.
What a platform calls a task is derived from position and metadata
hints, never enforced.

## ADR-007: `metadata` is the escape hatch

**Status:** Accepted.

**Context.** The model's field names are fixed single lowercase English
words, never renamed or translated. Tools track far more attributes
than title and status — due dates, priorities, tags, estimates,
platform statuses — and spell them differently. Standardizing each
attribute would make the model a union of every tool's taxonomy, and
new attributes keep arriving. An open field keeps tool-specific data
without standardizing it.

**Decision.** Every task may carry an optional `metadata` object;
tool- or concern-specific key-value pairs live there, preserved
verbatim. New concepts arrive as metadata, never as new top-level
fields.

**Consequences.** Round-trips stay lossless: unknown details ride along
in `metadata` instead of being dropped.

## ADR-008: metadata keys must match a charset

**Status:** Accepted.

**Context.** `metadata` is open, so key spelling decides
interoperability. Every tool spells the same concepts differently
(`dueDate`, `Due_Date`, `dd`, `fälligAm`); without an agreed spelling,
every key is opaque to every other tool. Self-explanatory snake_case
English keys cross tool boundaries, get compared and guessed by other
tools. Unconstrained keys can't be parsed by heuristics, so tools reach
for company- or tool-specific prefixes and suffixes. One constraint
fixes both: a charset — a key in plain lowercase letters, digits, and
underscores is clear enough to guess and simple enough to write.

**Decision.** Metadata keys must match `^[a-z0-9_]{3,}$` — lowercase
letters, digits, underscores, at least three characters; no dots,
dashes, or camelCase. The charset is enforced for readers and writers
alike: a document whose metadata keys break it is invalid. Flat,
self-explanatory keys are a recommendation, not a rule.

**Consequences.** Keys stay uniform, heuristic-parseable, and
self-explanatory across tools. Any key in the charset stays
expressible; keys outside it are rejected by conforming readers.

## ADR-009: `status` is a string enum, not a boolean

**Status:** Pending.

**Context.** "Done or not" is binary and could be a boolean —
`is_done: false`. But a boolean is a closed type: adding
`in_progress` or `blocked` later would change the field's type and
break every reader. A string enum carries the same binary today and
can gain values without a schema change. `status` also reads as a
state, not a predicate — `"open"` / `"done"` mirror the model's
"[ ] / [x]" line, where `is_done: false` asks the reader to negate.

**Consequences.** The binary stays explicit, and the field can grow
new values without breaking existing documents or readers.
