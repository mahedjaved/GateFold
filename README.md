# Verified Workflow Harness (working title)

"CI for AI workflows": ICM-style folder pipelines plus SDD-style executable contracts, with a run ledger, edit mining, and uncertainty-based confidence-gated review.

## What's in this package

| Path | Purpose |
|---|---|
| `.specify/memory/constitution.md` | Project principles (12), constraints, workflow, governance. Drop into a Spec Kit project. |
| `specs/001-contract-parser-verify-runner/spec.md` | First feature spec: 6 user stories, 27 requirements, 8 success criteria, 3 open clarifications. |
| `docs/01-research-synthesis.md` | What both papers say, how they connect, their caveats. |
| `docs/02-market-and-product.md` | Market landscape, product concept, buyers, EU AI Act timing, business model, risks, sources. |
| `docs/03-mvp-plan-and-speckit.md` | Why and how to use Spec Kit, MVP scope, spec sequence, plan-phase input, benchmark, roadmap. |

## Order of operations

1. `specify init` a new Spec Kit project (check the Spec Kit README for current commands).
2. Copy these files over it, preserving paths.
3. Clarify spec 001 (three `[NEEDS CLARIFICATION]` markers).
4. Plan using the input in `docs/03-mvp-plan-and-speckit.md` section 5.
5. Tasks, then implement, with acceptance scenarios as CI tests from day one.
