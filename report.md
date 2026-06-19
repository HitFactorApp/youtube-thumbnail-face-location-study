# Does Where the Face Sits on the Thumbnail Get More Views?

A common piece of thumbnail advice is about placement: put the face on the **left** (or the right),
not dead center, and the views follow. We tested it on the videos that have exactly one face.
Classifying that face as **left, center, or right** of frame and comparing each channel's videos
against its own, holding age constant, the **left-vs-right** difference in views is **essentially
zero**: a held-out **−0.3%** with the pooled estimator and **+0.1%** with the one-channel-one-vote
estimator (95% intervals −5.4% to +5.2% and −4.1% to +6.0%). Both intervals sit inside the ±10% we set
in advance as worth acting on, and both straddle zero. This is a precise null, not "we couldn't tell":
within a channel, which side the face is on does not move views.

This is the third in a thumbnail-face line. The first asked whether to use a face at all (a face is
associated with slightly *fewer* within-channel views). The second asked whether a bigger face helps
(no measurable change). This one asks where the face goes. The answer, for the left-vs-right placement
the advice is actually about, is the same: it doesn't matter.

## What we tested, precisely

The question is conditional and we keep it that way: **among videos a creator already chose to put a
single face on, does the face's horizontal placement relate to views, within the same channel?** It is
not "does moving a face help" (we cannot run that experiment), and we restrict to single-face
thumbnails because "location" is only well defined when there is one face.

- **Predictor:** the single detected face's horizontal position, the box center as a fraction of frame
  width, classified into three zones: left `[0, 1/3)`, center `[1/3, 2/3)`, right `[2/3, 1]`.
- **Comparison:** each channel against itself (channel fixed effects), controlling for video age. This
  removes channel size, niche, audience, and anything else stable about a channel, so the result cannot
  be "big channels just frame faces differently."
- **Outcome:** views at a single capture, age-controlled. The cohort sits on one measurement clock, so
  raw views-per-day would smuggle measurement timing into the result; controlling for age removes that
  (see [methodology.md](./methodology.md)).
- **Two co-equal estimators.** We compute every contrast two ways and claim a result only when both
  agree: the **median of per-channel contrasts** (one channel, one vote, immune to a few high-volume
  channels) and the **pooled fixed-effects contrast** (video-count weighted). This was fixed in the
  plan, not chosen after seeing the numbers.
- **Who it describes:** channels with at least five left and five right single-face videos (381
  channels, 16,561 videos on the held-out set). A within-channel left-vs-right contrast only exists for
  channels that vary which side they place the face, so the finding is about side-varying channels, not
  all of YouTube.

The analysis plan was registered and publicly timestamped before any placement-vs-views number was
queried (`004-v0.1-preregistration`), and the headline was read once off a sealed 20% holdout that was
never touched while building the analysis. The full plan is in
[preregistration.md](./preregistration.md).

## Key findings

1. **Left vs right is a precise null.** Held-out estimate **−0.3%** (pooled, 95% CI −5.4% to +5.2%) and
   **+0.1%** (median-per-channel, 95% CI −4.1% to +6.0%). Both intervals clear ±10% in both directions
   and include zero. The two estimators land on opposite sides of zero by 0.4 of a point, which is what
   two noisy estimates of a true zero look like, not a disagreement: both reach the same verdict, no
   left-vs-right effect of practical size.

2. **The forks agree.** Locating each video by the **largest** face instead of requiring a clean single
   face (so single-face conditioning cannot drive the result) gives +1.7% pooled / +3.4% median, both
   intervals crossing zero. Raising the face-size floor from 2% to 4% of the frame gives +3.4% pooled /
   +6.8% median, again crossing zero. Every fork stays inside the ±10% bar; left-vs-right does not move.

3. **A zone spread appears, but only in the volume-weighted view, so we claim none of it.**
   Placing all three zones side by side, relative to the typical placement on the same channel and
   counting every video equally, the pooled estimator shows a spread: left at +4.0% (interval excluding
   zero), center at −5.7% (excluding zero), right at +2.1% (crossing zero), so left up, center down,
   with even a small left-over-right edge. The one-channel-one-vote estimator does **not** reproduce any
   of it: left, center, and right all land within half a point of each other (−0.3% / −0.3% / +0.4%).
   Under our locked rule that a result is claimed only when both estimators agree, none of this is a
   finding, not the center dip and not the left edge. It is reported as unresolved shape. This is also
   why the left-vs-right headline is null: the apparent edge is a weighting artifact the one-vote count
   erases. See the next section. (This level view is computed on the broader set of channels that span
   at least two zones, 717 channels, so its center gap is smaller than the −15.7% pairwise left-vs-center
   contrast below, which uses only the channels that vary across that one pair.)

## The shape guard fired, and the center dip is reported, not claimed

Our plan included a mechanical shape guard: compute all three pairwise zone contrasts, and if a
center-involving contrast (left-vs-center or center-vs-right) is real by its own interval **and** larger
than the left-vs-right contrast, stop leading with the single left-vs-right number and **show the
curve**. The guard fired: the volume-weighted left-vs-center contrast (−15.7%) is real by its own
interval and far larger than the left-vs-right near-zero.

