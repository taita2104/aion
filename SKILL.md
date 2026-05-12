---
name: aion
description: >
  Token-efficient protocol for AI-to-AI document exchange. Load into the system prompt of every
  agent that produces or consumes AION output. Both producer and consumer load this skill —
  the skill is the contract. Also use when generating a .aion semantic digest of a document.
  Triggers: AI pipeline · agent to agent · AION format · .aion · generate aion · aion digest ·
  no human in the loop · interop between agents · produce output for another AI
---

# AION v3.3.3

Token-efficient artificial language for AI-to-AI document exchange. No natural language.
Both producer and consumer load this skill. The skill is the contract.
File headers declare `v=3`. v3.3.3 is fully backward compatible with all prior v3 revisions.

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
F.t= `find concl desc req claim def warn note quote corr evid refut item`
K.t= `sign approve review test deploy notify recommend act`
S.t= `introduction background scope methodology results analysis conclusion recommendation appendix next-steps risk financial section`
L.t= `cite ref attach replaces supersedes see-also`

`t=` is optional on F, K, S. Omitting it is equivalent to `t=item` / `t=act` / `t=section`.
Use specific subtypes when the classification adds semantic value a consumer can act on.

S: `S[id] t=SUBTYPE [+[ids]]` — `+[ids]` authoritative for membership; else positional. Do not mix.
L: `L[id] >URL|>TYPE[id]|>DOC:ID t=TYPE [n=LABEL]`

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
consequence: TYPE[id].prop=val  set property on referenced record (C records after => only)
```

`>>` logical dependency. `->` strict sequence. Not interchangeable.
`~` standalone modal. `~=` compound relational. Never ambiguous: `~=` has operands both sides.
`neg` / `neg=1`: equivalent; `neg` preferred. Asserts record content is negated/excluded.
Multi-source: `F[c] >>[F[a],F[b]]` (and) · `F[c] >>F[a] | >>F[b]` (or).
Parallel: `K[a] -> (K[b] || K[c]) -> K[d]`

## DATES

`YYYYMMDD  HHMM  YYYYMMDD.HHMM  +Nd  +Ndw  +Nmo` — relative to header `dt=`.
If `dt=` absent and relative dates present: emit `X field=dt reason=missing`, dates unresolvable.

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

`ep=` vs `t=`: `t=corr|evid|refut` implies ep= — omit it. Use `ep=` only when `t=` differs.
Header extras: `cov=0-1` coverage · `excl=[id,id]` excluded sections.
Extended standard: `jur= ven= pen= cnd= mth= fmt= via= ref= dur= qty=`
Use extended props over freestyle synonyms.

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
3. Lists → item|item|item — never commas
4. Sequences → step1 → step2 → step3
5. Key-value → key:value
6. Quantities inline: `50pct` `3h` `2km` — never prose
7. Entity references: `E[id]` mandatory — never prose name

## TMPL

```
TMPL[id] fields=[f1:unit,f2,f3]
id[row] f1=v f2=v f3=v          # named (order-independent)
id[row] v1 v2 v3                # positional (preferred for dense tables)
```

Mode from first instance row. Do not mix in same TMPL.
Field values: `E[id]` `Q[id]` inline quantities or literals.

## SCHEMA

```
SCHEMA
  E.t = subtype1 subtype2
  Q.unit = unit1 unit2
END
```

Additive only. No new primitive codes or operators.

## DELTA

`DELTA TYPE[id] prop=val ...` — merges listed props, preserves rest.

## ERRORS

`X src=REC field=F reason=REASON [got=V expected=V]`
Reasons: `missing unknown-type undeclared-subtype invalid-value parse-fail unsupported`

## DOCUMENT CLASSES

```
contract report spec email minutes proposal policy invoice brief manual
research plan form audit press whitepaper sdk rfp financial
article presentation memo ticket announcement doc
```
`doc` = generic fallback; use SCHEMA to extend subtypes for domain-specific docs.

## PRODUCER MUST

1. Header `AION v=3`
2. Globally unique `[id]` on every referenced record
3. Omit `s=0` and `p=2`
4. No natural language outside RAW blocks. Free text inside `<<<` requires `RAW` opener.
5. Emit `X` for anomalies before sending
6. Declare custom subtypes in SCHEMA before first use
7. `cf=` on probabilistic/inferred records
8. `neg` over `neg=1` · inline Q for single-use values
9. `wt=` not `p=` for numeric weight · `vs=`+`d=` for comparisons
10. Extended props (`jur=` `ven=` etc.) over freestyle synonyms
11. `ep=` on F records where epistemic status is meaningful
12. `valid=` on C records with bounded temporal applicability
13. `>>F[def-id]` on records that use a term defined elsewhere
14. Block type by content: tabular → TMPL · operators → pure `<<<` · prose → CMP · verbatim → RAW (last resort)
15. `dt=` in header whenever relative date expressions are present

## CONSUMER MUST

1. Parse `TYPE[id] prop=val` order-independent
2. Unknown prop: ignore, continue
3. Unknown 1-letter type: `X type=unknown`, skip
4. Unknown 2-letter type: check TMPL first
5. Missing `[id]` on referenced record: `X field=id reason=missing`, skip
6. DELTA: merge, preserve unlisted
7. Propagate `fr=` downstream
8. Version mismatch: `X field=v reason=unsupported`, halt

## EXAMPLE

```
AION v=3 dt=20260512 type=report lang=en cf=0.9

E[team] t=org n=Engineering
E[sys] t=system n=payments-api

S[s1]
F[f1] t=find n=latency cf=0.95
<<<
CMP
E[sys] p99-latency degraded: 420ms vs 180ms baseline · onset +3d · affects E[team]
>>>
F[f2] t=concl >>F[f1]
<<<
CMP
root-cause: connection-pool exhaustion under peak-load
>>>

K[k1] t=deploy by=team @<+5dw ! n=patch-v2.3.1
K[k2] t=review by=team @<+2dw ! >>K[k1]
```
