---
name: aion
description: >
  Token-efficient protocol for AI-to-AI document exchange. Load into the system prompt of every
  agent that produces or consumes AION output. Both producer and consumer load this skill —
  the skill is the contract. Also use when generating a .aion semantic digest of a document.
  Triggers: AI pipeline · agent to agent · AION format · .aion · generate aion · aion digest ·
  no human in the loop · interop between agents · produce output for another AI
---

# AION v3.3

Token-efficient artificial language for AI-to-AI document exchange. No natural language.
Both producer and consumer load this skill. The skill is the contract.
File headers declare `v=3`. v3.3 is fully backward compatible with v3, v3.1, v3.2.

## RECORD

```
TYPE[id] prop=val ... OPERATOR TARGET
```

TYPE: single uppercase letter. [id]: globally unique in file. Anonymous: omit [id].
Records separated by blank line or `---`.

## PRIMITIVES

```
E entity · F fact · Q quantity · K action · C condition · S section · L link · X error
```

Custom types: 2-letter uppercase codes declared in SCHEMA.

E.t= `person org system place product file concept role event`
F.t= `find concl desc req claim def warn note quote corr evid refut`
K.t= `sign approve review test deploy notify recommend`
S.t= `introduction background scope methodology results analysis conclusion recommendation appendix next-steps risk financial`
L.t= `cite ref attach replaces supersedes see-also`

## OPERATORS

```
relational:  >  to    <  from    <>  bilateral    >>  depends-on    >>[A,B]  multi-dep
             ->  precedes    =>  implies    +  includes    -  excludes
             =  equals    ~=  approx    *  foreach    ^  version-of
temporal:    @  at    @<  by    @>  from    @+Nd  days    @+Ndw  workdays    @+Nmo  months
             *d  *mo  *yr  *u  (frequency)
modal:       !  must    ~  may    /  must-not    ?  uncertain
combinators: &  and    |  or    ||  parallel    (max one nesting level)
             & | only in C records and <<< blocks. || only in -> chains.
```

`>>` logical dependency. `->` strict sequence. Not interchangeable.
`~` standalone modal. `~=` compound relational. Never ambiguous: `~=` has operands both sides.

## DATES

`YYYYMMDD  HHMM  YYYYMMDD.HHMM  +Nd  +Ndw  +Nmo` - relative to header `dt=`.

## QUANTITIES

Declared (multi-use): `Q[id] =VALUE UNIT [*FREQ]`
Inline (single-use):  `20000EUR  2pct*mo  500u*d  99.9pct`
Range:                `Q[id] =MIN..MAX UNIT`

Units: `EUR USD GBP JPY CHF · mo d dw h min · KB MB GB TB · pct · km m cm · kg g · C K F · u r`

## STATUS · PRIORITY

`s=` 0 not-started(omit) · 1 pending · 2 active · 3 review · 4 approved · 5 done · 6 cancelled · 7 blocked · 8 error
`p=` 1 high · 2 medium(omit) · 3 low

## PROPERTIES

```
t=  subtype          s=  status        p=  priority      v=  version
n=  label/note       by= responsible   to= destination   fr= source
cf= confidence       dt= date          tags= labels       neg negated (= neg=1)
wt= weight 0-100     vs= compare-to    d=  delta
ep= epistemic-status (F records): assert·claim·evid·corr·demo·estab·disput·refut
valid= temporal validity: YYYYMMDD:YYYYMMDD · YYYYMMDD: · :YYYYMMDD
```

Header extras: `cov=0-1` digest coverage fraction · `excl=[id,id]` intentionally excluded sections.
Extended standard: `jur= ven= pen= cnd= mth= fmt= via= ref= dur= qty=`
Use extended props over freestyle synonyms for inter-producer consistency.

## BLOCKS

```
RECORD
<<<
AION content - operators and identifiers only
RAW
verbatim multi-line text, opaque, ends at >>>
CMP
compressed prose, ends at >>>
>>>
```

One block per record. RAW and CMP must be last element in block.

## CMP

Alternative to RAW for compressible prose. Preserves meaning, reduces tokens ~60-70%.
Opener: `CMP` inside `<<<` block. Rules:

1. Drop articles, prepositions, auxiliary verbs, redundant punctuation
2. Multi-word concepts → hyphenated-token
3. Lists → item|item|item - never commas
4. Sequences → step1 → step2 → step3
5. Key-value → key:value
6. Quantities inline: 50pct 3h 2km - never prose (not "50 percento", not "tre ore")
7. Entity references: E[id] mandatory - never prose name

## TMPL

