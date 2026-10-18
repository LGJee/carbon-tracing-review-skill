# Workbook handoff

## Commands

Use the compatible tracing interpreter and the absolute helper path. These
examples use shell variables supplied by the agent, not hard-coded machine paths:

```bash
"$TRACING_PYTHON" "$SKILL_DIR/scripts/review_workbooks.py" inspect \
  /path/MyCulture_2_atom_mapping.xlsx /path/OtherCulture_2_atom_mapping.xlsx \
  --project "$PROJECT_DIR" > review_targets.json

"$TRACING_PYTHON" "$SKILL_DIR/scripts/review_workbooks.py" apply proposals.json \
  --project "$PROJECT_DIR" --output-dir "$CASE_OUTPUT_DIR" > review_report.json
```

`CASE_OUTPUT_DIR` is the selected analysis plan's resolved output directory,
for example `$PROJECT_DIR/results/results_by_AI/my_analysis`. Supply it for
custom cases; the fallback may use the publication plan.

The helper locates a project only in the explicit directory or input/workspace
ancestors. The scientific validator defaults to the sibling installed skill
`enzymatic-atom-mapping/scripts/validate_mapping.py`. Use `--mapping-skill` with
that skill's directory if installed elsewhere. Inspect needs openpyxl; apply
also needs RDKit and the scientific validator. A supplied project must expose
its existing reader/validation/save interfaces; incompatible APIs are reported.
No package installation, remote prediction or network lookup occurs in review_workbooks.py. For missing SMILES, first use the separate [structure-completion workflow](structure-completion.md), which can call the installed local mapper environments for affected reactions.

## Review targets and combined propagation

Review flagged rows and blank selected/final mappings even with FALSE flags;
explicit Boundary/NoCarbon coverage is exempt from blank-map targeting. A missing
ReactionSMILES remains a target with equation and metabolite context. Report the
reason if no supported replacement can be generated. Do not invent structures.

CarbonRules may save a reduced mapped subreaction. Its original coefficients and
position pairs are in Partial_Mapping; it is not a complete original-reaction map.
During correspondence review, only Final_Mapping may change relative to its staged input; a preceding explicit structure-completion stage has the separate contract below. New step 3 reads that copy for both
Enumeration and size-1 EMU-inspired source balances in a single result workbook.
Inspect the installed project: older projects may support Enumeration only.

## Proposal file

Copy paths, input hashes and reaction IDs from inspect's output. Do not copy the
input file hash into SourceInputSHA256: the former identifies the step-2 workbook;
the latter records the original step-1 input and must remain unchanged.

```json
{
  "workbooks": [
    {
      "input_file": "/path/MyCulture_2_atom_mapping.xlsx",
      "input_sha256": "<exact SHA256 from inspect>",
      "reviews": [
        {
          "ReactionID": "r456",
          "decision": "unresolved",
          "AIReviewed": true,
          "rationale": "<scientific analysis performed; why no replacement is justified>",
          "evidence": [],
          "limitations": ["<remaining uncertainty>"]
        }
      ],
      "replacements": [
        {
          "ReactionID": "r123",
          "MappedReactionSMILES": "<complete accepted mapped reaction>",
          "scope": "all",
          "evidence_status": "evidence-supported",
          "rationale": "<why the evidence supports these correspondences>",
          "limitations": [],
          "evidence": [
            {
              "url": "https://example.org/specific-entry",
              "accessed_on": "YYYY-MM-DD",
              "claim": "<specific correspondence or mechanism supported>"
            }
          ]
        }
      ]
    }
  ]
}
```

This is a schema example, not a reaction or evidence claim. Replace every
placeholder with the scientific skill's actual result. Accepted evidence statuses
are evidence-supported and partially-supported; the latter must be qualified in
the rationale/limitations. The all-atom validation still must pass. Proposals
with unresolved evidence, limited scope or failing checks remain unchanged.
Omit an unresolved reaction or supply it with evidence_status=unresolved; do not
invent a complete map merely to populate a cell.

