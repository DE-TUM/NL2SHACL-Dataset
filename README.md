# NL2SHACL-Dataset

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.20082565.svg)](https://doi.org/10.5281/zenodo.20082565)


A benchmark dataset of natural language to SHACL translation pairs, introduced as part of [NL2SHACL-Bench](https://de-tum.github.io/NL2SHACL-Bench). Each record pairs a natural language description of data constraints with a reference SHACL shapes graph and the relevant ontology terms, enabling systematic evaluation of NL-to-SHACL translation systems.

---

## Dataset Statistics

| Subset | # Records | # Node Shapes | # Property Shapes | Avg. NL Length | Avg. Shapes per Record |
|--------|----------:|-------------:|------------------:|---------------:|-----------------------:|
| CHEMROF | 74 | 74 | 441 | 154.5 | 6.96 |
| DCAT | 20 | 35 | 103 | 116.2 | 6.90 |
| ePO | 50 | 50 | 139 | 116.5 | 3.78 |
| Invoice | 78 | 78 | 113 | 75.1 | 2.45 |
| SNIK | 8 | 11 | 49 | 52.0 | 7.50 |
| DBpedia | 10 | 10 | 11 | 9.5 | 2.10 |
| **Overall** | **240** | **258** | **856** | **108.1** | **4.64** |

---

## Repository Structure

```
NL2SHACL-Dataset/
├── chemrof-dataset/
│   ├── chemrof-descriptions.jsonl
│   ├── chemrof-ontology_snippets.jsonl
│   └── shacl/
│       └── chemrof-1.ttl, chemrof-2.ttl, ...
├── dcat-dataset/
│   ├── dcat-descriptions.jsonl
│   ├── dcat-ontology_snippets.jsonl
│   └── shacl/
│       └── dcat-1.ttl, dcat-2.ttl, ...
├── epo-dataset/
│   ├── epo-descriptions.jsonl
│   ├── epo-ontology_snippets.jsonl
│   └── shacl/
│       └── epo-1.ttl, epo-2.ttl, ...
├── invoice-dataset/
│   ├── invoice-descriptions.jsonl
│   ├── invoice-ontology_snippets.jsonl
│   └── shacl/
│       └── invoice-1.ttl, invoice-2.ttl, ...
├── snik-dataset/
│   ├── snik-descriptions.jsonl
│   ├── snik-ontology_snippets.jsonl
│   └── shacl/
│       └── snik-1.ttl, snik-2.ttl, ...
└── dbpedia-dataset/
    ├── dbpedia-descriptions.jsonl
    ├── dbpedia-ontology_snippets.jsonl
    └── shacl/
        └── dbpedia-1.ttl, dbpedia-2.ttl, ...
```

Each subset folder contains three files:

- `*-descriptions.jsonl` — natural language descriptions, one record per line
- `*-ontology_snippets.jsonl` — ontology term definitions referenced by the shapes, one record per line
- `shacl/` — reference SHACL shapes in Turtle (`.ttl`) format, one file per record

Records across the three files are linked by a shared `id` field (e.g., `snik-1`).

---

## Data Format

### descriptions.jsonl

Each line is a JSON object with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique record identifier (e.g., `"snik-1"`) |
| `label` | string | Name of the node shape (e.g., `"meta:EntityTypeShape"`) |
| `description` | string | Natural language description of the constraints |

**Example:**
```json
{
  "id": "snik-1",
  "label": "meta:EntityTypeShape",
  "description": "Every entity type must have at least one human-readable label and is strictly limited to a specific set of allowed attributes. ..."
}
```

### ontology_snippets.jsonl

Each line is a JSON object with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique record identifier, matching the descriptions file |
| `label` | string | Name of the node shape |
| `ontology_snippet` | object | A dictionary keyed by ontology term URIs, each with `source`, `types`, `label`, and optionally `description` |

**Example:**
```json
{
  "id": "snik-1",
  "label": "meta:EntityTypeShape",
  "ontology_snippet": {
    "http://www.snik.eu/ontology/meta/EntityType": {
      "source": "local",
      "types": ["http://www.w3.org/2002/07/owl#Class"],
      "label": "entity type",
      "description": "An entity type is any kind of information that is consumed, produced or modified by a task."
    }
  }
}
```

### shacl/*.ttl

Each `.ttl` file contains the reference SHACL shapes graph for one record, serialized in Turtle format. The filename corresponds to the `id` field in the `.jsonl` files.

**Example (`snik-1.ttl`):**
```turtle
meta:EntityTypeShape a sh:NodeShape ;
    sh:closed true ;
    sh:targetClass meta:EntityType ;
    sh:property [ sh:path rdfs:label ] ;
    ...
```

---

## Subsets

| Subset | Domain | Description |
|--------|--------|-------------|
| **CHEMROF** | Chemistry | Models chemical entities and their relationships, from atoms to complex mixtures. Constraints include closed-world restrictions, mandatory identifiers, enumerated values, and hierarchical classifications. |
| **DCAT** | Open Data Metadata | Based on the DCAT-AP vocabulary for European data portals. Constraints cover metadata properties such as license, language, and publisher, often linking to controlled vocabularies. Includes nested constraints via `sh:node`. |
| **ePO** | Public Procurement | Based on the eProcurement Ontology (EU). Covers multiple procurement stages including ordering, invoicing, and payment. Constraints vary widely in complexity. |
| **Invoice** | Electronic Invoicing | Based on the EDIFACT standard for invoicing data. Constraints cover required fields, data formats, and identifier structures, with `sh:or` and `sh:not` used for alternative and negation constraints. |
| **SNIK** | Healthcare Information Management | Models healthcare information systems using a meta-level ontology. Constraints focus on class assignments and target declarations, resulting in structurally simple but semantically expressive shapes. |
| **DBpedia** | General (Cross-domain) | Curated using the DBpedia ontology. Covers a broad range of constraint types including existence, cardinality, datatype, logical, and conditional rules. Natural language descriptions are derived from existing `sh:description` annotations. |

---

## Usage

Records across the three files in each subset are linked by `id`. To load and align a subset in Python:

```python
import json

def load_subset(prefix):
    with open(f"{prefix}-descriptions.jsonl") as f:
        descriptions = {json.loads(l)["id"]: json.loads(l) for l in f}
    with open(f"{prefix}-ontology_snippets.jsonl") as f:
        ontologies = {json.loads(l)["id"]: json.loads(l) for l in f}
    return descriptions, ontologies

descriptions, ontologies = load_subset("snik-dataset/snik")

record_id = "snik-1"
nl_description = descriptions[record_id]["description"]
ontology_snippet = ontologies[record_id]["ontology_snippet"]

# Load the corresponding SHACL shapes graph
with open(f"snik-dataset/shacl/{record_id}.ttl") as f:
    shacl_graph = f.read()
```

---

## License

This dataset is released under the [Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/) license.

Individual subsets are derived from source materials with their own licenses. Please refer to the original sources listed in the paper for details.

---

## Citation

If you use this dataset, please cite:

```bibtex
@dataset{zhou2025nl2shacl,
  author    = {Zhou, Yuchen and Bobet, Niels and Acosta, Maribel},
  title     = {NL2SHACL-Dataset},
  year      = {2025},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.20082565},
  url       = {https://doi.org/10.5281/zenodo.20082565}
}
```

The dataset is also available on Zenodo: [https://doi.org/10.5281/zenodo.20082565](https://doi.org/10.5281/zenodo.20082565)