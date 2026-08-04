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
>    information not provided.
> 5. **Independent yardstick** — in your own words, without reference to any grading
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
[findings or "none found"]

## Resource Sufficiency Check
[findings]

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
> For each verifier, determine:
> 1. **Matches prompt** — does this verifier trace to a specific requirement in the
>    requirement list? Quote the requirement it matches, or state "no match found."
> 2. **Matches yardstick** — does it correspond to something on the independent
>    yardstick? State yes/no.
> 3. **Path lock-in** — does the verifier's language allow more than one reasonable
>    resolution for any ambiguity flagged in the ambiguity inventory that it
>    touches, or does it require one specific method/value where the instruction
>    left the choice open? Quote the specific rubric language you're basing this on.
> 4. **Embedded factual claims** — list every specific number, date, or figure the
>    verifier's own text asserts as fact (these will need independent verification
>    later; don't verify them now, just extract them).
> 5. **Shared inputs** — what underlying source figures or claims does this
>    verifier's pass/fail depend on? (You'll use this across verifiers next.)
>
> After going through every verifier individually, build a dependency matrix: which
> verifiers share which underlying input figures. Flag any input figure that, if
> wrong, would fail more than one verifier, and state whether the resulting
> weight concentration still seems proportionate to the severity of that single
> underlying error.
>
> Finally, list any yardstick item from Stage A that no verifier covers.
>
> [INSTRUCTION TEXT] [RUBRIC TEXT] [STAGE A OUTPUT]

**Output format (Stage B):**

```
## Per-Verifier Table
| Verifier ID | Matches Prompt (quote) | Matches Yardstick | Path Lock-in Risk | Embedded Factual Claims | Shared Inputs |
|---|---|---|---|---|---|
| v1 | ... | Y/N | Y/N + quote | [list] | [list] |
...

## Dependency / Double-Counting Matrix
[which verifiers share which inputs; flag concentration risk]

## Yardstick Coverage Gaps
[yardstick items with no matching verifier, or "none"]
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
> 1. **Band-tightness check.** For every numeric tolerance in the rubric, assess:
>    if this task were solved with a different but equally defensible methodology
>    than the golden solution used, how far from golden's specific value would that
>    alternative plausibly land? State whether the rubric's stated tolerance is wide
>    enough to admit that plausible alternative, or whether it appears sized to
>    golden's specific number rather than to the space of correct answers.
> 2. **Alternate-path substance check.** For every verifier that names an accepted
>    alternative to golden's method (e.g., "X or a justified alternative"), actually
>    work through what that alternative would produce numerically, using the same
>    attached inputs golden used. State whether it would actually pass the verifier
>    and every other verifier that depends on the same figure, or whether the
>    "alternative" is cosmetic and wouldn't really survive.
> 3. **Structural mirroring check.** Does the rubric's organization, section order,
>    or phrasing track golden's specific structure more closely than it tracks the
>    instruction's structure? Give examples if so.
> 4. **Fact verification.** For every embedded factual claim extracted in Stage B,
>    search for and verify it against a real independent source (not the golden
>    solution, not the rubric's own justification text). Report match / mismatch /
>    unable to verify, with the source, for each one. Prioritize by verifier weight
>    — verify every core-weighted claim.
>
> [STAGE B OUTPUT] [GOLDEN SOLUTION] [RUBRIC TEXT]

**Output format (Stage C):**

```
## Band-Tightness Findings
| Verifier ID | Golden Value | Plausible Alternative Range | Rubric Tolerance | Overfit Risk (Y/N) |
|---|---|---|---|---|

## Alternate-Path Substance Check
[per applicable verifier: does the named alternative actually survive]

## Structural Mirroring
[findings or "none found"]

## Fact Verification Table
| Verifier ID | Claim | Real Source | Match / Mismatch / Unverifiable |
|---|---|---|---|
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
> and only then (c) PASS or FAIL with a one-line reason.
> Do not skip straight to a verdict — a verdict without the quoted clause and
> quoted output passage side by side will not be accepted.
>
> If a real autograder result is provided for this output, compare your verdict to
> it for every verifier. For every disagreement, classify it as one of:
> - **rubric ambiguity** — the clause could reasonably be read either way, and you
>   and the autograder resolved it differently (quote the ambiguous language)
> - **grading error** — you misapplied a clause you understood correctly (state
>   what you missed)
> - **evidence-access gap** — you lacked visibility into part of the output the
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
