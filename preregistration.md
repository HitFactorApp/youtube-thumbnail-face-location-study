# Pre-registration: Thumbnail Face _Location_ vs. Views

> The analysis plan, fixed and publicly tagged (`004-v0-preregistration`) before any query relating
> face location to views, and before the detection pass that records face position. The tagged commit
> is the audit trail. Deviations after this point are allowed but logged in Transparent Changes at the
> bottom.

**Status:** DRAFT (becomes PRE-REGISTERED on the tagged commit)
**Design:** [design-spec.md](./design-spec.md)

## Prior exposure

We have studied thumbnail faces before on this dataset: a face being _present_ is associated with a
small within-channel view decline (on the order of 6% fewer), and a _bigger_ face is associated with no
within-channel view change (a precise null). This study asks the distinct placement question among
single-face videos. We have not computed a within-channel placement comparison on the holdout and do not
know its answer or sign. We pre-register to stop ourselves searching specifications after seeing the
outcome; the real protection is the sealed holdout.

## Could the predictor be circular?

No. Face location is the detector's bounding-box center, read from the thumbnail image. No view,
watch-time, or engagement data feeds it at any stage. The risk is not circularity but a biased detector:
a position-dependent center error (the detector mislocating faces along the horizontal axis) and a
zone-dependent miss rate. Both are addressed by the detector gate below. A separate provenance risk,
that a creator swapped the thumbnail since the earlier studies measured it, is handled by the
consistency check below.

## Sample conditioning (two colliders, reported conditionally)

The sample is conditioned twice, on two creator choices, and the result is reported strictly within that
population.

1. **Faced.** Videos a creator chose to put a face on. Keeping only faced videos conditions on a common
   effect of the packaging decision and unobserved quality, which can induce a correlation between the
   predictor and unobserved quality inside the faced subset. The leak scales with how strongly the face
   decision ties to the outcome, which the presence study bounds at roughly 6%. We expect it small;
   that magnitude bound, not a fork, is the defense.
2. **Single-face.** Videos with exactly one face clearing the floor. This conditions on a format proxy
   (solo content is single-face; podcasts and panels are multi-face) that is itself related to both
   zone and views, and it is NOT covered by the 6% bound above. Channel fixed effects absorb the stable
   part of format; the within-channel residual is bounded by the dominant-face fork below. The
   single-face-share diagnostic only _discloses_ how much format mixing the fixed effects must absorb;
   it is not itself a bound. The sharpest threat is asymmetric: a channel's edge-zone "single-face"
   videos may be disguised panels (a second face fell just under the floor, leaving the surviving face
   off-center), which have different baseline views than true solos and distort the edge zones. That
   primarily threatens center-vs-edge and threatens left-vs-right only if the dropped face is itself
   left/right asymmetric; the dominant-face fork is aimed at this path and the shape guard watches for
   it.

Consequences, fixed here:

- **The estimand is conditional.** "Among single-face thumbnails a creator chose to use, does horizontal
  placement relate to views," NOT "does moving a face change views," and NOT informative about the
  no-face to face margin or the one-face to many-face margin.
- **Floor-sensitivity fork (guards the floor and the single-face definition):** re-run the headline at
  `T = 4%`. The videos nearest the 2% floor are where measurement is shakiest, and raising the floor
  also reclassifies marginal tiny second faces (so a one-big-plus-one-tiny thumbnail becomes
  single-face), so this fork tests both the small-area floor artifact and the stability of the
  single-face definition.
- **Dominant-face fork (bounds the single-face conditioning):** re-run the headline on faced videos
  using the largest face's zone regardless of how many smaller faces exist. **Agreement in sign and
  within `FORK_AGREE_PP = 8` percentage points** means single-face conditioning is not driving the
  result; divergence beyond that means report both and lead with the conservative (smaller-magnitude)
  one. The tolerance is locked here rather than judged after seeing the pair.

## Hypotheses

