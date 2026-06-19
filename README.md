# Does Where the Face Sits on a Thumbnail Get More Views?, Replication Package

A [Hitfactor Labs](https://hitfactorapp.com) study on whether a single thumbnail face's horizontal placement (left, center, or right of frame) correlates with more views, measured **within the same channel** so the result is not just "big channels frame faces differently."

This repository is the public **replication package** for the study. It exists so anyone can audit the design, check that it was pre-registered before we looked at outcomes, and re-derive the published charts from the released aggregate tables and de-identified dataset.

## What's here

| File / dir | What it is | When published |
|---|---|---|
| `design-spec.md` | The study design: question, unit of analysis, clustering, confounders, the locked holdout, and the detector placement-validation gate. | At pre-registration, **before any outcome data was analyzed.** |
| `preregistration.md` | The pre-registered analysis plan: hypotheses, exact statistical methods, the two co-equal estimators and the claim rule, practical-significance thresholds. Committed and tagged before the first query touching views. | At pre-registration. |
| `aggregates/` | The binned tables behind every chart in the published report (`headline.csv`, `zone_curve.csv`, `zone_levels.csv`). | At publication. |
| `dataset/` | A **de-identified, row-level dataset** (see "The dataset" below) plus a runnable `replicate.qmd` notebook, so you can re-run our statistics yourself. | At publication. |
| `report.md` | The written study (prose + key takeaways), mirroring the interactive page on hitfactorapp.com. | At publication. |
| `methodology.md` | Full methodology and disclosures. | At publication. |

## The pre-registration timestamp

The `004-v0.1-preregistration` git **tag** marks the commit of `design-spec.md` + `preregistration.md` that was published **before any placement-vs-views data was queried.** That tag is the audit trail: it is what lets a skeptic verify the analysis plan was fixed in advance, not reverse-engineered to fit a result we had already seen.

## The dataset (de-identified, methods-reproduction)

`dataset/thumbnail_face_location_vpd_deidentified.csv` is one row per qualifying faced video, with each face's horizontal zone and center fraction, views, age, and a holdout flag. The YouTube IDs are removed: `channel_hash` and `video_hash` are keyed HMAC-SHA256 hashes computed with a secret salt we do not publish.

- **It preserves what you need to check our method.** Rows that share a `channel_hash` belong to the same channel, so you can reproduce the load-bearing part of our analysis: the within-channel zone contrasts, the channel-clustered intervals, and the bootstrap that resamples channels. `dataset/replicate.qmd` does exactly this and reproduces the published headline.
- **It removes what would make this a YouTube ToS violation.** A keyed hash is one-way in practice (YouTube IDs are short and enumerable, so a plain hash would be reversible; an HMAC with an unpublished salt is not). With no recoverable resource IDs, a row is a point in a distribution, not republished YouTube API data.

**Honest about the limit: this dataset reproduces our *method*, not our *measurement*.** Because the IDs are hashed, you cannot pull the thumbnail or re-fetch the view count, so you cannot independently verify that a given face placement or view count is correct. Our defense of the measurement is the **detector placement-validation gate** described in `methodology.md` (hand-marked face centers, error that does not drift with position), not the dataset.

See `dataset/README.md` for the full column definitions and the exact steps to reproduce the headline.

## How to replicate

- **Re-plot the charts:** each chart in `report.md` cites a table in `aggregates/` (plain CSV).
- **Re-run the statistics:** open `dataset/replicate.qmd` (or read it as a script). It loads the de-identified CSV, restricts to the identifying set, fits the within-channel left-vs-right contrast two ways, and bootstraps over channels. You should recover the published headline (left vs right ≈ −0.3% pooled / +0.1% one-vote, a precise null) and its interval.

## Citation

> Hitfactor (2026). *The location of the face in a thumbnail (left/center/right) changes views by essentially zero: a within-channel study of face placement across 381 YouTube channels.* hitfactorapp.com/labs/2026-thumbnail-face-location/
