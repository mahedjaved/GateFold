# Feature Specification: Contract Parser and Verify Runner

**Feature Branch**: `001-contract-parser-verify-runner`
**Created**: 2026-09-19
**Status**: Draft
**Input**: Build the foundation of a "CI for AI workflows" harness. A workspace is a folder of numbered stages. Each stage has a plain-text contract (`CONTEXT.md`) declaring Inputs, Process, Outputs, and a new Verify block. The runner parses contracts, validates them, and checks each stage's outputs against its Verify checks, failing loudly when they do not hold.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Add Verify checks to a stage and see pass/fail (Priority: P1)

A workflow author opens a stage's contract in a text editor, adds a few checks describing what a good output must satisfy, and runs verification. They immediately see which checks passed and which failed, with evidence.

**Why this priority**: This is the core value of the product. Without it there is no verification layer, only guidance.

**Independent Test**: Create a workspace with one stage, one output file, and a Verify block containing two deterministic checks. Run verification and confirm a per-check result and a stage verdict are produced.

**Acceptance Scenarios**:

1. **Given** a stage whose output satisfies every check, **When** verification runs, **Then** every check is reported as passed, the stage verdict is "pass", and the run reports overall success.
2. **Given** a stage whose output violates a blocking check, **When** verification runs, **Then** that check is reported as failed with evidence (what was expected, what was found, and where), the stage verdict is "fail", and the run reports overall failure.
3. **Given** a stage whose only failing checks are warning-severity, **When** verification runs, **Then** the stage verdict is "pass with warnings", the warnings are listed, and the run reports overall success.
4. **Given** a stage with no Verify block, **When** verification runs, **Then** the stage is reported as "unverified" with a visible notice, and it is never reported as "pass".

---

### User Story 2 - Get actionable errors when a contract is malformed (Priority: P1)

An author makes a mistake in a contract (a missing section, an input that does not exist, a duplicate check name). The tool tells them exactly what is wrong and where, before any verification happens.

**Why this priority**: Non-developers edit these files. Unclear errors would destroy the "anyone with a text editor can change the workflow" advantage.

**Independent Test**: Run validation against a set of deliberately broken contracts and confirm each problem is reported with file, location, and a plain-language explanation.

**Acceptance Scenarios**:

1. **Given** a contract missing its Outputs section, **When** validation runs, **Then** the error names the contract file, the missing section, and what the section must contain.
2. **Given** an Inputs entry that is not classified as reference material or a working artifact, **When** validation runs, **Then** the entry is rejected with an explanation of the two classes.
3. **Given** an Inputs entry that points to a file that does not exist, **When** validation runs, **Then** the error names the missing path and is reported as a contract error, not as a verification failure.
4. **Given** two checks in one stage with the same identifier, **When** validation runs, **Then** the duplicate is reported.
5. **Given** a contract with several problems, **When** validation runs, **Then** all problems are reported in one run rather than stopping at the first.

---

### User Story 3 - Block a merge in CI when verification fails (Priority: P1)

A team wires verification into their CI pipeline. A failing blocking check stops the pipeline, and the report is available in machine-readable form.

**Why this priority**: Enforcement is the gap this product exists to close. Spec-driven tooling that only advises is what already exists.

**Independent Test**: Run verification against a workspace with a known blocking failure and confirm a non-success exit status and a structured report.

**Acceptance Scenarios**:

1. **Given** any blocking failure, **When** verification runs, **Then** the exit status is non-success and distinct from the status used for invalid contracts.
2. **Given** all checks pass, **When** verification runs, **Then** the exit status is success.
3. **Given** a machine-readable report is requested, **When** verification finishes, **Then** a plain-text structured report is produced containing every check result, every stage verdict, and the overall verdict.
4. **Given** any verification run, **When** it finishes, **Then** a human-readable summary is shown.
5. **Given** any verification run, **When** it finishes, **Then** every file in the workspace (contracts, references, outputs) is byte-for-byte unchanged.

---

### User Story 4 - Check consistency across stages (Priority: P2)

An author declares that a later stage's output must stay consistent with an earlier stage's output (for example, an animation spec must reference every phrase in the script). The runner checks the two files together.

**Why this priority**: Misalignment between distant stages is the most common downstream error, and practitioners currently catch it only by manual tracing.

**Independent Test**: Create a two-stage workspace where the later output omits an item that the earlier output contains, declare a trace check, and confirm the omission is reported with locations.

**Acceptance Scenarios**:

