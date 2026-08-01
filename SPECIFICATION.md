---
status: draft
version: "0"
date: 2026-08-01
author: Adrian Mindak (aricma) <adrian@aricma.org>
---

# OPS Specifications

This document is the **OPS Specifications**: the specification of the
Open Productivity Standard (OPS).

A minimal, tool-agnostic data structure for exchanging tasks between
productivity tools. Any tool can export its tasks, any other tool can
import them. Platform-specific data that this model doesn't capture lives
in `metadata`, so nothing is lost in transit.

## The model

One type: a **task**. A task may contain other tasks in `tasks`, nested
arbitrarily deep. An export is one or more root tasks; every task descends
from exactly one root. There are no other concepts — no epics, features,
backlog items, or lists. What a platform called the task is carried in
`metadata`.

Nesting makes every graph a tree: acyclic by construction, each task has
exactly one parent, and no task is shared between roots. Multiple roots
form a forest; each tree stands alone.

## Design goals

At its core, a task is a written line on a piece of paper with a checkmark
next to it. The standard keeps exactly that: `title` is the line — it must
explain itself — and `status` says whether it's done. Those two are the
only required fields because they're the only information you need to
understand a task at any level.

Everything above that is structure the reader brings. More tasks make a
project; sort tasks by belonging and you have features; group features and
you have an epic. This game of nesting can be played to infinity, which is
exactly why the standard defines no such levels — only tasks, nested.

Everything beyond title and status — due dates, priorities, tags,
estimates, platform statuses — is real data, but every tool spells it
differently. Rather than standardize each one (and either lose the
differences or argue forever about what "high priority" means), the
standard keeps them in `metadata`, preserved verbatim.

This buys two things:

- **Lossless round-trips.** Exporting and re-importing into the same tool
  should lose nothing. Across different tools, some loss may be
  unavoidable — but aim for none: keep `metadata` around even when you
  don't understand it, and use self-explanatory keys (`due_date`,
  `status`, `priority`, `estimates`, `location`, `attachments`) so the
  next tool can guess their meaning.
- **The model stays the intersection, not the union.** As tools gain
  features, the standard doesn't have to grow. New concepts arrive as
  metadata, not as new top-level fields.

Hierarchy is a reading of the graph, not a type. What a platform calls a
task — epic, feature, PBI, story — can be derived from a task's position in
the tree (projects and epics tend to be near the top, leaf tasks are the
concrete work) and from hints in `metadata` — keys like `type`, `_type`, or
`kind` that the source tool attached. These heuristics are allowed but
never required: the standard defines no types, so no tool is wrong for not
knowing one.

## Fields

| Field      | Type                   | Required | Notes |
|------------|------------------------|----------|-------|
| `title`    | string                 | yes      | The human-readable summary: the line on the paper. |
| `status`   | `"open"` / `"done"`    | yes      | `open` or `done`. |
| `id`       | string                 | no*      | Identifier for formats that reference tasks by id. |
| `notes`    | string                 | no       | Long-form text, plain or Markdown. |
| `tasks`    | task[]                 | no       | Nested task objects or child-id references, per serialization. |
| `metadata` | object                 | no       | Tool- or concern-specific data, keyed by tool or concern. |
| `version`  | string                 | no       | The standard version a task or subtree uses. |

`no*` means conditionally required — see the rules.

## Rules

1. **Required fields** — `title` and `status` appear on every task.
2. **`id` and uniqueness** — ids are only needed when a format
   references tasks by id (the flat serializations, where `tasks` holds
   child ids): then every task except roots carries a unique id, unique
   across the whole graph. Nested serializations never need ids. Roots
   never need an id in any format, though one may be attached anyway.
3. **`status` values** — only `open` or `done`. Anything richer (in
   progress, backlog, "done by definition") is platform metadata: store
   it in `metadata`.
4. **`tasks` define hierarchy** — a task belongs to exactly one parent,
   across the whole export. Tasks are never shared between roots — a
   task exists in exactly one place. Cycles are impossible.
5. **`metadata` is the escape hatch** — the only place for data outside
   the model: due dates, tags, timestamps, priority, estimates, custom
   properties, platform statuses. Anything the model doesn't cover goes
   in `metadata` (prefer the recommended keys in the Metadata chapter);
   never invent new top-level fields — that would reinvent the model, and
   model changes belong in this spec, not in documents.
6. **`version` resolution** — optional on any task. It applies to that
   task and its whole subtree unless a deeper task overrides. A task's
   effective version is the nearest `version` on the path to the root;
   if none exists, the latest version applies.
7. **Empty export** — a root task with no tasks is a valid export,
   whether it represents an empty export or a single standalone task —
   the structure is identical.
