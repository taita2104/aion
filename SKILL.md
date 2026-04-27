---
name: aion
description: >
  AION (AI Interop Object Notation) is a shared communication protocol for AI-to-AI pipelines.
  Load this skill into the system prompt of every AI agent in a pipeline where one AI produces
  structured output that another AI consumes — without any human reading it in between.
  Use AION whenever you are acting as a producer (generating structured output for a downstream AI)
  or as a consumer (receiving and interpreting AION-formatted input from an upstream AI).
  Also use when a human asks you to generate a .aion file as a semantic digest of a document
  they intend to share with another party whose AI will read it.
  Triggers: "AI pipeline", "agent to agent", "AI workflow", "produce output for another AI",
  "consume AI output", "no human in the loop", "AION format", "interop between agents",
  "generate aion", "create aion file", "aion digest", ".aion".
---

# AION v3 — AI Interop Object Notation

Token-efficient artificial language for AI-to-AI information exchange.
No natural language. Every construct has exactly one interpretation.
Universal: seven primitives cover any document type.

**Core principle**: producer and consumer both load this skill. The skill is the contract.

---

## Design axioms

1. BPE tokenizers assign 1 token to short ASCII sequences. Unicode costs 2-4 tokens.
   AION uses only printable ASCII.
2. Every document is fully describable as seven semantic primitives. No more are needed.
3. The type system encodes *semantic role*, not domain.
4. Natural language never appears except inside explicit `RAW` escape blocks.
5. Unknown constructs are silently ignored, not rejected (forward compatibility).
6. Properties at default values MUST be omitted.

---

## Seven universal primitives

| Code | Name | What it encodes |
|------|------|-----------------|
| `E` | Entity | Any named thing: person, org, system, place, concept, product, file |
| `F` | Fact | Any stated truth: finding, claim, description, definition, result |
| `Q` | Quantity | Any measurable value with unit |
| `K` | Action | Anything that must or should be done |
| `C` | Condition | Any if-then, constraint, rule, obligation, permission |
| `S` | Section | A structural grouping of records |
| `L` | Link | A reference to an internal record or external resource |
| `X` | Error | Encoding or interpretation failure (reserved) |

Custom types: 2-letter uppercase codes declared in `SCHEMA`.

---

## File structure

```
AION v=3 dt=YYYYMMDD type=CLASS lang=XX src=ORIGIN cf=N
---
SCHEMA           # optional — domain subtype extensions
---
E[...] F[...]    # entity declarations
---
C[...] K[...]    # content records
```

`---` separates sections. Empty sections may omit their `---`.
The `AION` header line is the only required element.

### Header properties

| Prop | Format | Meaning |
|------|--------|---------|
| `v=` | integer | AION version (required) |
| `dt=` | YYYYMMDD | document date |
| `type=` | class code | document class |
| `lang=` | ISO 639-1 | source language |
| `src=` | string | origin filename or system |
| `cf=` | 0.0-1.0 | producer confidence in the digest as a whole |

### Document classes (`type=`)

```
contract    report      spec        email       minutes
proposal    policy      invoice     brief       manual
research    plan        form        audit       press    doc
```

---

## Record structure

```
TYPE[id] prop=val prop=val OPERATOR TARGET ...
```

- `TYPE`: single uppercase letter
- `[id]`: unique within the file; chars `a-z A-Z 0-9 - .`
  ID uniqueness is **global** across all types. `C[x]` and `E[x]` cannot coexist.
  References always include the type prefix: `E[alice]`, `C[c1]`.
- `prop=val`: space-separated, order-independent
- `[id]` may be omitted on anonymous records (no other record may reference them)
- Records are separated by a blank line or `---`

---

## Entity subtypes (`t=`)

```
person  org  system  place  product  file  concept  role  event
```

Entity-specific properties:

