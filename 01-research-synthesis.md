# 01 - Research Synthesis

Two papers underpin this project. This document records what each says, how they connect, and where their evidence is weak. It is background context for the constitution, specs, and plans. It is not itself a spec.

- **Paper 1:** *Spec-Driven Development: From Code to Contract in the Age of AI Coding Assistants*, Deepak Babu Piskala, arXiv:2602.00180 (30 Jan 2026). Referred to below as **SDD**.
- **Paper 2:** *Interpretable Context Methodology: Folder Structure as Agent Architecture*, Jake Van Clief and David McDermott, arXiv:2603.16021v2 (18 Mar 2026). Referred to below as **ICM**.

---

## Paper 1: Spec-Driven Development (Piskala)

**Core claim.** AI coding assistants are good at pattern completion but poor at guessing intent. So the specification should be the source of truth and code should derive from it.

### Three levels of rigor

| Level | Meaning | Notes |
|---|---|---|
| **Spec-first** | Write the spec before coding; the spec may then be abandoned. | Low maintenance; good for prototypes and one-off features; does not protect against drift. |
| **Spec-anchored** | Spec and code are maintained together, with tests enforcing alignment. | The paper calls this the sweet spot for most production systems. BDD (Cucumber) and OpenAPI plus contract testing (Specmatic) are examples. |
| **Spec-as-source** | Humans edit only the spec; code is fully generated and never hand-edited. | Standard in domains with mature generators (Simulink to certified C in automotive). Tessl aims to extend it to general software. Requires high trust in generation. |

### Workflow: Specify, Plan, Implement, Validate

1. **Specify:** what the software should do (behavior, requirements, acceptance criteria in Given/When/Then), without implementation detail.
2. **Plan:** how to build it. Architecture, data models, interfaces, technology choices, non-functional constraints. The plan declares constraints on implementation.
3. **Implement:** break the plan into discrete, reviewable tasks and work in small validated increments; human oversight remains.
4. **Validate:** confirm the code meets the spec. If it does not, either fix the code or revise the spec; the spec remains the authority.

Human review sits at each checkpoint. Good specs are behavior-focused, testable, unambiguous, and complete enough without over-specifying.

### Main argument

SDD is not new. It is BDD and TDD with better tooling, CI enforcement, and AI as a consumer of specs. The real difference from traditional HLD/LLD/SRS documents is that SDD specs are **enforced** (builds fail when code diverges) rather than **advisory** (read once, then drift).

### Practical advice

- Use the minimum rigor that removes ambiguity ("golden rule"): spec-first for AI-assisted initial development, spec-anchored for long-lived production systems, spec-as-source only when generation tooling is mature and trusted.
- Write specs at the level of detail needed to remove ambiguity; if there is only one reasonable interpretation, do not over-specify.
- Known pitfalls: over-specification, specification rot, specification as bureaucracy, tooling complexity, and false confidence (a passing test only proves the code matches the spec, not that the spec is right).
- Techniques for LLM non-determinism include property-based testing.
- The paper also describes emerging "self-spec" methods, where an LLM drafts a spec that humans refine before implementation.

### Tools surveyed

BDD frameworks (Cucumber, SpecFlow/Reqnroll, Behave); TDD frameworks; API specification (OpenAPI, GraphQL SDL, Protocol Buffers, AsyncAPI); contract testing (Pact, Specmatic); AI-assisted SDD (GitHub Spec Kit, Amazon Kiro, Tessl); model-based design (Simulink, SCADE).

---

## Paper 2: Interpretable Context Methodology (Van Clief and McDermott)

**Core claim.** For sequential, human-reviewed workflows, a multi-agent framework is unnecessary. Folder structure can do the orchestration.

### Structure

- Numbered folders are **stages**. Plain Markdown files (`CONTEXT.md`) are **stage contracts** with Inputs, Process, and Outputs. One agent reads the right files at the right moment. Local scripts handle non-AI work.
- Stages communicate through plain files (Markdown and JSON). Any human with a text editor can inspect or modify any artifact.

### Five context layers

| Layer | Purpose | Approx. size |
|---|---|---|
| 0: `CLAUDE.md` | "Where am I?" Global identity | ~800 tokens |
| 1: workspace `CONTEXT.md` | "Where do I go?" Task routing | ~300 tokens |
| 2: stage `CONTEXT.md` | "What do I do?" Stage contract | 200-500 tokens |
| 3: reference material | "What rules apply?" Stable across runs; "the factory" | 500-2k tokens |
| 4: working artifacts | "What am I working with?" Per-run; "the product" | varies |

