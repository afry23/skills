---
name: requirements-validation
description: Audits already-written requirements documents and/or functional specifications (requirements tables, glossaries, PlantUML class/activity diagrams, Gherkin acceptance criteria) against SOPHIST quality rules and reports concrete findings — it does not rewrite the documents. Use this skill whenever the user has an existing requirements doc, glossary, functional spec, or set of acceptance criteria and wants it reviewed, validated, checked for quality/consistency/completeness, or wants a second opinion before it goes to stakeholders or development — e.g. "can you review these requirements," "check this spec for gaps," "is this MASTER-template compliant," or "validate the traceability between my requirements and my acceptance criteria." This is a review/audit skill, distinct from `requirements-engineering` (which extracts requirements from raw text) and `functional-specification` (which generates a spec) — use this one when the artifacts already exist and need checking, not producing.
---

# Requirements & Specification Validation (SOPHIST method)

## Why this exists

Writing requirements well and checking requirements well are different skills, even though they draw on the same rules. Someone who just wrote a requirements table is the worst-positioned person to spot its own gaps — they already know what they meant, so ambiguous phrasing reads as clear to them. A validation pass needs the opposite stance: read every sentence as if you're seeing it for the first time and have no access to what the author intended, only to what's actually written down.

Take on the mindset of a skeptical reviewer — someone whose job is to find the places where a requirement, a glossary entry, or a diagram doesn't hold up to its own stated rules, not to fix them. This skill produces a findings report; it does not rewrite the source documents. That separation matters: silently "fixing" things while validating hides from the user how much was actually wrong, and conflates two different jobs that are worth keeping distinct.

## What you're validating

This skill covers whichever of the following artifacts the user provides — validate what's there, and note explicitly if an expected companion artifact (e.g. a glossary referenced by the requirements but not provided) is missing rather than silently skipping it.

- A requirements table (`REQ-xxx` rows)
- A glossary (`GLO-xxx` rows)
- An open-issues/contradictions log (`OP-xxx` rows)
- A functional specification: PlantUML class diagram (information model), PlantUML activity diagram(s) (process flows), Gherkin acceptance criteria

## Validation checks

Work through each artifact type systematically. Don't just skim for things that jump out — a rule-by-rule pass catches issues that a read-through misses, especially in a document long enough that early context has faded by the end.

### Requirements table

- **Atomicity** — does each row express exactly one obligation? A row that says "X and Y" bundles two requirements into one, which breaks independent testability.
- **MASTER-template compliance** — does each functional requirement name an explicit actor (not passive voice), an explicit modality (MUST/SHOULD/WILL/CAN), and a concrete object/action? Flag passive constructions ("data is validated") and vague generic actors ("the system," used where a specific component was surely meant) as findings, not as acceptable shorthand.
- **Quantifier scope** — do universal quantifiers ("all," "always," "every," "never") have their scope actually defined, or are they left as unbounded overreach?
- **Condition completeness** — for every "if/when X" requirement, is there a defined behavior for the case where X doesn't hold, or is that gap already captured in an open-issues log? An unhandled condition that isn't flagged anywhere is a finding.
- **Category correctness** — is Functional/Quality/Constraint applied consistently and correctly (e.g. a requirement stating a performance threshold shouldn't be filed as "Functional")?
- **Source/traceability** — does each requirement cite where it came from? A requirement with no traceable source is harder to challenge or update later.

### Glossary

- **No fabricated definitions** — does every definition trace to a quoted/paraphrased source passage, or to explicitly cited standard usage? A definition with no source and no standard-usage citation is a hallucination risk worth flagging even if the definition itself sounds plausible.
- **Context correctly scoped** — for any term that plausibly means different things in different parts of the domain (e.g. "client," "session," "queue"), is each meaning its own row with its own context, rather than one blended definition trying to cover both?
- **Synonym capture** — where the requirements/source material clearly uses two terms for the same concept, is that captured, or does the glossary silently treat them as unrelated?

### Open-issues / contradictions log

- **Coverage** — cross-check the requirements table for contradictions the log should have caught but didn't (e.g. two rows stating different numeric limits for what looks like the same constraint, without an open issue linking them).
- **Requirement linkage** — does each open issue reference the requirement ID(s) it affects, where that's determinable?

### Functional specification

- **Model-to-requirement traceability** — does every class attribute and every relationship in the information model trace back to something the requirements actually state or clearly imply? An attribute with no traceable origin is either a hallucination or an undocumented assumption — either way, worth flagging.
- **Decision completeness** — does every decision diamond in the activity diagram(s) show both branches? A diamond with only one labeled path (or an "undefined" branch that isn't cross-referenced to a gap in the requirements/open-issues) is incomplete.
- **Requirement-to-criterion coverage** — does every requirement ID that should have a testable outcome actually have at least one Gherkin scenario, and does every Gherkin scenario cite a real requirement ID? Orphans in either direction (a requirement with no test coverage, or a scenario with no traceable source) are findings.
- **Gherkin precision** — do any acceptance criteria still contain vague, unmeasurable quality terms ("fast," "user-friendly," "reliable") that should have been resolved to a concrete threshold or flagged as needing one?
- **Cross-diagram consistency** — do the entities referenced in the activity diagram's actions actually exist in the class diagram (and vice versa)? A process step that manipulates an entity the information model never defined is a modeling gap.

## Severity

Not every finding carries the same weight — say so explicitly rather than presenting a flat list that makes a missing Source citation look as serious as an unhandled payment-failure branch:

- **Blocker** — the artifact can't be safely acted on as-is (e.g. a requirement that contradicts another with no resolution, a decision branch with genuinely undefined behavior in a critical flow).
- **Major** — a real quality problem that should be fixed before handoff, but doesn't block understanding the document's overall intent (e.g. a glossary term missing its Context split, a vague quantifier).
- **Minor** — a polish issue (e.g. inconsistent ID formatting, a missing but inferable Source citation).

## Output format

Produce a single validation report, structured as follows.

**1. Summary**

A short paragraph stating what was reviewed (which artifacts were provided) and a rough headline (e.g. "14 findings: 2 blockers, 6 major, 6 minor").

**2. Findings**

A table:

| Column | Content |
|---|---|
| `ID` | Sequential, unique: `VAL-001`, `VAL-002`, ... |
| `Severity` | `Blocker`, `Major`, or `Minor` |
| `Artifact` | Which document/section the finding is in (e.g. "requirements.md", "class diagram", "Gherkin — REQ-004") |
| `Rule` | The specific check that failed (e.g. "MASTER-template — passive voice", "Glossary — no fabricated definitions") |
| `Finding` | What's wrong, stated concretely enough that someone could act on it without re-deriving the problem |
| `Reference` | The specific ID(s) affected (`REQ-xxx`, `GLO-xxx`, etc.), where applicable |

**3. What passed**

Briefly note the checks that were run and found no issues — this tells the user the absence of a finding in that category was a deliberate check, not an oversight. A validation report that only lists problems leaves the reader unsure whether everything else was actually checked or simply not looked at.

Do not edit the source artifacts as part of this skill. If the user wants the findings applied, that's a separate, explicit follow-up request.
