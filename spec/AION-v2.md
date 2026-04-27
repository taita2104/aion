# AION v2 — Formal Specification

This document is the authoritative reference for the AION v2 notation.
`SKILL.md` in the repository root is the deployable subset optimized for AI system prompts.
When the two conflict, this document takes precedence.

---

## 1. Purpose and scope

AION encodes the semantic content of documents for exchange between AI agents.
It is not a data serialization format (not a replacement for JSON or XML).
It is not a programming language. It is not designed to be parsed by code.
It is designed to be read, produced, and interpreted by large language models.

The unit of exchange is the `.aion` file: a plain text file containing a semantic
digest of a source document.

---

## 2. Encoding

- Character set: printable ASCII (0x20-0x7E) plus newline (0x0A)
- No BOM
- No tabs (use spaces for alignment only in comments)
- File extension: `.aion`
- MIME type (informal): `text/aion`

---

## 3. File structure

A valid `.aion` file consists of the following sections in order.
All sections except the header are optional.

```
HEADER
---
SCHEMA
---
ENTITIES
---
CONTENT
```

Section breaks use `---` on a standalone line. If a section is empty, its `---` may be omitted.

### 3.1 Header (required)

```
AION v=2 dt=YYMMDD type=CLASS lang=XX src=ORIGIN cf=N
```

The `AION` keyword must appear on the first line of the file.
`v=2` is required. All other header properties are recommended but not required.

| Property | Format | Meaning |
|----------|--------|---------|
| `v=` | integer | AION version |
| `dt=` | YYMMDD | document creation date |
| `type=` | class code | document class (section 4) |
| `lang=` | ISO 639-1 | source document language |
| `src=` | string | source filename or system identifier |
| `cf=` | 0.0-1.0 | producer confidence in the digest |

### 3.2 SCHEMA (optional)

Declares custom subtypes for the current file.

```
SCHEMA
  E.t: subtype1 subtype2 ...
  F.t: subtype1 subtype2 ...
  Q.unit: unit1 unit2 ...
  S.t: subtype1 subtype2 ...
END
```

`SCHEMA` may only add subtypes to existing primitives. It may not define new primitive codes,
new operators, or new properties.

### 3.3 Entities (optional)

`E` records that define named entities referenced throughout the file.
Placing them in a dedicated section before content improves readability for the consumer AI
but is not required. `E` records may appear anywhere in the content section.

### 3.4 Content (optional)

All other records. May contain any mix of primitives in any order.

---

## 4. Document classes

The `type=` header property identifies the document class.

| Class | Meaning |
|-------|---------|
| `contract` | Legal agreement between parties |
| `report` | Structured reporting of findings or status |
| `spec` | Technical specification or requirements |
| `email` | Electronic message |
| `minutes` | Meeting minutes |
| `proposal` | Business or project proposal |
| `policy` | Internal or external policy document |
| `invoice` | Financial invoice or billing document |
| `brief` | Short briefing document |
| `manual` | User or technical manual |
| `research` | Academic or scientific research document |
| `plan` | Project or strategic plan |
| `form` | Structured form with fields |
| `audit` | Audit report or compliance document |
| `press` | Press release or public communication |
| `doc` | Generic / unclassified |

---

## 5. Record syntax

### 5.1 Basic form

```
TYPE[id] prop=val prop=val OPERATOR TARGET
```

- `TYPE`: single uppercase letter (see section 6)
- `[id]`: unique alphanumeric identifier within the file; characters: `a-z A-Z 0-9 - .`
- `prop=val`: space-separated key-value pairs; order-independent
- Operators and targets follow properties; order-independent among themselves
- Records are separated by blank lines or `---`
- A record may span multiple lines; continuation is indicated by leading whitespace

### 5.2 Anonymous records

`[id]` may be omitted. Anonymous records cannot be referenced by other records.

```
F t=warn n=minor-caveat
```

### 5.3 Inline list values

```
prop=[val1,val2,val3]
```

No spaces inside lists. Nested lists are not supported.

