# AION v3 — Formal Specification

This document is the authoritative reference for AION v3.
`SKILL.md` in the repository root is the deployable subset optimized for AI system prompts.
When the two conflict, this document takes precedence.

---

## 1. Purpose and scope

AION encodes the semantic content of documents for exchange between AI agents.
It is not a data serialization format. It is not a programming language.
It is not designed to be parsed by code.
It is designed to be produced and interpreted by large language models.

The unit of exchange is the `.aion` file: a plain-text semantic digest of a source document.

---

## 2. Encoding

- Character set: printable ASCII (0x20-0x7E) plus newline (0x0A)
- No BOM, no tabs
- File extension: `.aion`
- MIME type: `text/aion`

---

## 3. Changes from v2

**Breaking (require version increment):**
- Date format changed: `YYMMDD` → `YYYYMMDD` (avoids century ambiguity)
- ID uniqueness changed: now global across all types, not per-type
- `SCHEMA` separator changed: `E.t:` → `E.t =` (consistency with record syntax)
- `RAW:` single-line rule removed: `RAW` now opens a multi-line verbatim sub-block

**Additive (no version increment in isolation, but bundled):**
- `->` sequence operator added
- `&` and `|` logical combinators added (conditions and blocks only)
- `neg=1` property added
- `TMPL[id]` template record type added
- `cf=` semantics clarified: header = digest-level, record = record-level
- Section membership rule clarified: explicit `+[...]` is authoritative; positional is fallback; mixing is forbidden
- `s=0` and `p=2` changed from SHOULD omit to MUST omit

---

## 4. File structure

```
HEADER
---
SCHEMA
---
ENTITIES
---
CONTENT
```

Section breaks use `---` on a standalone line.
Empty sections may omit their `---`.

### 4.1 Header (required)

```
AION v=3 dt=YYYYMMDD type=CLASS lang=XX src=ORIGIN cf=N
```

| Property | Format | Meaning | Required |
|----------|--------|---------|----------|
| `v=` | integer | AION version | yes |
| `dt=` | YYYYMMDD | document date | recommended |
| `type=` | class code | document class | recommended |
| `lang=` | ISO 639-1 | source language | recommended |
| `src=` | string | origin filename or system | optional |
| `cf=` | 0.0-1.0 | producer confidence in digest as a whole | optional |

### 4.2 SCHEMA (optional)

```
SCHEMA
  E.t = subtype1 subtype2 ...
  F.t = subtype1 subtype2 ...
  Q.unit = unit1 unit2 ...
  S.t = subtype1 subtype2 ...
END
```

`SCHEMA` adds subtypes to existing primitives only.
It cannot define new primitive codes, operators, or properties.
All `SCHEMA` declarations use `=` as separator.

### 4.3 Entities (optional but recommended)

`E` records before content improve navigation for the consumer.
`E` records may also appear anywhere in the content section.

### 4.4 Content

All other records in any order.

---

## 5. Document classes

| Class | Meaning |
|-------|---------|
| `contract` | Legal agreement |
| `report` | Structured findings or status |
| `spec` | Technical specification |
| `email` | Electronic message |
| `minutes` | Meeting minutes |
| `proposal` | Business or project proposal |
| `policy` | Policy document |
| `invoice` | Billing document |
| `brief` | Short briefing |
| `manual` | User or technical manual |
| `research` | Academic or scientific document |
| `plan` | Project or strategic plan |
| `form` | Structured form |
| `audit` | Audit or compliance document |
| `press` | Press release |
| `doc` | Generic / unclassified |

---

## 6. Record syntax

```
TYPE[id] prop=val prop=val OPERATOR TARGET ...
```

- `TYPE`: single uppercase letter
- `[id]`: unique across the entire file regardless of type; chars `a-z A-Z 0-9 - .`
- `prop=val`: space-separated, order-independent
- Records separated by blank line or `---`
- Continuation: leading whitespace on subsequent lines
- Anonymous records (no `[id]`): allowed; cannot be referenced

