---
name: requirements-engineering
description: Use whenever the user hands over messy real-world text — Slack/chat pastes, email threads, meeting or kickoff notes, dictated feature lists, stakeholder interviews, rough spec drafts, or German Lastenheft/Pflichtenheft/Konfluenz excerpts — and wants it turned into structured requirements engineering deliverables: an atomic requirements table (MASTER-template style), a terminology glossary, and an open-issues/contradictions log, following the SOPHIST methodology. Trigger on requests like "turn this into a requirements doc," "clean up this spec," "extract requirements and definitions," "find the contradictions/open questions," "make this official," or "list what I still need to ask him" — with or without naming requirements, glossary, SOPHIST, or MASTER. Also trigger on German equivalents: Anforderungen, Glossar, offene Punkte, Konfliktliste, Lastenheft/Pflichtenheft strukturieren. Treat these as signals even when unstated: passive voice, vague nouns ("the system," "handle it"), conflicting numbers or definitions across sections, undefined jargon, unnamed actors. Skip for: plain summarization, proofreading, translation, writing tests/acceptance criteria (e.g. Gherkin), or drafting a brand-new PRD from scratch with no source text to process.
---

# Requirements Engineering (SOPHIST method)

## Why this exists

Stakeholders write requirements the way they talk: passive voice, vague nouns ("the system," "the data"), universal quantifiers ("always," "all users"), unstated exceptions, requirements tangled together with commentary. That ambiguity is exactly what later causes rework, disputes about scope, and bugs from unhandled edge cases. The SOPHIST method exists to squeeze that ambiguity out at the source — by forcing every requirement into one unambiguous sentence shape, and by explicitly cataloguing everything that's still unclear rather than silently guessing.

Take on the mindset of an experienced requirements engineer: skeptical of vague language, precise about actors and objects, and unwilling to paper over contradictions in the source material.

## Workflow

1. Read the entire source text first. Don't process it linearly — skim for the domain, the actors involved, and any terms that repeat, before extracting anything.
2. Pull out three separate streams as you go: atomic requirements, domain terminology, and open issues (contradictions, gaps, ambiguities). A single sentence in the source can feed more than one stream (e.g., a sentence can yield a requirement *and* reveal a contradiction with an earlier statement).
3. Write the three output files as described below.

## Extracting requirements

**Separate signal from noise.** Source text mixes real requirements with rationale, background story, and opinions. Only pull sentences that describe a function or property the system must have. A justification like "because customers keep complaining" is context for a requirement, not a requirement itself — don't manufacture a REQ row out of it, but you may cite it in the source reference.

**One requirement, one sentence.** If a source sentence bundles two obligations ("the system must validate the input and log the result"), split it into two atomic requirements. This keeps each requirement independently testable — a reviewer or QA engineer should be able to point to one condition and say pass/fail, not have to untangle a compound sentence first.

**Resolve ambiguity instead of transcribing it:**
- *Passive voice hides the actor.* "The order will be validated" doesn't say by whom. Name the actor or system component explicitly — infer it from context if the text implies it (e.g., "the payment service"), and flag it as an open issue if it truly cannot be determined.
- *Universal quantifiers overreach.* Words like "all," "every," "always," "never" are rarely literally true and hide the real boundary. Replace them with the precise set they refer to, or flag the imprecision as an open issue if the source doesn't specify it.
- *Vague nouns underspecify.* "The data" or "the system" could mean five different things in a multi-component architecture. Name the concrete data entity or component. If the source genuinely doesn't disambiguate, note that in the open-issues file rather than inventing specificity that isn't there.
- *Conditions need an else.* If the text says "if X, then Y" without saying what happens otherwise, that's a real gap — most systems need defined behavior for the negative case too. Don't invent the missing behavior yourself; log it as an open issue (missing information) referencing the requirement it affects.

**Sentence shape (MASTER template).** Every functional requirement must follow this structure so that modality, actor, and action are always in the same place and never left implicit:

```
[Condition, e.g. "Once/If..."] MUST/SHOULD/WILL/CAN [system or component] provide [whom] THE ABILITY TO / BE CAPABLE OF [object] [action verb, infinitive].
```

Example: *"Once a customer submits an order, the checkout service MUST provide the payment gateway THE ABILITY TO validate the payment method before confirming the order."*

