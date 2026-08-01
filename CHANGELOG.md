# Changelog

## v0 — draft, unreleased

OPS is a draft. No version is released, nothing is stable, and the
model may change before the first release.

### Added

- First draft of the OPS Specifications (`SPECIFICATION.md`): the task
  model (`title`/`status` required, everything else optional), the
  nine rules, common metadata keys, serializations, and conformance
  obligations for readers, writers, and transformers.
- First draft of the decision records (`docs/ADR.md`): the reasoning
  behind each model decision (ADR-001–009; ADR-009 pending).
- Examples of every serialization (`examples/`): an empty export, a
  nested tree, and a multi-root forest, to support future tool
  development.
- Conformance test fixtures (`tests/`): valid and invalid documents
  for the nested and flat families, to support future tool
  development and documentation efforts. Every invalid fixture breaks
  exactly one rule.
- Everything under the MIT license for now (see `LICENSE`).
