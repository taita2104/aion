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

# AION v2 — AI Interop Object Notation

AION is a token-efficient artificial language for AI-to-AI information exchange.
No natural language. Every construct has exactly one interpretation.
Universal: applies to any document type without domain-specific extensions.

**Core principle**: producer and consumer both load this skill. The skill is the contract.
No parser, no middleware, no infrastructure.

---

## Design axioms

1. BPE tokenizers assign 1 token to short ASCII sequences and common short English words.
   Unicode symbols cost 2-4 tokens. AION uses only printable ASCII.
2. Every document -- regardless of type -- is fully describable as a combination of
   seven semantic primitives. No more are needed.
3. The type system encodes *semantic role*, not domain. A clause in a contract and a
   requirement in a spec are both `C`. A finding in a report and a stated fact in meeting
   minutes are both `F`.
4. Natural language never appears in AION except inside explicit `RAW:` escape blocks.
5. Unknown constructs are silently forwarded, not rejected (forward compatibility).

---

## Seven universal primitives

| Code | Name | What it encodes |
|------|------|-----------------|
| `E` | Entity | Any named thing: person, org, system, place, concept, product, file |
| `F` | Fact | Any stated truth: finding, claim, description, definition, result |
| `Q` | Quantity | Any measurable value with unit |
| `K` | Action | Anything that must or should be done |
| `C` | Condition | Any if-then, constraint, rule, obligation, permission |
| `S` | Section | A structural grouping of records within the document |
| `L` | Link | A reference to an internal record or external resource |

Errors use a reserved code:

| Code | Name | What it encodes |
|------|------|-----------------|
| `X` | Error | Encoding or interpretation failure |

These eight codes cover all document types. Domain extensions (see SCHEMA) add subtypes,
never new primitive codes.

---

## File header

Every `.aion` file begins with a header line:

```
AION v=2 dt=YYMMDD type=TYPE lang=XX src=ORIGIN
```

| Prop | Meaning | Required |
|------|---------|----------|
| `v=` | AION version | yes |
| `dt=` | document creation date | recommended |
| `type=` | document class (see below) | recommended |
| `lang=` | source language ISO 639-1 | recommended |
| `src=` | origin system or filename | optional |
| `cf=` | producer confidence 0.0-1.0 | optional |

### Document classes (`type=`)

```
contract    report      spec        email       minutes
proposal    policy      invoice     brief       manual
research    plan        form        audit       press
```

If none fits: `type=doc` (generic).

---

## Record structure

```
TYPE[id] prop=val prop=val OPERATOR TARGET ...
```

- `TYPE` is a single uppercase letter
- `[id]` is a unique alphanumeric identifier, no spaces, no special chars
- `prop=val` pairs are space-separated, order-independent
- Operators and targets encode relationships (see Operators)
- Records are separated by blank line or `---`
- `[id]` may be omitted on anonymous records (no other record may reference them)

---

## Entity subtypes (`t=`)

`E` records use `t=` to declare what kind of entity they represent.

```
person  org     system  place   product file    concept role    event
```

Examples:
```
E[alice] t=person role=ceo
E[acme] t=org sector=finance
E[api-gw] t=system lang=python
E[london] t=place
E[mvp] t=product v=1
```

---

## Fact subtypes (`t=`)

`F` records use `t=` to declare the nature of the assertion.

```
find    concl   desc    req     claim   def     warn    note    quote
```

Examples:
```
F[f1] t=find cf=0.94 >>Q[q3]
F[req-1] t=req p=1
F[d-scope] t=desc
F[w1] t=warn
```

---

## Operators

### Relational

| Op | Meaning |
|----|---------|
| `>` | transfers to / delivers to / sends / outputs |
| `<` | receives from / accepts from / inputs |
| `<>` | bilateral / mutual / between |
| `>>` | depends on / follows from / derived from |
| `=>` | implies / if-then / therefore |
| `+` | includes / contains / adds |
| `-` | excludes / removes / except |
| `=` | equals / is defined as / has value |
| `~=` | approximately equals |
| `*` | for each / repeated / applies to all |
| `^` | version of / revision of |

