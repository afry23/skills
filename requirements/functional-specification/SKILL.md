---
name: functional-specification
description: Turns customer-facing requirements (a requirements table, a raw requirements doc, or even loose stakeholder text with REQ-IDs) into a detailed functional specification — a PlantUML class diagram modeling the domain's entities and relationships, a PlantUML activity diagram modeling the process flow and business rules, a table of Gherkin (Given/When/Then) acceptance criteria traced back to source requirement IDs, and a gap analysis of missing business rules or undefined edge cases. Use this skill whenever the user has requirements and wants them turned into a "fachliche Spezifikation," "functional spec," domain model, process diagram, or acceptance criteria — especially when they ask for traceability back to requirement IDs, or want to hand a spec to developers/testers that's more concrete than the raw requirements text.
---

# Functional Specification (SOPHIST method)

## Why this exists

A requirements list tells you *what* the system must do, one sentence at a time. It doesn't tell you how those sentences fit together — which entities exist, how they relate, what order things happen in, or which business rule fires under which condition. That gap is exactly where implementations diverge from what stakeholders actually meant: two developers read the same requirement and build two different data models, or one QA engineer tests a happy path the requirement never actually specified.

A functional specification closes that gap by building an explicit domain model and process model *on top of* the requirements, and by turning each requirement into a concrete, testable scenario. Take on the mindset of an experienced business analyst: someone whose job is to spot the structure implied by a requirements text, make it explicit as diagrams, and flag exactly where that structure is still underspecified.

## Input expectations

This skill works best on requirements that already carry stable IDs (e.g. `REQ-001`) — the output of a prior requirements-extraction pass, or a numbered requirements table the user already has. If the input is raw, unstructured stakeholder text without IDs, assign sequential temporary IDs (`REQ-001`, `REQ-002`, ...) as you go, exactly as you would when extracting requirements from scratch, so every downstream artifact still has something concrete to trace back to.

## Workflow

1. Read all the requirements first, end to end, before modeling anything — the structure of a domain model only becomes visible once you've seen every entity and relationship mentioned across the whole document, not just the first few lines.
2. Build the information model: pull out every domain-specific noun (not generic words like "system" or "data") and the relationships between them.
3. Build the process model: pull out the chronological steps, decision points, and business rules that govern how those entities move through a workflow.
4. Derive acceptance criteria: for each functional requirement, write the concrete scenario(s) that would prove it's satisfied.
5. Run the gap analysis last, once you've tried to model everything — gaps are easiest to spot after you've attempted to build a complete picture and hit a place where the source text simply doesn't say what happens.

## Information modeling (class diagram)

Scan the requirements for domain nouns — the "things" the business talks about (e.g. `Order`, `Customer`, `Invoice`), as opposed to technical/generic nouns (`system`, `data`, `component`). For each one:

- Model it as a PlantUML class.
- Give it the attributes the source text actually mentions or clearly implies (don't invent attributes the requirements never reference — an empty or sparse class is more honest than a padded one).
- Model relationships between classes as associations, aggregations, or generalizations, using the cardinality/multiplicity the text supports (e.g. "an order contains multiple line items" → one-to-many).

If the source text doesn't give you enough to infer a relationship's cardinality confidently, model the relationship but flag the missing cardinality in the gap analysis rather than guessing.

## Process modeling (activity diagram)

Scan the requirements for anything chronological or conditional — sequences of steps, and business rules that branch behavior ("if X, then Y; otherwise Z"). Model this as a PlantUML activity diagram:

- Use a start node and end node(s).
- Model each meaningful step as an action.
- Model every business rule/condition as a decision diamond, with both branches shown. If the source only describes the "then" branch and not the "otherwise" branch (a common gap — see the `requirements-engineering` skill for more on this), still draw the decision diamond with the undefined branch labeled clearly (e.g. "? undefined") and log it in the gap analysis — don't silently invent what the missing branch does.

If the requirements describe more than one distinct process (e.g. "checkout" and "returns" are separate flows), produce a separate activity diagram for each rather than forcing them into one diagram — a diagram that tries to show two unrelated processes at once is harder to read than two focused ones.

## Acceptance criteria (Gherkin)

For each functional requirement, write one or more Given/When/Then scenarios that would concretely prove the requirement is met. A good scenario is specific enough that a tester could execute it without asking clarifying questions — vague scenarios ("Given a user, When they do something, Then it works") are not acceptance criteria, they're restated requirements with different words.

Where a requirement implies multiple distinct scenarios (a happy path and at least one edge case, e.g. success vs. failure), write both rather than only the happy path — the failure/edge-case scenario is usually the one that's actually useful to a tester, since the happy path is rarely where bugs hide.

Always cite the source requirement ID next to each scenario, so the chain from stakeholder need → requirement → concrete test scenario stays traceable in both directions.

## Gap analysis

While modeling, you'll naturally hit places where the requirements don't say enough to model confidently: an undefined "else" branch, a relationship whose cardinality isn't stated, a business rule mentioned once but never fully specified, an exception case nobody addressed. Collect these as you go rather than trying to reconstruct them at the end from memory. Each gap should name the specific requirement(s) it relates to and explain concretely what's missing — "REQ-004 doesn't specify what happens when the discount would bring the price below zero" is useful; "some things are unclear" is not.

## Tone and grounding

Write in a formal, domain-focused, precise register — this document typically gets handed to developers and testers who need to trust it as a working reference, not a rough draft. Stay strictly grounded in what the source requirements state or clearly imply; where you're inferring something not explicitly stated (like the acting component behind a passive-voice requirement), say so briefly rather than presenting an inference as a stated fact. It's better to under-model and flag the gap than to invent a plausible-sounding business rule the source never mentioned.

## Output format

Produce the functional specification with exactly these four sections, in this order.

**1. Information Model (PlantUML)**

A single fenced ` ```plantuml ` code block containing valid PlantUML class-diagram syntax (`@startuml` ... `@enduml`), with domain entities as classes, their attributes, and their relationships.

**2. Process Flows (PlantUML)**

One or more fenced ` ```plantuml ` code blocks, each valid PlantUML activity-diagram syntax (`@startuml` ... `@enduml`), one per distinct process identified in the requirements. Label each diagram with the process name it models.

**3. Acceptance Criteria**

A table:

| Column | Content |
|---|---|
| `Source REQ-ID` | The requirement ID this scenario validates |
| `Scenario` | Short descriptive title of the scenario |
| `Gherkin Criterion` | `Given [starting state] When [action/rule] Then [expected system behavior]` |

**4. Gaps and Open Business Rules**

A bullet list of the specific gaps found while modeling, each one naming the requirement(s) it relates to and stating precisely what's undefined (e.g. "For business rule in REQ-004, it is not defined what happens if condition Y does not hold").
