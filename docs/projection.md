# How the projected score works

The number at the top of the dashboard is a **projected total**, not your last score. It comes with a 95% interval (the `±` under it) and a confidence percentage. This page explains where each of those numbers comes from, what makes them move, and which parts are assumptions.

Everything here is computed in `dashboard.html` from the `DATA` block. The knobs live in `DATA.predict`.

## The short version

- **Level** is anchored on your most recent full practice test that reported both section scores. That is a real measurement, and nothing overrides it.
- **Movement** comes only from practice you log *after* that test. With no new practice, the projection equals your last score exactly.
- **The interval** is built from three sources of uncertainty: how little your new practice can say, how much a section score bounces between tests anyway, and how long it has been since anything was measured.
- **Confidence** is that interval's width compared against a ±150-point "knows nothing" band. A full test resets it upward. Elapsed time and logged practice both pull it down.

## Inputs

| Field | What it holds |
|---|---|
| `DATA.scores` | Each full test: date, label, `rw`, `math`. `null` means not taken yet. |
| `DATA.domains` | The four Reading and Writing and four Math domains, each with its share of the section (`weight`) and a `bands` list. `bands[i]` is that domain's `[low, high]` band on `scores[i]`. |
| `DATA.practice` | One row per practice set: date, domain code, `correct`, `total`, `timed`. |
| `DATA.predict` | The model's constants (see [The knobs](#the-knobs)). |

## Step 1: the anchor

The dashboard takes the **latest entry in `DATA.scores` with both `rw` and `math` filled in**. Its two section scores are the starting level. Only practice dated after it counts toward movement.

## Step 2: practice accuracy to a score band

For one domain, the dashboard pools every practice set logged since the anchor:

- `k` = total correct and `n` = total questions
- the **untimed share** is the fraction of those `n` questions done untimed

