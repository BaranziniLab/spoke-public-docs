# SPOKE public type documentation

Public, human- and machine-readable documentation for the node and edge **types**
in the public [SPOKE](https://spoke.ucsf.edu/) biomedical knowledge graph.

This repo is the **canonical fetch source** for the "?" type-documentation popups in
the SPOKEtest Explorer, and is designed to double as compact, structured schema/meaning
documentation for any external tool or LLM integrating with the public SPOKE graph.

## Layout

```
type_jsons/
  _manifest.json      # how to render a record (field → display primitive); the schema lives here
  <Type>.json         # one record per node/edge type; filename == the exact type name
```

Looking up documentation for a type is a plain file fetch: `type_jsons/<Type>.json`
(e.g. `type_jsons/Gene.json`, `type_jsons/INTERACTS_PiP.json`). No parsing, no fuzzy
matching.

## Consuming these files

Fetch raw over HTTPS:

```
https://raw.githubusercontent.com/<owner>/<repo>/main/type_jsons/<Type>.json
https://raw.githubusercontent.com/<owner>/<repo>/main/type_jsons/_manifest.json
```

A consumer renders a record by walking `_manifest.json`'s `layout` and applying the
named generic primitive to each field. Field names, order, and labels live in the
manifest — **not** in consumer code — so the record schema can change without any
consumer code change.

## Record shape (per-source sections)

A type is explicitly **multi-source** in SPOKE (every edge/node carries a `sources`
list). A record therefore keeps each contributing source's documentation in its own
section rather than flattening everything together:

| Field | Meaning |
| --- | --- |
| `type` | Exact type name (== filename). |
| `kind` | `node` or `edge`. |
| `reviewed` | Only `true` records are shown to users. Mirrors the modeling doc's `Human Approved` flag. |
| `endpoints` | (edges) `{from, to, direction}`. |
| `description` | Plain-English, type-level summary. Paraphrased from the modeling docs; confidence-gated (never asserts unstated domain knowledge). |
| `sources` | The sources that co-own this type. |
| `common_properties` | Properties present regardless of source (e.g. `sources`, `last_updated`). |
| `source_contributions[]` | One block per source: `source`, `scope`, `description`, `properties[]`. |
| `curator_notes` | Human-added context, woven into the natural description. `null` when none. |

Each property carries exact programmatic facts copied verbatim from the modeling docs
(`type`, cardinality via the type string, structural `notes`) kept distinct from the
paraphrased `description`, plus a `provenance` tag (`extracted` from a modeling doc, or
`curator` for human-added content) so regeneration never overwrites human edits.

> **Status:** skeleton / work in progress. `INTERACTS_PiP.json` is a first hand-authored
> record used to build and review the rendering pipeline. Content will be reviewed and
> refined; the eventual generation workflow will emit records in this same shape.
