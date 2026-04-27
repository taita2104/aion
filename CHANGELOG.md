# Changelog

All notable changes to the AION specification are documented here.

Format: `[VERSION] YYMMDD — description`

Breaking changes are marked **BREAKING**.

---

## [2] 260427 — Universal primitives

Complete redesign from v1.

**BREAKING**: all v1 type codes removed. v1 files are not compatible with v2 consumers.

### Changes from v1

- Replaced domain-specific types (DOC TSK MTG MSG EVT ENT ERR ACK REQ RES)
  with 7 universal primitives (E F Q K C S L) + error code X
- Document metadata moved from `D[id]` record to `AION` header line
- Added `type=` document class declaration in header
- Added `SCHEMA` block for domain-specific subtype extensions
- Added `S` (Section) primitive for structural grouping
- Added `L` (Link) primitive for references and citations
- Added `F.t=` fact subtypes: find concl desc req claim def warn note quote
- Added `E.t=` entity subtypes: person org system place product file concept role event
- Added `cf=` confidence property to all record types
- `RAW:` escape limited to one per block
- Delta syntax unchanged
- Error record syntax unchanged

---

## [1] 260415 — Initial release

First draft. Domain-specific type set:
DOC TSK MTG MSG EVT ENT ERR ACK REQ RES

Operator set established (unchanged in v2).
Date encoding established (unchanged in v2).
Quantity encoding established (unchanged in v2).
`<<<`/`>>>` block syntax established (unchanged in v2).
