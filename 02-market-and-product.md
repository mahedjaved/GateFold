# 02 - Market and Product

Research date: 2026-09-19. Market facts come from web sources listed at the end. Treat them as a snapshot; this space moves quickly.

## 1. Short answer

Yes, the two papers can be combined into a sellable product, but **not as another spec-driven coding tool**. That lane is crowded. The opportunity is where the two papers share a weakness: contracts that guide but do not fail the build.

## 2. Market landscape

- **Coding SDD is saturated.** GitHub's Spec Kit passed roughly 115,000 GitHub stars by June 2026, less than a year after launch. Competitors include Amazon Kiro, OpenSpec, BMAD-METHOD, Task Master AI, Tessl (venture-backed), Augment's Intent, and others.
- **Rough taxonomy** from one 2026 guide:
  - *Bucket A:* spec-anchored / spec-as-source (Tessl aspirationally, OpenSpec, Augment Intent, CodeMySpec).
  - *Bucket B:* spec-first scaffolding bolted onto a coding session (Spec Kit, Kiro, and many clones), the most crowded bucket. The spec is treated as a launch document and drifts once code generation starts.
  - *Bucket C:* agentic-agile orchestration with multi-role agent teams (BMAD-METHOD).
- **Enforcement is the gap.** One June 2026 comparison rates Spec Kit as having no enforcement or verification and Kiro as review-gated but not enforced. This is the weakness SDD says separates real spec-driven development from old design documents.
- **Kiro specifics** (same comparison): specs live in `.kiro/`, models route through Bedrock, and pricing is metered credits with a markup, which drew community backlash.
- **ICM's side of the map:** ICM targets sequential, human-reviewed, repeatable **non-code** workflows, a much less crowded space. It has no enforcement either: recovery is a manual re-run, and its `Verify` section is only proposed.

**Source bias warning.** Several comparison articles are published by vendors with competing products (for example, one sells a "mandatory BDD gate"). The enforcement ratings above are plausible but should be independently verified before being used in positioning.

**Conclusion:** both halves have the same hole. Nobody clearly owns "enforced contracts for AI workflows, with an audit trail and calibrated human escalation."

## 3. Product concept: CI for AI workflows

Combine ICM's folder-as-pipeline with SDD's executable contracts.

1. **Stage contracts with a Verify block.** Inputs, Process, Outputs, Verify. Checks can be deterministic scripts, schema and rule checks, cross-stage trace audits (ICM's stage n versus stage n-2 audit), or LLM-judge checks.
2. **Run ledger.** Records the files each stage loaded, the model version, the outputs, and every human edit. This is the audit trail.
3. **Edit mining.** When a human keeps making the same edit, the tool proposes a contract or reference-file change (ICM section 6.3). Neither paper ships this, and it makes workflows improve with use.
4. **Confidence-gated review.** Instead of a human checking every gate, calibrated uncertainty escalates only low-confidence outputs. Uncertainty quantification is the intended moat: it beats both "review everything" and "trust everything."
5. **Local-first, model-agnostic, git-native.** Data privacy is an enterprise selling point.
6. **Workspace builder plus vertical template packs.**

## 4. Who pays

- **Consultancies and agencies** that hand clients recurring deliverable pipelines. ICM's "copy the folder" handoff fits them well.
- **Regulated teams** in finance, medtech, pharma, legal, and policy, where traceability matters.

### EU AI Act timing (do not anchor the pitch to August 2026)

The Council approved the Digital Omnibus on AI on 29 June 2026 (Parliament voted 16 June 2026). The deferrals:

| Category | Old deadline | New deadline |
|---|---|---|
| Annex III standalone high-risk systems | 2 Aug 2026 | 2 Dec 2027 |
| Annex I high-risk AI embedded in products | 2 Aug 2027 | 2 Aug 2028 |
| Art. 50 watermarking (systems already on the market) | 2 Aug 2026 | 2 Dec 2026 |

The tailwind for audit trails and human-oversight evidence is real but slower than many assumed. Per the constitution (principle XII), the product provides evidence and never claims legal compliance.

## 5. Business model

**Open-core**, which is how Spec Kit spread.

- **Open source (MIT intended):** the contract format and CLI, for adoption.
- **Paid:**
  - shared workflow registries
  - SSO and audit exports
  - hosted runners
  - verified vertical template packs
  - support

## 6. Honest risks

- **Platform risk:** GitHub, AWS, or Anthropic can absorb generic features.
- **Thin evidence:** ICM's claims are self-reported. A benchmark showing that verification gates catch real errors is required. See `03-mvp-plan-and-speckit.md`.
- **Scale questions:** folder-based orchestration does not handle concurrency or automated branching, and enterprise buyers will ask.
- **Horizontal is hard to sell:** a general "workflow harness" pitch is difficult, so go vertical.
- **Open-source competitors are free.** Differentiation must come from enforcement, the ledger, edit mining, and calibrated gating.

## 7. Sources consulted

- ssojet.com, "7 Spec-Driven Development Tools: Spec Kit, Kiro, OpenSpec, Tessl & More" (26 Jun 2026): https://ssojet.com/blog/best-spec-driven-development-tools
- CodeMySpec, "Best Spec-Driven Development Tools (2026)": https://codemyspec.com/blog/best-spec-driven-development-tools
- CodeMySpec, "Spec-Driven Development in 2026: The Complete Guide and Tool Comparison": https://codemyspec.com/blog/spec-driven-development
- Augment Code, "6 Best Spec-Driven Development Tools for AI Coding in 2026": https://augmentcode.com/tools/best-spec-driven-development-tools
- Digital Omnibus / EU AI Act deferral: https://secureprivacy.ai/blog/eu-ai-act-digital-omnibus-the-new-high-risk-ai-deadlines-after-council-approval and https://pasqualepillitteri.it/en/news/5190/digital-omnibus-ai-act-eu-parliament