### Temporal

| Op | Meaning |
|----|---------|
| `@` | at (point in time or place) |
| `@<` | by / no later than |
| `@>` | from / no earlier than |
| `@+Nd` | in N calendar days from now |
| `@+Ndw` | in N working days from now |
| `@+Nmo` | in N months from now |
| `*d` | per day / daily |
| `*mo` | per month / monthly |
| `*yr` | per year / annually |

### Modal

| Op | Meaning |
|----|---------|
| `!` | mandatory / must |
| `~` | optional / may / should |
| `/` | prohibited / must not |
| `?` | uncertain / unverified / inferred |

---

## Date and time encoding

Pure numeric, no separators.

| Format | Meaning | Example |
|--------|---------|---------|
| `YYMMDD` | date | `261001` |
| `HHMM` | time 24h | `1430` |
| `YYMMDD.HHMM` | datetime | `261001.1430` |
| `+Nd` | relative N calendar days | `+30d` |
| `+Ndw` | relative N working days | `+15dw` |
| `+Nmo` | relative N months | `+6mo` |

---

## Quantities (`Q`)

`Q` records encode any measurable value.

```
Q[id] =VALUE UNIT *FREQ
```

| Unit | Meaning | Unit | Meaning |
|------|---------|------|---------|
| `EUR` `USD` `GBP` | currency | `pct` | percent |
| `mo` | months | `d` | days |
| `dw` | working days | `h` | hours |
| `MB` `GB` `TB` | storage | `r` | records/rows |
| `u` | generic units | `km` `m` | distance |
| `kg` `g` | weight | `C` `K` | temperature |

Examples:
```
Q[total] =120000 EUR
Q[rate] =10000 EUR *mo
Q[penalty] =5000 EUR *d
Q[dur] =12 mo
Q[size] =2 GB
Q[margin] =23 pct
```

---

## Status codes (`s=`)

| Code | Meaning |
|------|---------|
| `0` | not started |
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
| `2` | medium (default, omit) |
| `3` | low |

---

## Common properties (all types)

| Prop | Meaning |
|------|---------|
| `t=` | subtype |
| `s=` | status |
| `p=` | priority |
| `v=` | version |
| `to=` | destination / routing |
| `fr=` | source / provenance |
| `cf=` | confidence 0.0-1.0 |
| `n=` | short annotation |
| `dt=` | date YYMMDD |
| `by=` | responsible entity id |
| `tags=` | categorical labels list |

---

## Long content blocks

Append `<<<`/`>>>` immediately after a record for extended content.

Inside blocks: AION operators and identifiers only. No natural language.
`RAW:` prefix marks verbatim text that cannot be encoded without semantic loss.
One `RAW:` line per block maximum. Treat as opaque string, do not parse.

```
C[c1] E[supplier]>E[mvp] @<261001 ! +[E[doc-t],E[test],E[manual]]
<<<
E[client]<E[mvp]: ~ack @<@+15dw
C[c1a]: timeout => E[mvp].s=4
>>>

C[c2] by=legal-dept
<<<
RAW: Testo normativo verbatim non comprimibile senza perdita semantica.
>>>
```

---

## Section grouping (`S`)

`S` records group related records and map document structure.

```
S[intro] t=introduction
S[findings] t=results +[F[f1],F[f2],F[f3]]
S[actions] t=next-steps +[K[k1],K[k2]]
```

Section subtypes:
```
introduction    background    scope       methodology
results         analysis      conclusion  recommendation
appendix        next-steps    risk        financial
```

---

## Link records (`L`)

```
L[id] >URL|ID t=TYPE n=LABEL
```

Link types:
```
cite    ref     attach  replaces    supersedes    see-also
```