The optional `reviews` list records non-replacement decisions for flagged targets.
Use decision `unchanged` or `unresolved`, a real Boolean AIReviewed, and a nonempty
scientific rationale; include retrieved evidence and limitations when available.
AIReviewed is TRUE only after reaction-level scientific analysis, even if the
outcome remains unresolved. For automatic input triage alone, use FALSE and
describe the missing structure or inventory problem. Do not put an accepted
replacement in both lists. Accepted replacements automatically receive TRUE.
Unknown or duplicate IDs and invalid flags are rejected. A validation-rejected
proposal may have an unresolved review, but cannot be declared unchanged.

Apply independently processes workbooks and reactions. Its JSON report records
accepted/rejected/missing proposals, validator warnings and whether project
validation ran. An exit code of 2 signals errors/partial failure; inspect the
report for successful outputs instead of discarding them. Accepted replacements
or explicit review decisions produce a separate output workbook. A review-only
copy retains AIGenerated=FALSE unless that provenance already existed. Neither
accepted replacements nor explicit reviews means no output workbook. Existing destination files are preserved. Output
defaults to the project's configured results directory when discoverable, else
the input directory; --output-dir overrides delivery location. The input hash is
checked again before commit. The saved copy is checked for unchanged sheet data
outside intended edits, including Mapping_Info and raw prediction scores.
Within correspondence review, only Final_Mapping may change. Mapping_Selection remains the staged tool
comparison/selection record, including SelectionReason and its selected SMILES.
The AI-reviewed Final_Mapping may therefore differ from Mapping_Selection;
step 3 Enumeration reads Final_Mapping. Keep scientific rationale and evidence in the report
and immutable archive instead of synchronizing them into the selection sheet.

ReviewRequired is preserved, so a future run may still target an AI-assisted
reaction. AIGenerated is provenance, not a request to skip needed review.
The helper inserts AIReviewed immediately right of ReviewRequired in Final_Mapping.
Other rows default FALSE; existing TRUE values are retained. A missing final row
is added only for an accepted complete map, not for review metadata alone.

## Results reports

`apply` automatically writes `AI_mapping_review_<timestamp>.md` and `.json` to
the configured results/output directory, including zero-replacement and partial
failure runs. Their paths are returned in `report_files`; a write failure is
returned as `report_error` with a nonzero exit status. When one invocation spans
several input directories, reports go to the explicit output directory or the
first input's resolved results directory. Do not overwrite previous reports.

The Markdown includes per-workbook and total counts, every target's outcome and
rationale, validation rejections, evidence URLs/access dates, limitations, input
hashes, and output/archive paths. The JSON preserves complete validation results
and supplied reviews. Supply all decisions, including automatic triage, rather
than only successful proposals. Any omitted target remains explicitly unassessed.

Counts are workbook reaction records, not deduplicated chemical transformations.
Replaced + retained + unresolved + unassessed must equal initial review targets
(TRUE flags plus missing mappings). The legacy report field flagged counts this union.
AI-reviewed overlaps those categories; it must not be added to them. Current-run
replacement counts and scientific-review counts are distinct from saved workbook
totals if there were pre-existing flags or absent final rows. Do not clear
ReviewRequired or equate AIReviewed/AIGenerated with experimental correctness.
State the checks actually run and whether optional propagation was requested/run;
the helper itself performs no production propagation. If grouping is reported,
state the grouping key and give unique-group counts separately from workbook rows.

## Persistent review records

`apply` automatically writes an immutable JSON record in
`<project>/data/ai_mapping_reviews/`. Its filename is the SHA256 of its serialized
content, so separate reviews do not overwrite each other. The record contains
the original reaction SMILES, equation and direction, each proposal and its
evidence, accepted/rejected status, validation reports, workbook SHA256 and output
path. It is an audit record, not a new authority for biochemical correctness.

`inspect` returns `cached_candidates` only for earlier acceptances whose reaction
ID, name, equation, direction and input SMILES variants match exactly. This
conservative text match may miss equivalent SMILES traversals. It cannot reuse a
reversed reaction, another protonation or stereoisomer, or different stoichiometry.
Candidates are never applied automatically. Recheck evidence applicability and
submit the proposal with the current workbook hash; `apply` reruns the validators.
Malformed archives are reported in `cache_issues`. A failed cache write appears as
`archive_error` and returns exit code 2; an already delivered Excel is retained.

