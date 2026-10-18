# Carbon Tracing Review Skill

Review atom-mapping workbooks from
[Atom-Mapped Carbon Tracing](https://github.com/LGJee/atom-mapped-carbon-tracing),
complete missing molecular structures when justified, and deliver reviewed copies
with evidence and a detailed report. This repository replaces the former
carbon-tracing-ai-review-skill name.

## Install

Ask Codex:

```text
$skill-installer Install https://github.com/LGJee/carbon-tracing-review-skill as carbon-tracing-review-skill, with its required enzymatic-atom-mapping skill.
```

The scientific dependency is
[Enzymatic Atom Mapping](https://github.com/LGJee/enzymatic-atom-mapping-skill).
Keep existing compatible dependencies. Installing a skill does not install the
carbon-tracing project or its calculation environments.

## Use

Attach one or more `<model>_2_atom_mapping.xlsx` workbooks and send:

```text
$carbon-tracing-review-skill
Review these files and save the reviewed copies and report to my project's results directory.
```

Provide the project path if it cannot be located. The agent can communicate in
your language; generated files and reports are in English.

## Workflow

1. Check input workbooks, project interfaces and file hashes.
2. Inspect missing SMILES; research exact identities and record evidence.
3. Apply the project's Rules/preparation checks and remap affected ready reactions.
   Both full reactions and allowed known-structure subreactions are supported.
   All affected reactions require scientific review, regardless of tool confidence.
4. Re-extract the review queue and use enzymatic-atom-mapping for every flagged,
   blank or missing mapping and all newly affected reactions. Empty or failed
   mapper output does not prevent an evidence-based mapping attempt.
5. Validate replacement proposals and decisions. New AI replacements must cover
   the complete supplied reaction; reviewed existing partial maps can be retained
   as Partial. Unresolved atoms and generic pools are not fabricated.
6. Save a separate reviewed workbook, verify it and write detailed outcome reports.
7. Save accepted discoveries and evidence for later validated reuse.

Original inputs and unrelated records are preserved. Structure completion may
update structures and affected mapper/selection/final rows in a staging copy.
Subsequent correspondence review changes only Final_Mapping relative to that copy.
Keep the original equations, signed fluxes and SourceInputSHA256 throughout.

## Outputs and provenance

- `<model>_2_atom_mapping_AIGenerated.xlsx`: a separate reviewed copy; existing
  deliveries are not silently overwritten.
- `AI_mapping_review_<timestamp>.md` and `.json`: all target outcomes, including
  retained, unresolved, unassessed and unsuccessful replacements.
- `data/ai_smiles.csv`: Species, MetaboliteName, SMILES, Source.
- `data/ai_atom_mappings.csv`: Species, ReactionID, MappedReactionSMILES, Source.
- `data/ai_structure_reviews/` and `data/ai_mapping_reviews/`: detailed immutable
  evidence archives. Sources include evidence URLs and archive references.

Manual CSVs retain priority and are never updated implicitly by AI discovery.
AI mapping indexes are reviewable records, not automatic step-2 overrides.
Reuse archived candidates only after checking the exact input context and rerunning
validation. Unknown chemical identities remain unresolved.

In Final_Mapping, AIReviewed means scientific analysis occurred. AIGenerated means
an accepted replacement was applied. ReviewRequired is preserved by default.
These flags do not certify correctness. Scientific review can retain an existing
partial mapping without generating a complete replacement or setting AIGenerated.
Boundary/NoCarbon rows do not require mapping solely because their SMILES is blank.
No targets means a reported no-op, not an artificial AI copy.

## Interfaces and tests

[SKILL.md](SKILL.md) defines the agent workflow.
[Workbook handoff](references/workbook-handoff.md) documents inspect/apply commands,
proposal/review JSON and output validation.
[Structure completion](references/structure-completion.md) documents identity
proposals, affected-reaction remapping and staging provenance.

The correspondence helper requires openpyxl, RDKit and the scientific validator;
integrated checks use the selected project's tracing environment. Structure
completion/remapping additionally requires a compatible project and its mapper
environments. The helpers consume supplied evidence-backed proposals; running
Python alone does not perform literature research or AI prediction.

Run tests with compatible installed project/dependency paths:

```bash
"$TRACING_PYTHON" scripts/test_review_workbooks.py --project "$PROJECT_DIR" --mapping-skill "$MAPPING_SKILL_DIR"
"$TRACING_PYTHON" scripts/test_complete_structures.py "$PROJECT_DIR" "$MAPPING_SKILL_DIR"
```

Tests use isolated synthetic fixtures and mocked mapper outputs. They check
bookkeeping, preservation and handoffs, not biochemical prediction accuracy.
No user workbooks, cache data or credentials are distributed.

Carbon propagation is optional and runs only when requested, through the selected
project's step3_carbon_flow.py and analysis plan. Inspect its installed method
settings and naming; this skill does not implement a propagation engine or
upgrade/publish the computation project.