| Prop | Meaning |
|------|---------|
| `role=` | functional role in document context |
| `by=` | parent entity id |
| `sector=` | industry or domain |
| `vat=` | tax identifier |
| `email=` | contact email |

---

## Fact subtypes (`t=`)

```
find    concl   desc    req     claim   def     warn    note    quote
```

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

---

## Operators

### Relational

| Op | Meaning |
|----|---------|
| `>` | transfers to / delivers to / sends |
| `<` | receives from / accepts |
| `<>` | bilateral / mutual |
| `>>` | depends on / follows from (logical) |
| `->` | precedes / comes before (sequential order) |
| `=>` | implies / if-then |
| `+` | includes / adds |
| `-` | excludes / removes |
| `=` | equals / is defined as |
| `~=` | approximately equals |
| `*` | for each / applies to all |
| `^` | version of / revision of |

`>>` encodes logical dependency. `->` encodes strict temporal/procedural sequence.
They are not interchangeable.

### Temporal

| Op | Meaning |
|----|---------|
| `@` | at (point in time) |
| `@<` | by / no later than |
| `@>` | from / no earlier than |
| `@+Nd` | in N calendar days |
| `@+Ndw` | in N working days |
| `@+Nmo` | in N months |
| `*d` | per day |
| `*mo` | per month |
| `*yr` | per year |
| `*u` | per unit |

### Modal

| Op | Meaning |
|----|---------|
| `!` | mandatory / must |
| `~` | optional / may / recommended |
| `/` | prohibited / must not |
| `?` | uncertain / unverified / inferred |

`~` and `~=` are distinct: `~` is a standalone modal, `~=` is a compound relational operator.
They cannot be confused in context: `~=` always has an operand on both sides.

### Logical combinators (conditions only)

| Op | Meaning |
|----|---------|
| `&` | AND |
| `\|` | OR |

Used only inside `C` records and `<<<` blocks to combine conditions or triggers.

```
C[c1] (E[a]! & E[b]!) => E[mvp].s=4
C[c2] (E[a]~ \| E[b]~) => K[notify]
```

Parentheses `()` group sub-expressions. Maximum one level of nesting.

---

## Negation

Any record can be negated with the `neg=1` property. It means:
"the source document explicitly states this as false, excluded, or inapplicable."

```
F[f1] t=claim neg=1 n=warranty-excluded
C[c1] neg=1 n=force-majeure-does-not-apply
```

Without `neg=1`, a record asserts its content as true. With `neg=1`, it asserts the contrary.

---

## Date and time

4-digit year, no separators.

| Format | Meaning | Example |
|--------|---------|---------|
| `YYYYMMDD` | date | `20261001` |
| `HHMM` | time 24h | `1430` |
| `YYYYMMDD.HHMM` | datetime | `20261001.1430` |
| `+Nd` | relative N calendar days | `+30d` |
| `+Ndw` | relative N working days | `+15dw` |
| `+Nmo` | relative N months | `+6mo` |

Relative dates are computed from the document date in the header.

---

## Quantities (`Q`)

```
Q[id] =VALUE UNIT *FREQ
```

| Category | Units |
|----------|-------|
| Currency | `EUR USD GBP JPY CHF` |
| Time | `mo d dw h min` |
| Storage | `KB MB GB TB` |
| Ratio | `pct` |
| Distance | `km m cm` |
| Mass | `kg g` |
| Temperature | `C K F` |
| Generic | `u` (units) `r` (records/rows) |

---

## Status codes (`s=`)

| Code | Meaning |
|------|---------|
| `0` | not started — **default, must be omitted** |
| `1` | pending |
| `2` | active |
| `3` | review |
| `4` | approved / accepted |
| `5` | done |
| `6` | cancelled |
| `7` | blocked |
| `8` | error |

---

## Priority codes (`p=`)

| Code | Meaning |
|------|---------|
| `1` | high |
| `2` | medium — **default, must be omitted** |
| `3` | low |

---

## Common properties (all types)

