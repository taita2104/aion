# Changelog

All notable changes to the AION specification are documented here.

Format: `[VERSION] YYYYMMDD — description`

Breaking changes are marked **BREAKING**.

---

## [3.3.1] 20260428

File headers remain `v=3`. No breaking changes.

- PRODUCER MUST rule 4 strengthened: explicit prohibition on natural language inside `<<<` blocks without `RAW` opener
- Added wrong/right examples inline in rule 4 to prevent producer non-conformance on dense technical content
- `CMP` sub-block added: compressed prose alternative to `RAW`, ~60-70% token reduction
- CMP rules 7-9 added: `E[id]` mandatory for entity references, `|` mandatory for lists, inline quantities mandatory
- PRODUCER MUST rules 15-16 added: guidance on choosing TMPL / AION / CMP / RAW by content structure
- CONSUMER MUST rule 13 added: CMP interpretation rules including entity and quantity inverse mapping

---

## [3.3] 20260427

File headers remain `v=3`. All v3/v3.1/v3.2 syntax valid. No breaking changes.

- `ep=` epistemic status on F records: `assert·claim·corr·evid·demo·estab·disput·refut`
- F.t additions: `corr` (correlation) · `evid` (empirical evidence) · `refut` (refutation)
- `valid=YYYYMMDD:YYYYMMDD` temporal validity on C records and any bounded record
- `cov=0-1` and `excl=[ids]` in header: digest coverage metadata
- Producer convention: `>>F[def-id]` on records using defined terms

---

## [3.2] 20260427 — Additive improvements, no breaking changes

File headers remain `v=3`. All v3 and v3.1 constructs valid in v3.2.

### Multi-source dependency `>>[A,B]`
`>>` is now variadic. `F[c] >>[F[a],F[b]]` encodes dependency on A and B together.
Single-source `>>F[a]` unchanged.

### Parallel operator `||`
`K[b] || K[c]` = b and c execute concurrently.
`K[a] -> (K[b] || K[c]) -> K[d]` = sequence with parallel block.
Distinct from `->` (sequence) and `>>` (logical dependency).

### Quantity range `=MIN..MAX`
`Q[latency] =50..500 u` and inline `50..500u` for estimated or acceptable ranges.

### Comparison properties `vs=` and `d=`
`Q[rev-25] =48200000 EUR vs=Q[rev-24] d=+16.7pct` for multi-period or comparative data.

### Weight property `wt=`
`F[r1] t=req wt=40` — numeric weight 0-100, independent from priority `p=`.

### Document class vocabulary
Added: `whitepaper sdk rfp financial`

### Economic use threshold documented
AION is not economically advantageous for documents under ~3 pages (~500 tokens).
For short documents, structural overhead exceeds natural language savings.
This threshold is now normative in spec section 20.

### SKILL.md compressed
Skill token count reduced from ~8,700 to ~3,750 (57% reduction).
Session overhead (producer + consumer): 17,400 → 7,500 tokens.

---



File headers remain `v=3`. All v3 constructs are valid in v3.1.
v3.1 consumers MUST also accept all v3 syntax.

### `neg` flag shorthand
`neg` as a standalone flag is now equivalent to `neg=1`.
Producers should prefer `neg`. Consumers must accept both forms.

### Inline quantities
Quantities may appear inline anywhere a `Q[id]` reference is valid.
Format: `VALUEUNIT` with no space (`20000EUR`, `5000EUR*d`, `2pct*mo`).
Declared `Q[id]` records remain preferred when a value is referenced more than once.

### TMPL positional syntax
TMPL instances may use positional values in declaration order instead of `field=val` pairs.
TMPL field declarations may include unit annotations: `fields=[name:unit, ...]`.
Mixed named/positional syntax within the same TMPL is forbidden.
Consumers determine mode from the first instance row.

### Extended standard properties
Ten short-name standard properties added to reduce inter-producer inconsistency:
`jur=` `ven=` `pen=` `cnd=` `mth=` `fmt=` `via=` `ref=` `dur=` `qty=`
Freestyle synonyms (e.g. `jurisdiction=`, `penalty=`, `trigger=`) are now discouraged.

---

## [3] 20260427

### Breaking changes

- **Date format**: `YYMMDD` → `YYYYMMDD`. Two-digit years are rejected by v3 consumers.
- **ID uniqueness**: IDs are now globally unique across all types within a file.
  `C[x]` and `E[x]` in the same file is a v3 error. v2 files that relied on per-type
  uniqueness must be updated.
- **SCHEMA separator**: `E.t: subtypes` → `E.t = subtypes`. Colon syntax is rejected.
- **RAW single-line rule removed**: `RAW:` (colon, single line) replaced by `RAW` (block
  opener, multi-line until `>>>`). v2 `RAW:` lines are not valid v3.
- **`s=0` and `p=2`**: changed from SHOULD omit to MUST omit. v3 producers that emit
  default values are non-conforming.

### Additions

- `->` sequence operator: encodes strict temporal/procedural ordering.
  Distinct from `>>` (logical dependency).
- `&` and `|` logical combinators: usable inside `C` records and `<<<` blocks only.
- `neg=1` property: asserts that a record's content is explicitly negated in the source.
- `TMPL[id]` template records: compact encoding of repeated tabular structures.
- `cf=` semantics clarified: header `cf=` = digest-level confidence;
  record `cf=` = record-level confidence. Independent values.
- Section membership rule clarified: explicit `+[...]` is authoritative;
  positional is fallback; mixing both in the same file is forbidden.

---

## [2] 20260415

Complete redesign from v1.

**BREAKING**: all v1 type codes removed. v1 files are not compatible with v2 consumers.

### Changes from v1

- Replaced domain-specific types (DOC TSK MTG MSG EVT ENT ERR ACK REQ RES)
  with 7 universal primitives (E F Q K C S L) + error code X
- Document metadata moved from `D[id]` record to `AION` header line
- Added `type=` document class in header
- Added `SCHEMA` block for domain-specific subtype extensions
- Added `S` (Section) and `L` (Link) primitives
- Added `F.t=` fact subtypes: find concl desc req claim def warn note quote
- Added `E.t=` entity subtypes: person org system place product file concept role event
- Added `cf=` confidence property
- `RAW:` single-line escape introduced (replaced in v3)
- Delta syntax unchanged from v1
- Error record syntax unchanged from v1

---

## [1] 20260401

Initial release. Domain-specific type set:
DOC TSK MTG MSG EVT ENT ERR ACK REQ RES

Operator set established (core subset unchanged in v3).
Date encoding established (YYMMDD, changed to YYYYMMDD in v3).
Quantity encoding established (unchanged in v3).
`<<<`/`>>>` block syntax established (unchanged in v3).