Without a compatible project, records go in `data/ai_mapping_reviews/` below the
delivery directory. They are not auto-discovered there; deliver them to the actual
project's data directory if reuse is wanted. No project files or research records
are uploaded by these commands. Runtime cache files belong to local project data,
not the distributed skill.

## Optional carbon propagation

When the user requests it, place reviewed copies in the analysis plan's output
directory and run from the compatible project root:

```bash
python3 step3_carbon_flow.py
python3 step3_carbon_flow.py MyCulture --rounds 1
```

Enumeration reads available ordinary and `<model>_2_atom_mapping_AIGenerated.xlsx`
files for models in the plan and writes separate
`<model>_3_carbon_flow.xlsx` and
`<model>_3_carbon_flow_AIGenerated.xlsx`. It performs no AI review or
mapper prediction. A separate step4 and method selector are not required.
Either mapping version may exist alone. Model names, input hashes and ordered
sources must still match the
plan. Original step-2 and ordinary step-3 outputs remain separate. Inspect actual
installed interfaces before invoking; older versions may lack this entry.

## Local helper checks

```bash
"$TRACING_PYTHON" "$SKILL_DIR/scripts/test_review_workbooks.py" \
  --project "$PROJECT_DIR" --mapping-skill /path/to/enzymatic-atom-mapping -v
```

These tests create and remove temporary synthetic workbooks, check preservation
and validation failures, and exercise the current step-3 Enumeration code path. They do not
review real reactions or change project results.


## Saved carbon-rule assessments

Inspect includes `final` and `partial_mappings` for each target. On compatible
projects every original reaction is in Final_Mapping. Rule-derived Partial rows
may show a reduced unit mapped subreaction; their original amounts and exact
atom-index pairs are in Partial_Mapping. It is not a full original-reaction map. Missing
whole ReactionSMILES does not prevent scientific review of those pairs.

For a scientific rule review, supply a non-replacement review such as:

```json
{
  "ReactionID": "biomass",
  "decision": "unchanged",
  "AIReviewed": true,
  "rule_verdict": "accepted",
  "rationale": "Explain evidence supporting each stored carbon transfer.",
  "evidence": [],
  "limitations": ["Only the listed carbon skeletons are resolved."]
}
```

Use real evidence in actual reviews. `rule_verdict` is `accepted`, `rejected` or
`unresolved`. The latter two require `decision: "unresolved"`; the helper disables
the saved transfers by setting Final_Mapping.MappingCoverage=Unresolved and
clearing mapped SMILES. ReviewRequired stays TRUE and AIGenerated stays FALSE for
unchanged rule rows. A full replacement uses the existing replacements schema
and sets MappingCoverage=Complete. Do not submit both forms for one reaction.
Without scientific review, use AIReviewed=FALSE and omit rule_verdict. The skill
never edits Partial_Mapping, Mapping_Selection or raw predictions.


## Structure-completed input exception

Ordinary correspondence review preserves every sheet except Final_Mapping.
An explicit preceding structure-completion stage may change the copied metabolite
table and affected mapper/selection/final records, with its own evidence archive
and parent-workbook hash. Treat that staged workbook as the input to inspect/apply;
regenerate proposals and hashes rather than applying old original-input proposals.
See [structure-completion.md](structure-completion.md).


## Reduced mapper inputs

Mapping_Info may contain MappingInputEquation (active direction) and
OmittedComponentsJSON. Inspect returns omitted_components for each target.
Original Equation/Flux are unchanged. Review the whole biochemical reaction and
the omitted participants' roles; the reduced ReactionSMILES does not represent
complete coverage. Scientifically reviewed reduced Partial maps use rule_verdict
accepted/rejected/unresolved, as saved partial rules do. Unsupported reduced maps
are disabled in Final_Mapping only; original mapper and selection records remain.


Accepted AI replacements are indexed in data/ai_atom_mappings.csv with Species,
ReactionID, MappedReactionSMILES and Source (evidence URLs plus archive reference).
Only successfully applied proposals enter this table. Source and target are stored
in original equation direction; reverse-flux workbook mappings are reversed for
this index. It does not automatically override step-2 choices. Use immutable
exact-context archives for validated reuse; preserve separate manual data.
