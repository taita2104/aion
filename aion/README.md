# AION — AI Interop Object Notation

**A token-efficient artificial language for AI-to-AI document exchange.**

---

## The problem

When two people exchange a document, each may use an AI to read it. The document was written in natural language for humans -- but neither human will actually read it. The AI on one end produces it; the AI on the other end consumes it.

Natural language is expensive for this. It carries grammar, style, redundancy, and ambiguity -- none of which an AI needs. A 40-page contract costs ~8,000 tokens to process. Its semantic content fits in ~80.

## The solution

AION is a notation for encoding document semantics without natural language. It is:

- **Token-efficient**: no articles, prepositions, conjugations, or filler
- **Unambiguous**: every construct has exactly one interpretation
- **Universal**: 7 primitives cover any document type
- **Zero-infrastructure**: both AIs load the same skill file -- that is the entire contract
- **Extensible**: domain-specific subtypes via `SCHEMA` without touching the core

A `.aion` file is a semantic digest. It is not a replacement for the original document -- it is what an AI needs to reason about it.

---

## Quick example

A contract clause in natural language (~55 tokens):

```
The Supplier shall deliver the MVP product by October 1st 2026, including
technical documentation, acceptance tests and user manual. If the Client
does not respond within 15 working days, delivery is deemed accepted.
```

The same clause in AION (~14 tokens):

```
C[c1] E[supplier]>E[mvp] @<261001 ! +[E[doc-t],E[test],E[manual]]
<<<
E[client]<E[mvp]: ~ack @<+15dw
C[c1a]: timeout => E[mvp].s=4
>>>
```

---

## Seven universal primitives

Every document -- regardless of type -- is fully describable with seven semantic primitives:

| Code | Name | Encodes |
|------|------|---------|
| `E` | Entity | Any named thing: person, org, system, place, concept, product, file |
| `F` | Fact | Any stated truth: finding, claim, description, definition, result |
| `Q` | Quantity | Any measurable value with unit |
| `K` | Action | Anything that must or should be done |
| `C` | Condition | Any if-then, constraint, rule, obligation, permission |
| `S` | Section | A structural grouping within the document |
| `L` | Link | A reference to an internal record or external resource |

---

## How it works

```
Human A                          Human B
   |                                |
   | has a document                 |
   |                                |
   v                                |
[AI Producer]                       |
   | loads SKILL.md                 |
   | generates .aion digest         |
   |                                |
   |-------- .aion file ----------->|
                                    v
                              [AI Consumer]
                                loads SKILL.md
                                answers questions
                                takes actions
```

Both AIs load the same `SKILL.md`. The skill is the contract. No parser, no middleware, no API.

---

## File structure

```
AION v=2 dt=YYMMDD type=TYPE lang=XX
---
SCHEMA          # optional domain extensions
---
E[...] F[...]   # entity declarations
---
C[...] K[...]   # content records with optional <<< >>> blocks
```

---

## Supported document types

```
contract    report      spec        email       minutes
proposal    policy      invoice     brief       manual
research    plan        form        audit       press
```

Any other: `type=doc`

---

## Getting started

### 1. Install the skill

Copy `SKILL.md` into your AI system prompt, or install it as a skill if your platform supports the skill format.

### 2. Generate a `.aion` digest

Ask your AI (with SKILL.md loaded):

```
Generate an AION digest of this document.
```

### 3. Share the `.aion` file

Send it to your counterpart. Their AI (also with SKILL.md loaded) can now reason about the document without reading the original.

### 4. Query the digest

```
Based on this .aion file, what are the payment obligations and deadlines?
```

---

## Examples

See the [`examples/`](examples/) directory for complete `.aion` files across document types:

- [`contract.aion`](examples/contract.aion) — software development contract
- [`spec.aion`](examples/spec.aion) — API technical specification
- [`research.aion`](examples/research.aion) — research report with findings
- [`minutes.aion`](examples/minutes.aion) — meeting minutes with action items
- [`invoice.aion`](examples/invoice.aion) — invoice with line items

---

## Specification

The full formal specification lives in [`spec/AION-v2.md`](spec/AION-v2.md).

The `SKILL.md` in the repository root is the deployable version -- optimized for loading into AI system prompts.

---

## Versioning

AION uses integer versions. The current version is **2**.

Every `.aion` file declares its version in the header (`v=2`). Consumers that encounter an unsupported version emit `X field=v reason=unsupported got=N expected=2` and halt.

Breaking changes increment the version. Additive changes (new subtypes, new document classes) do not.

---

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

## License

MIT. See [`LICENSE`](LICENSE).
