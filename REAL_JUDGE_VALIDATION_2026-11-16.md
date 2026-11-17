# Real-judge validation — results, 2026-11-16

This lab's method notes have always said the rubric grader is a
deterministic proxy, and that the validation procedure should be
re-run with a real model judge before one is trusted. That re-run
has now happened. This file is its record.

**Provenance.** Prepared from a written run-spec dated 2026-11-16
and executed externally on the author's work account per that
run-spec; results returned the same day. The judge was Claude
Opus 5.5, reached through a hosted serving endpoint that does not
accept a temperature setting, so it ran at default sampling. All
inputs were this estate's public synthetic items — the frozen
150-item labelled subset, runreckon's 12-item grounding
calibration set, and a newly generated 410-item set. No other data
was used. Every number below was produced by the verdictpack
reporting scripts (`grader_report.py`, `judge_report.py`) from the
returned verdicts, or by runreckon's own `semantic.calibrate()`.

| Run record | Value |
|---|---|
| evalcrucible head at run time | `122812899c61b03bbb3e83a08b2d579b42fa7fb8` |
| runreckon head at run time | `cee771d4dc459ad75f3a6a932212bbda19b0e508` |
| Judge | Claude Opus 5.5 (hosted serving endpoint; temperature not settable) |
| Judge calls | 634 in total (150 + 12 + 410 scored items, plus review items later replaced and five replies re-run after truncation at a low token cap) |
| Judge tokens | 122,477 prompt, 52,224 completion |
| Extended-set generation | wording produced by a second model; every arithmetic answer and review label re-checked by a third, independent model — no checked item was dropped |

## The frozen 150: the real judge is ACCEPTED

Pre-registered bound H2, unchanged: agreement ≥ 0.90 AND
Cohen's kappa ≥ 0.75.

| Grader (150 items) | Agreement (Wilson 95%) | Kappa | False passes | False fails | Pass P / R | Fail P / R | H2 |
|---|---|---|---|---|---|---|---|
| real judge (Opus 5.5) | 99.3% (0.963–0.999) | 0.986 | 0 | 1 | 1.000 / 0.983 | 0.989 / 1.000 | ACCEPT |
| exact | 80.0% (0.729–0.856) | 0.545 | 0 | 30 | 1.000 / 0.500 | 0.750 / 1.000 | FALL BACK |
| normalized | 100.0% (0.975–1.000) | 1.000 | 0 | 0 | 1.000 / 1.000 | 1.000 / 1.000 | ACCEPT |
| rubric (stand-in) | 99.3% (0.963–0.999) | 0.986 | 1 | 0 | 0.984 / 1.000 | 1.000 / 0.989 | ACCEPT |

The real judge lands exactly where the rubric stand-in landed —
99.3% / 0.986, ACCEPT — so on this set the stand-in's verdict about
itself was honest. The three deterministic lines were re-derived
during the run and match `RESULTS.md`.

### The one disagreement is a gold error

The judge's single miss on the 150 is `arith-05-plain`, and it is
this lab's defect, not the judge's. The arith-05 template — "Tom
had {a} stickers. He bought {b} more packs of stickers" — never
states how many stickers are in a pack, so the gold answer rests
on an unstated one-sticker-per-pack assumption. The judge named
the gap on all five arith-05 items and failed the plain form for
it. **The gold is documented here as defective and is NOT
relabelled**: the labelled subset is frozen pre-registered ground
truth, and quietly repairing it after a measurement would destroy
exactly what the freeze protects. One inconsistency is also on
record: the judge passed the formatted twin answer "239.00" while
failing the plain "239", so its verdicts on equivalent outputs
were not uniform — a default-sampling endpoint will do that, and
any future re-run may flip borderline items.

Full per-item detail: `data/real_judge_disagreements.csv`.

## runreckon's 12-item grounding set

The same judge was run blind over runreckon's bundled grounding
calibration set. It agreed with gold on all 12 items: agreement,
kappa, precision and recall all 1.000, against the deterministic
stand-in's 0.833 / 0.667 / 0.833 / 0.833. Read that with the
small-N warning attached: at 12 items the Wilson interval is
0.758–1.000, far too wide to separate this judge from much weaker
ones. It is a smoke calibration, not a measurement.

## The extended set: 410 construction-labelled items

The run also generated a larger labelled set on a feature ×
scenario grid, to test whether standings earned on 150 items
survive a wider and less forgiving sample. Gold is fixed by
construction: every number in a prompt and every expected answer
was chosen by the generator program; the model assistance was
wording only, and an independent model re-checked every
arithmetic answer and review label. 410 items were generated and
all 410 survived deduplication (against the 150, against every
existing suite prompt, and within the set). Gold split: 170 pass /
240 fail.

| Family | Items | Shapes |
|---|---|---|
| arith | 100 | plain, formatted, delta, swap, empty |
| extract | 100 | plain, formatted, delta, swap, empty |
| classify | 100 | plain, formatted, delta, swap, empty (10 of 20 reviews carry their sentiment through a negation) |
| zero matches | 40 | none, paraphrased none, other field's number, invented 0 |
| one match | 30 | exact, an extra value added, the other record's value |
| several matches | 40 | in order, reordered, one dropped, one duplicated |