- **H1 (primary):** Among single-face videos (exactly one face clearing `T = 2.0%`), within the same
  channel and controlling for age, a face in the **left** zone is associated with a **different** number
  of views than a face in the **right** zone. The primary contrast is left vs. right. We register it
  two-sided because the side-preference advice points both ways depending on who is giving it, and we
  hold no private directional prior.
- **Secondary (reported, not the designated primary):** the left-vs-center and center-vs-right
  contrasts, and the full left/center/right curve.
- **Exploratory:** whether the contrasts differ by niche, channel size, and short vs. longer videos;
  whether they survive a length covariate and a face-size covariate.

## Primary analysis

- **Frame:** single-face videos (`exactly one detected face with area ≥ 2.0%`, `views ≥ 10`, `age ≥ 90
  days`) on the study's frozen data store, restricted to thumbnails whose image is unchanged since the
  earlier studies measured it (the consistency check below). The cohort is English-language
  business/creator-economy YouTube channels with at least 20 qualifying videos, public, organic, and
  long-form, published 90 days to 5 years ago.
- **Locked inclusion constants:** `T_PRIMARY = 2.0`, `VIEW_FLOOR = 10`, `MIN_AGE_DAYS = 90`,
  `ZONE_CUTS = (1/3, 2/3)` (the left/center/right boundaries on horizontal-center fraction), and
  `MIN_PER_ZONE = 5` (a channel needs at least five single-face videos in each contributing zone; for
  the left-vs-right headline, at least five left and at least five right). `MIN_PER_ZONE = 5` leaves
  well over the 30-channel bootstrap-stability floor.
- **Locked analysis constants:** `BOOTSTRAP_N = 10_000`, `BOOTSTRAP_SEED = 0`,
  `MEANINGFUL_PCT_STEP = 10.0` (the practical bar, a step), `FORK_AGREE_PP = 8` (dominant-face fork
  agreement tolerance), `POWER_HALFWIDTH_MAX = 7.0` (the power go/no-go threshold),
  `CENTERX_ERR_SLOPE_BOUND = 0.05`, `ZONE_RECALL_TOL = 0.05`, `T_FLOOR_FORK = 4.0`.
- **Unit:** the video, grouped by channel for every uncertainty calculation.
- **Predictor:** the horizontal zone (left, center, right) of the single face. **Outcome:** `y =
  ln(views)`.
- **Model:** within-channel (channel fixed effects) of `y ~ zone + ln(age)` on single-face videos. The
  headline is the within-channel left-vs-right contrast, holding `ln(age)` fixed, expressed as a percent
  (`100·(exp(contrast) − 1)`). Restricted to channels meeting `MIN_PER_ZONE` in the relevant zones. Both
  the headline channel-set (left and right both present) and the full-curve channel-set (any two zones
  present) are reported with their counts.
- **Two co-equal estimators (a result is claimed only if both agree).** The contrast is computed two
  ways: (i) the **median of per-channel left-vs-right contrasts** (one channel, one vote; immune to a
  few high-volume channels, since the single-face count distribution is heavy-tailed), and (ii) the
  **pooled fixed-effects contrast** (video-count weighted). A result is claimed only when the two agree
  in sign and rough size; on divergence we report both and lead with the smaller-magnitude (more
  conservative) one. This is fixed here, not chosen after seeing the pair.
- **Headline number:** the left-vs-right percent difference (both estimators), with a whole-channel
  bootstrap 95% interval (10,000 resamples, fixed seed = 0, synthetic cluster ids so a channel drawn
  twice stays two clusters). The same whole-channel resample feeds both estimators, and the channel
  demeaning is re-done inside every resample (each drawn channel demeaned by its own resampled mean);
  demeaning once and bootstrapping demeaned rows would understate the interval. We report the
  per-channel zone-count distribution and flag any single channel whose leverage dominates the pooled
  contrast. Computed on the holdout once; the exploration-set value is the confirming companion. Before
  the holdout read, we check the exploration-set interval half-width against the locked
  `POWER_HALFWIDTH_MAX = 7%` and, if it exceeds that, declare the study underpowered to separate a
  precise null from inconclusive and lead with the full curve and a bounded-magnitude statement rather
  than any single contrast (the pre-committed fallback). The headline identifying set is small by
  construction (a channel needs five left AND five right single-face videos, impossible below ten and
  usually needing well more since the center zone absorbs a large share), the predictor is coarser than
  a continuous dose, and only a channel's left and right videos feed the contrast, so a wider interval
  than the size study's, and a possibly inconclusive headline, is an expected and disclosed constraint,
  not a surprise.
