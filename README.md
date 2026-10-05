# onto_metrics

**Modular metrics and quality scoring for RDF/OWL ontologies.** Point it at a `.ttl`/`.owl`/`.rdf`/`.n3`/`.nt`/`.jsonld`/`.trig` file and a config, get back a stable JSON (and Markdown) report: raw measurements, pass/warn/fail status per metric, and a composite 0–100 score.

```bash
pip install -e .
onto-metrics init metrics.config.yaml --ontology ./my_ontology.ttl
onto-metrics run metrics.config.yaml
```

## Why

Most ontology tooling either reasons about correctness (Protégé, reasoners) or says nothing quantitative at all. `onto_metrics` sits in between: syntactic, fast, no reasoner required, and the output is a stable, versioned JSON schema suitable for a CI gate, a dashboard, or a one-off quality check before you ship an ontology.

**Metrics are measurements. Quality is interpretation.** Every metric reports its raw value regardless of whether you agree with the threshold applied to it. The composite score is a labeled heuristic for convenience, not a claim of ground truth.

## Quickstart

```bash
pip install -e ".[dev]"
cd examples/client
python run_metrics.py          # or: onto-metrics run metrics.config.yaml
```

This measures the bundled example ontology (`examples/ontologies/finance_ontology.ttl`) at `enhanced` level and writes `ontology_metrics.json` + `ontology_metrics.md` to `./out`.

## Levels

| | `normal` (default) | `enhanced` |
|---|---|---|
| Metrics | ~25, all O(triples) | +~20 more, including graph algorithms and itemized detail |
| Per-metric detail | value, status, message | also `details`: offending items, distributions, top-N lists |
| Cost | seconds, any size | slower on very large ontologies |

Run `onto-metrics list-metrics` to see every metric, its category, and which level it belongs to.

## Config

```yaml
ontology:
  path: ./ontology.ttl        # local path (resolved relative to the config file) or http(s):// URL
  format: auto                 # auto | turtle | xml (.owl/.rdf) | n3 | nt | json-ld | trig

metrics:
  level: normal                 # normal | enhanced
  include_categories: []        # optional allow-list, e.g. ["documentation", "structure"]
  exclude: []                    # metric ids, "PREFIX." groups, or category names
  thresholds: {}                  # per-metric warn/fail/target overrides
  profiles: []                     # opt-in profiles, e.g. ["onto_field_contract"]
  max_details: 50

output:
  dir: ./out
  metrics_file: ontology_metrics.json
  markdown_file: ontology_metrics.md   # set to null / omit to skip the Markdown report
```

Full schema: [`docs/CONFIG_REFERENCE.md`](docs/CONFIG_REFERENCE.md).

## Output

```json
{
  "schema_version": "1.0",
  "generated_at": "2026-...Z",
  "tool": {"name": "onto_metrics", "version": "0.1.0"},
  "ontology": {"path": "...", "format": "turtle", "sha256": "...", "triples": 797},
  "run": {"level": "enhanced", "metrics_run": 49, "metrics_errored": 0, "duration_ms": 12.3},
  "summary": {"overall_score": 93.8, "grade": "A",
              "dimensions": {"documentation": 62.9, "structure": 100.0, "...": "..."},
              "status_counts": {"pass": 16, "warn": 1, "info": 35}},
  "metrics": [{"id": "DOC.label_coverage", "value": 100.0, "unit": "percent",
               "status": "pass", "score": 100.0, "message": "18 of 18 entities have an rdfs:label."}]
}
```

Metric `id`s are permanent — a dashboard built against them won't break between versions. The full schema is validated by the test suite: [`src/onto_metrics/report/schema/metrics_report.schema.json`](src/onto_metrics/report/schema/metrics_report.schema.json).

## Extending: write your own metric

```python
from onto_metrics.core import Category, Measurement, Metric, Threshold, register_metric

@register_metric
class MyMetric(Metric):
    id = "CUSTOM.my_check"              # must look like "PREFIX.snake_case"
    name = "My check"
    category = Category.DOCUMENTATION
    unit = "percent"
    description = "What this measures."
    threshold = Threshold.higher(warn=80, fail=50, target=100)

    def compute(self, view, ctx):
        # `view` is a pre-indexed OntologyView: view.classes, view.labels,
        # view.subclass_of, view.property_domain, etc. — see docs/ARCHITECTURE.md
        return Measurement(value=..., message="...", details={...} if ctx.detailed else None)
```

Import your module before running the engine (or register it as an `onto_metrics.metrics` entry point) and it's picked up automatically — no registry file to edit. See [`docs/EXTENDING.md`](docs/EXTENDING.md).

## Project layout

```
onto_metrics/
├── src/onto_metrics/
│   ├── core/          base types (Level, Category, Status, Threshold, Metric, Registry, Engine)
│   ├── view.py         OntologyView — one-time graph indexing shared by every metric
│   ├── loader.py        multi-format ontology loading (rdflib)
│   ├── config.py         YAML/JSON config schema + loader
│   ├── metrics/           built-in metrics, one module per category
│   ├── profiles/           opt-in metric packs (e.g. onto_field_contract)
│   ├── scoring.py          composite 0-100 scorecard
│   ├── report/              Report dataclass, JSON writer, Markdown writer, JSON Schema
│   └── cli.py                onto-metrics command line
├── examples/client/      sample script + config
├── examples/ontologies/   bundled example ontology for the sample client
├── tests/                 unit + end-to-end tests, including fixtures with known answers
└── docs/                  architecture, metrics catalog, config reference, extending guide
```

## Honest limits

- **No reasoner.** Syntactic analysis only — it will not find an unsatisfiable class or an inferred contradiction.
- **Named classes only.** Hierarchy metrics use named `rdfs:subClassOf` edges; anonymous OWL restrictions are counted but not structurally analyzed.
- **Heuristic scoring.** Default thresholds are reasonable starting points, not an industry standard — override them per metric in the config, and treat the raw `value`/`details` as the ground truth.

## Development

```bash
pip install -e ".[dev]"
pytest tests/ -v
```

## License

MIT — see [`LICENSE`](LICENSE).

## References to Ontologies
For trying out this library you may explore your own ontologies or sites like the [Common Core Ontologies](https://github.com/CommonCoreOntology/CommonCoreOntologies)