Layer 3 is internalized as constraints. Layer 4 is processed as input. Each stage loads only what it needs, roughly 2-8k tokens, versus 40k+ for a monolithic prompt. The paper frames this as prevention rather than compression, motivated by "lost in the middle" findings.

### Five design principles

1. One stage, one job.
2. Plain text as the interface.
3. Layered context loading.
4. Every output is an edit surface.
5. Configure the factory, not the product.

### Human control

Every stage output is an editable file, so review gates sit at each stage boundary. The result is observability by default with no logging layer. The paper links this to mixed-initiative systems, interpretable-by-design arguments, and human-oversight regimes such as the EU AI Act. It explicitly declines to claim legal compliance.

### Stage contract shape (from the paper)

```
## Inputs
- Layer 4 (working):   ../01_research/output/
- Layer 3 (reference): ../../_config/voice.md
- Layer 3 (reference): references/structure.md

## Process
Write a script based on the research output. Follow structure.md. Match the tone in voice.md.

## Outputs
- script_draft.md -> output/
```

### Honest limits (the paper's own)

- Poor fit for real-time multi-agent collaboration, high-concurrency systems, and automated branching on AI decisions.
- Error recovery is manual re-run; branching is a human decision between stages; execution is sequential by design.

### Evidence quality (the paper's own admissions)

- An invite-only community of 52 practitioners; 33 reported on edit patterns. 30 of those 33 described a **U-shaped intervention pattern**: heavy editing at stage 1 (direction-setting), light in the middle, heavy again at the final stage (aligning output with earlier decisions, closer to debugging).
- Data is self-reported through conversation, not instrumented.
- Single model family tested (Claude Opus 4.6 and Sonnet 4.6).
- No controlled comparison against monolithic prompting; the quality benefit of scoped context rests on prior literature and practitioner judgment.
- Most use is content production; academic and policy deployments are early-stage.

### Future work the paper proposes (this project's opportunity)

- **Incremental compilation:** if a reference file changes, only stages that load it need re-running; a stage's declared inputs signal staleness.
- **Semantic debugging:** output provenance via identifiers (like source maps), cross-stage trace verification (an audit file checking stage n against stage n-2), and "breakpoints" in Markdown.
- **A `Verify` section** in stage contracts, listing which earlier outputs to check and against what criteria. Proposed, not implemented.
- **Edit-source principle:** editing outputs fixes one run; editing sources fixes every future run. Track repeated output edits and suggest contract or reference changes.

---

## How they connect

Both papers are about **moving intent out of prompts and code into durable, human-readable artifacts, with human checkpoints between phases.**

| | SDD | ICM |
|---|---|---|
| Authoritative artifact | Spec | `CONTEXT.md` contracts plus reference files |
| Staging | Specify, Plan, Implement, Validate | Numbered stage folders |
| Human review | At each phase | At each stage boundary |
| Anti-drift mechanism | Tests and CI enforce spec-code alignment | Mostly manual; a `Verify` section is only proposed |
| Fix at the source | "Spec remains the authority" | Edit-source principle (section 6.3) |

ICM's stage contracts work like specs for an agent, and its Layer 3 reference material resembles SDD's plan-phase constraints. The gap is **enforcement**. In SDD terms, ICM is spec-first or lightly anchored: contracts guide the agent, but nothing fails automatically when output diverges. SDD's contract tests and validation gates are what ICM's future-work section is reaching toward.

## Caveats

- **SDD:** quantitative claims are weakly sourced. The "up to 50% error reduction" comes from studies the paper itself calls nascent, and the 75% integration cycle-time reduction is an unattributed case study. The spec-as-source vision (Tessl) is unproven outside domains like Simulink.
- **ICM:** the 42k-token monolithic comparison is illustrative, not measured, and the central claim that scoped context improves output quality is unmeasured. The paper acknowledges this in its threats-to-validity section.

## What this project takes from each

| From SDD | From ICM |
|---|---|
| Enforced specs; drift fails the build | Folder-as-pipeline; plain-text contracts |
| Spec-anchored rigor with tests in CI | Reference (Layer 3) versus working (Layer 4) separation |
| Minimum-rigor discipline and named pitfalls | Every output is an editable file; review gates |
| Validate phase: fix code or revise spec | Edit-source principle; incremental staleness |
| | Cross-stage trace verification; provenance |
