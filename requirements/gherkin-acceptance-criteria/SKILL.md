---
name: gherkin-acceptance-criteria
description: Turns natural-language requirements or user stories (ideally with IDs) into precise, testable Gherkin acceptance criteria (Given/When/Then), with multiple scenarios per requirement covering both the happy path and error/edge cases, measurable parameters instead of vague adjectives, and traceability back to the source requirement ID. Use this skill whenever the user has requirements or user stories and wants acceptance criteria, test scenarios, "Given/When/Then," a definition of done, or something QA/testers can work from directly — even if they just say "write test criteria for this" or "how would we test this requirement" without naming Gherkin explicitly.
---

# Gherkin Acceptance Criteria (SOPHIST method)

## Why this exists

A requirement describes what the system should do; it doesn't by itself tell you how you'd know it actually does it. "The system must validate the order" is a requirement — it isn't yet something a tester can execute. Gherkin's Given/When/Then structure forces that translation: to fill it in, you have to name a concrete starting state, a concrete triggering action, and a concrete, observable outcome. That act of filling in the blanks is often what surfaces the fact that the original requirement was underspecified — take on the mindset of a test manager reading these requirements for the first time, whose job is to notice exactly where "how would I check this passed?" doesn't yet have an answer.

## Workflow

1. Take each requirement or user story as its own unit — don't try to write one criterion that covers several requirements at once, or the traceability back to a single ID breaks down.
2. For each one, write the standard/happy-path scenario first, since it's usually the most direct translation of what the requirement literally says.
3. Then deliberately look for what else could happen: what if the precondition isn't met, what if the action fails partway, what if the input is invalid, what if permissions are missing, what if a dependency is unavailable. Write a scenario for each of these that's actually plausible for this requirement — don't pad the list with implausible edge cases just to hit a quota.
4. Where the requirement uses a vague quality term, replace it with something measurable before writing the scenario around it (see below) — writing "Then the page loads quickly" just moves the vagueness from the requirement into the criterion instead of resolving it.

## Gherkin syntax

Every scenario follows this structure:

- **Given** [preconditions / starting state]
- **When** [triggering event / user or system action]
- **Then** [expected system behavior / measurable outcome]

Keep each clause concrete and singular — a `Given` that bundles three unrelated preconditions, or a `Then` that checks three unrelated outcomes, is harder to read and harder to debug when only one part fails. Split into separate scenarios or additional `And` clauses instead.

## Covering more than the happy path

For every requirement, deliberately ask "what's the standard flow, and what are the ways this could go differently?" A single positive scenario tells you the feature works when everything goes right, which is necessary but rarely where real bugs are found — most defects live in the error paths, boundary conditions, and exceptions that a "just write one test" pass would skip. This mirrors the same principle used when extracting requirements in the first place (see `requirements-engineering`): an unstated "else" branch is a gap, and here that gap becomes a missing test scenario instead of a missing sentence.

That said, use judgment about how many negative scenarios a given requirement actually warrants — a simple display-only requirement may only need one or two, while a payment or permissions-related requirement usually needs several. The goal is coverage of genuinely distinct failure modes, not an arbitrary scenario count.

## Precision over vague quality terms

Words like "fast," "user-friendly," "reliable," or "intuitive" can't be checked by a test — a tester reading "Then the system responds quickly" has no way to know whether 3 seconds counts as quick. Two ways to handle this:

- If the source material (or surrounding context) implies a concrete number, use it (e.g. a nearby requirement mentions a 2-second SLA — reuse that number here).
- If nothing in the context implies a number, propose a realistic placeholder value and mark it clearly as a suggested default that the team should confirm (e.g. "Then the search results are displayed within 2 seconds *(suggested threshold — confirm with the team)*"). Don't just quietly invent a number and present it as if it came from the requirement — the whole point of this rule is to make the missing precision visible, not to paper over it with an invented-but-uncited figure.

## Traceability

Every scenario must reference the ID of the requirement or user story it was derived from. If the input material has no IDs at all, assign sequential temporary ones as you go (`REQ-01`, `REQ-02`, ...) so the output still has something concrete to reference — don't leave scenarios floating with no way to trace them back to their source.

## Tone

Write in a factual, formal, test-oriented register. This output is meant to be handed directly to QA or used to write automated tests, so avoid hedging language, marketing-style adjectives, or explanatory asides inside the scenarios themselves — save any caveats or assumptions for a note outside the Given/When/Then structure.

## A note worth passing on to the user

Even well-constructed Gherkin scenarios only reflect what's inferable from the input material — they don't carry the fuller project context a human team has (unstated business priorities, regulatory constraints, prior incidents that shaped a requirement). Treat the scenarios you produce as a strong first draft and thinking aid for the team, not a final, unreviewable definition of done. It's worth saying this explicitly to the user when handing over the output, especially for anything safety-, payment-, or compliance-adjacent, so the criteria get the human sign-off (product owner, tester, developer) they need before being treated as authoritative.

## Output format

Generate the result as a single Markdown document. For each requirement:

```
**Reference ID:** [requirement ID]
**Requirement / User Story:** [brief restatement of the original text]

**Scenario 1: [title of the standard/happy-path flow]**
* **Given** [precondition]
* **When** [event]
* **Then** [expected outcome]

**Scenario 2: [title of an error/exception/alternate flow]**
* **Given** [precondition]
* **When** [event]
* **Then** [expected outcome]
```

Add as many additional scenarios per requirement as there are genuinely distinct flows worth testing (see "Covering more than the happy path" above).

## If the user also wants a domain model or process diagrams

This skill focuses specifically on acceptance criteria. If the user's requirements also need an information model, process flow diagrams, or a full functional specification, the `functional-specification` skill covers that ground (and includes its own Gherkin step) — mention it as a next step if the user's ask goes beyond acceptance criteria alone.