- **Confounder handling:** channel fixed effects absorb channel size, niche, audience, and whatever is
  stable about a channel's packaging and format; the `ln(age)` term removes the single-clock artifact
  and, because age relabels publish date under the single clock, also absorbs much of publish-era
  confounding (so the publish-year sensitivity is corroboration, not an independent check). We do not
  control click-through rate (mediator) or shelf placement (collider). Other packaging choices that may
  travel with placement, and that we do not observe (framing, topic, gaze direction), are a stated,
  unremovable residual confounder.

## Detection pass and the thumbnail-consistency check (before any outcome number)

Horizontal position was not recorded by the earlier studies, so this study runs a fresh detection pass
over the faced thumbnails to record each face's bounding-box position, using the same detection model
and configuration the earlier studies validated. The pass covers every thumbnail with a detected face
down to a 0.5% area floor (wider than the 2% analysis floor), so that the floor-sensitivity checks at
1%, 2%, and 4% are all recomputable from one pass without re-detecting; the analysis itself still uses
the locked `T_PRIMARY = 2.0%` floor. This pass touches images only, no outcome data.

Because creators can swap a thumbnail after publication, a re-fetched image may differ from the one the
earlier studies measured, which would break the link between the earlier faced flag and the new
position. The consistency check: recompute each face's area in this pass and compare the largest-face
area to the earlier study's stored value per video; if they diverge beyond a small tolerance, or the
detection count changes, or the thumbnail no longer loads, the video is flagged as changed and excluded
from the location analysis. The placement therefore always describes the same image the earlier studies
measured. The excluded count is reported as a provenance fact. One residual: the check verifies area and
count stability, so an area-preserving _horizontal reposition_ (a re-crop that keeps the largest-face
area but moves it) would pass undetected and change the zone. We accept this as a low-frequency residual
(area-preserving repositions are rare) rather than assume it away.

## Analyses fixed in advance

The headline is the **left-vs-right** within-channel contrast. These are fixed before the holdout is
touched; each can change or veto how the headline is reported, and each appears in "what has to be true
to publish":

1. **The left-vs-right contrast (headline):** `ln(views) ~ zone + ln(age)`, channel FE, single-face
   videos, expressed as a percent with a whole-channel bootstrap interval.
2. **Floor-sensitivity fork at `T = 4%`** (Sample conditioning above): tests both the small-area floor
   artifact and the single-face definition's stability.
3. **Dominant-face fork:** the headline re-run using the largest face's zone among faced videos
   regardless of count; agreement confirms the single-face conditioning is not driving the result.
4. **Shape guard (mechanical trigger):** compute all three pairwise zone contrasts (left-vs-right,
   left-vs-center, center-vs-right) with their whole-channel bootstrap intervals. The guard fires if a
   center-involving contrast (left-vs-center or center-vs-right) is real by its own interval AND larger
   in magnitude than the left-vs-right contrast (the largest real pairwise gap is not the left-vs-right
   one). On a trigger the result is reported as "shape, see the curve" with all three contrasts shown,
   not led by the single left-vs-right number. This is a mechanical pass/fail, not a judgment call made
   after seeing the curve, and it also catches the disguised-panel path (which surfaces as a
   center-vs-edge structure).

**No left-to-right trend test.** Left, center, right is a spatial classification, not an ordered dose,
so a monotone-trend test is not run; the result is the zone contrasts, primarily left vs. right.

Everything else is descriptive: FDR-corrected at `q = 0.10` within its family, cannot veto the headline,
logged in the lab-wide run registry.