### 5.4 Property values

- String values: no quoting required unless the value contains spaces
- Values with spaces: enclose in `"..."` (double quotes)
- Numeric values: digits only, no thousands separators
- Boolean values: `1` (true) `0` (false)

---

## 6. Primitive types

### 6.1 E — Entity

Any named thing that other records refer to.

Subtypes (`t=`): `person org system place product file concept role event`

Core properties beyond common set:

| Prop | Meaning |
|------|---------|
| `role=` | functional role in document context |
| `sector=` | industry or domain |
| `vat=` | VAT or tax identifier |
| `email=` | contact email |
| `by=` | parent entity id |

### 6.2 F — Fact

Any assertion, finding, claim, or stated truth.

Subtypes (`t=`): `find concl desc req claim def warn note quote`

| Subtype | Meaning |
|---------|---------|
| `find` | empirical finding or result |
| `concl` | conclusion derived from findings |
| `desc` | descriptive statement |
| `req` | requirement (functional or non-functional) |
| `claim` | asserted but unverified statement |
| `def` | definition |
| `warn` | warning, caveat, or limitation |
| `note` | annotation or aside |
| `quote` | verbatim attribution (use with `by=`) |

### 6.3 Q — Quantity

Any measurable value. Always includes a numeric value and a unit.

```
Q[id] =VALUE UNIT *FREQ
```

Standard units:

| Category | Units |
|----------|-------|
| Currency | `EUR USD GBP JPY CHF` |
| Time | `mo d dw h min` |
| Storage | `KB MB GB TB` |
| Ratio | `pct` |
| Distance | `km m cm` |
| Mass | `kg g` |
| Temperature | `C K F` |
| Generic | `u r` (units, records) |

Frequency modifiers: `*d *mo *yr *u` (per day, per month, per year, per unit)

### 6.4 K — Action

Anything that must or should be done by an agent.

| Prop | Meaning |
|------|---------|
| `by=` | responsible entity |
| `t=` | action subtype: `sign approve review test deploy notify` |

Modal operators apply: `!` (must), `~` (should), `/` (must not)

### 6.5 C — Condition

Any if-then logic, constraint, rule, obligation, or permission.

Conditions use `=>` for implication and `!` `~` `/` for modal force.

```
C[id] TRIGGER => CONSEQUENCE
C[id] E[actor] OBLIGATION OBJECT @<DEADLINE !
```

### 6.6 S — Section

Structural grouping. Does not carry semantic content itself.

```
S[id] t=SUBTYPE +[RECORD_IDS]
```

Subtypes: `introduction background scope methodology results analysis conclusion
           recommendation appendix next-steps risk financial`

`+[...]` lists the record IDs that belong to the section. If omitted, the section
boundary is determined by position in the file.

### 6.7 L — Link

Reference to internal or external resource.

```
L[id] >TARGET t=TYPE n=LABEL
```

Target format:
- External URL: `>https://...`
- Internal record: `>E[id]` `>F[id]` etc.
- External document identifier: `>DOC:IDENTIFIER`

Link types: `cite ref attach replaces supersedes see-also`

### 6.8 X — Error

Emitted by consumers when a construct cannot be interpreted.

```
X src=RECORD field=FIELD reason=REASON got=VAL expected=EXPECTED
```

Error reasons: `missing unknown-type undeclared-subtype invalid-value parse-fail unsupported`

---

## 7. Operators

### 7.1 Relational operators

| Op | Meaning |
|----|---------|
| `>` | transfers to / delivers to |
| `<` | receives from |
| `<>` | bilateral / mutual |
| `>>` | depends on / derived from |
| `=>` | implies / if-then |
| `+` | includes / adds |
| `-` | excludes / removes |
| `=` | equals / is defined as |
| `~=` | approximately equals |
| `*` | for each / applies to all |
| `^` | version of / revision of |

### 7.2 Temporal operators

| Op | Meaning |
|----|---------|
| `@` | at |
| `@<` | no later than |
| `@>` | no earlier than |
| `@+Nd` | in N calendar days |
| `@+Ndw` | in N working days |
| `@+Nmo` | in N months |