Quality requirements and constraints don't always fit the same clause order naturally — keep the same discipline (explicit actor, explicit modality, no vague nouns) even where the exact template doesn't apply verbatim.

## Extracting the glossary

A glossary is only useful if every reader trusts it — that means no invented definitions and no guessing at meaning. Apply these constraints:

- **Stay factual, not creative.** Define terms strictly from what the source text (plus, where a term is a standard software-engineering concept, established terminology) actually supports. If the source uses a term without ever explaining it and it isn't a well-known standard term, don't fabricate a definition — flag it as an open issue ("term used but not defined") instead of guessing.
- **Anchor to a source.** Every definition should be traceable to where it came from — either a quote/paraphrase from the input text, or a note that it follows standard usage (e.g., ISO/IEC/IEEE 24765 vocabulary for software and systems engineering terms, when the source itself doesn't define a standard term). This is what lets a reader trust the glossary instead of wondering if it was hallucinated.
- **Tone: formal, technical, precise.** A glossary sets the shared vocabulary for the whole project — write definitions the way a standard would, not the way a blog post would. Avoid hedging language ("might mean," "could refer to") in the definition itself; if there's genuine uncertainty, that belongs in the open-issues file, not smuggled into the glossary as hedge words.
- **Capture synonyms and usage, not just the definition.** If the source uses two different terms for the same concept (or the same term inconsistently), record that — it's exactly the kind of inconsistency that causes miscommunication later. A short example sentence showing the term used in context also makes the glossary far more useful than a bare definition.
- **Scope the definition to its context.** Many terms are overloaded — "client" means something different in a networking section than in a sales/CRM section, "session" differs between an auth context and a UX-research context. State explicitly which context/domain the given definition applies in (e.g., the module, process step, or business area it was used in). If the same term appears in the source with a different meaning in a different context, give it its own row rather than merging the two into one blurred definition — that's precisely the kind of ambiguity a glossary exists to resolve, not hide.

## Extracting open issues

Look for three distinct kinds of problems, and don't force one into another:

- **Contradiction** — the text says two things that can't both be true (e.g., a limit stated as "5" in one place and "10" in another).
- **Ambiguity** — the text is technically or logically unclear, even without an outright contradiction (e.g., a term that could plausibly mean two different things given the context).
- **Missing information** — something a complete requirement needs is simply absent (e.g., no error-handling behavior specified, an actor left unnamed, a condition without its "else" branch).

Link each open issue back to the requirement ID(s) it affects wherever that's determinable — an open issue floating with no connection to a requirement is much less actionable for whoever resolves it later.

## Output format

Produce exactly three Markdown files. Output each one in its own fenced Markdown code block, with the intended filename as a heading directly above the block.

**File 1: `requirements.md`**

| Column | Content |
|---|---|
| `ID` | Sequential, unique: `REQ-001`, `REQ-002`, ... |
| `Requirement` | The cleaned-up requirement, formulated per the MASTER template |
| `Category` | `Functional`, `Quality`, or `Constraint` |
| `Source` | Short quote or pointer to where in the input text this came from |

**File 2: `glossary.md`**

| Column | Content |
|---|---|
| `ID` | Sequential, unique: `GLO-001`, ... |
| `Term` | The identified domain term or acronym |
| `Definition` | Clear, unambiguous, formally worded explanation |
| `Context` | The domain/module/process area in which the term carries this specific meaning — important because the same term may mean something else elsewhere (add a separate row for that other meaning rather than blending them) |
| `Synonyms` | Other terms used for the same concept in the source, if any (else "—") |
| `Example` | A short sentence showing the term used in context |
| `Source` | Exactly where this definition came from: a quote/pointer to the input text passage, or "Standard usage (ISO/IEC/IEEE 24765)" if the source itself didn't define it |

**File 3: `open_issues.md`**

| Column | Content |
|---|---|
| `ID` | Sequential, unique: `OP-001`, ... |
| `Type` | `Contradiction`, `Ambiguity`, or `Missing Information` |
| `Description` | What's unclear or contradictory, and why |
| `Affected Requirement IDs` | Related `REQ-xxx` IDs, if determinable, else "—" |

If a stream genuinely has nothing to report (e.g., the text is clean enough that no open issues exist), still emit the file with just its header row — an empty file left out entirely looks like the step was skipped rather than deliberately empty.
