# Design Spec: Thumbnail Face _Location_ vs. Views

> Written before the pre-registration and before any query relating face location to views. This is
> the locked design for how face _location_ enters the model and how the headline is read off a sealed
> holdout exactly once. Because horizontal position was not captured by the earlier face studies, this
> spec also locks a fresh detection pass (content only, no outcome) and a position-specific detector
> gate that runs first.

## The question

**Among videos that have a single face on the thumbnail, does _where_ the face sits horizontally
(left, center, or right of the frame) relate to more views, within a channel?** Placement advice
circulates among creators, including a side preference (faces on one side of the frame said to beat the
other). This study tests placement on the videos that already use one face. Face presence and face size
are the earlier studies' questions, not this one. The estimand is a within-channel comparison across
horizontal zones, among single-face videos only.

**Prior exposure.** We have studied thumbnail faces before on this dataset: a face being _present_ is
associated with a small within-channel view decline (on the order of 6% fewer), and a _bigger_ face is
associated with no within-channel view change (a precise null). Those are different questions. We have
not computed a within-channel placement comparison on the holdout, and we do not know its answer or
sign. The protection is the sealed holdout, locked before exploration and read once.

## 1. Observational, not experimental

Every finding is correlational. Face placement is one creator choice among many and may travel with
others we do not observe. The within-channel design strengthens the association; it does not make it
causal.

## 2. The predictor: a single face's horizontal zone

- **Single-face thumbnails only.** "Location" is well defined only when there is one face: a panel or
  montage of several faces has no single horizontal position. We therefore restrict to thumbnails with
  **exactly one** detected face clearing the `T = 2.0%` area floor (the same floor the earlier studies
  used to define "a face"). This restriction also removes face count as a confound by construction,
  rather than as an afterthought.
- **Three zones: left, center, right.** The one face is classified by where its bounding-box center
  falls along the frame's width, in equal thirds: **left** for a center in `[0, 1/3)`, **center** for
  `[1/3, 2/3)`, **right** for `[2/3, 1]`. The zone is the predictor. The raw horizontal-center fraction
  is retained for diagnostics and the detector gate.
- **Why a categorical zone, not a continuous position.** The advice is stated in zone terms (a face on
  the left, on the right, centered), and a horizontal axis has no natural "more is better" direction:
  it is a location, not a dose. We report the three zones and the contrasts among them, not a single
  left-to-right slope.

## 3. Unit and clustering

The row is the video; the channel is the resampling unit. Videos in a channel are not independent, so
every interval resamples whole channels (10,000-resample bootstrap, fixed seed), never videos. The
headline is within-channel (channel fixed effects), so it cannot be dismissed as "the channels that
place faces on one side are just bigger."

A within-channel zone comparison only exists for channels whose single-face videos appear in more than
one zone. We require a channel to have single-face videos in **at least two of the three zones, with at
least `MIN_PER_ZONE = 5` single-face videos in each contributing zone**, so that two near-identical
videos cannot inject a high-leverage noise contrast into the bootstrap. For the primary left-vs-right
contrast specifically, a channel contributes only if it has at least five left-zone and at least five
right-zone single-face videos. The channels meeting that left-and-right rule (the headline estimand)
and the channels meeting the any-two-zones rule (the full-curve estimand) are different sets; we lock
and report both counts. The estimand therefore skews toward channels that vary their face placement;
that is a generalization caveat, stated in the headline, not a bias in the contrast.

**Two co-equal estimators, reported side by side.** The within-channel left-vs-right contrast is
reported two ways, and a result is claimed only if both agree in sign and rough size:

- **Median of per-channel contrasts (one channel, one vote).** Compute the left-vs-right view
  difference within each qualifying channel, then report the median across channels. This is immune
  by construction to a few high-volume channels: a channel with 500 single-face videos and a channel
  with 12 each contribute one value to the median. It is the standard-preferred estimator for an
  early study because the resampling unit (the channel) is also the unit of the point estimate.
- **Pooled within-channel fixed-effects contrast.** The pooled coefficient from demeaning
  `log(views)` on the zone dummies inside channels. This is video-count weighted, so high-volume
  channels carry more leverage; we report it as the companion to the median, not as a standalone
  headline, precisely because the single-face count distribution is heavy-tailed (a small number of
  channels have hundreds of single-face videos).

The single-face count distribution is heavy-tailed, so the two can diverge if the high-volume
channels place faces differently from the rest. **A result is claimed only when the median and the
pooled contrast agree in sign and rough size; on divergence we report both and lead with the
smaller-magnitude (more conservative) one**, a rule fixed here rather than chosen after seeing the
pair. We also report the distribution of per-channel zone counts and flag any single channel whose
leverage dominates the pooled contrast.