| Prop | Meaning |
|------|---------|
| `t=` | subtype |
| `s=` | status (omit if 0) |
| `p=` | priority (omit if 2) |
| `v=` | version |
| `to=` | destination / routing |
| `fr=` | source / provenance |
| `cf=` | confidence in this record, 0.0-1.0 |
| `n=` | human-readable label or short annotation |
| `dt=` | date YYYYMMDD |
| `by=` | responsible entity id |
| `tags=` | labels list |
| `neg=` | `1` = this record asserts the negative |

`cf=` on a record is independent from `cf=` in the file header.
Header `cf=` = confidence in the digest as a whole.
Record `cf=` = confidence in that specific record's encoding.

---

## Long content blocks

Append immediately after a record. A record may have at most one block.

```
RECORD_LINE
<<<
AION content — operators and identifiers only, no natural language
>>>
```

### RAW escape

For verbatim content that cannot be encoded without semantic loss.
`RAW` opens a verbatim sub-block inside `<<<`/`>>>`. It ends at `>>>`.
Multiple lines are allowed. `RAW` must be the last element in the block.

```
C[c1] E[supplier]>E[mvp] @<20261001 !
<<<
E[client]<E[mvp]: ~ack @<+15dw
C[c1a]: timeout => E[mvp].s=4
RAW
Il Fornitore si impegna a consegnare il Prodotto MVP entro il 1 ottobre 2026,
comprensivo di documentazione tecnica, test di accettazione e manuale utente.
La consegna si intende effettuata nel momento in cui il Cliente rilascia
accettazione scritta. In assenza di risposta entro 15 giorni lavorativi,
la consegna si intende tacitamente accettata.
>>>
```

---

## Sequence chains (`->`)

For ordered steps where sequence matters, not just dependency:

```
K[step-1] -> K[step-2] -> K[step-3]
```

Or as a property on each record:

```
K[step-2] ->K[step-1]   # step-2 follows step-1
```

`->` is directional: `A -> B` means A comes before B.

---

## Template records (`TMPL`)

For repeated structures (tables, line items, test cases, observations).

Declare the template once, then instantiate:

```
TMPL[id] fields=[f1,f2,f3,f4]
id[row-id] f1=val f2=val f3=val f4=val
```

Instances use the template name as the type. Fields are named (order-independent).
Field values may reference AION records (`E[id]`, `Q[id]`) or be literals.

```
TMPL[inv-line] fields=[item,qty,rate,total]
inv-line[l1] item=E[svc-1] qty=40h rate=EUR150 total=EUR6000
inv-line[l2] item=E[svc-2] qty=8h rate=EUR250 total=EUR2000
inv-line[l3] item=E[svc-3] qty=12h rate=EUR200 total=EUR2400
```

A consumer that does not recognize a template instance type should check declared `TMPL`
records before emitting `X`. If the type matches a `TMPL[id]`, parse as template instance.

---

## Section grouping (`S`)

```
S[id] t=SUBTYPE +[RECORD_IDS]
```

If `+[...]` is present, it is authoritative for membership.
If `+[...]` is absent, membership is determined by physical position in the file
(records following this `S` until the next `S` or `---`).
The two modes must not be mixed in the same file.

Section subtypes:
```
introduction  background  scope       methodology  results
analysis      conclusion  recommendation  appendix  next-steps
risk          financial
```

---

## Link records (`L`)

```
L[id] >TARGET t=TYPE n=LABEL
```

Target: `>https://...` or `>E[id]` or `>DOC:IDENTIFIER`
Types: `cite ref attach replaces supersedes see-also`

---

## SCHEMA block

Declare custom subtypes before first use. Syntax uses `=` throughout.

```
AION v=3 type=medical
SCHEMA
  E.t = patient doctor hospital drug procedure
  F.t = diagnosis prognosis symptom allergy
  Q.unit = mmHg bpm mgdl
END
```