- **Length covariate / short-long split:** the headline with and without `ln(duration)`, reported
  separately for short (≤ ~3 min) and longer videos.
- **Face-size covariate:** the headline with and without `ln(largest_face_area_pct)`, since size and
  placement can co-vary; the size study found size itself a null, so this is a check, not an expected
  mover.
- **Per-niche / per-channel-size spread:** the contrasts across niches and size bands (median + range).
- **Outlier legs:** winsorize at the 99th percentile and drop the top 1% of channels.

## Outlier handling

We work in log space (outlier-robust by construction) and report the headline three ways (plain log,
winsorized at the 99th percentile of views, top-1%-of-channels dropped), requiring agreement in sign and
rough size. Viral videos are signal and never dropped on outlier grounds; the only rows removed are
sub-floor view drops, the non-single-face and Shorts exclusions defined by the cohort and the
single-face restriction, and thumbnails flagged changed by the consistency check.

## Practical-significance thresholds

- **Decision rule:** (a) the 95% interval excludes 0, AND (b) the point reaches ±10% (left vs. right).
  At this N (a) is near-automatic, so (b) is the operative bar. "Present but trivial" = real but under
  ±10%. "Precise null" = the interval includes 0 but lies entirely within ±10% (any placement effect
  bounded under the practical bar). "Inconclusive" = ±10% falls inside the interval (a power failure,
  reported as inconclusive, never as a null and never as placement support).
- **±10% is a step contrast, not a rate.** It is the whole left-versus-right difference, the same kind
  of bar as the face-presence study's presence step. The face-size study's ±10% was per doubling of a
  continuous dose, a rate-shaped bar; the matching number does not imply matching stringency. We state
  this so the two are not conflated.
- **Shape guard (mechanical):** the headline is reported as "shape, see the curve" if a center-involving
  pairwise contrast (left-vs-center or center-vs-right) is real by its own interval AND larger in
  magnitude than the left-vs-right contrast. A mechanical pass/fail, not a post-hoc judgment.
- **Power go/no-go:** if the exploration-set left-vs-right half-width exceeds `POWER_HALFWIDTH_MAX = 7%`,
  the study is declared underpowered to separate a precise null from inconclusive on the single contrast,
  and the result is led with the full curve and a bounded-magnitude statement (the pre-committed
  fallback), not any single contrast.

## Locked holdout

- The fixed whole-channel holdout established by the earlier face studies: a channel is held out if a
  deterministic function of its id places it in the held-out fifth, AND it was not examined in any prior
  work (prior-seen channels forced to exploration). This deterministic rule regenerates the split, which
  is whole-channel and locked before any exploration.
- All exploration, the detection pass, the detector gate, and the consistency check run on the
  exploration side; the headline contrast is read off the holdout once, with the analysis code fixed in
  advance.

## Detector gate (before any placement-vs-views number)

The detector and cohort are shared with the earlier face studies, whose presence and count gate passed
on about 1,100 hand-labeled thumbnails (high precision and recall, no miss-rate trend with views). That
carries over. The new, not-yet-run part is the position gate:

1. **Position-dependent center error (the zone-killer).** Regress the detector's horizontal-center error
   (`detected_center − true_center`, as a fraction of frame width) on the true center on a hand-labeled
   sample; a center-pulling bias shows as a nonzero slope. **Pre-committed bound: `|slope| ≤ 0.05`
   frame-widths**, gated on the point estimate, not significance; the interval is reported for
   transparency but does not gate. A face center is cheap and reliable to hand-mark, so this numeric
   check is feasible and is the primary gate.
2. **Zone-dependent miss rate.** Recall stratified by true zone; **pre-committed bound: each zone's
   recall within `0.05` of the overall recall.** Evenness across zones is the load-bearing property.
3. **Non-constant center-error spread (errors-in-variables).** Whether the _spread_ of the
   horizontal-center error grows toward the frame edges; rising spread in the outer thirds attenuates the
   left-vs-right contrast toward a false null even with zero mean bias, and the mean-slope check (1) does
   not catch it. Compare the center-error standard deviation in the outer thirds to the center third;
   report it, and on a materially larger outer-third spread fall back to faces whose box is fully inside
   the frame.