**The interval on accuracy.** It takes a 95% [Wilson score interval](https://en.wikipedia.org/wiki/Binomial_proportion_confidence_interval#Wilson_score_interval) on `k / n`. Wilson is used instead of the usual normal approximation because `n` is small and accuracy is high, where the normal interval can run past 100%.

**Accuracy to points.** A straight line through two anchor points (by default 50% → 480 and 100% → 800):

```
score(acc) = 480 + (acc − 0.50) × (800 − 480) / (1.00 − 0.50)
```

Then:

- it subtracts `untimed × untimedShare` points, because untimed work flatters accuracy
- it clamps the result to 200–800 and rounds to the nearest 10

Applying this to the low end, the midpoint and the high end of the Wilson interval gives the domain's estimated band and midpoint.

> **This map is an assumption.** College Board publishes no conversion from domain accuracy to the 200–800 scale. That's why `calibrated` is `false`. Score reports show a bar per domain, not a numeric band, so the domain bands in `DATA.domains` are estimated from your miss counts. Calibrating the map against them would feed the model's own output back into it as if it were a measurement.

## Step 3: movement

For each domain with post-anchor practice:

```
move = estimated midpoint − midpoint of that domain's band on the anchor test
```

Each section's movement is the weight-averaged sum of its domains' moves:

```
section delta = Σ (weight / 100) × move
projected section = anchor section score + section delta
projected total   = projected RW + projected Math
```

A domain with no practice since the anchor contributes nothing, so it stays where the test put it.

## Step 4: the interval

Every quantity below is a **95% half-width in points**, so they can be combined directly.

| Source | How it's computed |
|---|---|
| **Practice** (per section) | Σ `(weight / 100) × (band width / 2) × (1 + untimedShare)`. Summed directly rather than in quadrature, because domains within a section tend to move together. Untimed work widens it on top of the points it already loses. |
| **Section noise** | `sectionNoise` per section: how much a score bounces between tests anyway. |
| **Drift** | `drift × weeks since the anchor`. Uncertainty grows the longer nothing has been measured. |

These combine in quadrature:

```
half = √( rwPractice² + mathPractice² + 2 × sectionNoise² + drift² )
```

The dashboard shows `total ± half`, with the interval rounded to the nearest 10.

## Step 5: confidence

```
confidence = 1 − half / confRef        (clamped to 0–99%)
```

With the default `confRef` of 150, a ±150 interval reads as 0% and a ±35 interval as about 76%.

## What makes the numbers move

| Event | Projection | Confidence |
|---|---|---|
| You log a full practice test | Resets to the new test's scores | **Jumps up**: drift and practice width reset to zero |
| You log practice after a test | Moves by what the practice says | **Goes down**, less so for timed work and larger sets |
| Nothing gets logged | Stays put | **Drifts down** about 8 points of width per week |

**Confidence is not a progress metric between tests.** Logging practice always widens the interval at first. A pile of question-bank work is weaker evidence than a full test, and the model says so. As you log more, confidence climbs back toward the no-practice level from below, but only a full test raises it past that. At 75% accuracy, for example:

| Logged since the last test | Interval | Confidence |
|---|---|---|
| nothing | ±35 | 76% |
| 20 untimed | ±69 | 54% |
| 60 untimed | ±51 | 66% |
| 200 untimed | ±41 | 73% |
| 20 timed | ±46 | 69% |
| 100 timed | ±38 | 75% |

Timed work costs much less confidence than untimed work, which is one more reason to time your sets once the basics are in place.

## The ~20-question rule

**Don't pool a single domain into `DATA.practice` from one sitting with fewer than about 20 questions in that domain.** Below that, the Wilson interval is so wide, and the linear map so steep, that one miss can read as a 70-point drop. An 8-question math block split four ways says almost nothing about any one domain.

Small blocks still matter. Their misses go into the error log, where they drive the repeat-offender table. They just don't move the projection. The sample dashboard leaves its 15-question Set I out of `DATA.practice` for this reason and says so in a comment.

## Worked example (the sample dashboard)

On day 20 of the sample plan:

**Anchor.** Practice #2: RW 660 + Math 670 = **1330**.

**Practice since then.** Three timed 20-question sets:

| Domain | Result | Wilson 95% | Band | Midpoint | Band on the anchor test | Move |
|---|---|---|---|---|---|---|
| Information and Ideas | 17/20 | 64%–95% | 570–770 | 700 | 650–710 (mid 680) | **+20** |
| Craft and Structure | 16/20 | 58%–92% | 530–750 | 670 | 620–680 (mid 650) | **+20** |
| Standard English Conventions | 16/20 | 58%–92% | 530–750 | 670 | 630–690 (mid 660) | **+10** |

**RW delta.** 0.26 × 20 + 0.28 × 20 + 0.26 × 10 = **+13.4**. No math practice was pooled, so Math stays at 670.

**Projection.** 1330 + 13.4 = **1343**.

**Interval.**
- RW practice: 0.26 × 100 + 0.28 × 110 + 0.26 × 110 = 85.4
- Section noise: 25 per section
- Drift: 8 × 1.7 weeks ≈ 13.7
- Total: √(85.4² + 2 × 25² + 13.7²) ≈ **±93**

**Confidence.** 1 − 93 / 150 ≈ **38%**.

So the header reads **1343, ±93, 38% confidence**. The projection is up a little on good timed practice, and confidence is low because three 20-question sets can't pin down a score the way a full test can.

## The knobs

All of these live in `DATA.predict`. They are judgment calls, not measured facts.

| Knob | Default | Meaning |
|---|---|---|
| `anchors` | 50% → 480, 100% → 800 | The two points of the accuracy-to-score line |
| `untimed` | 40 | Points docked from fully untimed practice |
| `z` | 1.96 | Wilson z-value (1.96 = 95%) |
| `calibrated` | `false` | Stays `false` until the map is fitted against real numeric domain scores |
| `sectionNoise` | 25 | ±points one section score bounces between tests |
| `drift` | 8 | ±points of uncertainty added per week without a measurement |
| `confRef` | 150 | The ±interval that reads as 0% confidence |

If you change a knob, change it in `DATA.predict`, never in the render code, so the header and the Topics tab keep agreeing.