| Grader (410 items) | Agreement (Wilson 95%) | Kappa | False passes | False fails | Pass P / R | Fail P / R | Bound on 410 | Bound on 150 |
|---|---|---|---|---|---|---|---|---|
| real judge (Opus 5.5) | 98.0% (0.962–0.990) | 0.960 | 1 | 7 | 0.994 / 0.959 | 0.972 / 0.996 | ACCEPT | ACCEPT |
| exact | 80.5% (0.764–0.840) | 0.568 | 0 | 80 | 1.000 / 0.529 | 0.750 / 1.000 | FALL BACK | FALL BACK |
| normalized | 95.1% (0.926–0.968) | 0.898 | 0 | 20 | 1.000 / 0.882 | 0.923 / 1.000 | ACCEPT | ACCEPT |
| rubric (stand-in) | 85.9% (0.821–0.889) | 0.711 | 34 | 24 | 0.811 / 0.859 | 0.896 / 0.858 | FALL BACK | ACCEPT |

### The headline: the rubric stand-in FALLS BACK at scale

The one standing that changed belongs to this lab's own proxy.
Accepted on the frozen 150, the rubric grader falls to FALL BACK
on the 410 (85.9% / 0.711), and the failure is not noise — it is
mechanism. Thousands separators in formatted numbers break its
token overlap (14 formatting rejections among its disagreements).
Worse, its two generosity terms turn dangerous on the new shapes:
first-number closeness plus credit for prompt material produced
32 numeric-closeness errors and 2 prompt-material false passes —
34 false passes in total, waving through answers that dropped a
value, duplicated one, or reported the wrong record's number.
The lesson is the lab's thesis stated from the other side: a
grader's acceptance is a property of the set it was measured on.
The rubric grader was never "a 99.3% grader"; it was a grader
that scored 99.3% on 150 narrow items.

The normalized grader holds ACCEPT (95.1% / 0.898) but is not
spotless: all 20 of its misses are paraphrase misses — paraphrased
empty answers and reordered lists it cannot recognise. Exact
stays FALL BACK (80.5% / 0.568), rejecting correct answers for
their shape, as before.

Error modes across every disagreement on the 410 (all of them are
in `data/extended_set_disagreements.csv`, with mechanism notes):

| Grader | formatting rejection | paraphrase miss | negation false-pass | numeric-closeness error | prompt-material false-pass | gold error |
|---|---|---|---|---|---|---|
| judge | 0 | 1 | 0 | 0 | 0 | 7 |
| exact | 60 | 20 | 0 | 0 | 0 | 0 |
| normalized | 0 | 20 | 0 | 0 | 0 | 0 |
| rubric | 14 | 10 | 0 | 32 | 2 | 0 |

### Known defects in the extended set — documented, NOT relabelled

The judge's dominant error mode on the new set is, again, gold
error: 7 of its 8 disagreements are construction defects in the
set itself, and one is a paraphrase miss. The zero-match
generator asked three record bases for a field that is derivable
from, or synonymous with, a field the record does state:

- `syn-card0-04-zero-none`, `syn-card0-04-zero-paraphrase` — a
  booking asked for its "total due", which is derivable as
  nights × room rate (7 × 656 = 4592), so "none" is not the only
  defensible answer.
- `syn-card0-06-zero-none`, `syn-card0-06-zero-paraphrase`,
  `syn-card0-06-zero-otherfield` — a membership asked for its
  "fee charged", a near-synonym of the stated "renewal cost"
  (597); on the otherfield variant the judge passed the answer
  597 against a fail gold, for the same reason.
- `syn-card0-10-zero-none`, `syn-card0-10-zero-paraphrase` — a
  booking asked for its "total due", derivable as 3 nights × a
  326 room rate = 978.

These seven items are named here as known defects. They are NOT
relabelled and NOT removed: the set is committed as generated,
defects disclosed, so anyone re-using it can exclude them with
full knowledge. Bases syn-card0-04, syn-card0-06 and syn-card0-10
should be rebuilt with a field that is neither derivable nor
synonymous before the set is used for a fresh measurement. For
reference only: without those 7 items the judge agrees on 402 of
403.

The remaining judge disagreement, `syn-cardN-08-many-reordered`,
is a genuine paraphrase miss with a policy wrinkle: the three
amounts were correct but listed in reverse order with no labels,
and the judge read the sequence as mapping values to the wrong
orders. The set's construction treats order as immaterial; the
prompt never said so.

Across all three sets, 8 of the judge's 9 disagreements are gold
errors. A validated judge audits the benchmark right back — which
is, in the end, the finding this whole estate exists to produce.

## Data artefacts

Committed with this record, under `data/`:

- `real_judge_disagreements.csv` — every judge disagreement from
  the frozen-150 run and the runreckon 12-item run, with error
  mode, reason, and the judge's rationale.
- `extended_labelled_set.csv` — the 410 construction-labelled
  items (`item_id,prompt,output,expected,gold,source`; source
  `synthetic-2026-11`), including the seven known-defective items
  named above, unrelabelled.
- `extended_set_disagreements.csv` — every disagreement by all
  four graders on the 410 items, with error mode and mechanism
  notes.

## What would make these numbers wrong

- The judge ran at its endpoint's default sampling — no
  temperature control exists there — and it treated like cases
  differently twice (the plain "239" failed while "239.00"
  passed; one reordered list rejected while nine were accepted).
  A re-run can flip a few borderline verdicts.
- The frozen 150 are program-built and narrow — three task
  families. The bound was met on these items, not on open-ended
  answers.
- runreckon's set has 12 items; its interval (0.758–1.000) is too
  wide to separate the judge from much weaker ones.
- Construction gold is only as good as the construction: seven
  items in the extended set are defective (named above), and the
  reordered-list items carry a policy choice — order does not
  matter — that the prompt did not state.