`SCHEMA` adds vocabulary only. It cannot define new primitive codes or operators.

---

## Delta updates

```
DELTA E[id] prop=val prop=val
```

Merges listed properties into existing record. Unlisted properties unchanged.
`DELTA` targeting a non-existent `[id]` is an error: emit `X reason=missing`.

---

## Error records

```
X src=RECORD field=FIELD reason=REASON got=VAL expected=EXPECTED
```

| Reason | Meaning |
|--------|---------|
| `missing` | required field absent |
| `unknown-type` | unrecognized primitive code |
| `undeclared-subtype` | subtype not in SCHEMA |
| `invalid-value` | value outside allowed range |
| `parse-fail` | block content uninterpretable |
| `unsupported` | version not supported |

---

## Producer rules (MUST)

1. Begin every file with `AION v=3`
2. Every referenced record must have `[id]`; IDs are globally unique in the file
3. Omit `s=0` and `p=2` (defaults)
4. Use `<<<` blocks only when record-level properties are insufficient
5. No natural language outside `RAW` blocks
6. Declare custom subtypes in `SCHEMA` before first use
7. Emit `X` for anomalies detected before sending
8. Add `cf=` to records whose content is probabilistic or inferred
9. Use `->` for sequence, `>>` for logical dependency — never interchange them

## Consumer rules (MUST)

1. Parse `TYPE[id] prop=val` as order-independent
2. Unknown property: ignore, continue
3. Unknown 1-letter type: emit `X type=unknown`, skip record
4. Unknown 2-letter type: check `TMPL` declarations before emitting `X`
5. Missing `[id]` on referenced record: emit `X field=id reason=missing`, skip record
6. `DELTA`: merge, preserve unlisted properties
7. `RAW` block: treat all lines until `>>>` as opaque verbatim string
8. Propagate `fr=` when forwarding records downstream
9. Version mismatch: emit `X field=v reason=unsupported`, halt

---

## Examples

### Contract

```
AION v=3 dt=20260428 type=contract lang=it src=ctr-2026-001.pdf cf=0.95

E[acme] t=org role=client vat=IT12345678901
E[beta] t=org role=supplier vat=IT98765432100
E[mvp] t=product
E[doc-t] t=file n=technical-documentation
E[test] t=file n=acceptance-tests
E[manual] t=file n=user-manual
---
Q[total] =120000 EUR
Q[rate] =10000 EUR *mo
Q[deposit] =20000 EUR
Q[penalty-del] =5000 EUR *d
Q[penalty-pay] =2 pct *d

S[obligations]
C[c1] E[beta]>E[mvp] @<20261001 ! +[E[doc-t],E[test],E[manual]]
<<<
(E[acme]~ack @<+15dw) | (timeout => E[mvp].s=4)
>>>
C[c2] E[acme]>Q[rate] E[beta] @<*mo.01 @>20260601 !
C[c3] law=it-civil forum=trib-mi
C[c4] t=warranty neg=1 n=implied-warranties-excluded

S[risk]
C[r1] C[c1] trigger=delay => Q[penalty-del] E[beta]>E[acme]
C[r2] C[c2] trigger=delay => Q[penalty-pay] E[acme]>E[beta]

S[actions]
K[k1] by=acme t=sign @<20260501 !
K[k2] by=beta t=sign @<20260501 !
K[k3] by=acme E[acme]>Q[deposit] E[beta] @<20260515 ! >>K[k1]
```

---

### Technical specification

