# AION v3 — Formal Specification (rev. 3.1)

This document is the authoritative reference for AION v3.
`SKILL.md` in the repository root is the deployable subset optimized for AI system prompts.
When the two conflict, this document takes precedence.

> **Revision 3.1** is fully backward compatible with v3. File headers continue to declare `v=3`.
> Consumers conforming to v3.1 MUST also accept all valid v3 constructs.

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
- Section membership rule clarified
- `s=0` and `p=2` changed from SHOULD omit to MUST omit

---

## 3.1 Changes in revision 3.1 (additive only)

All changes in this revision are backward compatible. File headers remain `v=3`.
v3.1 consumers MUST accept all v3 syntax in addition to the new forms below.

### 3.1.1 `neg` flag shorthand

`neg` as a standalone property flag is equivalent to `neg=1`.
Producers SHOULD prefer `neg`. Consumers MUST accept both.

```
F[f1] t=claim neg n=warranty-excluded        # preferred
F[f1] t=claim neg=1 n=warranty-excluded      # also valid
```

### 3.1.2 Inline quantities

Quantities may appear inline anywhere a `Q[id]` reference is valid.
Format: `VALUEUNIT` with no space. Frequency modifier appended directly.

```
K[k1] E[acme]>20000EUR E[beta] @<20260515 !
C[r1] pen=5000EUR*d
```

Producers SHOULD use declared `Q[id]` records when the same value appears in two or more records.
Producers SHOULD use inline quantities for single-use values and TMPL instances.

### 3.1.3 TMPL positional syntax

TMPL instances may use positional values instead of named `field=val` pairs.
Field positions follow the order declared in `fields=`.
A TMPL declaration may use typed fields with `:UNIT` annotation.

```
TMPL[line] fields=[item,h:h,rate:EUR,total:EUR]

# positional (v3.1 preferred for dense tables)
line[l1] E[svc-1] 40 150 6000

# named (v3 compat, still valid)
line[l1] item=E[svc-1] h=40h rate=EUR150 total=EUR6000
```

Mixed syntax within the same TMPL (some rows named, some positional) is not permitted.
Consumers determine the mode from the first instance row encountered.

### 3.1.4 Extended standard properties

Standard short-name properties for domain-frequent concepts.
Producers SHOULD prefer these over freestyle synonyms.

| Prop | Meaning | Example |
|------|---------|---------|
| `jur=` | applicable law / jurisdiction | `jur=it-civil` |
| `ven=` | venue / competent forum | `ven=trib-mi` |
| `pen=` | penalty or sanction value | `pen=5000EUR*d` |
| `cnd=` | trigger condition | `cnd=delay` |
| `mth=` | method or procedure | `mth=rct` |
| `fmt=` | format or media type | `fmt=pdf` |
| `via=` | transmission channel | `via=email` |
| `ref=` | external reference code | `ref=ISO-9001` |
| `dur=` | duration | `dur=12mo` |
| `qty=` | count without unit | `qty=3` |

---

## 3.2 Changes in revision 3.2 (additive only)

All changes in this revision are backward compatible. File headers remain `v=3`.
v3.2 consumers MUST also accept all v3 and v3.1 syntax.

### 3.2.1 Multi-source dependency `>>[A,B]`

`>>` becomes variadic. A record may declare dependency on multiple sources simultaneously.

```
F[c] >>[F[a],F[b]]          # c follows from A and B together (conjunction)
F[c] >>F[a] | >>F[b]        # c follows from A or B (use | combinator)
```

Single-source `>>F[a]` remains valid. `>>[A]` with one element is equivalent.

### 3.2.2 Parallel operator `||`

Encodes concurrent / parallel execution. Distinct from `->` (sequence) and `>>` (dependency).

```
K[b] || K[c]                     # b and c execute concurrently
K[a] -> (K[b] || K[c]) -> K[d]  # sequence with parallel block
```

Parentheses group parallel sets. Maximum one level of nesting.

### 3.2.3 Quantity range `=MIN..MAX`

Any `Q` record or inline quantity may declare a range instead of a single value.

```
Q[latency] =50..500 u n=ms
Q[guidance] =52000000..58000000 EUR
F[f1] t=claim cf=0.85 >>[Q[eff-a],Q[baseline]]
<<<
Q[eff-a] =28..36 pct cf=0.88
>>>
```

Inline range: `50..500u` `28..36pct`

### 3.2.4 Comparison properties `vs=` and `d=`

`vs=` references a comparison baseline. `d=` encodes the delta value.
Both apply to `Q` records and inline quantities.

```
Q[rev-25] =48200000 EUR vs=Q[rev-24] d=+16.7pct
Q[err-rate] ~=2.3 pct vs=Q[err-baseline] d=-0.8pct
```

### 3.2.5 Weight property `wt=`

Numeric weight 0-100 for requirements, evaluation criteria, risk factors, decisions.

```
F[r1] t=req wt=40 n=security
F[r2] t=req wt=15 n=scalability
```

`wt=` is independent from `p=`. Priority `p=` is ordinal (1/2/3); weight `wt=` is numeric.

### 3.2.6 Document class vocabulary additions

Added to `type=` recognized values: `whitepaper sdk rfp financial`

---

---

