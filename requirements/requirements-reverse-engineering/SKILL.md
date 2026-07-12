---
name: requirements-reverse-engineering
description: Reconstructs requirements and a functional specification directly from a source code repository — no stakeholder text needed. Explores the codebase (tests, API routes, data models, business logic, config/feature flags) and produces the same artifacts as requirements-engineering (requirements table, glossary, open issues) and functional-specification (PlantUML class/activity diagrams, Gherkin acceptance criteria), but sourced from file:line citations instead of quoted text. Use this skill whenever the user has an existing codebase with missing, outdated, or absent requirements documentation and wants to know "what does this system actually do," needs a spec reconstructed before a rewrite/migration/audit/onboarding, or asks to "reverse engineer requirements from this repo" or "document this legacy codebase." Distinct from the text-based skills: use this one when the primary source of truth is code, not prose.
---

# Reverse-Engineering Requirements & Functional Specs from Code (SOPHIST method)

## Companion skills

This skill reuses output conventions from two sibling skills in this repo — read them for the exact table/diagram formats before producing output:
- `../requirements-engineering/SKILL.md` (requirements table, glossary, open-issues log)
- `../functional-specification/SKILL.md` (PlantUML class/activity diagrams, Gherkin acceptance criteria)

## Why this exists

The other skills in this family start from prose — meeting notes, stakeholder interviews, existing requirements text — because a human already tried to write down what the system should do. Plenty of real systems don't have that: the documentation is missing, wrong, or was never written, and the only reliable source of truth left is the code itself. Reverse-engineering requirements from code is a different exercise from extracting them from text, because code tells you *what the system currently does*, not *what it was supposed to do* or *why*. A requirement reconstructed this way documents observed behavior — it is evidence about intent, not a stakeholder's direct statement of intent, and that distinction needs to stay visible in the output rather than getting flattened into a normal-looking requirements table.

Take on the mindset of a business analyst doing an archaeology pass on an unfamiliar codebase: curious about what the system is actually for, skeptical of comments and docs that might be stale, and treating executable behavior (especially tests) as the most trustworthy evidence available.

## Where to look, roughly in order of reliability

1. **Automated tests** (unit/integration/e2e). These are usually the single best source: a test explicitly states an expected input, an expected outcome, and often the edge cases someone cared enough to write down. Read these early — they orient you to intended behavior faster than reading implementation code line by line, and they're a natural source for Gherkin scenarios later.
2. **API routes / controllers / CLI entry points**, and their validation and authorization logic — this is the actual contract the system exposes, which is usually closer to "requirements" than internal implementation.
3. **Data models** (DB schema, migrations, ORM classes, type definitions) — ground truth for the information model, more reliable than a written data dictionary that may have drifted.
4. **Business/service logic layer** — where domain rules, calculations, and branching actually live; this is where most of the process model and business rules come from.
5. **UI components/forms**, if present — often reveal user-facing behavior and validation that isn't visible from the API layer alone.
6. **Config, feature flags, environment-based branching** — frequently hide conditional requirements ("this behavior only applies for the enterprise tier," "disabled in production"). Easy to miss if you only read the main code path.
7. **Comments, docstrings, README, commit messages, CHANGELOG** — useful for rationale and context, but treat as secondary evidence. Code behavior wins when a comment and the code disagree; log the mismatch as a gap rather than trusting the comment (stale comments describing removed or changed behavior are extremely common, and are themselves worth flagging, not silently resolving in the comment's favor).

## Deciding what's a requirement vs. an implementation detail

Not every function or branch belongs in the requirements table — a caching layer, a retry loop, a variable name, or an internal refactor generally isn't something a stakeholder would recognize as "something the system does for them." Ask: would a business stakeholder care about this behavior, or is it purely a mechanism for delivering an already-captured requirement? When genuinely unsure, lean toward including it — a requirements doc missing a real business rule is a worse failure than one with an extra boundary-case entry the reader can disregard.

## Workflow

1. **Map the shape of the codebase first**, before reading line by line: find the entry points (routes, CLI commands, queued jobs, scheduled tasks), the data models, and the test suite's structure. You need the overall shape before individual details make sense — reading files in an arbitrary order tends to produce a requirements doc that reflects whatever you happened to read first rather than the system's actual structure.
2. **Read the tests next.** They're usually the fastest, most reliable route to intended behavior, including the edge cases and error handling that are easy to miss from implementation alone. A skipped or clearly-outdated test is itself a signal worth noting.
3. **Trace each business-relevant entry point** through the service/business-logic layer to its outcome: what triggers it, what conditions branch its behavior, what it persists or returns or causes as a side effect, and what validation/authorization guards it.
4. **Produce the requirements-engineering artifacts** (requirements table, glossary, open-issues log) using that skill's MASTER template, glossary Context/Source conventions, and gap categories — with one change: the `Source` column cites `file:line` (or a fully-qualified symbol name) instead of a quoted sentence.
5. **Produce the functional-specification artifacts** (PlantUML class diagram sourced from the real data models, PlantUML activity diagram(s) sourced from the real control flow, Gherkin acceptance criteria) — adapt existing test cases into Gherkin scenarios directly where they already express the right shape, rather than writing new scenarios from scratch when a good one already exists in the test suite.
6. **Flag observed-vs-intended distinctions explicitly.** If a requirement is inferred purely from what the code currently does, and you have no independent way to confirm this matches actual business intent (as opposed to, say, a long-standing bug the system now silently depends on), say so in the gap analysis rather than presenting it with the same confidence as a requirement backed by a test or a clear API contract.

## Gap analysis: code-specific additions

Beyond the standard gap categories (contradiction, ambiguity, missing information), watch specifically for:

- **Dead or unreachable code**, or a feature flag permanently set to disabled — implies a requirement that used to be active but may no longer be. Flag rather than silently omitting; it may be an intentionally retired feature worth documenting as "currently inactive" rather than erasing from the historical record.
- **Comment/code mismatch** — a comment or docstring describing behavior the code doesn't actually implement (anymore, or ever).
- **Untested business-relevant behavior** — a branch that looks business-relevant but has no corresponding test. Worth calling out explicitly: the "requirement" you're documenting here has no automated safety net, which matters if this doc is feeding into a rewrite.
- **Implicit business rules buried in error handling** — a validation block or try/catch often encodes a rule nobody wrote down anywhere else (e.g., "orders under a certain amount are silently rejected"). These are easy to miss reading only the happy path, and are usually exactly the kind of undocumented behavior this exercise exists to surface.

## Output format

Produce the same file/section set as the two companion skills combined, following their exact table and diagram conventions (read those files for the precise structure): a requirements table, a glossary, an open-issues log, a PlantUML class diagram, PlantUML activity diagram(s), and a Gherkin acceptance-criteria table. The one structural difference throughout: every `Source` reference is a code location (`path/to/file.ext:123` or a symbol name) rather than a quoted text passage. Where a requirement's confidence is "inferred from observed behavior, not independently confirmed," say so directly in that row or in the gap analysis rather than leaving it implicit.
