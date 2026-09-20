# Verified Workflow Harness Constitution

<!-- "Verified Workflow Harness" is a working title. Rename freely; nothing below depends on the name. -->

## Core Principles

### I. Plain Text Is the Interface
Contracts, reports, ledgers, and configuration MUST be human-readable text (Markdown, JSON, JSONL, or equivalent). No binary or proprietary formats. No required database server. Any person with a text editor MUST be able to open and inspect any artifact the system produces or consumes.

*Rationale:* Observability comes for free when every intermediate artifact is a readable file. It also lets non-developers change workflow behavior by editing text, which is a core adoption advantage over framework-based orchestration.

### II. Local-First and Private by Default
Core functionality MUST run on the user's machine with no network access. Data leaves the machine only when the user explicitly configures an external provider (for example, a judge model). A workspace is a folder: copyable, emailable, and versionable in Git, with no server to deploy.

*Rationale:* Privacy is an enterprise selling point, and portability is what makes handing a workflow to a client as simple as copying a folder.

### III. Deterministic Before Probabilistic
Deterministic checks MUST run before judged (LLM-based) checks. A judged result MUST NEVER override a deterministic failure. A check that was skipped, errored, or never defined MUST NEVER be reported as a pass.

*Rationale:* A verification layer that can silently pass unverified work is worse than none. Trust depends on the difference between "checked and passed" and "not checked."

### IV. Model-Agnostic
No feature may depend on a specific model vendor. Model providers sit behind a neutral interface, and every result that involves a model MUST record the identity and version of that model.

*Rationale:* Portability, no vendor lock-in, and reproducibility. The reference research was tested on one model family only, so cross-model behavior is an open question that the product must not assume away.

### V. Human in Command
Every stage output is an editable file. The system MUST read whatever is on disk, including human edits, and MUST NEVER silently overwrite or alter them. Verification is read-only with respect to workspace artifacts. Review gates MUST support proceeding, re-running a prior stage, or abandoning the run. When automation is uncertain, the default MUST be to escalate to a human.

*Rationale:* Human-in-the-loop review at stage boundaries is the core interaction model. Automation exists to focus human attention, not to remove it.

### VI. Append-Only Evidence
Run records (the ledger) MUST be append-only and tamper-evident through content fingerprints. They capture the files each stage loaded, the model identity, the outputs produced, and every human edit. Corrections are new entries, never rewrites.

*Rationale:* Audit trails and edit mining both depend on an honest history. Editing history destroys the value of both.

### VII. Calibrated Uncertainty
Any confidence signal used to gate human review MUST declare its method, MUST be evaluated for calibration on benchmark data before it is relied on, and MUST default to conservative behavior (escalate to a human) when confidence is unavailable or unvalidated. Start with a simple signal (for example, disagreement across repeated samples) and measure it before adding sophistication.

*Rationale:* Uncertainty quantification is the product's intended differentiator. An uncalibrated confidence score that looks authoritative is a liability, not a feature.

### VIII. Specs Are Authority, Enforced by Tests
Development is spec-anchored. Every acceptance scenario in a spec MUST become an automated test that runs in CI, and drift between spec and code MUST fail the build. When validation fails, either fix the code or revise the spec; the spec remains the authority either way. Specs describe what and why, not how.

*Rationale:* The gap this product exists to close is enforcement. Spec-driven tooling commonly stops at convention, so this project applies to itself the discipline it will sell.

### IX. Minimum Rigor
Use the least specification and process that removes ambiguity. Actively guard against:
- **Over-specification:** if a spec reads like code, it has gone too far.
- **Specification rot:** specs and code drifting apart; automated enforcement is the remedy.
- **Specification as bureaucracy:** forms filled in without improving understanding.
- **Tooling complexity:** drowning in generated plans, task lists, and checklists; start simple.
- **False confidence:** a passing test proves the code matches the spec, not that the spec is right. Specs get the same careful review as code.

*Rationale:* These are the predictable failure modes of spec-driven development. Naming them keeps the process proportionate.

### X. Measured Claims Only
No claim that scoped context, verification gates, or uncertainty-based gating improves output quality may appear in documentation or marketing unless it is backed by a reproducible benchmark that ships with the repository. Unmeasured claims MUST be labeled as hypotheses.

*Rationale:* The research this product builds on rests largely on self-reported practitioner experience and has no controlled comparison against monolithic prompting. Producing that evidence is part of the product's value and its credibility.

### XI. The Contract Format Is a Product
The stage-contract format (a `CONTEXT.md` with Inputs, Process, Outputs, and Verify sections) is designed on its own terms, independent of any other tool's directory conventions. It MUST be versioned. Changes MUST be backward compatible or ship with a migration path. Inputs MUST distinguish reference material (stable across runs) from working artifacts (per-run). Importers for other tools (for example, Spec Kit) are adapters, never dependencies.

*Rationale:* The contract format is the core intellectual property and the potential open standard. Coupling it to another project's layout would forfeit control of it.

### XII. Evidence, Not Legal Compliance
The system produces inspectable evidence: audit trails, review records, and verification results. It MUST NOT claim that using it makes anyone compliant with any regulation (for example, the EU AI Act). Compliance is a legal determination outside the tool's scope.

*Rationale:* Honest positioning. Staged review, audit trails, and defined intervention points are structurally useful for oversight requirements, but structural alignment is not legal compliance.

## Additional Constraints

**Scope discipline.** The MVP consists of: the contract parser and Verify runner, the run ledger, and a first-cut confidence gate. Edit mining is deferred to v0.2 because it needs several runs of history before patterns are detectable.

**Technology.** Technology choices are made in the plan phase, not in specs. Default direction, overridable with a justification recorded in the plan: Python 3; typed schemas for contract validation (pydantic-style); a command-line interface; a JSONL or SQLite ledger with content hashes; pytest for automated tests.

**Security.**
- Paths in contracts MUST resolve inside the workspace root; anything else is rejected.
- Script-based checks run with a bounded time limit.
- Checks make no network calls except through an explicitly configured judge provider.
- Secrets MUST NEVER be written to reports or the ledger.

**Portability.** Support Windows, macOS, and Linux. Reports use workspace-relative paths only, so they are portable across machines.

**Licensing intent.** The contract format and CLI are intended to be open source (MIT). Team and cloud features are separate modules and MUST NOT be required for local use.

## Development Workflow

- **Lean Spec Kit pass:** constitution, spec, clarify, plan, tasks, implement. Skip heavy checklists unless a specific risk justifies them.
- **One spec per capability**, built in this order: (1) contract parser and Verify runner, (2) run ledger, (3) confidence-gated review, (4) edit mining (v0.2), (5) workspace builder and templates.
- **Clarify before plan.** Resolve every `[NEEDS CLARIFICATION]` marker before the plan phase begins.
- **Small, validated increments.** Each task delivers a testable piece of functionality, with human review at each phase boundary.
- **Acceptance scenarios become pytest tests** in CI from the first commit.
- **Constitution check.** Every plan states how it satisfies each principle above and justifies any deviation.
- **Benchmark before belief.** The confidence gate ships with a benchmark comparing monolithic prompting, staged execution, and staged execution with verification, measured by defect catch rate.
- **Dogfooding.** Once the runner works, express this project's own build workflow as stage contracts and verify it with the tool.

## Governance

This constitution supersedes other practices for this project. Amendments require a written rationale, a version bump, and a review of dependent specs and plans. Principles are numbered for reference in reviews; a plan or pull request that violates a principle MUST cite the principle and justify the exception.

**Version**: 1.0.0 | **Ratified**: 2026-09-19 | **Last Amended**: 2026-09-19