So we show the curve. But the guard is a one-sided tripwire that reads only the volume-weighted
estimator; firing tells us how to *report*, not that the dip is *real*. The claim test is the
two-estimator rule, and the center contrasts fail it: the one-channel-one-vote estimator sees no center
dip (both center contrasts cross zero). A decisive volume-weighted result sitting on a null
one-channel-one-vote result is the signature of an effect that is real in the aggregate but not in the
typical channel.

We checked which it was. Removing one channel at a time from the left-vs-center contrast, no single
channel exceeds 3.3% of the videos, and dropping the three most influential channels moves the contrast
only from −15.7% to −8.4%, still far from the one-vote estimator's −3.7%. So this is **not** a few
outlier channels. It is a broad weighting difference: the video-count distribution is heavy-tailed
(typical channel 28 single-face videos, largest 506), and higher-volume channels show a stronger
center-vs-edge gap than the typical channel does. The volume-weighted view sees it; one-channel-one-vote
does not. We report it as unresolved shape and stop there.

An exploratory three-way test, added after the sealed read and labeled as such, says the same thing in
one number: a joint test of "does zone matter at all" is decisive in the volume-weighted form
(driven entirely by the center coefficient, with right-vs-left near zero) and null in the
one-channel-one-vote form. It sharpens the picture without changing the verdict.

## Big picture

1. **"Put the face on the left (or right)" does not hold up within a channel.** For the left-vs-right
   placement the advice is about, the effect is a precise zero. Combined with the presence study (a face
   is associated with slightly fewer within-channel views) and the size study (face size is flat), the
   broader "faces sell, and here's how to frame them" advice does not survive a within-channel test at
   scale: presence is slightly negative, size is flat, and side is null.

2. **It is a precise null, not an absence of data.** With ~380 side-varying channels and ~16,600 videos
   the held-out estimate is tight enough to rule out any left-vs-right effect bigger than about 6% (the
   widest edge of the two estimators' intervals is +6.0%). The
   honest response is to stop treating face side as a growth lever and, as always, to A/B test packaging
   on your own channel, which is the test this study cannot run for you.

3. **There is one loose thread we are honest about.** Center-placed faces may underperform both edges,
   but only in the volume-weighted view, and the typical channel shows no such gap. We flag it as
   unresolved rather than bury it or oversell it.

4. **The result is correlational.** Face placement is one packaging choice bundled with others we do not
   observe; "associated with," never "causes."

## Limitations

- **Correlational, not causal.** Where a face sits travels with framing, topic, and the rest of a
  packaging choice we do not measure. The within-channel design removes everything stable about a
  channel, not the choices that vary within it.
- **A conditional estimand with a selection caveat.** Restricting to single-face videos conditions on
  two creator choices (use a face; use exactly one) that are themselves related to views. The
  dominant-face fork bounds the single-face conditioning, and the faced conditioning is bounded small by
  the presence study; we do not claim either is zero.
- **Who it describes.** Channels with at least five left and five right single-face videos, drawn from
  our English-language business/creator-economy funnel: not a random sample of YouTube.
- **Views, not watch-time,** captured once; deleted and private videos are unobservable.
- **Short-form leakage is bounded, not eliminated.** Short vertical videos whose thumbnails are padded
  can place a face oddly; we drop everything under 90 seconds to trim the most likely cases, but the
  duration field alone does not catch every short, so a small residual remains. Because the headline is
  a null, a little such noise widens the interval rather than manufacturing an effect.
- **The detector placement check is the load-bearing validation here, and it passed.** Unlike face
  *size* (which a human cannot mark precisely), a face's horizontal *center* is cheap to mark by hand,
  so we ran the numeric check: on 150 hand-marked faces the detector's center error does not drift with
  true position (slope −0.002, bound ±0.05), the miss rate is even across zones, and the error does not
  widen at the edges. Full detail in [validation-log.md](./validation-log.md).

## Credits & disclosures

- **Data:** Hitfactor's proprietary dataset of YouTube channels and videos. Face detection and placement
  by a standard open model, validated for presence in the first study and for horizontal placement here.
- **Conflicts:** Hitfactor builds tools for video teams; this study was pre-registered to fix the
  analysis before the outcome was seen, and the headline was read once off a sealed holdout.
- **Replicate it:** the three pairwise zone contrasts behind the curve and the headline/fork table are
  in [`aggregates/`](./aggregates). A de-identified row dataset (hashed ids, so channel structure
  survives but no raw YouTube data is republished) carrying the face's zone and horizontal-center
  fraction accompanies the public release, on the same terms as the prior studies' datasets.
- **Suggested citation:** Hitfactor (2026). *The location of the face in a thumbnail (left/center/right)
  changes views by essentially zero: a within-channel study of face placement across 381 YouTube
  channels.* hitfactorapp.com/labs/2026-thumbnail-face-location/
