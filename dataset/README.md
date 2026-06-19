# De-identified dataset: Thumbnail Face Location vs. Views

This is a **methods-reproduction** dataset, not a measurement-verification one. It lets a
replicator re-run our statistics (the within-channel zone contrasts, the channel clustering, the
bootstrap) and catch a coding or specification error. It does **not** let anyone re-verify our
face placements or view counts, because the real YouTube identifiers and thumbnails are not in it.
The dataset proves the *method*; the *measurement* is defended by the detector placement-validation
gate described in the methodology, not by this file.

## File

`thumbnail_face_location_vpd_deidentified.csv`: one row per qualifying **faced** video (a thumbnail
with at least one face clearing the 2%-of-frame size floor, the thumbnail unchanged since the
placement pass, with the locked view, age, and duration filters applied).

| Column | Meaning |
|---|---|
| `channel_hash` | One-way keyed hash of the channel id (see "De-identification"). Rows sharing a channel share this value: the channel-clustering structure is preserved. |
| `video_hash` | One-way keyed hash of the video id. |
| `zone` | The single (or largest) face's horizontal zone: `left` `[0, 1/3)`, `center` `[1/3, 2/3)`, `right` `[2/3, 1]` of frame width. |
| `xcenter_frac` | The face's bounding-box horizontal center as a fraction of frame width (0 = left edge, 1 = right edge). The raw predictor `zone` is binned from. |
| `single_face` | 1 if exactly one face clears the 2% floor (the headline frame, located by that face); 0 if two or more faces clear it (located by the largest face). Filter to `single_face = 1` to reproduce the headline; keep all rows to reproduce the dominant-face fork. |
| `n_faces` | Number of faces clearing the 2% size floor. |
| `largest_face_area_pct` | The largest face's bounding-box area, as a percent of the thumbnail. Lets you restrict to videos whose largest face clears a higher floor (e.g. 4%). Note: this is a weaker restriction than the published 4% size-floor fork, which re-decides which videos are *single-face* at the higher floor; that needs every face's area, and only the largest is released here. |
| `views` | Total views at the single capture time. |
| `age_days` | Days between publish and the view capture. |
| `publish_month` | Publish year-month (`YYYY-MM`). |
| `view_capture_date` | The date the view count was captured (almost the entire cohort falls in a single short window in June 2026). |
| `holdout` | 1 if the row is in the held-out 20% of channels (the headline number is computed on these), 0 if exploration. |

## Reproducing the headline

The headline is the within-channel **left-vs-right** view contrast among single-face videos, on the
holdout, reported two ways:

1. Filter to `single_face = 1 AND holdout = 1`.
2. Keep channels with at least five `left` and five `right` rows (the identifying set: a within-channel
   left-vs-right contrast only exists for channels that vary which side the face is on). This is **381
   channels / 16,561 videos**.
3. Regress `log(views)` on a right-vs-left dummy plus `log(age_days)` with channel fixed effects
   (the pooled estimator), and separately take the median over channels of each channel's
   age-adjusted (right mean − left mean) (the one-channel-one-vote estimator).
4. Bootstrap by resampling **channels** (not videos) with replacement.

Both estimators land inside ±10% and straddle zero: a precise null. The three pairwise zone
contrasts behind the curve, and the headline/fork table, are in `../aggregates/`.

## De-identification

`channel_hash` and `video_hash` are `HMAC-SHA256(secret_salt, id)`, truncated, with a secret
salt we generated and **do not publish**. (A plain hash of a YouTube id would be reversible, since
the ids are short and enumerable, so we key the hash with an unpublished salt: it's stable, so
which-rows-share-a-channel survives for replicators, but it isn't reversible back to a real id.)

No raw YouTube data (real ids, titles, descriptions, thumbnails) is in this file. That is the
point of the de-identification: YouTube's API Developer Policies don't allow republishing API
data keyed to real resource ids. With the ids hashed and the raw fields gone, each row is a point
in a distribution rather than "the view count *for video X*": the same category as the published
aggregate tables.

## Suggested citation

> Hitfactor (2026). *The location of the face in a thumbnail (left/center/right) changes views by
> essentially zero: a within-channel study of face placement across 381 YouTube channels.*
> hitfactorapp.com/labs/2026-thumbnail-face-location/
