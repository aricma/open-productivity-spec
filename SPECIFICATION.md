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
import them.

## The model

One type: the **task**. A task may contain other tasks, nested
arbitrarily deep. An export is one or more root tasks; every task descends
from exactly one root. There are no other concepts — no epics, features,
backlog items, or lists. What a platform calls a task is carried in
`metadata`.

## Design goals

At its core, a task is a written line on a piece of paper with a checkmark
next to it.

```plaintext
[x] Buy Milks
[ ] Clean Room
[ ] Find Purpose
```

The standard keeps exactly that: `title` is the line — it must
explain itself — and `status` says whether it's done. Those two are the
only required fields because they're the only information you need to
understand a task at any level.

Everything else is metadata the reader brings. More tasks make a
project; group tasks and you have features; group features and
you have an epic. This game of nesting can be played to infinity, which is
exactly why the standard defines **no** such levels — only tasks, nested.

Everything beyond title and status — due dates, priorities, tags,
estimates, platform statuses — is real data, but every tool spells it
differently. Rather than standardize each one, the
standard keeps them as metadata, preserved verbatim.

This enables us to get three things:

- **Interoperability** — one open task structure that any productivity
  tool can export and import, so tasks travel between tools without
  losing title, status, notes, or hierarchy. Keep `metadata` around even
  when you don't understand it, and use self-explanatory keys
  (`due_date`, `status`, `priority`, `estimates`, `location`,
  `attachments`) so the next tool can guess their meaning. The model is
  the intersection of what tools share, not the union: new concepts
  arrive as metadata, never as new top-level fields.
- **Abstraction** — the structure is open and stable, so tools can
  expose their tasks through OPS-based APIs, and automations and
  analysis are written once against the OPS shape and work across every
  OPS-capable tool, instead of being re-implemented per platform.
- **Ownership** — your productivity data stays yours. Exports are
  lossless: re-importing into the same tool loses nothing, across
  tools the aim is none either — unknown details ride along in
  `metadata` instead of being dropped, so nothing stops you from
  leaving or returning to a tool when you want to.

Hierarchy is a reading of the graph, not a type. What a platform calls a
task — epic, feature, PBI, story — can be derived from a task's
position in the tree (projects and epics tend to be near the top, leaf
tasks are the concrete work) and from hints in `metadata` — keys like
`type` or `kind` that the source tool attached.
These heuristics are allowed but never required: the standard defines no
types, so no tool is wrong for not knowing one.

## Fields

| Field      | Type                   | Required | Notes |
|------------|------------------------|----------|-------|
| `title`    | string                 | yes      | The human-readable summary: the line on the paper. |
| `status`   | `"open"` / `"done"`    | yes      | `open` or `done`. |
| `version`  | string                 | no*      | The version the graph is modeled after. |
| `id`       | string                 | yes*     | Identifier for formats that reference tasks by id. |
| `notes`    | string                 | no       | Long-form text, plain or Markdown. |
| `subtasks` | task[]                 | no       | Nested task objects or child-id references, per serialization. |
| `metadata` | object                 | no       | Tool- or concern-specific data, keyed by tool or concern. |

`yes*` means required under conditions; `no*` means optional but
conditional — an asterisk always points at the rules.

## Rules

1. **Required fields** — `title` and `status` appear on every task.
2. **`id` and uniqueness** — task ids are required under conditions:
   only formats that reference tasks by id need them. Ids have to be
   unique across the whole graph. Nested serializations never need ids.
   Roots never need an id in any format. Though optional, ids can be
   attached anyway.
3. **`status` values** — can only be `open` or `done`.
4. **`subtasks` define hierarchy** — a task can have subtasks and
   exactly one parent. Tasks build an acyclic directed graph.
5. **`metadata` is the escape hatch** — anyone may add any key-value
   pairs, but keys must match `^[a-z0-9_]{3,}$` (lowercase letters,
   digits, and underscores, at least three characters; no dots, dashes,
   camelCase, etc.).
