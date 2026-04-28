# AION v3 - Formal Specification (rev. 3.3.1)

This document is the authoritative reference for AION v3.
`SKILL.md` in the repository root is the deployable subset optimized for AI system prompts.
When the two conflict, this document takes precedence.

> **Revision 3.3.1** is fully backward compatible with v3, v3.1, v3.2, v3.3. File headers continue to declare `v=3`.

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

### 3.1.1 `neg` flag shorthand

`neg` as a standalone property flag is equivalent to `neg=1`.
Producers SHOULD prefer `neg`. Consumers MUST accept both.

### 3.1.2 Inline quantities

Quantities may appear inline anywhere a `Q[id]` reference is valid.
Format: `VALUEUNIT` with no space. Frequency modifier appended directly.

```
K[k1] E[acme]>20000EUR E[beta] @<20260515 !
C[r1] pen=5000EUR*d
```

### 3.1.3 TMPL positional syntax

TMPL instances may use positional values instead of named `field=val` pairs.

```
TMPL[line] fields=[item,h:h,rate:EUR,total:EUR]
line[l1] E[svc-1] 40 150 6000
```

### 3.1.4 Extended standard properties

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

### 3.2.1 Multi-source dependency `>>[A,B]`

`>>` becomes variadic. `F[c] >>[F[a],F[b]]` means c follows from A and B together.
`F[c] >>F[a] | >>F[b]` means c follows from A or B. Single-source `>>F[a]` unchanged.

### 3.2.2 Weight property `wt=`

Numeric weight 0-100 on any record. `F[r1] t=req wt=40` - requirement weighted at 40/100.
`p=` (1/2/3 ordinal) and `wt=` (0-100 cardinal) are independent; both may appear.

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
```

### 3.2.5 Parallel operator `||`

```
K[b] || K[c]                    # b and c are concurrent
K[a] -> (K[b] || K[c]) -> K[d] # sequence with parallel block
```

### 3.2.6 New document classes

Added to `type=` vocabulary: `whitepaper sdk rfp financial`

---

## 3.3 Changes in revision 3.3 (additive only)

File headers remain `v=3`. All v3, v3.1, v3.2 syntax remains valid.

### 3.3.1 Epistemic status property `ep=`

Applies to `F` records. Encodes the epistemic standing of the assertion.

| Value | Meaning |
|-------|---------|
| `assert` | stated by a party, not independently verified |
| `claim` | asserted but disputed or unverified |
| `corr` | correlation established, causation not claimed |
| `evid` | empirical evidence |
| `demo` | demonstrated by experiment or formal argument |
| `estab` | established / settled / uncontested |
| `disput` | explicitly contested; use `src=` to identify the contesting party |
| `refut` | refutes another record; use `>>` to identify the target |

### 3.3.2 F subtype additions: `corr`, `evid`, `refut`

| Subtype | Meaning |
|---------|---------|
| `corr` | correlation finding (non-causal) |
| `evid` | empirical evidence supporting a claim |
| `refut` | refutation of another record |

### 3.3.3 Temporal validity property `valid=`

```
valid=YYYYMMDD:YYYYMMDD   # applies from date A to date B (inclusive)
valid=YYYYMMDD:            # applies from date A onward
valid=:YYYYMMDD            # applies until date B
```

### 3.3.4 Digest coverage metadata: `cov=` and `excl=`

| Property | Format | Meaning |
|----------|--------|---------|
| `cov=` | 0.0-1.0 | fraction of source document semantically covered |
| `excl=` | `[id,id]` | section names or identifiers intentionally excluded |

### 3.3.5 Definition linking convention

When a record uses a term defined by an `F[id] t=def` record elsewhere in the file,
the producer SHOULD add `>>F[def-id]` to make the semantic link explicit.

---

## 3.3.1 Changes in revision 3.3.1 (additive only)

File headers remain `v=3`. All v3, v3.1, v3.2, v3.3 syntax remains valid.

### PRODUCER MUST rule 4 strengthened

Explicit prohibition on natural language inside `<<<` blocks without `RAW` opener.
Wrong/right examples added inline to prevent producer non-conformance on dense technical content.

### CMP sub-block

`CMP` opens a compressed prose region inside `<<<` blocks. Preserves meaning with
~60-70% token reduction. Use instead of RAW whenever verbatim reproduction is not required.

### PRODUCER MUST rules 15-16

Guidance on choosing TMPL / AION / CMP / RAW by content structure.

### CONSUMER MUST rule 13

CMP interpretation rules including entity and quantity inverse mapping.

### Section 20 rewritten

Separates data-exchange use case (non-ambiguity, interoperability) from internal
digest use case (token efficiency). Content composition threshold added.

### Section 22 added

The Vision: future of professional document exchange without natural language ceremony.

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

### 4.3 Entities (optional but recommended)

`E` records before content improve navigation for the consumer.

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
| `whitepaper` | White paper |
| `sdk` | SDK or developer documentation |
| `rfp` | Request for proposal |
| `financial` | Financial document |
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
- Anonymous records (no `[id]`): allowed; cannot be referenced

---

## 7. Primitive types

### 7.1 E - Entity

Subtypes (`t=`): `person org system place product file concept role event`

### 7.2 F - Fact

Subtypes (`t=`): `find concl desc req claim def warn note quote corr evid refut`

### 7.3 Q - Quantity

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

### 7.4 K - Action

Subtypes (`t=`): `sign approve review test deploy notify recommend`
Modal operators apply: `!` `~` `/`

### 7.5 C - Condition

If-then logic, constraints, rules, obligations, permissions.

```
C[id] TRIGGER => CONSEQUENCE
C[id] E[actor] MODAL OBJECT @<DEADLINE
```

Logical combinators `&` and `|` are permitted inside `C` records and `<<<` blocks.
Parentheses `()` group sub-expressions. Maximum one level of nesting.

### 7.6 S - Section

```
S[id] t=SUBTYPE +[RECORD_IDS]
```

If `+[...]` is present: authoritative for membership.
If `+[...]` is absent: membership by physical position until next `S` or `---`.
Mixing both modes in the same file is forbidden.

Subtypes:
```
introduction background scope methodology results analysis
conclusion recommendation appendix next-steps risk financial
```

### 7.7 L - Link

```
L[id] >TARGET t=TYPE n=LABEL
```

Targets: `>https://...` | `>TYPE[id]` | `>DOC:IDENTIFIER`
Types: `cite ref attach replaces supersedes see-also`