---

## 7. Primitive types

### 7.1 E — Entity

Subtypes (`t=`): `person org system place product file concept role event`

Properties beyond common set:

| Prop | Meaning |
|------|---------|
| `role=` | functional role |
| `by=` | parent entity id |
| `sector=` | industry or domain |
| `vat=` | tax identifier |
| `email=` | contact email |

### 7.2 F — Fact

Subtypes (`t=`): `find concl desc req claim def warn note quote`

### 7.3 Q — Quantity

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
| Generic | `u r` |

Frequency modifiers: `*d *mo *yr *u`

### 7.4 K — Action

Subtypes (`t=`): `sign approve review test deploy notify recommend`
Modal operators apply: `!` `~` `/`

### 7.5 C — Condition

If-then logic, constraints, rules, obligations, permissions.

```
C[id] TRIGGER => CONSEQUENCE
C[id] E[actor] MODAL OBJECT @<DEADLINE
```

Logical combinators `&` and `|` are permitted inside `C` records and `<<<` blocks.
Parentheses `()` group sub-expressions. Maximum one level of nesting.

### 7.6 S — Section

```
S[id] t=SUBTYPE +[RECORD_IDS]
```

If `+[...]` is present: authoritative for membership.
If `+[...]` is absent: membership by physical position until next `S` or `---`.
Mixing both modes in the same file is forbidden; emit `X` if detected.

Subtypes:
```
introduction background scope methodology results analysis
conclusion recommendation appendix next-steps risk financial
```

### 7.7 L — Link

```
L[id] >TARGET t=TYPE n=LABEL
```

Targets: `>https://...` | `>TYPE[id]` | `>DOC:IDENTIFIER`
Types: `cite ref attach replaces supersedes see-also`

### 7.8 X — Error

```
X src=RECORD field=FIELD reason=REASON got=VAL expected=EXPECTED
```

Reasons: `missing unknown-type undeclared-subtype invalid-value parse-fail unsupported`

---

## 8. Operators

### 8.1 Relational

| Op | Meaning |
|----|---------|
| `>` | transfers to / delivers to |
| `<` | receives from |
| `<>` | bilateral / mutual |
| `>>` | depends on / follows from (logical) |
| `->` | precedes (sequential order) |
| `=>` | implies / if-then |
| `+` | includes / adds |
| `-` | excludes / removes |
| `=` | equals / is defined as |
| `~=` | approximately equals |
| `*` | for each / applies to all |
| `^` | version of |

`>>` (logical dependency) and `->` (sequence) are not interchangeable.

`->` may form chains: `K[a] -> K[b] -> K[c]`
Direction: left precedes right.

`~=` is a compound relational operator. `~` standalone is a modal.
They are syntactically unambiguous: `~=` always has operands on both sides.

### 8.2 Temporal

| Op | Meaning |
|----|---------|
| `@` | at |
| `@<` | no later than |
| `@>` | no earlier than |
| `@+Nd` | in N calendar days |
| `@+Ndw` | in N working days |
| `@+Nmo` | in N months |

### 8.3 Modal

| Op | Meaning |
|----|---------|
| `!` | mandatory |
| `~` | optional / recommended |
| `/` | prohibited |
| `?` | uncertain / inferred |

### 8.4 Logical combinators

| Op | Scope | Meaning |
|----|-------|---------|
| `&` | C records, blocks | AND |
| `\|` | C records, blocks | OR |

Not permitted in record property values or outside conditions.

---

## 9. Negation (`neg=1`)

Any record may carry `neg=1` to assert that the source document
explicitly states its content as false, excluded, or inapplicable.

Without `neg=1`: the record asserts its content as true.
With `neg=1`: the record asserts the negated form.

```
F[f1] t=claim neg=1 n=warranty-excluded
C[c1] neg=1 n=jurisdiction-clause-inapplicable
```

