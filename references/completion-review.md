# Whole-reaction completion review

Read this before deciding whether a flagged, empty or partial mapping can be
replaced. The goal is the most complete evidence-supported correspondence for
the original reaction, not merely a nonempty cell or a valid cofactor fragment.

## Reconstruct the actual input

- Read the original Equation, signed Flux, direction and saved metabolite
  identities/structures. Compare all raw mapper inputs and candidates with that
  equation; a raw ReactionSMILES can itself be reduced.
- Inventory every participant and its coefficient on both sides. Distinguish
  missing molecules, unexpanded multiplicities and unmapped positions within a
  molecule. Include carbon dioxide, bicarbonate, formate and other small products.
- Identify carbon-bearing functional groups separately from carrier skeletons.
  Correct CoA or NAD(P) assignments do not establish the fatty-acyl, nucleotide
  or other transferred-group correspondence.
- Resolve missing exact structures through the existing completion stage and
  rerun only newly ready affected mapper inputs. Do not replace a protein,
  polymer or generic biomass pool with an invented small-molecule structure.

The original equation is the completeness reference. Preserve its compartment
identities, direction, coefficients, charges, isotopes and stereochemistry.
Expand integer copies for the existing reaction-SMILES validator. Fractional
coefficients can be rescaled only when exact proportional equivalence and
project support are demonstrated; never round or reduce them to one copy.
A prepared-input limit is a compatibility blocker, not permission to omit atoms.

## Attempt the whole correspondence

Give enzymatic-atom-mapping the original equation, full available structures,
all coefficients, candidates, omissions and the requested completion scope.
Ask it to account for the carbon fate of every participant, then construct the
complete all-represented-atom replacement required by the current apply helper.
Mapper failure or a correct cofactor-only map does not finish this task.

Use retrieved reaction-specific biochemical evidence and mechanism to connect
the missing groups. Inspect cleavage products, decarboxylation, condensation,
carrier release and repeated substrate copies. Explain symmetry-equivalent
representatives; do not invent a unique physical order among equivalent copies.
A balanced formula or a minimum-edit graph match alone does not establish atom
correspondence. Do not distribute unknown carbon uniformly between products.

If all three mappers fail, the scientific skill still attempts the original
reaction directly. If a proposal fails validation, inspect the concrete error
and correct justified input assembly or correspondence mistakes, then recheck.
Stop when the identified evidence, structure, balance or interface blocker
cannot be resolved with available information. Do not loop over unsupported
guesses or weaken validation to obtain an accepted result.

## Acceptance and unresolved outcomes

Before apply, compare original and proposed component identities and counts,
and account for every original carbon atom on both sides. Then run the existing
all-atom validator and compatible project checks. A carbon-complete explanation
can still fail the all-atom contract; report that distinction.

A new accepted replacement must cover the original reaction and pass those
checks. Never validate a reduced proposal solely against a reduced mapper input
and call it complete. Do not silently add water, protons or other participants
to repair balance; document any upstream correction required.

Retaining a legacy partial correspondence is a last-resort unresolved completion
outcome, allowed only when the installed project still supports it and the
retained pairs have scientific support. State exactly what remains unmapped and
why a complete replacement was not delivered. If the project has disabled partial
mapping, do not restore its sheets, hidden records or propagation behavior.

## Completion audit in existing reports

For each analyzed incomplete target, put the following in its existing rationale
and limitations so both the Markdown and JSON outcomes retain it:

- Original omissions/unmapped groups, including coefficients and small products.
- Evidence consulted and the fate of each previously missing carbon-bearing group.
- Outcome: complete replacement, unchanged complete map, unresolved completion
  with a supported legacy partial map, or unresolved/unusable mapping.
- Remaining carbon positions/participants and the precise blocker, if any.
- Checks actually run; distinguish scientific analysis, structural validation
  and downstream execution.

Do not count a retained fragment as a completed repair. AIReviewed records real
analysis; AIGenerated records a successfully applied replacement. Neither flag
establishes completeness by itself. Link the existing reports and archives;
do not invent new workbook columns or another mapping implementation.

## Example: fatty-acyl-CoA synthase with CO2 products

For an original reaction containing one acetyl-CoA, eight malonyl-CoA and a
stearoyl-CoA product plus eight CO2 molecules, a CoA/NADP-only map is incomplete.
The review must investigate the acyl-carbon construction and all eight released
carbons, not only retention of the carrier skeletons.

A useful carbon-accounting check for the acyl groups is:
2 + 8 * 3 = 18 + 8. Reaction-specific evidence is still needed to establish
which malonyl carbon is released and its positional correspondence. Useful
starting entries are [fatty-acyl-CoA synthase, EC 2.3.1.86](https://iubmb.qmul.ac.uk/enzyme/EC2/3/1/86.html) and
[acetyl-CoA carboxylase, EC 6.4.1.2](https://iubmb.qmul.ac.uk/enzyme/EC6/4/1/2.html).
An enzyme equation supports the overall transformation; it does not validate
every atom pair. This example is a review obligation, not a hardcoded map.
