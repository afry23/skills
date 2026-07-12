# Requirements Engineering Skills

A family of Claude Code skills covering the requirements-engineering lifecycle, built on the SOPHIST methodology (MASTER template, Kano model, atomic/testable requirements). Each skill is a focused tool for one stage of the process — from a vague idea, or messy source text, or even an existing codebase, all the way to a validated functional specification with acceptance criteria.

## Skills

### [`requirements-interview-coach`](requirements-interview-coach/SKILL.md)
Interactive, turn-by-turn interview for a vague, early-stage idea with no document yet. Uses the funnel technique (general → specific), the Kano model (basic/performance/delight factors), and live SOPHIST probing of vague answers. Produces a MASTER-template requirements table.

**Use when:** there's no source text — just a person with an idea, talking it through.

### [`requirements-engineering`](requirements-engineering/SKILL.md)
One-shot extraction from existing, messy real-world text (chat logs, meeting notes, email threads, rough spec drafts, Lastenheft/Pflichtenheft excerpts). Produces an atomic requirements table, a terminology glossary (with context and source), and an open-issues/contradictions log.

**Use when:** a document already exists and needs to be turned into structured requirements.

### [`requirements-reverse-engineering`](requirements-reverse-engineering/SKILL.md)
Reconstructs requirements and a functional specification directly from a source code repository — no stakeholder text at all. Explores tests, API routes, data models, business logic, and config in order of reliability, and produces the same artifacts as `requirements-engineering` and `functional-specification`, but with `file:line` citations instead of quoted text, and explicit observed-vs-intended flagging.

**Use when:** the codebase is the only source of truth (missing/outdated docs, pre-migration audit, legacy-system onboarding).

### [`functional-specification`](functional-specification/SKILL.md)
Turns a requirements table (ideally with `REQ-IDs`) into a detailed functional spec: a PlantUML class diagram (information model), PlantUML activity diagram(s) (process flow and business rules), a Gherkin acceptance-criteria table traced back to requirement IDs, and a gap analysis.

**Use when:** requirements exist and need to become a concrete domain model + process model + test scenarios for developers/testers.

### [`gherkin-acceptance-criteria`](gherkin-acceptance-criteria/SKILL.md)
Turns requirements or user stories into precise Given/When/Then acceptance criteria — happy path plus deliberately-sought edge cases, vague quality terms resolved to measurable thresholds, full traceability to the source requirement ID.

**Use when:** only acceptance criteria are needed, without the full diagram-based functional spec (a lighter-weight subset of what `functional-specification` produces).

### [`requirements-validation`](requirements-validation/SKILL.md)
Audits already-written requirements, glossaries, functional specs, or acceptance criteria against SOPHIST quality rules and reports findings — it does **not** rewrite anything. Checks atomicity, MASTER-template compliance, quantifier scope, condition completeness, glossary sourcing, model traceability, decision-diamond completeness, and bidirectional requirement↔criterion coverage. Findings are ranked Blocker/Major/Minor.

**Use when:** artifacts already exist and need a second, skeptical pass before they go to stakeholders or development.

## How they relate

```
                    ┌─────────────────────────────┐
                    │  requirements-interview-coach │  (no source text — live conversation)
                    └──────────────┬───────────────┘
                                   │
  messy source text  ──────►  requirements-engineering        requirements-reverse-engineering  ◄────── source code repo
  (chat/notes/Lastenheft)         │                                        │
                                   │        both produce: requirements table, glossary, open issues
                                   ▼                                        │
                          functional-specification  ◄──────────────────────┘
                          (class diagram, activity diagram,
                           Gherkin acceptance criteria, gaps)
                                   │
                          ┌────────┴─────────┐
                          ▼                  ▼
              gherkin-acceptance-criteria   (used standalone too, when only
              (subset: criteria only,        test scenarios are needed —
               no diagrams)                  no full spec)

                          any of the artifacts above
                                   │
                                   ▼
                        requirements-validation
                    (audits, does not rewrite; run at
                     any point once artifacts exist)
```

**Typical pipelines:**

- **From scratch, no docs at all:** `requirements-interview-coach` → `functional-specification` → `requirements-validation`
- **From messy stakeholder text:** `requirements-engineering` → `functional-specification` (or just `gherkin-acceptance-criteria` if diagrams aren't needed) → `requirements-validation`
- **From an existing, undocumented codebase:** `requirements-reverse-engineering` (covers both the requirements-extraction and functional-spec steps in one pass) → `requirements-validation`
- **Acceptance criteria only, requirements already exist:** `gherkin-acceptance-criteria` directly

**Traceability convention:** every artifact downstream of a requirements table cites the originating `REQ-ID` (or, for reverse-engineered output, a `file:line` code location), so the chain from stakeholder need (or code behavior) → requirement → diagram → test scenario stays auditable in both directions. `requirements-validation` checks specifically that this chain isn't broken anywhere.