### 7.8 X - Error

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

---

## 9. Negation

Two equivalent forms:
- `neg` as a standalone flag (preferred)
- `neg=1` as a property (v3 compat, still valid)

Consumers MUST accept both forms.

---

## 10. Date and time

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

One block per record.

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

### 11.2 CMP sub-block

`CMP` opens a compressed prose region. Preserves meaning with ~60-70% token reduction.
Use instead of RAW whenever the text does not need to be reproduced verbatim.
`CMP` must be the last element in the block.

Compression rules:
1. Drop articles, prepositions, auxiliary verbs, redundant punctuation
2. Multi-word concepts → hyphenated-token
3. Lists → `item|item|item` - never commas
4. Sequences → `step1 → step2 → step3`
5. Key-value → `key:value`
6. Quantities inline: `50pct` `3h` `2km` - never prose ("50 percento", "tre ore")
7. Entity references: `E[id]` mandatory - never prose name

Consumer interpretation: apply inverse rules to reconstruct meaning.
CMP is not a lossless encoding - minor stylistic detail may be dropped.
Semantic content must be fully preserved.
Inverse rules: hyphenated-token → multi-word concept, item|item → list,
step → step sequence, key:value → key-value pair,
E[id] → entity name, inline quantities → natural language quantities.

```
F[f1] t=desc
<<<
CMP
phase-1:collect-data → phase-2:validate → phase-3:store-results
>>>

F[f2] t=warn
<<<
CMP
risk-factors: factor-a|factor-b|factor-c. mitigation ! → escalate-if-unresolved
>>>
```

Producer MUST use RAW (not CMP) for verbatim text: legal clauses, quotations, citations.

---

## 12. Template records (`TMPL`)

```
TMPL[id] fields=[f1,f2,f3]
```

- Instances use the `TMPL[id]` name as their type
- **Named syntax**: fields are `name=val` pairs, order-independent (v3 compat)
- **Positional syntax**: values in declaration order, no field names (v3.1, preferred for tables)
- Mixed syntax within the same TMPL is forbidden

