# Task Quality Audit — LLM Prompt Set

This is a **four-stage** prompt set, not one prompt. Staging exists to preserve the
audit's core discipline: forming an independent view of the prompt and the expected
answer *before* seeing the rubric, and forming a view of the rubric *before* seeing
the golden solution. Feeding everything to the model in one call defeats this — the
model will silently let the rubric's phrasing define "good" before it has formed its
own view. Run the stages as separate LLM calls (fresh context each time, or at
minimum a clear instruction to disregard anything from later attachments), and pass
each stage's output forward as input to the next.

A human reviews the output of every stage before releasing the next one's inputs.
This is where the human stays in the loop — not by re-doing the work, but by
deciding whether each stage's output is sound enough to build on.

---

## What you need before starting

Gather all of this up front; you won't need all of it for every stage, but having it
ready avoids mid-audit delays:

1. **The instruction/prompt text** — the full task as given to the model being
   evaluated, including any runtime file-mapping or format instructions.
2. **The rubric / verifier file** — every verifier, with its full prompt text,
   weight, importance (core/secondary), grading_role, and expected value.
3. **The golden solution** — all deliverable files/content produced as the
   reference-quality answer.
4. **The model output(s) to be graded**, if grading is in scope for this run — one
   or more full deliverable sets from actual model attempts.
5. **The real autograder result(s)**, if available — per-verifier pass/fail/score,
   for reconciliation in Stage D.
6. **Search/tool access** for Stage C, if any rubric verifier embeds a specific
   factual claim (a number, date, ratio, named figure) that should be checked
   against a real external source.

---

## Stage A — Prompt-only review and independent yardstick

**Give the model:** only item 1 (the instruction). Do not include the rubric, the
golden solution, or any model output. If your tool doesn't let you literally hide
files, explicitly instruct the model not to look ahead and don't paste the other
materials into this call at all.

**Prompt:**

> You are auditing a task instruction for quality, before seeing how it will be
> graded. You will be shown only the instruction. Do not assume anything about how
> this will be scored — you have not seen a rubric and should not guess at one.
>
> Read the instruction below and produce:
>
> 1. **Requirement list** — every distinct thing the instruction asks for, numbered,
>    quoting the exact sentence or clause it comes from. Include conditional
>    requirements (anything of the form "if X, then Y") as their own numbered items.
> 2. **Ambiguity inventory** — every place the instruction leaves a methodological,
>    scope, or format choice open without saying so explicitly. For each, state
>    what the open choice is and what 2–3 different reasonable resolutions would
>    look like.
> 3. **Consistency check** — any contradictions in dates, units, defined terms, or
>    scope across the instruction. State "none found" if genuinely none.
> 4. **Resource sufficiency check** — given only the named/attached inputs, is every
>    requirement in your list actually answerable? Flag anything that would require
>    information not provided. Give an overall label, chosen from exactly this list
>    (pick one, do not invent a new label): `Sufficient` / `Partially Sufficient —
>    gaps noted` / `Insufficient`.
> 5. **Consistency verdict** — an overall label for the consistency check in item 3,
>    chosen from exactly this list: `Consistent` / `Minor Inconsistencies` / `Major
>    Inconsistencies`.
> 6. **Independent yardstick** — in your own words, without reference to any grading
>    scheme, what would a genuinely good answer to this instruction contain? Write
>    this as a bulleted list, one bullet per substantive component a good answer
>    needs.
>
> [INSTRUCTION TEXT HERE]

**Output format (Stage A):**

```
## Requirement List
1. [requirement] — source: "[quoted clause]"
...

## Ambiguity Inventory
- [open choice] — reasonable resolutions: [A], [B], [C]
...

## Consistency Check
Findings: [details or "none found"]
Verdict (pick one): Consistent / Minor Inconsistencies / Major Inconsistencies

## Resource Sufficiency Check
Findings: [details]
Verdict (pick one): Sufficient / Partially Sufficient — gaps noted / Insufficient

## Independent Yardstick
- [component]
- [component]
...
```

**Human checkpoint before Stage B:** read the yardstick and edit it. Don't just
approve it — this is the one point in the process where your judgment isn't yet
anchored by the rubric's language, so it's worth spending real time here.

---

