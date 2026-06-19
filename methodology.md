# Methodology: Thumbnail Face Location vs. Views

> The full methods for the study. The plan was registered and publicly timestamped before any
> placement-vs-views number was queried (see `preregistration.md`, tag `004-v0.1-preregistration`).
> This document describes what we did; the [design spec](./design-spec.md) explains why each choice
> was made.

## The question

Among videos whose thumbnail has a single human face, does *where* that face sits horizontally
(left, center, or right of the frame) relate to more views, within the same channel? "Put the face
on the left, not dead center" is a recurring piece of thumbnail advice, usually a left-versus-right
side argument, and rarely measured at scale with the channel difference removed. We take the
single-face population as given and ask only about horizontal placement.

## Dataset

The cohort is English-language business and creator-economy YouTube channels drawn from our own
dataset, each with at least 20 qualifying videos. They are **not a random sample of YouTube** (see
Limitations): they entered through our funnel and skew toward English-speaking, business and
creator-economy, mid-to-large channels.

A video qualifies if it is:

- a regular long-form upload (not a Short), decided by the platform's own video-type label;
- public (not unlisted, private, hidden, or deleted);
- organic (we exclude videos our traffic classifier flags as primarily ad-driven, because
  ad-driven views don't test whether the thumbnail earned the click);
- published between 90 days and 5 years before measurement (90 days so views have settled; 5 years
  so it stays comparable across thumbnail and algorithm eras);
- at least 90 seconds long. We add this duration floor on top of the platform's long-form label to
  trim the most-likely short-form tail, because a short vertical video whose thumbnail is padded can
  place a face oddly. The floor was fixed in the plan; it is a logged refinement of the qualifying
  filter.

From the qualifying videos this study keeps the **faced** ones (a thumbnail with at least one face
clearing the size floor below) and, for the headline, the **single-face** ones (exactly one face
clearing the floor). "Location" is only well defined for a single face, so the single-face frame is
the estimand; a dominant-face fork (locate any faced video by its largest face) is reported beside
it so the single-face conditioning cannot drive the result.

## Outcome

The outcome is **views**, measured at a single point in time (one capture, for almost the entire
cohort within a short window), reported alongside each video's age so the comparison holds age
constant (see "Why views, age-controlled" below). We work in log space because view counts are
heavily skewed by viral videos; a single outlier cannot swing a log-based comparison.

## The placement measurement

Each thumbnail is run through a face-detection model. A face counts only if its bounding box is at
least **2% of the thumbnail area** (the locked floor); we also report the headline at a **4%** floor
so the finding cannot hinge on one cutoff. For each kept face we record its bounding-box horizontal
center as a fraction of frame width, and we classify the locating face into three zones: **left**
`[0, 1/3)`, **center** `[1/3, 2/3)`, **right** `[2/3, 1]`.

Because horizontal position was not recorded when faces were first detected for face-presence work,
this study runs a fresh detection pass over the faced thumbnails to capture each face's box position.
Thumbnails that changed since the earlier pass are detected and excluded, so the placement always
describes the same image the presence work measured.

**Detector placement validation (run before any placement-to-views number).** Unlike a face's
*size*, a face's horizontal *center* is cheap to mark by hand, so we ran the numeric check directly.
On 150 hand-marked faces, the detector's center error does **not** drift with the face's true
horizontal position (slope −0.002, within a ±0.05 bound), the miss rate is even across the three
zones, and the error does not widen toward the frame edges. A detector whose placement error
correlated with position could manufacture a zone-to-views pattern out of nothing; ours does not.

## The comparison

**The unit is the video; the channel is the cluster.** Videos in the same channel are not
independent (shared creator, audience, niche, cadence), so we never treat videos as independent
observations. Every confidence interval uses the **channel** as the resampling unit.

**The headline is within-channel.** We compare a channel's left-faced videos to its own right-faced
videos, which cancels every stable channel trait (size, niche, audience, production quality) and so
can't be dismissed as "big channels just frame faces differently." A within-channel left-versus-right
contrast only exists for channels that use *both* sides; channels that always place the face on one
side drop out, so the headline describes side-varying channels (we report that set: 381 channels,
16,561 videos on the holdout).

**Two co-equal estimators, fixed in the plan.** We compute every zone contrast two ways and claim a
result only when both agree in sign and rough size:

- the **median of per-channel contrasts** (one channel, one vote; immune to a few high-volume
  channels), and
- the **pooled fixed-effects contrast** (video-count weighted).

This was fixed before seeing the numbers, not chosen after. The two estimators answering the same
question differently is the study's main guard against a result that lives only in the weighting.

**Why views, age-controlled, rather than raw views-per-day.** The whole cohort was measured on a
single clock, so a video's age is really *when we captured its views*, not a behavior of the video,
and dividing by it would put that measurement timing into the outcome. The estimator therefore
removes age explicitly instead: a channel-fixed-effects regression of **log(views) on the zone
contrast plus log(age)**. Channel fixed effects absorb every stable between-channel difference; the
age term removes the measurement-timing artifact.

**Confidence intervals.** A whole-channel bootstrap: resample channels with replacement (never
videos), re-demean within channel, refit, repeat 10,000 times (fixed seed). With this many videos,
statistical significance is automatic for any non-zero effect, so we de-emphasize p-values and report
effect sizes, channel-clustered intervals, and a practical-significance threshold (a ±10% step)
declared in advance.

## The shape guard

The advice we test is specifically about side (left versus right), so the headline is the
left-versus-right contrast. But to avoid leading with a single number while a different zone pattern
hides behind it, the plan included a mechanical shape guard: compute all three pairwise zone
contrasts and, if a center-involving contrast (left-versus-center or center-versus-right) is real by
its own interval **and** larger than the left-versus-right contrast, stop leading with the single
number and **show the full three-zone curve**.

The guard is a one-sided tripwire that reads only the volume-weighted estimator; firing tells us how
to *report*, not that a contrast is *real*. The claim test is still the two-estimator rule. In this
study the guard fired (the volume-weighted left-versus-center contrast is real and large), so we show
the curve; but the one-channel-one-vote estimator does not corroborate the center dip, so under the
locked rule we report the center pattern as unresolved shape, not a finding.

## The holdout

A fixed 20% of channels were set aside by a deterministic rule before any exploration, and the
headline number was computed on them **once**, with the identical analysis code, after the spec was
frozen on the other 80%. The exploration result is the confirming companion; the held-out number is
the headline.

## The decision rule

Each contrast is graded against a four-state rule fixed in advance, with a ±10% step as the
practical bar:

- **present and meaningful**: the interval excludes zero and the point is at least ±10%;
- **present but trivial**: the interval excludes zero but the point is under ±10%;
- **inconclusive**: ±10% sits inside the interval (the estimate is too wide to separate a real effect
  from a null);
- **precise null**: the interval includes zero and lies entirely within ±10%.

A pre-holdout power check on the exploration set required the left-versus-right interval to be tight
enough to tell a precise null apart from inconclusive before the holdout was read.

## Robustness

The headline was re-checked under two pre-registered forks: locating each video by its **largest**
face instead of requiring a clean single face (so single-face conditioning cannot drive the result),
and raising the face-size floor from 2% to 4%. Both leave the left-versus-right contrast inside the
±10% bar. A leave-one-channel-out pass on the center-involving contrast confirmed the
volume-weighted center dip is a broad weighting difference, not a few outlier channels.

## Reproducing the statistics

The `aggregates/` directory holds the binned tables behind every chart and the headline/fork table,
and `dataset/` holds a de-identified row dataset (channel and video identifiers replaced by keyed
one-way hashes, so the channel-clustering structure survives but no real YouTube identifier is
republished). A replicator can re-run our within-channel zone contrasts, clustering, and bootstrap
and catch a coding error. They **cannot** independently re-verify our face placements or view counts
from the dataset (the real identifiers and thumbnails are not in it): that is what the detector
placement-validation gate is for. The dataset proves the *method*, not the *measurement*. See
`dataset/README.md`.

## Limitations (stated plainly)

- **Observational, not causal.** Where a face sits is a creator's choice; we randomized nothing.
  Within a channel, left and right faced videos still differ in bundled ways (framing, topic, title,
  packaging) we cannot fully separate. Read every result as "associated with," never "causes."
- **A conditional estimand with a selection caveat.** Restricting to single-face videos conditions on
  two creator choices (use a face; use exactly one) that are themselves related to views. The
  dominant-face fork bounds the single-face conditioning; we do not claim it is exactly zero.
- **Sampling frame.** English business and creator-economy channels from our own funnel, skewed
  mid-to-large; specifically, the headline describes channels that vary which side they place the
  face. The finding may not transfer to other languages, niches, or very small channels.
- **Views, not watch-time,** captured once; deleted and private videos are unobservable.
- **Short-form leakage is bounded, not eliminated.** The 90-second floor trims the most likely cases,
  but the duration field alone does not catch every short, so a small residual remains. Because the
  headline is a null, such noise widens the interval rather than manufacturing an effect.