**Bootstrap procedure (fixed here).** Each of the 10,000 resamples draws whole channels with
replacement and assigns each draw a synthetic cluster id, so a channel drawn twice is two clusters, not
one collapsed group. The channel fixed-effects demeaning is **re-done inside every draw** (each
resampled channel is demeaned by its own resampled mean before the pooled contrast is computed);
demeaning once on the full sample and bootstrapping the already-demeaned rows would treat the
within-channel centering as known and understate the interval. The same whole-channel resample feeds
both estimators (the median-of-per-channel contrast is recomputed inside each draw over the resampled
channels, and the pooled contrast is refit inside each draw), so both carry channel-clustered
intervals.

## 4. The selection caveats (conditioning on creator choices)

The frame is restricted twice, on two creator choices, and we report the result strictly within that
restricted population.

**Conditioning on faced (carried over from the earlier studies).** Putting a face on a thumbnail is a
creator decision tied to things we cannot see (the presence study found a face associated with a small
view decline, so the decision correlates with the outcome). Keeping only faced videos conditions on a
common effect (a "collider") of the packaging decision and the video's unobserved quality, which can
induce a correlation between our predictor and unobserved quality inside the faced subset that is not
there in the full population. The leak scales with how strongly the face decision ties to the outcome,
which the presence study bounds at roughly 6%. A weak decision-to-outcome link can only induce a weak
correlation, so we expect this leak to be small, though we cannot put an exact number on it. That
magnitude bound, not a robustness fork, is the defense, exactly as in the face-size study.

**Conditioning on single-face (new to this study).** Restricting to exactly one face is a second
creator choice, and it is not neutralized by the 6% bound above, because it conditions on a different
thing. "Exactly one face" correlates with format: a solo talking-head or vlog is single-face; a
podcast, panel, collaboration, or montage is multi-face. These formats have different baseline views
_and_ different placement conventions (a solo face is often centered or on a third; a two-up podcast
forces faces toward the edges). So single-face conditioning is conditioning on a format proxy that is
itself related to both the predictor (zone) and the outcome (views). Channel fixed effects absorb the
part of format that is stable within a channel (a podcast channel is mostly two-up), but not the part
that varies within a channel (a channel that mixes solo and panel videos).

We do not wave this away, and we are careful about which of the two below is a _disclosure_ and which
is the actual _bound_:

- **A within-channel single-face-share diagnostic (disclosure, not a bound).** On the exploration data
  (image only, no outcome), we report what fraction of a channel's faced videos are single-face, and
  how that share varies across channels. This tells the reader how much format mixing the fixed effects
  must absorb; it does **not** by itself bound the bias, because a channel can be mostly single-face and
  still carry all of its biasing structure in the within-channel solo-versus-panel split. It is context,
  not a defense.
- **A dominant-face sensitivity fork (the actual bound, locked in §8).** We re-run the headline on faced
  videos using the _largest_ face's zone regardless of how many smaller faces exist. This directly
  attacks the disguised-panel path below: a thumbnail built for two people, with the second face landing
  just under the floor, is treated by its dominant face's position rather than reclassified as a clean
  single-face. If the single-face headline and the dominant-face version **agree in sign and fall within
  `FORK_AGREE_PP = 8` percentage points** of each other, the single-face conditioning is not driving the
  result; if they diverge beyond that, we report both and lead with the smaller-magnitude (more
  conservative) one.

**The specific bias to watch (disguised panels leaking into the edge zones).** The sharpest version of
the format threat is asymmetric: a channel's center-zone single-face videos may be its genuine solo
clips, while some of its edge-zone "single-face" videos are really two-person compositions where the
second face fell just below the floor, leaving the surviving face off-center. Those disguised panels
have different baseline views than true solos, so they can distort the edge zones specifically. This
primarily threatens the center-versus-edge contrast; it threatens the left-versus-right headline only if
the dropped second face is itself left/right asymmetric. The dominant-face fork is aimed at exactly this
path, and the shape guard (§8) watches for a center-versus-edge structure that would betray it.