`neg=1` has no effect on `X` (error) records.

---

## 10. Date and time

All dates use 4-digit year.

| Format | Meaning |
|--------|---------|
| `YYYYMMDD` | date |
| `HHMM` | time (24h, UTC unless stated) |
| `YYYYMMDD.HHMM` | datetime |
| `+Nd` | N calendar days from document date |
| `+Ndw` | N working days |
| `+Nmo` | N months |

---

## 11. Long content blocks

```
RECORD_LINE
<<<
AION content
>>>
```

One block per record. Block content: AION operators and identifiers only.

### 11.1 RAW sub-block

`RAW` opens a verbatim region. All lines from `RAW` until `>>>` are opaque.
`RAW` must be the last element in the block.

```
C[c1] E[supplier]>E[mvp] @<20261001 !
<<<
E[client]<E[mvp]: ~ack @<+15dw
C[c1a]: timeout => E[mvp].s=4
RAW
Verbatim text spanning
multiple lines.
Still verbatim.
>>>
```

---

## 12. Template records (`TMPL`)

```
TMPL[id] fields=[f1,f2,f3]
id[row-id] f1=val f2=val f3=val
```

- Instances use the `TMPL[id]` name as their type
- Fields are named and order-independent in instances
- Field values: AION record references (`E[id]`, `Q[id]`) or literals
- Anonymous instances allowed (`id[]` omitted)
- A consumer encountering an unknown 2-letter type MUST check `TMPL` declarations
  before emitting `X type=unknown`

---

## 13. Delta updates

```
DELTA TYPE[id] prop=val prop=val
```

Merges listed properties. Unlisted properties unchanged.
`DELTA` on non-existent `[id]`: emit `X reason=missing`.

---

## 14. Common properties

| Prop | Meaning | Default |
|------|---------|---------|
| `t=` | subtype | — |
| `s=` | status | `0` (must omit) |
| `p=` | priority | `2` (must omit) |
| `v=` | version | — |
| `to=` | destination | — |
| `fr=` | source | — |
| `cf=` | record-level confidence | — |
| `n=` | human label or annotation | — |
| `dt=` | date | — |
| `by=` | responsible entity | — |
| `tags=` | label list | — |
| `neg=` | `1` = negated assertion | — |

---

## 15. Status codes

`0` = not started (default, must omit)
`1` = pending · `2` = active · `3` = review · `4` = approved
`5` = done · `6` = cancelled · `7` = blocked · `8` = error

---

## 16. Priority codes

`1` = high · `2` = medium (default, must omit) · `3` = low

---

## 17. Producer normative requirements

MUST:
1. Begin every file with `AION v=3`
2. Assign unique `[id]` globally to every referenced record
3. Omit `s=0` and `p=2`
4. No natural language outside `RAW` blocks
5. Emit `X` for anomalies detected before sending
6. Declare custom subtypes in `SCHEMA` before first use
7. Add `cf=` to records whose content is probabilistic
8. Use `->` for sequence, `>>` for logical dependency

SHOULD:
1. Place `E` declarations before content
2. Use `S` to mirror source document structure
3. Include `dt=`, `lang=`, `type=` in header

---

## 18. Consumer normative requirements

MUST:
1. Parse `TYPE[id] prop=val` as order-independent
2. Unknown property: ignore, continue
3. Unknown 1-letter type: emit `X type=unknown`, skip
4. Unknown 2-letter type: check `TMPL` declarations first
5. Missing `[id]` on referenced record: emit `X field=id reason=missing`, skip
6. `DELTA`: merge, preserve unlisted
7. `RAW` block: treat all lines until `>>>` as opaque string
8. Propagate `fr=` when forwarding downstream
9. Version mismatch: emit `X field=v reason=unsupported`, halt

---

## 19. Versioning policy

- Current version: **3**
- Breaking changes increment the integer
- Additive changes (new subtypes, new document classes) do not
- Consumers declare supported versions; multi-version support is permitted