### 7.3 Modal operators

| Op | Meaning |
|----|---------|
| `!` | mandatory |
| `~` | optional / recommended |
| `/` | prohibited |
| `?` | uncertain / inferred |

Operators are not composable with each other except `~=` (one unit).

---

## 8. Date and time

| Format | Meaning |
|--------|---------|
| `YYMMDD` | date |
| `HHMM` | time (24h, UTC unless otherwise declared) |
| `YYMMDD.HHMM` | datetime |
| `+Nd` | relative: N calendar days from document date |
| `+Ndw` | relative: N working days |
| `+Nmo` | relative: N months |

All dates in the file are relative to the document date declared in the header unless
a specific date is provided.

---

## 9. Long content blocks

```
RECORD_LINE
<<<
BLOCK_CONTENT
>>>
```

A block immediately follows its parent record. A record may have at most one block.
Block content uses AION operators and identifiers only.

### 9.1 RAW escape

```
<<<
RAW: verbatim text that cannot be encoded without semantic loss
>>>
```

Rules:
- At most one `RAW:` line per block
- `RAW:` must be the last line in the block if present
- Content after `RAW:` on the same line is treated as an opaque string
- Consumers must preserve `RAW:` content verbatim

---

## 10. Delta updates

```
DELTA TYPE[id] prop=val prop=val
```

Merges listed properties into the existing record identified by `TYPE[id]`.
Properties not listed remain unchanged.
`DELTA` for a non-existent `[id]` is an error: emit `X reason=missing`.

---

## 11. Common properties (all types)

| Prop | Meaning | Format |
|------|---------|--------|
| `t=` | subtype | code |
| `s=` | status | 0-8 |
| `p=` | priority | 1-3 |
| `v=` | version | integer |
| `to=` | destination | entity id |
| `fr=` | source | entity id |
| `cf=` | confidence | 0.0-1.0 |
| `n=` | annotation | short string |
| `dt=` | date | YYMMDD |
| `by=` | responsible | entity id |
| `tags=` | labels | [list] |

Default values (omit when using default):

| Prop | Default |
|------|---------|
| `p=` | `2` |
| `s=` | `0` |

---

## 12. Status codes

| Code | Meaning |
|------|---------|
| `0` | not started (default) |
| `1` | pending |
| `2` | active |
| `3` | review |
| `4` | approved / accepted |
| `5` | done |
| `6` | cancelled |
| `7` | blocked |
| `8` | error |

---

## 13. Producer normative requirements

A conforming producer MUST:

1. Begin every `.aion` file with `AION v=2`
2. Assign a unique `[id]` to every record that may be referenced by another record
3. Emit `X` records for anomalies detected before transmission
4. Declare custom subtypes in `SCHEMA` before first use
5. Use `cf=` on any record whose content is probabilistic or inferred
6. Limit blocks to one `RAW:` line
7. Use only printable ASCII

A conforming producer SHOULD:

1. Omit properties at their default values
2. Place `E` declarations in the entity section before content
3. Use `S` records to mirror the source document structure
4. Include `lang=` and `dt=` in the header

### 14. Consumer normative requirements

A conforming consumer MUST:

1. Parse `TYPE[id] prop=val` as order-independent
2. Ignore unknown properties (forward compatibility)
3. Emit `X type=unknown` and skip records with unrecognized primitive codes
4. Emit `X field=id reason=missing` and skip records missing required `[id]`
5. On `DELTA`: merge listed properties, preserve unlisted ones
6. Treat `RAW:` block content as opaque strings
7. Propagate `fr=` when forwarding records downstream
8. Emit `X field=v reason=unsupported` and halt on unsupported version

---

## 15. Versioning policy

- The current version is **2**
- Breaking changes (removed constructs, changed semantics) increment the version integer
- Additive changes (new document classes, new subtypes) do not increment the version
- Implementations declare the versions they support; multi-version support is permitted
