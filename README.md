# Carbon Tracing Review Skill

Review atom-mapping workbooks from
[Atom-Mapped Carbon Tracing](https://github.com/LGJee/atom-mapped-carbon-tracing),
complete missing molecular structures when justified, and deliver reviewed copies
with evidence and a detailed report, then run the project's step-3 carbon analysis
by default. This repository replaces the former
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
Review these files, save the reviewed copies and report, and run step 3 for these models.
```

Uploading the files and invoking the skill is sufficient when the compatible
project, environments and matching analysis plan can be located. Provide the
project path or plan if requested. Step 3 uses that plan's carbon sources and
the installed calculation settings; it does not run unrelated models.

To stop after review instead, send:

```text
$carbon-tracing-review-skill
Review these files only. Do not run step 3.
```

If propagation cannot run, completed review outputs are still delivered with the
actual blocker. The agent can communicate in your language; files and reports
are in English.

## Workflow

1. Check input workbooks, project interfaces and file hashes.
2. Inspect missing SMILES; research exact identities and record evidence.
3. Apply the project's Rules/preparation checks and remap affected ready reactions.
   Both full reactions and allowed known-structure subreactions are supported.
   All affected reactions require scientific review, regardless of tool confidence.
4. Re-extract the review queue and use enzymatic-atom-mapping for every flagged,
   blank or missing mapping and all newly affected reactions. Empty or failed
   mapper output does not prevent an evidence-based mapping attempt.
5. Attempt completion of the original reaction, including omitted carbon-bearing
   groups, small products and repeated molecules. Validate complete replacements.
   A supported legacy fragment is not a completed repair; report remaining gaps
   and blockers instead of fabricating atoms or generic-pool structures.
6. Save a separate reviewed workbook, verify it and write detailed outcome reports.
7. Save accepted discoveries and evidence for later validated reuse.
8. Run step 3 for the supplied models, verify current outputs and link them in the
   report, unless review-only was requested.

Original inputs and unrelated records are preserved. Structure completion may
update structures and affected mapper/selection/final rows in a staging copy.
Subsequent correspondence review changes only Final_Mapping relative to that copy.
Keep the original equations, signed fluxes and SourceInputSHA256 throughout.

## Outputs and provenance

- `<model>_2_atom_mapping_AIGenerated.xlsx`: a separate reviewed copy; existing
  deliveries are not silently overwritten.
- `AI_mapping_review_<timestamp>.md`: only actual AI analyses and accepted mapping
  changes, including retained and reviewed-but-unresolved cases.
- Matching `.json`: exact queue counts, machine checks and unfinished ReactionIDs
  for continuation. An incomplete review returns exit code 2; writing a copy or
  screening every input does not mean scientific review is complete.
- `<model>_3_carbon_flow_AIGenerated.xlsx`: step-3 output for reviewed mappings;
  available original mappings may also produce `<model>_3_carbon_flow.xlsx`.
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
These flags do not certify correctness or complete coverage. Before retaining a
legacy partial map, review must attempt whole-reaction completion and explain
any blocker. Legacy partial retention requires support in the installed project;
it does not count as a completed repair or set AIGenerated.
Boundary/NoCarbon rows do not require mapping solely because their SMILES is blank.
No targets means a reported no-op, not an artificial AI copy.
The conversation briefly reports added/corrected SMILES and applied mapping
replacements, with a report link and an incomplete-review reminder if necessary.
Unprocessed rows are not presented as AI findings; no accuracy comparison is
implied by the number of changes.

## Interfaces and tests

[SKILL.md](SKILL.md) defines the agent workflow.
[Workbook handoff](references/workbook-handoff.md) documents inspect/apply commands,
proposal/review JSON and output validation.
[Whole-reaction completion](references/completion-review.md) defines the carbon
accounting audit and completion obligations, including omitted CO2 products.
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

Carbon analysis runs by default after review, through the selected project's
step3_carbon_flow.py and matching analysis plan. An explicit review-only request
skips it. The workflow skill requests review-only and then runs step 3 once itself.
Inspect installed method settings and naming; this skill does not implement a
propagation engine or upgrade/publish the computation project. The helper scripts
alone do not perform AI research or launch step 3; the skill coordinates these
stages.