## Stage B — Rubric vs. prompt and yardstick

**Give the model:** item 1 (instruction), item 2 (rubric), and the human-edited
Stage A output. Still withhold the golden solution and any model outputs.

**Prompt:**

> You previously audited (or are now given) an independent requirement list and
> yardstick for this instruction, produced without seeing any rubric. You will now
> be shown the rubric used to grade this task. For every verifier in the rubric,
> check it against the instruction and the yardstick — do not let the rubric's own
> framing tell you whether it's correct; check it against the independent materials
> below.
>
> For each verifier, determine, and for every labeled judgment below pick exactly
> one value from the given list — do not invent new labels or hedge between two:
>
> 1. **Matches prompt** — does this verifier trace to a specific requirement in the
>    requirement list? Quote the requirement it matches (or "none" if not found).
>    Label (pick one): `Full Match` / `Partial Match` / `No Match`.
> 2. **Matches yardstick** — does it correspond to something on the independent
>    yardstick? Label (pick one): `Full Match` / `Partial Match` / `No Match`.
> 3. **Path lock-in** — does the verifier's language allow more than one reasonable
>    resolution for any ambiguity flagged in the ambiguity inventory that it
>    touches, or does it require one specific method/value where the instruction
>    left the choice open? Quote the specific rubric language you're basing this
>    on. Label (pick one): `None` / `Low` / `Moderate` / `High`.
> 4. **Embedded factual claims** — list every specific number, date, or figure the
>    verifier's own text asserts as fact (these will need independent verification
>    later; don't verify them now, just extract them).
> 5. **Shared inputs** — what underlying source figures or claims does this
>    verifier's pass/fail depend on? (You'll use this across verifiers next.)
>
> After going through every verifier individually, build a dependency matrix: which
> verifiers share which underlying input figures. For every input figure shared by
> more than one verifier, label the resulting weight concentration (pick one):
> `Proportionate to Error Severity` / `Disproportionate to Error Severity`.
>
> Finally, list any yardstick item from Stage A that no verifier covers, and give an
> overall coverage label (pick one): `Full Coverage` / `Minor Gaps` / `Major Gaps`.
>
> [INSTRUCTION TEXT] [RUBRIC TEXT] [STAGE A OUTPUT]

**Output format (Stage B):**

```
## Per-Verifier Table
| Verifier ID | Matches Prompt | Evidence Quote | Matches Yardstick | Path Lock-in Risk | Lock-in Evidence Quote | Embedded Factual Claims | Shared Inputs |
|---|---|---|---|---|---|---|---|
| v1 | Full/Partial/No Match | "..." | Full/Partial/No Match | None/Low/Moderate/High | "..." | [list] | [list] |
...

## Dependency / Double-Counting Matrix
| Shared Input Figure | Verifiers Affected | Weight Concentration Label |
|---|---|---|
| [figure] | [v#, v#, ...] | Proportionate / Disproportionate |

## Yardstick Coverage
Gaps: [yardstick items with no matching verifier, or "none"]
Overall coverage label: Full Coverage / Minor Gaps / Major Gaps
```

**Human checkpoint before Stage C:** manually review every "no match found" and
every "path lock-in risk: Y" row — these are the highest-value flags. Spot-check at
least 20–30% of the rows marked clean, since a model that's too generous here will
under-flag rigidity and you won't catch that pattern by only reading the flagged
rows.

---

## Stage C — Overfitting check and factual grounding

**Give the model:** everything from Stage B, plus item 3 (golden solution), plus
search/tool access.

**Prompt:**

