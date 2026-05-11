# AION Governance

AION is an open specification. This document defines how it evolves.

---

## 1. Roles

### Editor

The Editor is the final decision authority for the specification. The Editor:

- Accepts or rejects AIPs
- Publishes spec revisions
- Maintains `SKILL.md` and the extension registry

**Current Editor:** [GitHub handle]

A single Editor may appoint co-editors by opening a PR that updates this file.

### Contributor

Anyone may propose a change by opening an AIP (see section 3). No prior approval required.

---

## 2. Scope

This governance covers:

| Artifact | Description |
|---|---|
| `spec/AION-v*.md` | Formal specification — authoritative |
| `SKILL.md` | Deployable skill file |
| `examples/` | Reference `.aion` files |
| `REGISTRY.md` | SCHEMA extension registry (voluntary) |

It does not govern implementations, tooling, or systems built on top of AION.

---

## 3. Change categories

| Category | Version bump | Multi-LLM test |
|---|---|---|
| **Breaking** | Yes — integer increment (`v=N+1`) | Required |
| **Additive** | No — header stays `v=N` | Required |
| **Editorial** | No | Not required |
| **SKILL.md compression** | No | Required (verify no normative loss) |

**Breaking** — any change that causes a conforming `v=N` file to fail with a `v=N+1` consumer.

**Additive** — new subtypes, properties, document classes. Does not invalidate existing files.

**Editorial** — wording, examples, formatting. Zero normative effect.

**SKILL.md compression** — token reduction without normative change. Behavior must be verified unchanged.

---

## 4. AIP process

An **AION Improvement Proposal** (AIP) is the formal unit of change.

### Step 1 — Proposal

Open a GitHub issue titled `AIP: [short description]`. Include:

- **Motivation** — what problem this solves and why `SCHEMA` extension is insufficient
- **Proposal** — the exact change to the spec text and/or `SKILL.md`
- **Compatibility** — breaking or additive?
- **Reference file** — at least one `.aion` example using the proposed construct
- **Universality argument** — why this belongs in the core, not in a `SCHEMA` extension

### Step 2 — Discussion

Minimum 7 days open for comment. The Editor may extend to 14 days for significant proposals. Silence is not consensus — the Editor must issue an explicit decision.

### Step 3 — Multi-LLM verification

Required for all normative changes (breaking and additive). The proposer must demonstrate consistent behavior across at least **two distinct LLM families** (e.g., GPT-class and Claude-class), acting as both producer and consumer.

Attach results to the issue. Results must show:
- A producer generating a conforming `.aion` file with the proposed construct
- A consumer correctly interpreting that file
- No divergence in interpretation between the two LLM families

### Step 4 — Editor decision

The Editor accepts, rejects, or requests revision with documented reasons. Accepted AIPs are assigned a revision label and scheduled for the next release.

### Step 5 — PR

One PR per AIP. Must update:

- `spec/AION-v*.md`
- `SKILL.md` (if normative change or compression)
- `CHANGELOG.md`
- At least one file in `examples/`

---

## 5. Decision criteria

The primary test for any proposed addition is **universality**:

> Does this make AION more universal, or more specific to a domain?

A construct passes if it applies meaningfully across many document types without domain coupling. Domain-specific needs belong in `SCHEMA` extensions, not the core.

Secondary tests:

- **Non-ambiguity** — does the change introduce any construct with more than one valid interpretation under any LLM?
- **Token budget** — does the SKILL.md footprint increase? If so, is the normative value proportional to the cost?
- **Backward compatibility** — breaking changes require exceptional justification; additive changes are strongly preferred

### What AION does not accept

- New primitive type codes — E F Q K C S L X are closed
- New operators — the operator set is closed
- Domain-specific additions to the core — use `SCHEMA`
- Natural language outside `RAW` blocks in any normative example
- Any change that increases SKILL.md token count without proportional normative value

---

## 6. Versioning policy

AION uses a single integer version declared in every file header (`v=N`).

| Change type | Effect on `v=` |
|---|---|
| Breaking | `N` → `N+1` |
| Additive | unchanged |
| Editorial / compression | unchanged |

Revision labels (3.1, 3.2, ...) are informational — they track the additive release sequence within a major version. File headers always declare the major integer only.

### Deprecation policy

A minimum of **one published additive revision** must exist between a deprecation notice and the corresponding breaking change. The deprecation must be documented in `CHANGELOG.md` and noted inline in `spec/AION-v*.md`.

---

## 7. Stability guarantees

| Component | Status |
|---|---|
| Primitive type codes (E F Q K C S L X) | Frozen |
| Core operators | Frozen |
| Standard properties (`t=` `s=` `p=` `cf=` etc.) | Stable — additions additive, removals breaking |
| Extended standard properties (`jur=` `ven=` etc.) | Stable — additions additive |
| Document classes (`type=`) | Open — additions are additive |
| `SKILL.md` token count | May decrease between revisions; never increases without AIP |

---

## 8. SCHEMA extension registry

Producers may define custom subtypes via `SCHEMA`. This is valid without registration.

Registration in `REGISTRY.md` is voluntary and serves one purpose: preventing two producers from using the same subtype name with different meanings across the ecosystem.

To register an extension, open an AIP of type **Extension** with:

- The subtype names and their definitions
- The document types they apply to
- A reference `.aion` example

Registered extensions do not become part of the core specification. They remain in `REGISTRY.md` as documented conventions.

---

## 9. MIME type

The declared MIME type is `text/aion`. Registration with IANA will be pursued once the specification has reached stability across at least two major versions with documented multi-LLM conformance. Until then, `text/aion` should be treated as an unregistered vendor type.

---

## 10. Conflict resolution

1. Disagreements are documented on the AIP issue thread
2. The Editor's decision is final for the current revision
3. A rejected AIP may be re-opened in a future revision with new evidence or a revised proposal

---

## 11. Amendments to this document

Changes to `GOVERNANCE.md` follow the same AIP process as spec changes, with one exception: they require only an editorial review period (7 days) and no multi-LLM testing.