6. **`version` resolution** — optional; only root tasks may carry it.
   Each root defines the version its subtree is modeled after, and a
   version never appears below a root — different roots may declare
   different versions. It must be a released version of the OPS
   Specifications. If none is set, the latest released version applies.
7. **Attachments** — attachments are meant to be added as paths in
   `metadata` and resolved at the next import.
8. **Empty export** — a root task with no subtasks is a valid export,
   whether it represents an empty export or a single standalone task —
   the structure is identical.
9. **Field names** — the top-level field names are fixed: single
   lowercase English words, never renamed or translated.

## About metadata
*Common metadata keys and general recommendations*

`metadata` is open — anyone may add key-value pairs, as long as the
keys match the character set of rule 5. We recommend keeping them
flat: it keeps the metadata simple, self-explanatory, and readable.
The specifications define a task with very few required fields. This
allows us to keep the interaction with the data simple, fast, and
direct. However, many productivity tools already have a long list of
other information that they track. To support these and to make tool
interoperability simple, we want to give a few suggestions for common
attributes and how to spell them.

Recommended keys for the most common task attributes in the wild:

| Key            | Value |
|----------------|-------|
| `due_date`     | ISO 8601 date |
| `start_date`   | ISO 8601 date |
| `priority`     | free-form string (tool scales differ) |
| `tags`         | list of strings |
| `status`       | the platform's original status (e.g. "In review") — distinct from the model's `status` field |
| `estimates`    | number or string (tool units differ) |
| `location`     | string |
| `assignee`     | string naming a team member (name, id, or email) |
| `attachments`  | list of paths — relative to the export, or global (a URL, a path on the machine) |
| `created_at`   | ISO 8601 timestamp |
| `updated_at`   | ISO 8601 timestamp |
| `completed_at` | ISO 8601 timestamp |
| `url`          | string, link to the task in its source tool |

If teams follow the convention of using these attributes, parsing
tasks across tools will already be much simpler.

## Serializations

The model can be carried in any format that tools already speak. These
are general guidelines, not a closed list: the standard does not mandate
a single format, and any mapping that stays lossless and unambiguous is
welcome.

Formats fall into two families, and the use case decides which to pick:

- **Tree-preserving** — JSON, YAML, Markdown, XML. `subtasks` holds
  nested task objects, so the graph structure is explicit and recursive
  traversal is straightforward. Best when the whole tree lives in memory
  anyway, and computation uses graph traversal algorithms for
  performant results.
- **Flat and streamable** — JSONL, CSV. `subtasks` holds child ids and
  each record appears on its own line, so a processor handles one record
  at a time and rebuilds the graph by resolving id lists. Best for large
  exports, logs, and pipe-style processing — not every format is
  data-transfer efficient, and these exist to be streamable and to
  perform local, context-based transformations in a very efficient way.

All examples live in [`examples/`](examples/) (see
[`examples/README.md`](examples/README.md) for what each file shows).

## Conformance

How can software claim OPS conformance? In the following, we look at
what a reader, writer, and transformer must do. The obligations below
apply to whatever formats a tool claims to support.

**OPS Reader** — imports OPS documents.

- Must accept every valid document following the OPS Specifications.
- Must reject every invalid document claiming to follow the OPS
  Specifications.

**OPS Writer** — exports OPS documents.

- Must output only valid documents following the OPS Specifications.

**OPS Transformer** — imports and exports (converters, sync tools,
round-trippers).

- Everything the Reader and Writer must do.
- Must preserve the number and relation of all nodes and graphs per dataset.
- Must preserve all valid `metadata` verbatim across import → export cycles.

Conformance never means understanding metadata. No tool is required to
know what another tool's metadata means or to support every serialization.

The fixtures in [`tests/`](tests/) express rules 1–9 as concrete valid
and invalid documents, one per rule and shape; they are the working
definition of "valid" for implementers.