> You will now see the golden (reference) solution for this task. Your job is to
> check whether the rubric was built independently of this specific solution, or
> whether it was fitted to it — and to verify every factual claim the rubric embeds
> against real external sources.
>
> For every labeled judgment below, pick exactly one value from the given list.
>
> 1. **Band-tightness check.** For every numeric tolerance in the rubric, assess:
>    if this task were solved with a different but equally defensible methodology
>    than the golden solution used, how far from golden's specific value would that
>    alternative plausibly land? Label (pick one): `Wide Enough for Plausible
>    Alternatives` / `Borderline` / `Fitted to Golden's Specific Value`.
> 2. **Alternate-path substance check.** For every verifier that names an accepted
>    alternative to golden's method (e.g., "X or a justified alternative"), actually
>    work through what that alternative would produce numerically, using the same
>    attached inputs golden used. Label (pick one): `Substantive — Alternative
>    Would Pass` / `Partially Substantive — Alternative Passes Some Dependent
>    Verifiers Only` / `Cosmetic — Alternative Would Not Survive` / `Not
>    Applicable — No Alternative Named`.
> 3. **Structural mirroring check.** Does the rubric's organization, section order,
>    or phrasing track golden's specific structure more closely than it tracks the
>    instruction's structure? Give examples if so. Label (pick one): `None` /
>    `Some` / `Extensive`.
> 4. **Fact verification.** For every embedded factual claim extracted in Stage B,
>    search for and verify it against a real independent source (not the golden
>    solution, not the rubric's own justification text). Label each (pick one):
>    `Match` / `Minor Discrepancy` / `Mismatch` / `Unable to Verify`. Prioritize by
>    verifier weight — verify every core-weighted claim.
>
> [STAGE B OUTPUT] [GOLDEN SOLUTION] [RUBRIC TEXT]

**Output format (Stage C):**

```
## Band-Tightness Findings
| Verifier ID | Golden Value | Plausible Alternative Range | Rubric Tolerance | Label |
|---|---|---|---|---|
| v2 | ... | ... | ... | Wide Enough / Borderline / Fitted to Golden |

## Alternate-Path Substance Check
| Verifier ID | Named Alternative | Numeric Result if Applied | Label |
|---|---|---|---|
| v2 | ... | ... | Substantive / Partially Substantive / Cosmetic / Not Applicable |

## Structural Mirroring
Findings: [details or "none found"]
Label: None / Some / Extensive

## Fact Verification Table
| Verifier ID | Claim | Real Source | Label |
|---|---|---|---|
| v3 | ... | ... | Match / Minor Discrepancy / Mismatch / Unable to Verify |
```

**Human checkpoint before Stage D:** this stage is the least automatable — confirm
every "overfit risk: Y" and every "mismatch" personally before drawing conclusions.
Treat this stage's output as a prioritized list of things to personally check, not
as a verdict.

---

## Stage D — Model output grading and reconciliation

**Give the model:** the rubric, one model output to grade, and (if available) the
real autograder result for that output. Run this stage once per model output.

**Prompt:**

> Grade the attached model output against every verifier in the rubric. For each
> verifier, you must produce, in this exact order, before stating a verdict:
> (a) the exact clause or sub-clause of the rubric you are checking,
> (b) the exact passage, figure, or absence thereof in the model output that
> is relevant to that clause,
> and only then (c) a verdict, picked from exactly this list: `PASS` / `FAIL`.
> Do not skip straight to a verdict — a verdict without the quoted clause and
> quoted output passage side by side will not be accepted.
>
> If a real autograder result is provided for this output, compare your verdict to
> it for every verifier. For every disagreement, classify it using exactly one of
> these labels — do not invent a new category:
> - `Rubric Ambiguity` — the clause could reasonably be read either way, and you
>   and the autograder resolved it differently (quote the ambiguous language)
> - `Grading Error` — you misapplied a clause you understood correctly (state
>   what you missed)
> - `Evidence-Access Gap` — you lacked visibility into part of the output the
>   autograder could see
>
> [RUBRIC TEXT] [MODEL OUTPUT] [AUTOGRADER RESULT, IF AVAILABLE]

**Output format (Stage D):**

```
## Per-Verifier Grading
| Verifier ID | Clause Quoted | Output Passage Quoted | Verdict | Reason |
|---|---|---|---|---|

## Autograder Reconciliation (if applicable)
| Verifier ID | My Verdict | Autograder Verdict | Match? | Disagreement Type |
|---|---|---|---|---|
| v1 | PASS/FAIL | PASS/FAIL | Yes/No | Rubric Ambiguity / Grading Error / Evidence-Access Gap / N/A |

## Score Summary
Binary reward (weighted): ...
Continuous reward (unweighted pass rate): ...
```

**Human checkpoint:** spot-check 100% of core-importance verifiers and any verdict
where the model's own language hedges ("likely," "appears to," "probably"); sample
~30% of secondary-importance verdicts. For any output with a real autograder
result, review every logged disagreement — a pattern of "rubric ambiguity"
disagreements across multiple outputs is itself a finding worth escalating, distinct
from any single grading error.

---

## Stage E — Final synthesis and judgment

**Give the model:** the full outputs of Stages A through D (all outputs produced so
far for this task, across however many model outputs were graded in Stage D). No
new source material is introduced at this stage — Stage E only synthesizes.

**Prompt:**

> You have the complete audit trail for this task: the independent requirement list
> and yardstick (Stage A), the rubric-vs-prompt alignment findings (Stage B), the
> overfitting and fact-verification findings (Stage C), and the model-grading and
> autograder-reconciliation results (Stage D). Do not introduce new judgments not
> grounded in these — every label you assign below must cite which stage and which
> specific row/finding it is based on.
>
> Assign exactly one label from each list below. Do not invent new labels, do not
> pick more than one, and do not hedge between two — if the evidence is genuinely
> mixed, pick the label that best matches the balance of evidence and say so in the
> justification.
>
> 1. **Prompt quality** (based on Stage A): `Well-Specified` / `Specified with Minor
>    Ambiguity` / `Specified with Significant Ambiguity` / `Poorly Specified`.
> 2. **Rubric-prompt alignment** (based on Stage B): `Fully Aligned` / `Mostly
>    Aligned — Minor Gaps` / `Partially Aligned — Notable Gaps` / `Misaligned`.
> 3. **Rubric independence from golden solution** (based on Stage C items 1–3):
>    `No Evidence of Overfitting` / `Minor Overfitting Risk` / `Moderate
>    Overfitting Risk` / `Significant Overfitting`.
> 4. **Factual grounding** (based on Stage C item 4): `Fully Verified` / `Mostly
>    Verified — Minor Gaps` / `Significant Unverified Claims` / `Contradicted by
>    Source`.
> 5. **Real-world robustness** (based on Stage D, across all graded outputs):
>    `Robust — Rubric Performs as Designed` / `Robust with Minor Gaps` / `Fragile —
>    Frequent Rubric-Model Disagreement` / `Unreliable`.
> 6. **Overall task verdict**, weighing all five findings above (weight #2, #3, and
>    #4 most heavily, since a misaligned, overfitted, or factually wrong rubric
>    invalidates scoring regardless of how clear the prompt is or how the models
>    happened to perform):
>    `Approved — Ready to Use` / `Approved with Revisions Needed` / `Not Approved —
>    Requires Rework` / `Rejected`.
>
> For each of the six labels, give a one-to-three sentence justification that cites
> specific stage findings (e.g., "Stage B: v2, v7 — path lock-in risk High" or
> "Stage C: fact verification — 2 core-weighted mismatches"). Then list, separately,
> the specific, actionable revisions needed to move the task to the next-better
> label — e.g., what exact wording change would take a verifier from `High` lock-in
> risk to acceptable, or what fact needs correcting.
>
> [STAGE A OUTPUT] [STAGE B OUTPUT] [STAGE C OUTPUT] [STAGE D OUTPUT(S)]

**Output format (Stage E):**

```
## Final Judgment Summary
| Dimension | Label | Justification (cite stage + finding) |
|---|---|---|
| Prompt Quality | [pick from list] | ... |
| Rubric-Prompt Alignment | [pick from list] | ... |
| Rubric Independence | [pick from list] | ... |
| Factual Grounding | [pick from list] | ... |
| Real-World Robustness | [pick from list] | ... |
| **Overall Task Verdict** | **[pick from list]** | ... |

## Required Revisions (if not "Approved — Ready to Use")
1. [specific, actionable change] — targets: [dimension] — moves label from X to Y
2. ...

## Open Items Requiring Human Judgment
[anything Stage E could not resolve on its own — e.g., a genuine methodological
disagreement between auditor and autograder that needs a task-owner decision, not
just a re-read]
```

**Human checkpoint:** this is the release gate. A human reads the full Final
Judgment Summary and either signs off on the Overall Task Verdict or overrides it —
but an override must itself cite which specific stage finding the model weighed
incorrectly, using the same discipline the model was held to throughout. The task
does not ship on an unreviewed Stage E output, regardless of how confident it reads.