## 3.2 Changes in revision 3.2 (additive only)

File headers remain `v=3`. All v3 and v3.1 syntax remains valid.

### 3.2.1 Multi-source dependency `>>[A,B]`

`>>` extended to accept a list of sources. `F[c] >>[F[a],F[b]]` means c follows from A and B together.
`F[c] >>F[a] | >>F[b]` means c follows from A or B. Single-source `>>F[a]` unchanged.

### 3.2.2 Weight property `wt=`

Numeric weight 0–100 on any record. Replaces `p=` when cardinal (not ordinal) weight is needed.
`F[r1] t=req wt=40` — requirement weighted at 40/100.
`p=` (1/2/3 ordinal) and `wt=` (0–100 cardinal) are independent; both may appear.

### 3.2.3 Comparison properties `vs=` and `d=`

`vs=` references another record for comparison. `d=` encodes the delta value.
```
Q[rev-25] =48200000 EUR vs=Q[rev-24] d=+16.7pct
F[f1] t=find vs=F[baseline] d=+32pct cf=0.96
```

### 3.2.4 Quantity ranges

Q values may declare a range with `..`:
```
Q[guidance] =52000000..58000000 EUR
Q[latency]  =50..500 u n=ms
Q[temp]     =36.5..37.5 C
```
Lower bound is inclusive. Upper bound is inclusive. Both must share the same unit.

### 3.2.5 Parallel operator `||`

For concurrent tasks or simultaneous events in a sequence chain:
```
K[b] || K[c]                    # b and c are concurrent
K[a] -> (K[b] || K[c]) -> K[d] # sequence with parallel block
```
`||` is only valid in `->` chains and `<<<` blocks. Not valid as a standalone record.

### 3.2.6 New document classes

Added to `type=` vocabulary: `whitepaper sdk rfp financial`

### 3.2.7 SKILL.md rewritten for token efficiency

The deployable `SKILL.md` was rewritten from ~4374 tokens (spec-format) to ~1511 tokens (compact-format).
All normative content preserved. Tables replaced with compact inline notation.
Examples reduced to one minimal illustrative example; full examples remain in `examples/`.
Per-session saving for a two-agent pipeline: ~5725 tokens.

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

## 9. Negation

Any record may carry a negation marker to assert that the source document
explicitly states its content as false, excluded, or inapplicable.

Two equivalent forms (v3.1):
- `neg` as a standalone flag (preferred)
- `neg=1` as a property (v3 compat, still valid)

Consumers MUST accept both forms.

```
F[f1] t=claim neg n=warranty-excluded
C[c1] neg n=jurisdiction-clause-inapplicable
```

Without negation, the record asserts its content as true.
With negation, it asserts the contrary.
`neg` has no effect on `X` (error) records.

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
- **Named syntax**: fields are `name=val` pairs, order-independent (v3 compat)
- **Positional syntax**: values in declaration order, no field names (v3.1, preferred for tables)
- Mixed syntax within the same TMPL is forbidden
- Consumers determine mode from the first instance row

Typed field declarations (v3.1): `fields=[name:unit, ...]`

```
TMPL[obs] fields=[subject,value:pct,period:mo,cf:0-1]
obs[o1] E[grp-int] 32 6 0.96
obs[o2] E[grp-sr]  41 6 0.88
```

- Field values: AION record references (`E[id]`, `Q[id]`), inline quantities, or literals
- Anonymous instances allowed (id omitted)
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
| `neg` | negated assertion — standalone flag or `neg=1`, both accepted | — |

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
4. Prefer `neg` flag over `neg=1` (v3.1)
5. Use inline quantities for single-use values (v3.1)
6. Use positional TMPL syntax for dense tabular data (v3.1)
7. Use extended standard properties (`jur=` `ven=` `pen=` `cnd=` `mth=`) over freestyle synonyms (v3.1)

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
10. Accept both `neg` and `neg=1` as equivalent (v3.1)
11. Accept both named and positional TMPL instance syntax (v3.1)
12. Accept inline quantities (`20000EUR`, `2pct*d`) wherever `Q[id]` is valid (v3.1)

---

## 20. Economic use threshold

AION is not universally token-efficient. The skill must be loaded by both producer and
consumer (~7,500 tokens combined overhead per session). This overhead is recovered only
when the documents being exchanged are long enough to benefit from compression.

| Document length | Compression ratio | Recommendation |
|----------------|-------------------|----------------|
| < 3 pages / < 500 tokens | < 1x (overhead exceeds savings) | do not use AION |
| 3-10 pages / 500-2,000 tokens | 2x-5x | use if multiple docs per session |
| 10-30 pages / 2,000-8,000 tokens | 5x-10x | use |
| > 30 pages / > 8,000 tokens | 10x-15x | strongly recommended |

For short documents (invoices, brief emails, short minutes), pass the original text directly.
The structural overhead of entity declarations, section markers, and property syntax
exceeds the natural language savings for documents under ~500 tokens.

The overhead amortizes quickly across a session: loading the skill once and processing
5 long documents in a session yields ~5x net token savings per document.

-e 
## 21. Versioning policy

- Current version: **3**
- Breaking changes increment the integer
- Additive changes (new subtypes, new document classes) do not
- Consumers declare supported versions; multi-version support is permitted

