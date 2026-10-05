# Changelog

## 0.1.0 — initial release

- Core plugin architecture: `Metric` base class, decorator-based `Registry`,
  `MetricsEngine` with level filtering, category/id exclusion, and per-metric
  threshold overrides, all validated at construction time (typos fail loudly).
- `OntologyView`: one-time graph indexing (classes, hierarchy, properties,
  labels/comments, used-vs-declared) shared by every metric.
- 46 built-in metrics across 8 categories (size, structure, documentation,
  properties, consistency, naming, richness, metadata), split across `normal`
  (≈25, O(triples)) and `enhanced` (+≈20, including graph algorithms and
  itemized detail) levels.
- Composite 0–100 scoring with letter grade, clearly a labeled heuristic —
  raw metric values and pass/warn/fail status are the ground truth.
- Multi-format ontology loading via `rdflib` (Turtle, RDF/XML, N3,
  N-Triples, JSON-LD, TriG, N-Quads), local path or `http(s)://` URL.
- Versioned, schema-validated JSON output (`schema_version: "1.0"`) plus a
  Markdown report.
- `onto-metrics` CLI: `init`, `run`, `list-metrics`.
- Opt-in `onto_field_contract` profile for projects using that annotation
  vocabulary, with zero coupling to the `onto_field` package itself.
- 27 tests: hand-built fixture ontologies with known exact answers, config
  validation, engine error isolation, JSON Schema conformance, and full CLI
  end-to-end runs. Verified against two independently-authored real-world
  ontologies in addition to the test fixtures.
- GitHub Actions CI across Python 3.9–3.12.