1. **Given** a trace check naming an earlier stage's output and a criterion, **When** verification runs, **Then** the runner evaluates both files and reports which items matched and which did not, with locations in each file.
2. **Given** a human has edited the earlier stage's output, **When** verification runs, **Then** the check uses the edited file as it exists on disk.
3. **Given** a trace check names a stage that is not earlier in the sequence or a file that does not exist, **When** validation runs, **Then** it is reported as a contract error.

---

### User Story 5 - Optional judged checks (Priority: P2)

For qualities that rules cannot capture (tone, completeness, faithfulness), an author declares a judged check evaluated by a language model. It is optional and never allowed to hide a deterministic failure.

**Why this priority**: Many valuable quality criteria are not rule-checkable, but they must not compromise the trustworthiness of the deterministic layer.

**Independent Test**: Declare a judged check, run once with a judge configured and once without, and confirm the two results are reported honestly and differently.

**Acceptance Scenarios**:

1. **Given** a judge is configured, **When** a judged check runs, **Then** the result includes pass or fail, a short rationale, and the identity and version of the judge used.
2. **Given** no judge is configured, **When** a judged check would run, **Then** it is reported as "skipped" with a reason, and if the check was blocking the stage verdict is "incomplete", never "pass".
3. **Given** a blocking deterministic check failed in the same stage, **When** verification runs, **Then** judged checks are not run and are reported as skipped because of the earlier failure.
4. **Given** the judge becomes unavailable or returns an error mid-run, **When** the check is evaluated, **Then** it is reported as "error", never "pass".

---

### User Story 6 - Verify a whole workspace or one stage (Priority: P3)

An author verifies every stage in order, or just one stage they are working on, and sees an aggregated overall verdict.

**Why this priority**: Convenience and speed for day-to-day authoring; the core behavior is already covered by the earlier stories.

**Independent Test**: Verify a five-stage workspace, then verify one named stage, and confirm ordering and aggregation.

**Acceptance Scenarios**:

1. **Given** a multi-stage workspace, **When** the whole workspace is verified, **Then** stages are processed and reported in numeric order and an overall verdict is aggregated.
2. **Given** a stage name is provided, **When** verification runs, **Then** only that stage is verified and the report says which stages were not included.

---

### Edge Cases

- A stage's output folder is empty, or an output file is zero bytes.
- An output file is binary or unreadable as text: the check reports an error with the reason, not a pass.
- A script-based check exceeds its time limit, exits non-zero, or cannot be launched.
- A script-based check attempts to modify workspace files.
- A contract contains sections or fields the tool does not recognize (must not break older tools reading newer contracts).
- Two stage folders share the same number, or the numbering has gaps.
- An Inputs path is a directory, a symlink, or resolves outside the workspace root.
- A contract file uses an unusual text encoding or line-ending style.
- Paths differ between Windows and Unix styles.
- An output file is very large.
- A check's identifier changes between runs (identifiers must be stable so later features can track them).

## Requirements *(mandatory)*

### Functional Requirements

**Workspace and contract handling**

- **FR-001**: The system MUST discover a workspace's stages as numbered folders and process them in numeric order.
- **FR-002**: The system MUST parse each stage's contract into four sections: Inputs, Process, Outputs, and Verify.
- **FR-003**: Each Inputs entry MUST be classified as either reference material (stable across runs) or a working artifact (per-run), and MUST identify a path and optionally the relevant part of the file.
- **FR-004**: The system MUST validate contracts and report all problems in a single run, each with the file, location, a plain-language description, and a suggested fix.
- **FR-005**: The system MUST distinguish contract errors (the contract is invalid) from verification failures (the contract is valid but the output does not satisfy it).
- **FR-006**: The contract format MUST declare a format version, and unknown sections or fields MUST be preserved and reported as warnings rather than errors.
- **FR-007**: Paths that resolve outside the workspace root MUST be rejected.

**Checks**

