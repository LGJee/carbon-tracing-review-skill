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
position pairs are in saved partial correspondence records; it is not a complete original-reaction map.
During correspondence review, only Final_Mapping may change relative to its staged input; a preceding explicit structure-completion stage has the separate contract below. New step 3 reads that copy for both
Enumeration and size-1 EMU-inspired source balances in a single result workbook.
Inspect the installed project: older projects may support Enumeration only.

## Whole-reaction completion before a verdict

Follow [the completion review](completion-review.md) for every incomplete target.
The original equation and coefficients define completeness, even when raw mapper
ReactionSMILES contain only a subreaction. Attempt missing carbon correspondences
before retaining a fragment; CO2 and other small products are not exempt.
Record the omitted groups, completion attempt, evidence, remaining gaps and
validation/interface blockers in the existing rationale and limitations fields.
Do not add a separate proposal schema or treat a retained partial as a repaired
reaction. Legacy partial verdicts below apply only when the installed project
supports them; never re-enable disabled partial propagation.

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
validation ran. An exit code of 2 signals errors or incomplete scientific review, including a
screening-only remainder; inspect successful outputs and remaining_review_ids
instead of discarding them. Zero exit status does not imply every map was repaired:
analyzed but unresolved reactions can complete review. Accepted replacements
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

`apply` writes one `AI_mapping_review_<model>_<timestamp>.md` and matching audit
JSON per input workbook, including zero-replacement and failed inputs. One model's
SMILES and mapping changes stay in the same readable report. Never combine models
into one report. Reports go to the explicit output directory or each input's own
resolved results directory; previous files are not overwritten.

`report_files` is always a list, including single-input runs. Each entry has
`input_file`, `markdown` and `json`, so consumers can associate paths with models.
A report write failure is recorded on that workbook as `report_error` with a
nonzero invocation exit status; other models' reports still get delivered.

The Markdown is a fuller version of the brief conversation, with only:

1. The summary table for this model only.
2. SMILES changes: model, metabolite ID/name, change type and new SMILES.
3. Applied mapping changes: model, ReactionID, reaction name and equation.
4. Delivered workbook and detailed JSON links.

Keep full mapped SMILES in the workbook and JSON. Do not expand unchanged,
unresolved or unprocessed reactions, per-reaction reasoning, hashes or validation
logs in Markdown. An incomplete run gets a short reminder. Preserve those detailed
analyses, evidence and limitations in JSON instead.

When the input is a structure-completed staging workbook, apply reads its sibling
`.structures.json` and verifies the staged workbook hash, immutable archive hash,
parent hash and structure count before listing additions. It does not scan the
whole AI cache or call old discoveries new. A missing sidecar yields no recorded
structure changes; an inconsistent sidecar yields a warning and nonzero status.
The existing completion helper fills missing structures only. A claim that an
existing SMILES was corrected requires a separate validated correction.

The JSON retains exact input hashes, outcomes and continuation metadata:

- `target_counts`: TRUE flags in each sheet, their ReactionID union,
  `missing_mapping_only` and the full queue size `targets`.
- `review_counts`: actual scientific review and replacement counts, with
  `remaining` separate from reviewed-but-unresolved decisions.
- `remaining_review_ids`: targets not scientifically analyzed in this run.
- `review_status`: `complete`, `incomplete` or `no_targets` per valid workbook.

The legacy `flagged` field counts the target union, not just TRUE flags; use
`target_counts.review_required_union` for the latter. Reconcile targets =
scientifically reviewed + remaining. Retained and unresolved counts include only
actual analysis; screening-only or rejected proposals without scientific review
remain pending. Existing AIReviewed flags and cache hits do not establish analysis
in the current run. Counts are workbook records; shared reactions across models
count separately. Input-processing failures have unknown counts, never zero
successful targets. A complete review can still contain scientifically unresolved
reactions. Keep continuation metadata when working in batches; do not stop after
an initial subset without a real blocker or explicit user limit.

The conversation must have a separate concise block per model/condition, giving
that model's initial target count, actual reviewed count, added/corrected SMILES,
applied mapping replacements and its own report link. Add that model's remaining
count only as a short incomplete-review reminder. Do not replace these blocks
with batch-wide totals, add accuracy/coverage comparisons, or enumerate unprocessed
rows as AI results.

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