Examples:
```
L[l1] >https://example.com/doc.pdf t=attach n=original-contract
L[l2] >F[f3] t=see-also
L[l3] >DOC:2024-SPEC-001 t=replaces
```

---

## Delta updates

Update an existing record by sending only changed properties:

```
DELTA E[mvp] s=4 dt=260429 n=accepted-by-client
```

Consumer merges listed properties into existing record. Unlisted properties unchanged.

---

## SCHEMA block (domain extensions)

Declare custom subtypes at the top of the file. Core primitive codes are never redefined.

```
AION v=2 type=medical
SCHEMA
  E.t: patient doctor hospital drug procedure
  F.t: diagnosis prognosis symptom allergy
  Q.unit: mmHg bpm mgdl
END
```

SCHEMA only adds vocabulary. It never adds new primitive codes or operators.

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

On missing `[id]` in a referenced record: emit `X`, skip record.
On unknown primitive code: emit `X type=unknown`, skip record.
On unknown property: ignore, continue (forward compatibility).
On unknown subtype not in SCHEMA: emit `X reason=undeclared-subtype`, use `t=unknown`.

---

## Producer rules

1. Every referenced record must have `[id]`
2. Omit `p=2` (default priority) and `s=0` (default status)
3. Use `<<<` blocks only when record-level properties are insufficient
4. No natural language outside `RAW:` lines
5. One `RAW:` line per block maximum
6. Declare custom subtypes in `SCHEMA` before first use
7. Emit `X` for anomalies detected before sending
8. Add `cf=` when output is probabilistic or derived by inference

## Consumer rules

1. Parse `TYPE[id] prop=val` as order-independent key-value
2. Unknown property: ignore, continue
3. Unknown primitive code: emit `X type=unknown`, skip record
4. Missing `[id]` on referenced record: emit `X field=id reason=missing`, skip record
5. `DELTA`: merge, preserve unlisted properties
6. `<<<` block: parse as AION; `RAW:` lines are opaque strings
7. Propagate `fr=` when forwarding records downstream
8. On version mismatch: emit `X field=v reason=unsupported got=N expected=2`

---

## Examples across document types

### Contract

```
AION v=2 dt=260428 type=contract lang=it src=ctr-2026-001.pdf

E[acme] t=org role=client
E[beta] t=org role=supplier
E[m.rossi] t=person role=ceo by=acme
E[j.smith] t=person role=ceo by=beta

Q[total] =120000 EUR
Q[rate] =10000 EUR *mo
Q[deposit] =20000 EUR
Q[penalty-del] =5000 EUR *d
Q[penalty-pay] =2 pct *d

S[obligations]
C[c1] E[beta]>E[mvp] @<261001 ! +[E[doc-t],E[test],E[manual]]
<<<
E[acme]<E[mvp]: ~ack @<+15dw
C[c1a]: timeout => E[mvp].s=4
>>>
C[c2] E[acme]>Q[rate] @<*mo.01 @>260601 !
C[c3] law=it-civil forum=trib-mi

S[risk]
C[r1] C[c1] trigger=delay => Q[penalty-del] E[beta]>E[acme]
C[r2] C[c2] trigger=delay => Q[penalty-pay] E[acme]>E[beta]

S[actions]
K[k1] by=acme t=sign @<260501 !
K[k2] by=beta t=sign @<260501 !
K[k3] by=acme E[acme]>Q[deposit] @<260515 !
```

---

### Technical specification

```
AION v=2 dt=260415 type=spec lang=en src=api-spec-v3.md

E[api] t=system v=3 by=backend-team
E[client] t=system
E[db] t=system t=product n=postgres-15

S[requirements]
F[r1] t=req E[api] p=1
<<<
E[client]>E[api]: auth=jwt ! @<200ms
E[api]>E[client]: response=json !
>>>
F[r2] t=req E[api] p=1
<<<
E[api]>E[db]: pool=20 timeout=30d !
E[api]/store plaintext-credentials
>>>
F[r3] t=req p=2
<<<
E[api] rate-limit=1000r *d *E[client]
E[api] => X[429] ~=F[r1]
>>>

S[constraints]
C[c1] E[api] deploy=container !
C[c2] E[api] lang=[python,go] ~
C[c3] E[db] / expose=public !

Q[uptime] =99.9 pct
Q[latency-p99] =500 ms
Q[storage] =2 TB

S[next-steps]
K[k1] by=backend-team t=review F[r1] @<260422 !
K[k2] by=arch t=approve @<260425 !
```

