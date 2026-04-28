# AION - AI Interop Object Notation

**An unambiguous language for AI-to-AI document exchange.**

---

## The problem

When two people exchange a document, each may use an AI to read it. The document was written in natural language for humans -- but neither human will actually read it. The AI on one end produces it; the AI on the other end consumes it.

Natural language is a poor medium for this. It carries grammar, style, redundancy, and ambiguity -- none of which an AI needs. Two AI systems from different vendors can read the same clause and reach different conclusions. A 40-page contract costs ~8,000 tokens to process. Its semantic content fits in ~80.

## The solution

AION is a notation for encoding document semantics without natural language. It is:

- **Unambiguous**: every construct has exactly one interpretation, regardless of which AI reads it
- **Interoperable**: any two AIs loading the same `SKILL.md` interpret the same file identically
- **Token-efficient**: no articles, prepositions, conjugations, or filler
- **Universal**: 7 primitives cover any document type
- **Zero-infrastructure**: both AIs load the same skill file -- that is the entire contract
- **Extensible**: domain-specific subtypes via `SCHEMA` without touching the core

A `.aion` file is a semantic digest. It is not a replacement for the original document -- it is what an AI needs to reason about it without ambiguity.

---

## When to use AION

### Exchanging documents between parties

When two parties exchange a document and both use an AI to read it, AION is advantageous regardless of document length. The value is non-ambiguity and interoperability, not compression. Different AI systems interpreting the same natural language clause can reach different conclusions. AION eliminates that variance -- the skill is the contract.

Use AION for inter-party exchange even on short documents.

### Internal digest for query sessions

When a single AI uses AION as a compressed digest for internal queries, token efficiency matters. The skill costs ~7,500 tokens to load (producer + consumer combined). This overhead is recovered only on documents long enough to benefit from compression -- roughly 3 pages or more. For shorter documents, pass the original text directly.

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
AION v=3 dt=YYYYMMDD type=TYPE lang=XX
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
whitepaper  sdk         rfp         financial
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

- [`contract.aion`](examples/contract.aion) - software development contract
- [`spec.aion`](examples/spec.aion) - API technical specification
- [`research.aion`](examples/research.aion) - research report with findings
- [`minutes.aion`](examples/minutes.aion) - meeting minutes with action items
- [`invoice.aion`](examples/invoice.aion) - invoice with line items

---

## Specification

The full formal specification lives in [`spec/AION-v3.md`](spec/AION-v3.md).

The `SKILL.md` in the repository root is the deployable version -- optimized for loading into AI system prompts.

---

## Versioning

AION uses integer versions. The current version is **3**.

Every `.aion` file declares its version in the header (`v=3`). Consumers that encounter an unsupported version emit `X field=v reason=unsupported got=N expected=3` and halt.

Breaking changes increment the version. Additive changes (new subtypes, new document classes) do not.

---

## The Vision

Documents exist because humans need to exchange meaning. Contracts, medical records, technical specifications, meeting minutes: all are attempts to encode intent, obligation, and fact in a medium another human can read and interpret.

Natural language was the only option. It no longer is.

When both parties to a document exchange use an AI to read it, the document no longer needs to be written for humans. It needs to be written for machines: unambiguously, verifiably, without the redundancy that natural language requires to compensate for the imprecision of human interpretation.

A 40-page contract is long not because it contains 40 pages of substance, but because it must anticipate every possible misreading by every possible human reader. It defines terms both parties already understand. It repeats obligations in multiple forms to ensure at least one formulation lands unambiguously. It includes boilerplate no one reads but everyone includes because omitting it creates risk.

The semantic content of that contract fits in roughly 80 tokens of AION. The rest is defensive redundancy against ambiguity.

If both parties use an AI that loads the same `SKILL.md`, that redundancy becomes unnecessary. Conditions are `C[id]`. Deadlines are `@<YYYYMMDD !`. Penalties are `pen=5000EUR*d`. There is no room for divergent interpretation because there is no natural language.

The same applies across professional documents:

- **Medical records** - structured with `ep=`, `cf=`, explicit dependencies between diagnoses and treatments. An AI reasons on the full clinical history without loss of meaning between practitioners or systems.
- **Technical specifications** - requirements as `F[id] t=req wt=` with explicit dependencies, automatically verifiable against implementation.
- **Meeting minutes** - generated during the meeting. Actions are `K[id] by= @< !`. Decisions are `F[id] t=concl ep=estab`. Shared immediately as `.aion`.

The technical foundation exists today. The remaining barrier is institutional: legal and regulatory systems built around human readability as the guarantee of transparency.

That argument inverts under AION. A formal, machine-readable document is *more* verifiable than prose - `ep=`, `cf=`, `>>` dependencies, and `valid=` ranges are checkable in ways that a paragraph of contractual language is not.

When formal verifiability is recognized as a stronger guarantee than human readability, the document as we know it changes. Not because the substance changes, but because the ceremony around it finally falls away.

---

## Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

## License

MIT. See [`LICENSE`](LICENSE).
