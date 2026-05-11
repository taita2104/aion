# Contributing to AION

AION is a specification project. Contributions fall into four categories.

---

## What you can contribute

### 1. Bug reports
A bug is a case where the specification is ambiguous, contradictory, or produces
different interpretations across AI systems. Open an issue using the Bug Report template.
Include the `.aion` snippet that triggers the ambiguity and the two differing interpretations.

### 2. New document classes
If a common document type is not covered by the existing `type=` list, propose it.
Open an issue using the Feature Request template. Include at least one complete `.aion`
example demonstrating why existing classes are insufficient.

### 3. New subtypes
Entity subtypes (`E.t=`), fact subtypes (`F.t=`), and section subtypes (`S.t=`) can be
extended without breaking the core. Propose new subtypes via issue before opening a PR.

### 4. Examples
New `.aion` files in `examples/` are always welcome. They must:
- Be complete and self-contained
- Use only constructs defined in the current specification
- Include a comment header with the source document type and language

---

## What AION does not accept

- New primitive type codes (E F Q K C S L X are fixed)
- New operators (the operator set is closed in v3)
- Natural language in record bodies outside `RAW` blocks
- Domain-specific forks -- use `SCHEMA` instead

The primitive and operator sets are intentionally closed. Stability across AI systems
depends on both producer and consumer reading the same skill. Proliferation of codes
breaks that guarantee.

---

## Process

1. Open an issue before writing any code or spec changes
2. Discuss the proposal -- the bar is: does this make AION more universal, not more specific?
3. If approved, open a PR against `main` with changes to `spec/AION-v3.md` and `SKILL.md`
4. PRs must include at least one new or updated example in `examples/`
5. Breaking changes require a version bump and a migration note in `CHANGELOG.md`

---

## Style guide for spec text

- No natural language in AION examples (enforce what the spec preaches)
- All AION snippets must be syntactically valid
- Properties listed in examples must appear in the spec
- Use `YYYYMMDD` dates, never ISO 8601 or written dates
- Quantities always have units

---

## Questions

Open a Discussion, not an Issue, for questions about intended behavior or usage patterns.
