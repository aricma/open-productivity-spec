# Tests

Conformance fixtures for the OPS Specifications
(`../SPECIFICATION.md`): documents that a compliant tool must accept or
reject. Write your parser against these states, not against guesswork.

Two families, mirroring the serializations chapter:

- [`nested/`](nested/) — tree-preserving formats (canonical JSON):
  hierarchy via nested `tasks` objects.
- [`flat/`](flat/) — flat, streamable formats (canonical JSONL):
  hierarchy via child-id lists in `tasks`.

Each family has:

- `valid/` — documents that MUST parse and import cleanly.
- `invalid/` — documents that MUST be rejected (each breaks exactly one
  rule).

### Naming

Every case states the expected depth in its name — for valid and invalid
cases alike:

- `<count>-<roots>-with-<n>-levels-of-tasks` — depth is the task levels
  *below* the root; a root alone is `zero-levels`. For multi-root cases
  it is the maximum depth across roots.
- Valid: `<count>-<roots>-with-<n>-levels-of-tasks[-with-<feature>]`,
  with features like `id`, `ids`, `version`, `version-override`,
  `rich-metadata`, `mixed-statuses`.
- Invalid: same shape pattern, with the broken rule as the feature
  (`with-missing-title`, `with-shared-child`, `with-cyclic-reference`).
- Examples: `one-open-root-with-zero-levels-of-tasks`,
  `one-root-with-two-levels-of-tasks-with-higher-leaf-version`,
  `two-roots-with-one-level-of-tasks-with-shared-child`.

The pattern is open-ended — extend it the same way instead of inventing
new schemes.

### Adding fixtures

- One scenario per file; the name states what it demonstrates.
- Valid fixtures must not depend on lenient readers; invalid fixtures
  must not depend on strict ones beyond the rule they break.
- Keep the canonical format per family (JSON for nested, JSONL for flat);
  other serializations are covered by the examples folder.

### Version cases

`version` exists per rule 6 of the specs, but only `"0"` is real today. The fixtures
under `one-root-with-*-version` are theoretical: they pin the mechanism
down so parsers agree on it before later versions exist.

- `one-root-with-zero-levels-of-tasks-with-version` — a root declaring
  `version` for the whole graph.
- `one-root-with-one-level-of-tasks-with-version-override` — a node
  overwrites the version for itself and its subtree (the root stays on
  `"0"`, the child moves to `"1"`).
- `one-root-with-two-levels-of-tasks-with-higher-leaf-version` — a
  leaf on a higher version than its ancestors; parents must not clamp or
  reject it.
- No version anywhere (e.g. `one-open-root-with-zero-levels-of-tasks`) —
  the latest/highest version applies by default.

The effective version of any task is the nearest `version` on the path
from the task to the root; a parent's version is the default for parsing
its children.

### Attachments

No test cases for attachments yet. Attachments are metadata-only (rule
8), and a portable fixture needs a concrete packaging decision (relative
paths inside a ZIP, how to point at them). We will add fixtures on
request when the first tool actually needs to exchange files.