- **FR-008**: The Verify block MUST support four check kinds: rule (deterministic content, format, or structure rules), script (a local deterministic script), trace (consistency with an earlier stage's output), and judged (evaluated by a language model).
- **FR-009**: Every check MUST have an identifier unique within its stage, a kind, a target, a criterion, and a severity of blocking or warning.
- **FR-010**: Within a stage, deterministic checks MUST run before judged checks. If a blocking deterministic check fails, judged checks MUST be skipped and reported as skipped with that reason.
- **FR-011**: Each check MUST end in exactly one of four statuses: pass, fail, error, or skipped.
- **FR-012**: A skipped, errored, or undefined check MUST NEVER be reported as a pass. A stage with a skipped or errored blocking check MUST have the verdict "incomplete".
- **FR-013**: Every failure and error MUST include evidence: what was expected, what was found, and where.
- **FR-014**: Deterministic checks MUST be reproducible: identical files and contract MUST yield identical results.
- **FR-015**: Script checks MUST run within a bounded time limit, configurable with a sensible default. A non-zero exit is a fail; failure to launch or a timeout is an error. Captured output MUST be included as evidence.
- **FR-016**: Trace checks MUST reference an earlier stage's output and MUST read that file as it exists on disk, including human edits.
- **FR-017**: Judged checks MUST use a provider-neutral interface and MUST record the identity and version of the judge and a short rationale. If no judge is configured, judged checks MUST be reported as skipped.

**Verdicts and reporting**

- **FR-018**: Each stage MUST receive exactly one verdict: pass, pass with warnings, fail, incomplete, unverified, or contract invalid. A stage with no Verify block MUST be "unverified".
- **FR-019**: The system MUST produce a human-readable summary and, on request, a machine-readable plain-text report. The report MUST include every check result, every stage verdict, and the overall verdict.
- **FR-020**: The report MUST identify each verified file by workspace-relative path and content fingerprint, so a later run can tell whether anything changed.
- **FR-021**: The report MUST expose results with stable check and stage identifiers so that later features (run ledger, confidence gate) can consume them without re-running verification.
- **FR-022**: The command MUST use distinct, documented exit statuses for: overall success, verification failure or incomplete, invalid contract, and usage or internal error.

**Safety and scope of action**

- **FR-023**: Verification MUST NOT create, modify, or delete any contract, reference, or output file. The only file it may write is a report at a user-specified location outside the verified artifacts.
- **FR-024**: The system MUST support verifying the whole workspace or a single named stage, and MUST state which stages were not included.

**Open decisions**

- **FR-025**: The Verify block MUST be authorable by a non-developer using only a text editor, using [NEEDS CLARIFICATION: which syntax: a Markdown table (consistent with the Inputs table) or a fenced structured block?].
- **FR-026**: Existing workspaces built with the reference methodology's conventions MUST be accepted [NEEDS CLARIFICATION: strictly, or should the tool also accept contracts whose section names differ from Inputs / Process / Outputs?].
- **FR-027**: A failing judged check is treated as [NEEDS CLARIFICATION: warning by default with blocking opt-in, or blocking by default? Recommended: warning by default, because judged results are probabilistic].

### Key Entities

- **Workspace**: A folder containing numbered stages, shared configuration, and identity and routing files. It is the unit that is copied, versioned, and verified.
- **Stage**: One numbered step of a workflow, with a contract, reference material, and an output folder.
- **Contract**: The plain-text file that declares a stage's Inputs, Process, Outputs, and Verify block, plus a format version.
- **Input Reference**: A pointer from a contract to a file, classified as reference material or a working artifact, optionally scoped to part of the file.
- **Check**: One declared condition on a stage's output, with identifier, kind, target, criterion, and severity.
- **Check Result**: The outcome of one check: status, evidence, and (for judged checks) judge identity and rationale.
- **Stage Verdict**: The single summarized outcome of a stage.
- **Verification Report**: The complete record of a run: results, verdicts, overall verdict, and content fingerprints of what was verified.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In a usability test with five non-developers, at least four add a working rule check to an existing contract using only a text editor in under 10 minutes.
- **SC-002**: 100% of contract errors in the test suite name the file and the location of the problem.
- **SC-003**: 100% of blocking failures in the test suite produce a non-success exit status.
- **SC-004**: Zero test scenarios exist in which a skipped, errored, or undefined blocking check yields a "pass" verdict.
- **SC-005**: Repeating a deterministic verification 100 times on identical files yields identical results every time.
- **SC-006**: After any verification run, content fingerprints of all workspace files are identical to those taken before the run.
- **SC-007**: A five-stage workspace with 20 deterministic checks completes verification in under five seconds on a typical laptop.
- **SC-008**: Every acceptance scenario in this spec has a corresponding automated test that runs in CI.

## Assumptions

- This feature verifies stage outputs. It does not execute stages or invoke an AI agent; producing outputs is outside its scope.
- Stage outputs are text files (Markdown, JSON, or similar).
- The workspace layout follows the reference methodology: numbered stage folders, a `CONTEXT.md` per stage, `references/` and `output/` folders per stage, and a shared `_config/` folder.
- Verification runs locally for a single user. No network access is required except when the user configures a judge.
- Judge providers are configured by the user. Provider selection is not part of this spec.

## Out of Scope (handled by later specs)

- Run ledger persistence, history, and staleness detection (spec 002)
- Confidence-gated review and uncertainty estimation (spec 003)
- Edit mining and contract-change suggestions (spec 004, v0.2)
- Workspace builder and vertical templates (spec 005)
- Importing Spec Kit artifacts
- Hosted or team features