```
TMPL[id] fields=[f1:unit,f2,f3]
id[row] f1=v f2=v f3=v          # named (order-independent)
id[row] v1 v2 v3                # positional (preferred for dense tables)
```

Mode determined from first instance row. Do not mix in same TMPL.
Field values: `E[id]` `Q[id]` inline quantities or literals.
Consumer: check TMPL before emitting `X` on unknown 2-letter type.

## SEQUENCE · PARALLEL

```
K[a] -> K[b] -> K[c]           # a precedes b precedes c
K[b] || K[c]                   # b and c concurrent
K[a] -> (K[b] || K[c]) -> K[d] # sequence with parallel block
```

## NEGATION

`neg` flag (preferred) or `neg=1` (compat). Both accepted. Asserts record content is false/excluded.

## MULTI-DEPENDENCY

`F[c] >>[F[a],F[b]]` - c follows from A and B together.
`F[c] >>F[a] | >>F[b]` - c follows from A or B.

## SECTION

`S[id] t=SUBTYPE [+[ids]]` - explicit `+[ids]` authoritative; else positional. Do not mix.

## LINK

`L[id] >URL|>TYPE[id]|>DOC:ID t=TYPE [n=LABEL]`

## SCHEMA

```
SCHEMA
  E.t = subtype1 subtype2
  Q.unit = unit1 unit2
END
```

Additive only. Cannot define new primitive codes or operators.

## DELTA

`DELTA TYPE[id] prop=val ...` - merges listed props, preserves rest.

## ERRORS

`X src=REC field=F reason=REASON [got=V expected=V]`
Reasons: `missing unknown-type undeclared-subtype invalid-value parse-fail unsupported`

## DOCUMENT CLASSES

```
contract report spec email minutes proposal policy invoice brief manual
research plan form audit press whitepaper sdk rfp financial doc
```

## PRODUCER MUST

1. Header `AION v=3`
2. Globally unique `[id]` on every referenced record
3. Omit `s=0` and `p=2`
4. No natural language outside RAW blocks. Free text inside `<<<` requires `RAW` opener.
5. Emit `X` for anomalies before sending
6. Declare custom subtypes in SCHEMA before first use
7. `cf=` on probabilistic/inferred records
8. `->` for sequence · `>>` for dependency · never interchange
9. `neg` over `neg=1` · inline Q for single-use values
10. `wt=` not `p=` for numeric weight · `vs=`+`d=` for comparisons
11. Extended props (`jur=` `ven=` etc.) over freestyle synonyms
12. `ep=` on F records where epistemic status is meaningful
13. `valid=` on C records with bounded temporal applicability
14. `>>F[def-id]` on records that use a term defined elsewhere
15. Block type by content: tabular → TMPL · AION operators → pure `<<<` · compressible prose → CMP · verbatim only → RAW (last resort)

## CONSUMER MUST

1. Parse `TYPE[id] prop=val` order-independent
2. Unknown prop: ignore, continue
3. Unknown 1-letter type: `X type=unknown`, skip
4. Unknown 2-letter type: check TMPL first
5. Missing `[id]` on referenced record: `X field=id reason=missing`, skip
6. DELTA: merge, preserve unlisted
7. RAW block: all lines until `>>>` are opaque
8. Propagate `fr=` downstream
9. Version mismatch: `X field=v reason=unsupported`, halt
10. Accept `neg` and `neg=1` as equivalent
11. Accept named and positional TMPL syntax
12. Accept inline quantities anywhere `Q[id]` is valid
13. CMP block: interpret compressed tokens as meaning-preserving prose reduction.
    Apply rules inverse to production: hyphenated-token → multi-word concept,
    item|item → list, step → step sequence, key:value → key-value pair,
    E[id] → entity name, inline quantities → natural language quantities.

## EXAMPLE

```
AION v=3 dt=20260428 type=contract lang=it cf=0.95 cov=0.9 excl=[schedule-b]

E[a] t=org role=client vat=IT12345678901
E[b] t=org role=supplier vat=IT98765432100
E[mvp] t=product
---
Q[fee] =10000 EUR *mo

C[c1] E[b]>E[mvp] @<20261001 ! +[E[doc],E[test],E[man]]
<<<
E[a]~ack @<+15dw | (timeout => E[mvp].s=4)
>>>
C[c2] E[a]>Q[fee] E[b] @<*mo.01 ! jur=it-civil ven=trib-mi
C[c3] cnd=delay pen=5000EUR*d E[b]>E[a] valid=20260601:20271231

K[k1] by=a t=sign @<20260501 !
K[k2] by=b t=sign @<20260501 !
K[k3] by=a E[a]>20000EUR E[b] @<20260515 ! >>K[k1]
```