4. **Style scope.** State the per-style cell sizes; if a style cell holds fewer than 30 labeled faces,
   fall back to photographic-only rather than lean on a low-power pass.
5. **On any failure:** narrow to where the position estimate is even (photographic only, or faces whose
   box is fully inside the frame), or report inconclusive, before any outcome number.
6. **Escalation:** a stratified visual spot-check suffices for a null, trivial, or inconclusive holdout;
   a present-and-meaningful holdout makes (1), (2), and (3) load-bearing for a positive claim, and
   publication then requires all three. Committed now, before the holdout read.

## Statistical stance

At our scale statistical significance is automatic, so we de-emphasize p-values and report effect sizes,
channel-clustered intervals, and the practical thresholds above. Every relationship is correlational;
the data is observational. The headline describes only single-face channels with within-channel
placement variation; it does not generalize to channels whose single-face videos are all in one zone,
nor to multi-face or no-face videos.

## Planned robustness checks

- [ ] Position detector gate passed (center-error slope within ±0.05; zone recall even within ±0.05; outer-third error spread not materially larger) before any of the below.
- [ ] Thumbnail-consistency check run; changed thumbnails excluded; excluded count reported.
- [ ] Power go/no-go: exploration-set half-width checked against `POWER_HALFWIDTH_MAX = 7%`; underpowered branch taken if exceeded.
- [ ] Both estimators (median-of-per-channel and pooled FE) agree in sign and rough size; divergence ⇒ report both, lead with the conservative one.
- [ ] Headline holds under alternative minimum-age cutoffs (30 / 90 / 180 days).
- [ ] Headline holds winsorized at the 99th percentile and dropping the top 1% of channels.
- [ ] Holdout replicates the exploration-set left-vs-right contrast (sign + rough size).
- [ ] Shape guard checked (mechanical trigger): no center-involving contrast is real and larger than left-vs-right (else report as shape).
- [ ] Floor-sensitivity: contrast stable between `T = 2%` and `T = 4%`.
- [ ] Dominant-face fork agrees in sign + within `FORK_AGREE_PP = 8` points with the single-face headline; divergence ⇒ report both, lead with the conservative one.
- [ ] Deletion/private rate by zone reported where observable, else flagged as unfixable within-channel survivorship limitation.
- [ ] Length covariate and face-size covariate on/off agree; short vs. long reported separately.
- [ ] Naive (raw views-per-day) and age-controlled `log(views)` reported side by side.
- [ ] Subgroup direction holds in a majority of niche/size tiers; a flip is reported, not suppressed.
- [ ] `verdict()` forks the corrected (post-fix) version: precise-null and inconclusive kept rigidly distinct; both ±bar checks two-sided.

## What has to be true to publish

The left-vs-right contrast is publishable as a positive ("placement matters") finding only if the whole
confirmatory bundle holds: the holdout contrast meets the decision rule on **both** estimators
(median-of-per-channel and pooled FE agree in sign and rough size); the half-width cleared the
`POWER_HALFWIDTH_MAX = 7%` go/no-go (else the curve-fallback branch is taken); the dominant-face fork
agrees in sign and within `FORK_AGREE_PP = 8` points; the `T = 4%` fork does not overturn it; the
mechanical shape guard did not fire; it holds under at least two alternative specifications and within
major subgroups; and the position detector gate passed (with the numeric center-error bound, zone-recall
evenness, and error-spread check load-bearing under the escalation rule). A contrast that flips, sits
under the ±10% bar, or comes back inconclusive is reported as exactly that, including the genuinely
interesting "we found no side preference," which is itself a publishable correction to the advice. A
reported null and a reported inconclusive are kept rigidly distinct: a private prior is never allowed to
round an underpowered inconclusive up to a satisfying null.

---

## Transparent Changes (post-registration deviations)

> A pre-registration is a plan, not a prison. Every change after the tag is logged here with date and
> reason. An empty section means none.

_None yet._
