# Changelog

## [Unreleased] — v0 (draft / alpha)

The standard is being drafted; nothing is stable. This version exists to
shape the spec and test the model before any tool depends on it.

### State

- Data model: one `task` type — `title` and `status` required; `id`,
  `notes`, `tasks`, `metadata`, `version` optional. An export is one or
  more root tasks (a forest); tasks are never shared between trees.
- Design goals: title + status are all you need to understand a task at
  any level; everything else lives in `metadata` for lossless
  round-trips; hierarchy is derived from graph position, not types.
- Metadata chapter: recommended self-explanatory snake_case keys
  (`due_date`, `priority`, `tags`, `estimates`, ...) to keep tool
  metadata interoperable instead of tool-specific namespaces.
- Serializations: JSON (canonical), YAML, Markdown (frontmatter +
  checkbox tree, metadata in hidden comments), JSONL and CSV (flat,
  children as id lists), and XML in two forms — nested (ids optional,
  metadata as attributes) and flat (id references via a `tasks`
  attribute).
- Flat serializations reference children by id lists in `tasks`; roots
  carry no `id`; processors rebuild the graph by resolving id lists.
- Casing and language: top-level fields are fixed single lowercase
  English words; `metadata` is free (any keys, casing, language), with a
  preference for self-explanatory snake_case English keys — a
  preference, not a rule.
- Examples for every serialization: an empty export, a nested tree, the
  same tree in each mapping, and a multi-root forest.
- Conformance fixtures in `tests/`: valid and invalid documents for the
  nested and flat families, named by graph shape and expected depth.
  Not part of the spec itself — they exist to help developers implement
  a parser against the rules.