```
AION v=3 dt=20260415 type=spec lang=en src=api-spec-v3.md

E[api] t=system v=3
E[client] t=system
E[db] t=system n=postgres-15
---
Q[uptime] =99.9 pct
Q[latency-p99] =500 u n=ms
Q[rate-limit] =1000 r *d
Q[pool] =20 u

S[requirements]
F[r1] t=req E[api] p=1
<<<
E[client]>E[api]: auth=jwt ! @<500ms
E[api]>E[client]: format=json !
E[api]/plaintext-credentials
>>>
F[r2] t=req E[api] p=1
<<<
E[api]>E[db]: pool=Q[pool] !
E[api] rate-limit=Q[rate-limit] *E[client] !
>>>

S[constraints]
C[c1] E[api] deploy=container !
C[c2] E[db] / expose=public !
C[c3] E[api] uptime>=Q[uptime] !
F[nf1] t=req latency-p99<Q[latency-p99] !

S[actions]
K[k1] by=backend t=review F[r1] @<20260422 !
K[k2] by=arch t=approve @<20260425 ! >>K[k1]
K[k3] by=devops t=deploy @<20260501 ~ >>K[k2]
K[k1] -> K[k2] -> K[k3]
```

---

### Research report

```
AION v=3 dt=20260301 type=research lang=en cf=0.91

E[study] t=concept n=llm-efficiency-2026
E[grp-ctrl] t=concept n=control-group
E[grp-int] t=concept n=intervention-group
---
Q[n] =1240 u
Q[period] =6 mo
Q[eff-main] ~=32 pct cf=0.96
Q[eff-sr] ~=41 pct cf=0.88
Q[eff-jr] ~=21 pct cf=0.88
Q[p-val] =0.003 u

S[methodology]
F[m1] t=desc method=rct parts=[E[grp-ctrl],E[grp-int]] +[Q[n],Q[period]]

S[results]
F[f1] t=find cf=0.96
<<<
E[grp-int] Q[eff-main] vs E[grp-ctrl]
Q[p-val] < 0.05 => statistically-significant
>>>
F[f2] t=find cf=0.88
<<<
E[grp-sr] => Q[eff-sr]
E[grp-jr] => Q[eff-jr]
>>>
F[f3] t=warn cf=0.72
<<<
? selection-bias & ? self-report-bias
>>>

S[conclusion]
F[c1] t=concl >>F[f1] cf=0.94
F[lim] t=warn >>F[f3] n=generalizability-limited

S[actions]
K[k1] t=recommend n=replication-study @<20270101 ~
K[k2] t=recommend n=double-blind >>F[f3] ~
```

---

### Meeting minutes

```
AION v=3 dt=20260420 type=minutes lang=en

E[alice] t=person role=pm
E[bob] t=person role=dev
E[carol] t=person role=qa
E[sprint-7] t=concept @<20260430
---
S[decisions]
F[d1] t=claim E[sprint-7]
<<<
E[feat-x] => S[sprint-8]
deadline=unchanged @=20260430
E[alice]<>E[bob]: agreed
>>>

S[actions]
K[k1] by=bob t=implement @<20260424 ! s=2
K[k2] by=carol t=test @<20260426 ! >>K[k1]
K[k3] by=alice t=update @<20260422 ~
K[k1] -> K[k2]

S[risks]
F[r1] t=warn K[k1] ? p=1
<<<
K[k1] delay => K[k2].s=7
K[k2].s=7 => E[sprint-7].s=7
>>>
```

---

### Invoice

```
AION v=3 dt=20260430 type=invoice lang=it src=INV-2026-042.pdf

E[beta] t=org role=supplier vat=IT12345678901
E[acme] t=org role=client vat=IT98765432100
---
TMPL[line] fields=[item,h,rate,total]
line[l1] item=E[svc-1] h=40 rate=EUR150 total=EUR6000
line[l2] item=E[svc-2] h=8 rate=EUR250 total=EUR2000

Q[subtotal] =8000 EUR
Q[vat] =22 pct
Q[vat-amt] =1760 EUR
Q[total] =9760 EUR

C[pay] E[acme]>Q[total] E[beta] @<+30dw ! method=bank-transfer
C[late] trigger=timeout => penalty=2pct *mo E[acme]>E[beta]
K[k1] by=acme t=pay Q[total] @<20260530 !
```