```
TMPL[obs] fields=[subject,value:pct,period:mo,cf:0-1]
obs[o1] E[grp-int] 32 6 0.96
obs[o2] E[grp-sr]  41 6 0.88
```

---

## 13. Delta updates

```
DELTA TYPE[id] prop=val prop=val
```

Merges listed properties. Unlisted properties unchanged.

---

## 14. Common properties

| Prop | Meaning | Default |
|------|---------|---------|
| `t=` | subtype | (none) |
| `s=` | status | `0` (must omit) |
| `p=` | priority | `2` (must omit) |
| `v=` | version | (none) |
| `to=` | destination | (none) |
| `fr=` | source | (none) |
| `cf=` | record-level confidence | (none) |
| `n=` | human label or annotation | (none) |
| `dt=` | date | (none) |
| `by=` | responsible entity | (none) |
| `tags=` | label list | (none) |
| `neg` | negated assertion - standalone flag or `neg=1`, both accepted | (none) |

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
4. No natural language outside `RAW` blocks. `<<<` blocks contain AION only.
   Free text inside `<<<` without `RAW` opener = non-conforming output.

   Non-conforming:
   ```
   F[f1] t=def n=consegna
   <<<
   La consegna avviene quando il fornitore trasferisce il prodotto al cliente.
   >>>
   ```

   Conforming - free text requires `RAW`:
   ```
   F[f1] t=def n=consegna
   <<<
   RAW
   La consegna avviene quando il fornitore trasferisce il prodotto al cliente.
   >>>
   ```

   Conforming - pure AION inside `<<<`, no `RAW` needed:
   ```
   C[c1] E[b]>E[mvp] @<20261001 !
   <<<
   E[a]~ack @<+15dw | (timeout => E[mvp].s=4)
   >>>
   ```
5. Emit `X` for anomalies detected before sending
6. Declare custom subtypes in `SCHEMA` before first use
7. Add `cf=` to records whose content is probabilistic
8. Use `->` for sequence, `>>` for logical dependency
9. Choose block content by structure:
   - Tabular/repeated structure → TMPL
   - Operators, dependencies, conditions → AION pure inside `<<<`
   - Compressible prose → CMP
   - Verbatim text (legal clauses, quotations, citations) → RAW
   RAW is last resort. Default to TMPL, AION, or CMP.
10. Prefer CMP over RAW for compressible prose. Use RAW only for verbatim text
    that must be reproduced exactly.

SHOULD:
1. Place `E` declarations before content
2. Use `S` to mirror source document structure
3. Include `dt=`, `lang=`, `type=` in header
4. Prefer `neg` flag over `neg=1`
5. Use inline quantities for single-use values
6. Use positional TMPL syntax for dense tabular data
7. Use extended standard properties (`jur=` `ven=` `pen=` `cnd=` `mth=`) over freestyle synonyms

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
10. Accept both `neg` and `neg=1` as equivalent
11. Accept both named and positional TMPL instance syntax
12. Accept inline quantities (`20000EUR`, `2pct*d`) wherever `Q[id]` is valid
13. CMP block: interpret compressed tokens as meaning-preserving prose reduction.
    Apply rules inverse to production: hyphenated-token → multi-word concept,
    item|item → list, step → step sequence, key:value → key-value pair,
    E[id] → entity name, inline quantities → natural language quantities.

---

## 20. When to use AION

AION serves two distinct use cases with different cost/benefit profiles.

### 20.1 Data exchange between parties

When two parties exchange a document and both use an AI to read it, AION is
advantageous regardless of document length. The value is not compression - it is:

- **Non-ambiguity**: every construct has exactly one interpretation. Natural language
  leaves dates, obligations, and conditions open to divergent reading by different models.
- **Interoperability**: two AI systems from different vendors with different training
  interpret the same AION file identically, because the skill is the contract.
- **Auditability**: an AION file is verifiable. `ep=`, `cf=`, `>>` dependencies,
  and `valid=` ranges are checkable in ways that prose is not.

For data exchange, use AION even on short documents. The non-ambiguity value
is independent of token savings.

### 20.2 Internal digest for query sessions

When a single AI uses AION as a compressed digest of a document for internal
query purposes, token efficiency determines suitability. Both producer and
consumer load the skill (~7,500 tokens combined overhead per session).
This overhead is recovered only when documents are long enough to benefit
from compression.