Net: the estimand is stated strictly conditionally ("among single-face thumbnails a creator chose to
use"), never as "moving a face changes views"; the faced collider is bounded small by the presence
study's ~6% edge; the single-face conditioning's residual is bounded by the dominant-face fork's measured
agreement (the share diagnostic only discloses the magnitude of mixing, it does not bound it).

## 5. Outcome: views, age-controlled

The cohort sits on a single measurement clock (almost the entire set captured within one short window),
so a video's age is effectively _when we measured it_, and raw views-per-day would smuggle measurement
timing into the outcome. The headline outcome is therefore `log(views)` controlling for `log(age)`,
inside channel fixed effects. This is algebraically the same model as an age-controlled
views-per-day regression (moving from views-per-day to views shifts only the age coefficient by 1,
leaving the zone contrasts identical), with the timing artifact moved out. We report the raw
views-per-day version alongside as a cross-check.

Because the capture date is nearly fixed, a video's age is essentially a relabeling of its publish
date, so `log(age)` here doubles as a publish-era axis: controlling for it removes the timing artifact
and absorbs a chunk of era confounding for free. The flip side is that the publish-year sensitivity
check is not a fully independent control (it leans on the same axis the headline already controls); we
report it as corroboration, not as orthogonal evidence.

Limitations: views, not watch-time; a single capture; deleted and private videos are unobservable
(survivorship). The within-channel version of the survivorship threat is the one that bites: if a
creator deletes its weak edge-zone videos more readily than its weak center-zone videos (or vice
versa), the surviving zones are differentially upward-selected and the contrast inherits a
survivorship gradient that channel fixed effects cannot remove, because it lives in the within-channel
residual. We report the deletion/private rate by zone where it is observable and otherwise state it as
an unfixable limitation, named at the within-channel level, not just as an aggregate rate. We work in
medians and log space because views are outlier-dominated.

## 6. Confounders

| Variable | Role | Control? |
| --- | --- | --- |
| Channel size / niche / audience | confounder | Absorbed by channel fixed effects. |
| Video age / measurement clock | artifact | Removed by the explicit `log(age)` term (§5). |
| Era / publish period | confounder | Largely absorbed by `log(age)` (under the single clock, age relabels publish date). Publish-year sensitivity as corroboration, not an independent check (§5). |
| Face count / multi-face format | selection, addressed by design | Removed by the single-face restriction (§2); the residual within-channel format mixing is bounded by the §4 diagnostic and the dominant-face fork. |
| Video length / format | weak proxy only | Sensitivity covariate, with and without. A check, not a fix. |
| Face size | possible covariate | A face's size and its placement can co-vary (a big centered face vs. a small cornered one). Reported with and without a face-size covariate; the size study found size itself a null, so this is a check, not an expected mover. |
| **Other packaging choices** (framing, topic, style, eye-gaze direction) | **core residual confounder** | Placement may travel with other choices in the thumbnail and the video that we do not observe. We cannot separate them, so we cannot rule them out. Not removable by design; the headline's stated limitation. |
| Click-through rate | mediator | No (on the causal path from placement to views). |
| Recommendation-shelf placement | collider | Never (controlling it manufactures correlation). |

**The residual-confounder limitation.** Face placement is one of many choices a creator makes about a
thumbnail and a video, and those choices can move together. We do not measure framing, topic, gaze
direction, or any other packaging variable, so when placement differs and views move, we cannot say the
placement did it rather than something that traveled with it. Channel fixed effects remove whatever is
stable about a channel, but not variation in these choices within a channel. We claim only the
within-channel association, never the causal effect of moving a face.

## 7. Detector gate (before any outcome number)

The detector and cohort are shared with the earlier face studies, whose presence and count gate already
passed on a hand-labeled sample of about 1,100 thumbnails (high precision and recall, with no miss-rate
trend across the views range). That carries over. What is new, and not yet run, is the position-specific
gate, because the predictor is the face's horizontal _position_:

- **(a) Position-dependent center error (the zone-killer).** The danger is a detector that systematically
  mislocates faces along the horizontal axis, for example pulling edge faces toward the center, which
  would misclassify true-left and true-right faces as center and contaminate the zone contrasts. Test:
  on a hand-labeled sample, regress the detector's horizontal-center error (`detected_center −
  true_center`, as a fraction of frame width) on the true center; a center-pulling bias shows as a
  nonzero slope. **Pre-committed bound: `|slope| ≤ 0.05` frame-widths**, which caps zone
  misclassification near the one-third and two-thirds boundaries to a narrow marginal band. The gate is
  on the point estimate of the slope, not on significance (a few hundred labeled faces can pass an
  underpowered significance test with a real bias present); we report the interval for transparency but
  do not gate on it. Unlike a face-_area_ error, a face-_center_ is something a human can mark cheaply
  and reliably, so this numeric check is feasible and is the primary gate, not a fallback.
- **(b) Zone-dependent miss rate (selection on the predictor).** If the detector misses faces more often
  at one end of the frame (edge faces are more often cropped), those videos leave the single-face set or
  get miscounted, which selects on the predictor rather than mismeasuring it. Test: recall stratified by
  true zone on the labeled set. **Pre-committed bound: each zone's recall within `ZONE_RECALL_TOL =
  0.05` of the overall recall.** Evenness across zones is the load-bearing property.
- **(c) Non-constant center-error spread (errors-in-variables).** Beyond the mean bias in (a), check
  whether the _spread_ of the horizontal-center error grows toward the frame edges (edge faces are
  partially cropped, lower-confidence, and their boxes are less stable). Rising error spread in the
  outer thirds does two things the mean-slope check in (a) cannot catch: it blurs zone membership near
  the one-third and two-thirds cuts even with zero mean bias, and it differentially attenuates the
  left-vs-right contrast toward zero (classic errors-in-variables attenuation), biasing the study
  toward a false null. Test: compare the center-error standard deviation in the outer thirds to the
  center third on the labeled sample; report it, and if the outer-third spread is materially larger,
  fall back to faces whose box is fully inside the frame (where the center is most trustworthy).
- **(d) Style scope.** The named worry is illustrated faces, whose box convention may differ from
  photographic faces. State the per-style cell sizes; if a style cell holds fewer than 30 labeled faces,
  fall back to photographic-only rather than lean on a low-power pass.

On any failure: narrow to where the position estimate is even (for example, photographic only, or faces
whose box is fully inside the frame so the center is most trustworthy) or report inconclusive. No
placement-vs-views number is computed before (a) through (d) pass.

**Escalation rule.** A stratified visual spot-check of detector boxes across the frame suffices for a
null, trivial, or inconclusive holdout. If the holdout comes back present and meaningful, the numeric
center-error bound (a) and the zone-stratified recall (b) become load-bearing for a positive
"placement matters" claim, and publication then requires both. This is committed now, before the
holdout read, so it cannot be a post-hoc dodge either way.

## 8. Thresholds and decision rules (fixed in advance)

At this scale everything is statistically significant, so the threshold is the study.

- **Headline = the within-channel, age-controlled left-vs-right view difference** among single-face
  videos, expressed as a percent (`100·(exp(contrast) − 1)`), with a whole-channel bootstrap 95%
  interval. It is reported as **two co-equal estimators** (the median of per-channel contrasts and the
  pooled fixed-effects contrast, §3); a result is claimed only when they agree in sign and rough size,
  and on divergence we lead with the more conservative one. The left-vs-center and center-vs-right
  contrasts and the full left/center/right curve are reported alongside.
- **Decision rule:** (a) the 95% interval excludes 0, AND (b) the point reaches **±10%** (left vs. right)
  to count as meaningful. At this N, (a) is near-automatic, so (b) is the operative bar. "Present but
  trivial" = real but under ±10%. "Precise null" = the interval includes 0 but lies entirely within
  ±10%, so the design has bounded any placement effect under the practical bar. "Inconclusive" = ±10%
  sits inside the interval (a power failure), kept rigidly distinct from a precise null.
- **±10% is a step contrast, not a rate.** It is the whole left-versus-right difference, the same kind
  of bar as the face-presence study's presence step. (The face-size study's ±10% was per _doubling_ of
  a continuous dose, a different and rate-shaped bar; the matching number does not imply a matching kind
  of stringency. We state this so the two are not conflated.)
- **No left-to-right trend test.** Left, center, and right are a spatial classification, not an ordered
  dose, so a monotone-trend test across them would test a hypothesis we do not hold. The result is the
  set of zone contrasts (primarily left vs. right), not a single ordered slope. This is a design choice,
  stated plainly, not a claim about the world.
- **Shape guard (mechanical trigger, fixed in advance).** A single left-vs-right number can hide
  structure the center zone carries (left and right roughly equal while center stands clearly apart, or
  one side an outlier while the other matches center). To keep this from being a judgment call applied
  after seeing the curve, the trigger is mechanical: compute all three pairwise zone contrasts
  (left-vs-right, left-vs-center, center-vs-right) with their whole-channel bootstrap intervals. **The
  guard fires if a center-involving contrast (left-vs-center or center-vs-right) is real by its own
  interval AND larger in magnitude than the left-vs-right contrast** (that is, the largest real pairwise
  gap is not the left-vs-right one). On a trigger the result is reported as "shape, see the curve" with
  all three contrasts and their intervals shown, not led by the single left-vs-right number. The guard
  is confirmatory: it can change how the result is led, and it appears in "what has to be true to
  publish." It also catches the disguised-panel path (§4), which would surface as a center-vs-edge
  structure.

The headline is the **left-vs-right** within-channel contrast, fixed in advance. The analyses that can
change or veto how it is reported, all computed on the holdout: the left-vs-right contrast itself; the
`T = 4%` floor-sensitivity fork (which also tests the single-face definition, since raising the floor
reclassifies marginal tiny second faces); the dominant-face fork (§4); and the shape guard.

Everything else is descriptive, FDR-corrected at `q = 0.10` within its family, logged in the run
registry, and cannot veto the headline: the length covariate on and off, the face-size covariate on and
off, short and long split, publish-year sensitivity, per-niche and per-channel-size spread, winsorize
and drop-top-1%.

## 9. Locked holdout

The headline is read off the fixed whole-channel holdout established by the earlier face studies: a
channel is held out if a deterministic function of its id places it in the held-out fifth, AND it was
not examined in any prior work (prior-seen channels forced to exploration). This deterministic rule
regenerates the split. It is whole-channel (a channel is entirely in or entirely out), so a
within-channel contrast on the holdout is never contaminated by exploration rows from the same channel.
All exploration, the detector gate, and threshold confirmation run on the exploration side; the headline
contrast is computed on the holdout once, with the analysis code fixed in advance.

**A disclosed power constraint (stated now, not discovered after the read).** The headline identifying
set is small by construction: a channel contributes to the left-vs-right contrast only if it has at
least five single-face videos in the left zone AND at least five in the right zone, which is impossible
below ten single-face videos and, because the center zone absorbs a large share of single-face
thumbnails, typically needs well more than ten. On the held-out channels, only the subset with enough
left-and-right single-face videos enters the headline, a set materially smaller than the per-doubling
size study's. The predictor is also coarser (a three-level zone rather than a continuous dose) and only
the left and right videos of a contributing channel feed the contrast (center videos do not). All three
push the interval wider than the size study's. We expect a wider interval and possibly an inconclusive
headline; this is a known constraint of the design, not a surprise.

**Power go/no-go before the holdout read (locked threshold).** On the exploration set we first compute
the left-vs-right contrast's bootstrap interval half-width. **If that half-width exceeds
`POWER_HALFWIDTH_MAX = 7%` (the locked threshold), we declare the study underpowered to separate a
precise null from inconclusive on the single contrast, and we lead with the full left/center/right curve
and the bounded-magnitude statement rather than presenting any single contrast as confirmatory.** This
branch is the pre-committed expected fallback, fixed here so an inconclusive outcome reads as "we said
this might happen," not as improvisation after the read. The threshold is a reporting rule, not a reason
to re-spec the headline.

## 10. Generalization

The frame is English-language business and creator-economy YouTube channels from our own funnel, skewed
mid-to-large: not a random sample of YouTube. The contrasts are reported as a spread across niches
(median and range), not one blended number. The zone distribution is reported for all single-face videos
and for the within-channel-varying subset that drives the contrasts, so the disclosed representativeness
matches the estimand.

## 11. Data release (de-identified, ToS-compliant)

Per the lab protocol: a de-identified row dataset (one-way keyed hashes of channel and video id, secret
salt never published; no raw ids, titles, or thumbnails), the binned aggregate tables, and a replication
notebook. The released rows carry the horizontal zone and the raw horizontal-center fraction so the
placement comparison is replicable; both are derived from the image, not from any YouTube id, so they
carry no re-identification path. De-identification exists to satisfy the YouTube API terms (no
republishing data keyed to real ids).

## Sign-off

- [x] Within-channel association; prior exposure stated once
- [x] Unit (video) + clustering (channel as resampling unit, whole-channel bootstrap)
- [x] Predictor = single-face horizontal zone (left/center/right), `T = 2.0%` floor
- [x] Estimand = within-channel zone comparison; "who it's about" = channels varying placement, ≥5 per contributing zone
- [x] Two co-equal estimators (median-of-per-channel + pooled FE), claim only on agreement; heavy-tail leverage addressed
- [x] Outcome = age-controlled `log(views)`, single-clock artifact removed
- [x] Selection caveats classified: faced collider (bounded ~6%) + single-face conditioning (bounded by dominant-face fork; share diagnostic is disclosure not bound); disguised-panel path named
- [x] Confounders classified; unobserved packaging choices named as the unremovable residual; within-channel survivorship named
- [x] Position-specific detector gate locked (center-error slope, zone-recall evenness, error-spread) before any outcome number
- [x] Threshold = ±10% left-vs-right step; real/trivial/precise-null/inconclusive rule; no trend test; mechanical shape guard; locked power go/no-go; analyses that can veto the headline named
- [x] Locked whole-channel holdout, read once; consistency-check residual named; ToS-checked release
- [ ] → ready to write the pre-registration, tagged before touching placement-vs-views
