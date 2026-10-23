# Missing structures and incremental mapper handoff

After checking the input workbooks, inspect all missing structures before building
the final scientific review queue. Use this workflow when step-2 metabolites lack
SMILES. It is an explicit exception to Final_Mapping-only edits: a derived staging
workbook stores completed structures and new predictions for affected reactions.
The original workbook, step-1 input, fluxes and equations remain unchanged.

## Inspect and establish identity

Run with the compatible project's tracing Python, using the installed skill path:

```bash
python <skill>/scripts/complete_structures.py inspect <model>_2_atom_mapping.xlsx --project <project>
```

The result lists missing metabolite identities, available formula/charge/InChIKey,
and exact-context cached proposals from `data/ai_structure_reviews/`. Revalidate
cached proposals; an archive is provenance, not independent evidence. Corrupt
archive hashes are reported. Existing nonempty but wrong SMILES need a separate
explicit correction; this helper does not silently override them.

Resolve identity from the model's database identifiers and annotations first,
then inspect authoritative compound entries (for example ChEBI, PubChem or model
source annotations) and relevant primary evidence. Record the exact entry URL,
database identifier, access date and the identity each source supports. Check
stereoisomers, protonation/charge, formula and polymer/chain-length conventions.
Name equality or matching formula alone does not prove a unique compound. Do not
invent structures, turn protein/biomass/tRNA pools into monomers, or choose an
arbitrary candidate when identity remains ambiguous. Report such cases as unresolved.
Use retrieved evidence rather than claiming model-generated SMILES are database facts.

## Complete and remap

Write one proposal JSON per workbook, binding it to the inspect hash:

```json
{
  "input_file": "/path/to/MyCulture_2_atom_mapping.xlsx",
  "input_sha256": "<SHA256 from inspection>",
  "structures": [
    {
      "MetaboliteID": "<exact saved ID>",
      "MetaboliteName": "<exact compartment-qualified name>",
      "SMILES": "<evidence-backed structure>",
      "database_id": "<database and compound ID>",
      "rationale": "<why this exact structure matches the modeled compound>",
      "evidence": [
        {
          "url": "https://<exact compound entry>",
          "accessed_on": "YYYY-MM-DD",
          "supported_claim": "<identity, stereochemistry and chemical form established>"
        }
      ]
    }
  ]
}
```

```bash
python <skill>/scripts/complete_structures.py complete structures.json --project <project> --output-dir <project>/.cache/ai/structures/<unused-run-directory>
```

Omitting --output-dir uses a fresh <project>/.cache/ai/structures/<run>/ directory.
Keep the staging workbook and its .structures.json together there; publish only
the later validated AI mapping copy to the case's result folder.

Run completions sequentially within one project, because the AI structure table is
shared. This invokes installed mapper environments; it is not a remote prediction
service. The helper imports the selected project's `step2_atom_mapping.py` settings
and existing preparation, mapper, selection, fallback and export functions. It
requires all three mapper sheets and compatible project APIs. It does not install
or substitute missing environments, ignore atom limits, change fluxes or run step 3.
If environments or interfaces are unavailable, report the blocker; do not claim
that mapping completed. Validated identity discoveries may already be archived
and saved for the next attempt.

Machine checks require valid specified structures, matching saved names, model
formula/charge and InChIKey where available, evidence fields and unchanged input
hashes. They cannot prove the scientific correctness or unique identity of a
proposal; that remains the agent's evidence-based responsibility.

Validated AI structures are persisted in ai_smiles.csv beside the configured
manual table (normally data/ai_smiles.csv), using Species, MetaboliteName, SMILES,
Source. Source contains evidence URLs and an immutable hash-named JSON record
under data/ai_structure_reviews/. The manual CSV is never changed. Conflicting
identities raise an error rather than being overwritten. Future ordinary step-2
runs can fill missing input structures from the AI table without repeating lookup,
with StructureReviewRequired=TRUE and formula/charge validation. Archive
candidate reuse requires the same model/ID/name/formula/charge/InChIKey context;
other cases require identity reconciliation rather than blind cache reuse.

The staging workbook keeps its original basename for the existing review helper.
It updates the metabolite sheet and only affected rows in the mapper,
Mapping_Selection and Final_Mapping sheets. Unrelated row values are retained.
Affected partial rules and mapper-status records are regenerated. Original
SourceInputSHA256 is preserved, and Mapping_Info records the parent workbook hash
and structure archive. The helper checks the step-3 reader before delivery.

Only affected reactions now accepted as ready by the project's Rules/preparation
checks are passed to the three mappers, under existing CPU/memory limits. Inputs
may cover the full original reaction or an allowed known-structure subreaction.
Reduced inputs remain Partial; retain omitted-component context for scientific
review. All affected reactions remain ReviewRequired=TRUE, including high-confidence
or agreeing predictions. Rebuild the scientific review queue after this stage. Pure identity transports need no
mapper call. Reactions still lacking other structures or blocked by fractional
stoichiometry/size limits remain reviewable. At least one nonempty mapper result
must survive selection; an invalid retained candidate is not a propagation input.
Only when all predictions are empty may project CarbonRules apply.

New metabolite rows have `AIStructureCompleted=TRUE`; their structure provenance
is recorded separately. Mapper-produced atom maps have `AIGenerated=FALSE`, even
when AI discovered an input structure. Affected reactions remain
`ReviewRequired=TRUE` until scientific review; do not confuse automatic remapping
with actual AI reaction review.

## Continue scientific review and deliver

Inspect the staging workbook with `review_workbooks.py inspect`, then perform
reaction-level analysis with `enzymatic-atom-mapping`. This is required even when
all three mappers returned empty predictions or no candidate passed validation.
Analyze the entire original reaction using its newly completed structures and
biochemical evidence, not just a partial fallback pair. Mapper success is not a
gate for AI analysis. Re-extract hashes and targets;
never apply proposals or reaction caches bound to the original workbook unchanged.
Run the usual review helper against this staged input and deliver its
`<model>_2_atom_mapping_AIGenerated.xlsx` copy to the case's result directory.
Ordinary correspondence review still changes Final_Mapping only; the explicit
structure-completion stage is responsible for its other-sheet changes. Do not
save the staging workbook over an existing original or AI delivery.

Keep the structure handoff JSON (`*.structures.json`) beside its staging workbook
so correspondence review can verify and list the added SMILES. The readable report
only needs the summary and actual compound/reaction changes. Keep original hashes,
missing/unresolved compounds, cache reuse, evidence, affected reactions, mapper
submissions/results and blocked operations in the detailed JSON and archive. If no
reaction-level AI review was performed, state this and do not set AIReviewed or
AIGenerated merely to produce a file. Keep staging private while an operation is
in progress; preserve its provenance beside the delivery when needed for audit.
The completion helper does not launch propagation. After reaction review and
validated delivery, the skill continues to step 3 by default, unless review-only
was requested or a workflow orchestrator owns the subsequent run.

The manual CSV has exactly Species, MetaboliteName, SMILES. Species is the full
Latin binomial with underscores, not the analysis condition name. non_IUPD/IUPD
conditions and compartment suffixes share the same species/name entry. Proposal
JSON and immutable archives still retain workbook IDs, names, model and chemical
context for binding; those audit fields are not extra manual CSV columns.
Source models live under full-species directories in data/. Resolve the species
from the case plan's source model directory or the full analysis-name prefix.