| Document length | Compression ratio | Recommendation |
|----------------|-------------------|----------------|
| < 3 pages / < 500 tokens | < 1x (overhead exceeds savings) | do not use AION |
| 3-10 pages / 500-2,000 tokens | 2x-5x | use if multiple docs per session |
| 10-30 pages / 2,000-8,000 tokens | 5x-10x | use |
| > 30 pages / > 8,000 tokens | 10x-15x | strongly recommended |

For short documents (invoices, brief emails, short minutes), pass the original
text directly. The structural overhead of entity declarations, section markers,
and property syntax exceeds the natural language savings for documents under
~500 tokens.

The overhead amortizes quickly across a session: loading the skill once and
processing 5 long documents yields ~5x net token savings per document.

### 20.3 Content composition threshold

Length alone does not determine AION suitability. Consider content composition:

| Content type | RAW/CMP ratio | Recommendation |
|---|---|---|
| Structured data, conditions, actions | <20% | AION strongly recommended |
| Mixed structured + prose | 20-60% | AION recommended if >10 pages |
| Dense prose, definitions, procedures | >60% | evaluate against use case |

High RAW/CMP ratio means structural overhead is not recovered in compression.
For data exchange (20.1), AION retains full value even at high RAW/CMP ratios
because non-ambiguity is the primary goal. For internal digest use (20.2),
consider passing the original text directly when RAW/CMP ratio exceeds 60%.

---

## 21. Versioning policy

- Current version: **3**
- Breaking changes increment the integer
- Additive changes (new subtypes, new document classes) do not
- Consumers declare supported versions; multi-version support is permitted

---

## 22. The Vision

Documents exist because humans need to exchange meaning. Contracts, medical records,
technical specifications, meeting minutes: all of these are attempts to encode
intent, obligation, and fact in a medium that another human can read and interpret.

Natural language was the only option. It no longer is.

When both parties to a document exchange use an AI to read it, the document no longer
needs to be written for humans. It needs to be written for machines: unambiguously,
verifiably, without the redundancy that natural language requires to compensate for
the imprecision of human interpretation.

AION is a step toward that future.

### Documents without ceremony

A 40-page contract today is long not because it contains 40 pages of substance,
but because it must anticipate every possible misreading by every possible human reader.
It defines terms that both parties already understand. It repeats obligations in multiple
forms to ensure at least one formulation is unambiguous. It includes boilerplate that
no one reads but everyone includes because not including it creates risk.

The semantic content of a 40-page contract fits in roughly 80 tokens of AION.
The rest is defensive redundancy against ambiguity.

If both parties use an AI that loads the same `SKILL.md`, that redundancy becomes
unnecessary. The contract becomes a `.aion` file. Conditions are `C[id]`. Deadlines
are `@<YYYYMMDD !`. Penalties are `pen=5000EUR*d`. There is no room for divergent
interpretation because there is no natural language.

### What this enables

**Contracts** - generated, reviewed, and signed in minutes. No legal boilerplate.
No ambiguous clauses. Obligations, deadlines, and penalties are machine-verifiable
before signing.

**Medical records** - structured with `ep=`, `cf=`, explicit dependencies between
diagnoses and treatments, precise timelines. An AI consumer reasons on the full
clinical history without loss of meaning in translation between practitioners or systems.

**Technical specifications** - requirements as `F[id] t=req wt=` with explicit
dependencies, automatically verifiable against implementation. No 200-page Word
documents that engineers read in diagonal.

**Meeting minutes** - generated during the meeting, not after. Actions are `K[id] by=
@< !`. Decisions are `F[id] t=concl ep=estab`. Shared as `.aion` immediately.

### The missing piece

The technical foundation exists today. The remaining barrier is institutional:
legal and regulatory systems built around human readability as the guarantee of
transparency and validity.

The argument for natural language as the standard of record is that any party
can read it and verify its meaning without specialized tools. AION inverts this:
a formal, machine-readable document is *more* verifiable than prose, not less.
`ep=`, `cf=`, `>>` dependencies, and `valid=` ranges are checkable in ways that
a paragraph of contractual language is not.

When formal verifiability is recognized as a stronger guarantee than human readability,
the document as we know it changes. Not because the substance changes, but because
the ceremony around it finally falls away.