## Default carbon propagation

The skill agent runs step 3 after completed scientific review and validated
delivery by default. An incomplete delivery does not trigger propagation unless
the user explicitly requests running with those disclosed limitations. The
inspect/apply helpers do not launch propagation. Skip this stage for an explicit
review-only request or an orchestrator's review-only handoff; report the handoff
so only the owner runs step 3.

1. Locate the compatible project and tracing environment. Find the case's actual
   analysis_plan.json from the supplied path or nearby case directory; use
   data/analysis_plan.json only when it matches the supplied models. Confirm model
   names, SourceInputSHA256, actual input hashes, ordered carbon sources and output
   location. Unknown source names or conflicting plans need clarification; never
   rewrite hashes to bypass validation.
2. Ensure validated reviewed workbooks are in the plan's resolved output folder,
   under <model>_2_atom_mapping_AIGenerated.xlsx. Do not silently replace existing
   deliveries. A delivered copy elsewhere is not discovered automatically: resolve
   that handoff through supported interfaces, or report the blocker. Preserve
   earlier completed results before rerunning outputs that step 3 will replace.
3. From the project root, select only the supplied models and the actual plan:

   ```bash
   python3 step3_carbon_flow.py MyCulture OtherCulture --plan path/to/analysis_plan.json
   ```

   Substitute verified names and the real plan path; paths inside the plan use
   the installed project's conventions. Group files by compatible plan when
   necessary. Reuse the tracing interpreter, installed method/round/resource
   settings and size guards. Do not call all models by omitting the names.
4. Check the exit status and this invocation's carbon_flow_run.json, input/mapping
   hashes, saved tables and files. A prior workbook or Summary collection is not
   proof that this run succeeded. Record actual mapping versions, selected
   methods, saved Enumeration rounds, stop reasons and errors in the English
   review report, with links to the outputs. Preserve the review report's original
   decisions and counts when adding this execution status.

Step 3 reads available ordinary and AI-reviewed mappings independently for those
models. It writes <model>_3_carbon_flow.xlsx and
<model>_3_carbon_flow_AIGenerated.xlsx as applicable; either version may exist
alone. Current versions combine Enumeration and size-1 EMU-inspired source
balances according to script settings. Inspect older interfaces before running;
do not invent a method flag or a separate step4.

No review targets means no fabricated AI copy: propagate the valid supplied
mapping with its actual provenance. A retained/unresolved review-only copy can
still be a legitimate input without newly TRUE AIGenerated flags. Unassessed
targets and disabled partial maps must remain explicit limitations in the report.
Missing compatible integration blocks only propagation; deliver completed review
outputs and explain what is needed. Do not claim downstream success.

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
atom-index pairs are in saved partial correspondence records. It is not a full original-reaction map. Missing
whole ReactionSMILES does not prevent scientific review of those pairs.

Only after a documented complete-mapping attempt remains blocked, and only if
legacy partial propagation is supported, a scientific non-replacement review may
use the following form. It records unresolved completion, not a completed repair:

```json
{
  "ReactionID": "biomass",
  "decision": "unchanged",
  "AIReviewed": true,
  "rule_verdict": "accepted",
  "rationale": "Record the whole-reaction completion attempt, its blocker and evidence supporting the retained pairs.",
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
never edits saved partial correspondence records, Mapping_Selection or raw predictions.


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

Partial context may be stored in an older Partial_Mapping sheet or in hidden
Mapping_Info rows with RecordType=PartialMapping. Inspect the installed helper's
supported layouts; report a compatibility gap instead of asserting coverage.
Do not recreate partial outputs in a project that has disabled that feature.

### Editing or clearing AI cache tables

The two AI CSV files are the active, user-editable cache lists. Keep their four-column
headers; the first three columns match the corresponding manual table. Rows are
sorted and saved as UTF-8 with BOM for Excel. Delete data rows to disable those old
AI candidates, or remove a CSV to disable that AI cache. Archived evidence alone
must not restore deleted or edited entries. New accepted research may add a new
record later. Clearing a cache does not alter existing workbooks, manual tables,
model annotations or the independent database structure_cache.json. AI mapping
rows remain review candidates, not automatic step-2 overrides.
