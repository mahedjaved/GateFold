# 03 - MVP Plan and Spec Kit Playbook

## 1. Decision: use Spec Kit as scaffolding, not as the foundation

Use Spec Kit for the *process* of building the MVP. Do not let it define the *product*.

### Why it fits

- **Free and portable.** MIT-licensed and works with 30+ coding agents, so there is no lock-in. It carries an experimental label, so expect rough edges.
- **You are building what it lacks.** Spec Kit has no enforcement or verification, so it cannot stop spec drift during the build. This product exists to fix that gap.
- **The pain it targets is real.** A vague prompt like "build a CI runner with UQ gating" leaves the coding agent guessing at dozens of decisions.

### Where to be careful

- **Do not let `.specify/` define the product format.** The contract format (`CONTEXT.md` plus Verify) is the core IP and potential standard. Design it on its own terms (constitution principle XI).
- **Watch the ceremony.** Tooling complexity and over-specification are named pitfalls. For an MVP, run a lean pass: constitution, spec, clarify, plan, tasks, implement. Skip heavy checklists.
- **Add enforcement yourself.** Turn each spec's acceptance scenarios into pytest tests in CI, so drift fails the build. This is spec-anchored discipline, which Spec Kit leaves to you.

## 2. Setup and command sequence

> Spec Kit is experimental and changing. The SDD paper lists the phase commands as `/specify`, `/plan`, `/tasks`; current documentation uses a `speckit.*` command namespace and a `specify init` bootstrap. **Check the repository README (https://github.com/github/spec-kit) for the current install command and command names before starting.**

1. Bootstrap a project with `specify init`, choosing your coding agent.
2. Copy this package's files over the scaffold, preserving paths:
   - `.specify/memory/constitution.md`
   - `specs/001-contract-parser-verify-runner/spec.md`
   - `docs/`
3. Run the clarify step on spec 001 to resolve its three `[NEEDS CLARIFICATION]` markers (see section 4).
4. Run the plan step with the plan-phase input in section 5.
5. Run tasks, then implement task by task, reviewing at each phase boundary.
6. Wire the acceptance scenarios into CI as tests from the first commit.

Tip: tell the coding agent to read `docs/` before planning. The docs are split by topic so it can load only what it needs, which is the same scoped-context principle the product is built on.

## 3. MVP scope and spec sequence

The MVP is: **runner, Verify, ledger, and a first-cut gate.**

| Spec | Capability | Notes |
|---|---|---|
| **001** | Contract parser plus Verify runner | Drafted in this package. |
| **002** | Run ledger | Files loaded, model identity, outputs, human edits; append-only with content fingerprints; staleness detection (if an input changed, the stage's output may be stale). |
| **003** | Confidence-gated review | Start simple: disagreement across repeated samples. Measure calibration before investing in anything fancier. Default to escalating to a human when confidence is unavailable. |
| **004** | Edit mining (**v0.2**) | Needs several runs of history before patterns are detectable. Suggests contract or reference changes when the same kind of edit recurs. |
| **005** | Workspace builder and templates | Vertical template packs. |

## 4. Open questions in spec 001

1. **Verify syntax:** Markdown table (consistent with the ICM Inputs table) or a fenced structured block? Tables are friendlier to non-developers; structured blocks handle richer check parameters.
2. **ICM compatibility:** accept only the strict Inputs/Process/Outputs layout, or also tolerate different section names from existing ICM workspaces?
3. **Judged-check severity:** recommended default is *warning*, with blocking as an explicit opt-in, because judged results are probabilistic.

## 5. Plan-phase input (paste into the plan step)

These are technology decisions, kept out of the spec on purpose.

- Python 3, typed schemas for contract validation (pydantic-style), a CLI entry point, pytest for tests.
- Ledger (spec 002): JSONL or SQLite with content hashes; no server.
- Judge providers behind a neutral interface; no vendor SDK in the core.
- Cross-platform paths (Windows, macOS, Linux); workspace-relative paths in all reports.
- Script checks run in a subprocess with a time limit; workspace is treated read-only during verification.
- Justify any departure from these defaults in the plan's constitution check.

## 6. Benchmark plan (constitution principle X)

Neither paper contains a controlled comparison, so producing one is both a credibility requirement and a marketing asset.

- **Conditions:** (a) monolithic prompt, (b) staged execution, (c) staged execution with Verify gates.
- **Metric:** defect catch rate, meaning the fraction of injected or naturally occurring defects that are caught before final output.
- **For spec 003:** evaluate the confidence signal's calibration on held-out data before relying on it, and report the method alongside the result.
- **Ship it with the repository** so anyone can reproduce it.

## 7. Suggested path

1. **Pick one regulated vertical** where you have real domain credibility.
2. **Build a 6-8 week MVP:** CLI runner, contract format with Verify, run ledger, and one vertical template pack.
3. **Run the benchmark** in section 6.
4. **Recruit 5-10 design partners** before building any cloud tier.

## 8. Two later moves

- **Dogfood.** Once the runner works, express this project's own build workflow as stage contracts and verify it with the tool. It makes a strong demo.
- **Spec Kit importer.** Spec Kit has a huge installed base, so a verification layer that reads its specs could be the adoption on-ramp. It is an adapter, never a dependency (principle XI).