8. **Attachments** — files are not a model field; they live in
   `metadata` under a self-explanatory key such as `attachments`, as
   paths — relative to the export when the files travel with the data
   (the ZIP case), or global (a URL for remote files, a path on the
   machine). An export that carries files should be
   packaged as a ZIP holding the task data and the files together, so a
   receiving tool can re-upload them and reconnect the paths. Attachments
   are a use case, not the norm: a tool that receives a graph whose
   `metadata` points at files it was not given may discard those
   references — no tool is required to handle attachments.
9. **Field names and metadata keys** — the model's fields are the fixed
   words in the Fields chapter: single lowercase English words, never
   renamed or translated. Reading and writing are asymmetric:
   `metadata` is read leniently — a document is never invalid for using
   any keys, casing, or language, and readers must accept it all — but
   written strictly: keys a tool writes must match `^[a-z0-9_]{3,}$`
   (lowercase letters, digits, and underscores, at least three
   characters; no dots, dashes, camelCase, or other scripts).

## Metadata

`metadata` is open — anyone may add keys. But the point of a
shared standard is that the keys are shared too: where a concept is
common, use the recommended keys below instead of inventing a tool- or
company-specific namespace. Otherwise every tool spells the same thing
differently (`dueDate`, `due_date`, `dd`, `fälligAm`) and imports and
exports quietly become incompatible.

Recommended keys — all optional, snake_case (rule 9):

| Key            | Value |
|----------------|-------|
| `due_date`     | ISO 8601 date |
| `start_date`   | ISO 8601 date |
| `priority`     | free-form string (tool scales differ) |
| `tags`         | list of strings |
| `status`       | the platform's original status (e.g. "In review") — distinct from the model's `status` field |
| `estimates`    | number or string (tool units differ) |
| `location`     | string |
| `assignee`     | string (name or id) |
| `attachments`  | list of paths — relative to the export, or global (a URL, a path on the machine) |
| `created_at`   | ISO 8601 timestamp |
| `updated_at`   | ISO 8601 timestamp |
| `completed_at` | ISO 8601 timestamp |
| `url`          | string, link to the task in its source tool |

Use the exact spellings above, not close variants — a reader that
recognizes a recommended key can act on it; an unrecognized one is opaque.
Tool-specific keys are welcome, but namespace them by tool name (e.g.
`linear`, `notion`) so they never collide with generic keys. Keys like
`type`, `_type`, or `kind` carry role hints for the heuristics described
in the design goals.

## Serializations

The model can be carried in any format that tools already speak. These
are general guidelines, not a closed list: the standard does not mandate
a single format, and any mapping that stays lossless and unambiguous is
welcome.

Formats fall into two families, and the use case decides which to pick:

- **Tree-preserving** — JSON, YAML, Markdown, XML. `tasks` holds nested
  task objects, so the graph structure is explicit and recursive
  traversal is straightforward. Best when the whole tree lives in memory
  anyway.
- **Flat and streamable** — JSONL, CSV. `tasks` holds child ids and each
  record appears on its own line/row, so a processor handles one record
  at a time and rebuilds the graph by resolving id lists. Best for large
  exports, logs, and pipe-style processing — not every format is
  data-transfer efficient, and these exist to be.

Some formats can play both roles: XML nests children as `<task>`
elements, or lists them flat with child ids in a `tasks` attribute — the
nested form needs no ids at all.

How `metadata` is encoded is per-format and must not be assumed: simple
values may become attributes (XML) or plain keys (YAML), lists may be
comma-separated (XML), complex values may be JSON-encoded (CSV, XML
objects). Readers must not assume one encoding.

All examples live in [`examples/`](examples/) (see
[`examples/README.md`](examples/README.md) for what each file shows).

## Conformance

How can software claim OPS conformance? In the following, we look at
what a reader, writer, and transformer must do. The obligations below
apply to whatever formats a tool claims to support.

**OPS Reader** — imports OPS documents.

- Must accept every valid document in its claimed formats (rules 1–9).
- Must not fail on or strip unknown metadata — including keys that
  break rule 9's charset.
- Must reject a document for an unknown `version`.

**OPS Writer** — exports OPS documents.

- Must output only valid documents following the OPS Specifications.
- Must write only data following a released OPS version.
- Must write metadata keys per rule 9: `^[a-z0-9_]{3,}$`.

**OPS Transformer** — imports and exports (converters, sync tools,
round-trippers).

- Everything the Reader and Writer must do.
- Must preserve the graph: the same tasks with the same titles, notes,
  statuses, hierarchy, and ids where the format carries them.
- Must preserve `metadata` verbatim across import → export cycles — keys
  and values, understood or not. Normalizing, translating, or cleaning
  up unrecognized metadata is a loss, and lossless round-trips are the
  point of the standard.

Conformance never means understanding metadata. No tool is required to
know what another tool's metadata means, to handle attachments (rule 8),
or to support every serialization — only to carry metadata along,
unmodified.

The fixtures in [`tests/`](tests/) express rules 1–9 as concrete valid
and invalid documents, one per rule and shape; they are the working
definition of "valid" for implementers.