---

### Research report

```
AION v=2 dt=260301 type=research lang=en src=study-2026-llm.pdf cf=0.91

E[study] t=concept n=llm-efficiency-2026
E[team] t=org n=research-group
Q[n] =1240 u n=sample-size
Q[period] =6 mo

S[methodology]
F[m1] t=desc E[study] method=rct n=randomized-controlled
F[m2] t=desc E[study] +[Q[n],Q[period]]

S[results]
F[f1] t=find cf=0.96
<<<
Q[efficiency] ~= +32 pct >>intervention-group
Q[baseline] = control-group
C[significance] p-val=0.003 < 0.05
>>>
F[f2] t=find cf=0.88
<<<
E[subgroup:senior] => Q[efficiency] ~= +41 pct
E[subgroup:junior] => Q[efficiency] ~= +21 pct
>>>
F[f3] t=warn cf=0.72 n=confound
<<<
C[confound-1] E[study] ? selection-bias
C[confound-2] E[study] ? self-report-bias
>>>

S[conclusion]
F[c1] t=concl >>F[f1] cf=0.94
F[c2] t=concl >>F[f2] cf=0.85
F[lim] t=warn >>F[f3]

S[next-steps]
K[k1] t=recommend replication-study @<270101 ~
K[k2] t=recommend larger-n >>F[lim] ~
```

---

### Meeting minutes

```
AION v=2 dt=260420 type=minutes lang=en src=standup-260420

E[alice] t=person role=pm
E[bob] t=person role=dev
E[carol] t=person role=qa
E[sprint-7] t=concept

S[decisions]
F[d1] t=claim E[sprint-7] scope=reduced -[E[feat-x]]
<<<
E[alice]<>E[bob]: E[feat-x] => S[sprint-8]
C[d1a]: deadline E[sprint-7] unchanged @=260430
>>>
F[d2] t=claim by=carol
<<<
E[qa] E[sprint-7] +regression-suite @<260425
>>>

S[actions]
K[k1] by=bob t=implement E[feat-auth] @<260424 ! s=2
K[k2] by=carol t=test E[feat-auth] @<260426 ! s=0 >>K[k1]
K[k3] by=alice t=update E[roadmap] @<260422 ~ s=0

S[risks]
F[r1] t=warn K[k1] ? delay
<<<
C[r1a]: K[k1] delay => K[k2] blocked
C[r1b]: K[k2] blocked => E[sprint-7].s=7
>>>
```

---

### Invoice

```
AION v=2 dt=260430 type=invoice lang=it src=INV-2026-042.pdf

E[beta] t=org role=supplier vat=IT12345678901
E[acme] t=org role=client vat=IT98765432100
E[inv] t=file n=INV-2026-042

Q[subtotal] =8000 EUR
Q[vat] =22 pct
Q[vat-amt] =1760 EUR
Q[total] =9760 EUR
Q[due] =30 dw

F[f1] t=desc E[beta]>E[acme] E[inv]
<<<
E[svc-1] t=concept n=sviluppo-modulo-A h=40 rate=150 EUR =6000 EUR
E[svc-2] t=concept n=consulenza-arch h=8 rate=250 EUR =2000 EUR
>>>

C[pay] E[acme]>Q[total] E[beta] @<+30dw ! method=bonif
C[late] timeout => Q[penalty] =2 pct *mo E[acme]>E[beta]

K[k1] by=acme t=pay Q[total] @<260530 !
```
