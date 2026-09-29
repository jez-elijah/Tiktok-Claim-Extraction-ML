# Classifying claims and opinions in TikTok videos

A data analysis and machine learning project that explores TikTok video data and builds a model to classify each video as a **claim** or an **opinion**. It follows the PACE workflow (Plan, Analyze, Construct, Execute).

**In short:** claim videos get about 100x more views than opinion videos, and a text model can separate the two classes perfectly on this dataset. Testing showed that the perfect score depends on about ten templated cue words. With those words masked, accuracy falls to roughly 68%.

## Business question

Can we use machine learning to identify claims and opinions in TikTok videos and comments, so that content review can prioritize videos that make verifiable claims?

## Data

`tiktok_dataset.csv`: 19,382 rows and 12 columns.

| Column | Description |
|---|---|
| `claim_status` | Target: claim or opinion |
| `video_id` | Unique identifier |
| `video_duration_sec` | Length in seconds (max 60) |
| `video_transcription_text` | Transcription of the video (about 16 words on average) |
| `verified_status` | Whether the author is verified |
| `author_ban_status` | active, under review, or banned |
| `video_view_count`, `video_like_count`, `video_share_count`, `video_download_count`, `video_comment_count` | Engagement counts |

## Cleaning decisions

- **Dropped 298 rows (1.5%)** that had no label, no transcription and no engagement counts. The gaps all sit in the same rows, so these are empty records, not random missing values. Result: 19,084 rows.
- **Dropped the `#` index column.** `video_id` is used only as an identifier (no duplicates).
- **Kept outliers.** Likes, shares, downloads and comments each have 1,700 to 2,800 high outliers, all of them claim videos. These look like real viral videos, so I kept them and used log transforms instead.
- **No resampling needed.** Classes are balanced: 9,608 claims and 9,476 opinions.

## Key findings from EDA

- **Views almost separate the classes.** Median views are 501,555 for claims and 4,953 for opinions. No opinion video has more than 10,000 views.
- **Every engagement metric favors claims.** Median likes are 123,649 vs 823, shares 17,998 vs 121, downloads 1,140 vs 7, and comments 286 vs 1.
- **Author status matters.** Banned authors post 88% claims, authors under review 78%, active authors 43%. Verified authors post 83% opinions.
- **Engagement metrics are correlated** (for example likes with views 0.80, likes with shares 0.83), so collinearity matters for regression. Video duration is unrelated to anything (correlation about 0.01).

## Hypothesis test: views by verified status

- H0: mean views are equal for verified and non-verified authors. H1: they differ (two-sided, alpha 0.05).
- Welch's t-test (unequal variances): verified authors average 91,439 views vs 265,664 for non-verified authors, a gap of about 174,000 (95% CI about 160,800 to 187,600), t = -25.5, p < 0.001, Cohen's d = -0.54.
- Robustness: a Mann-Whitney U test and a t-test on log views give the same conclusion.
- **Confounding:** the gap is explained by claim status. Within claim videos, verified and non-verified authors average about the same views (p = 0.98). The same holds within opinion videos (p = 0.81). Verified authors simply post more opinions, which get fewer views (chi-square = 554, p < 0.001).

## Models

**Logistic regression on engagement and author features** (log-transformed counts, standardized, 5-fold stratified CV):

| Features | Accuracy | Claim recall | AUC |
|---|---|---|---|
| All features | 99.2% | 98.4% | 0.997 |
| Without views | 97.7% | 96.0% | 0.992 |

The model never labeled an opinion as a claim. Its errors were missed claims. It cannot be used at upload time, because a new video has no engagement yet.

**Final model: TF-IDF on transcription text (1 to 2 word phrases) plus duration and author status, logistic regression.** 100% accuracy in 5-fold CV and 0 errors on a 4,771-video hold-out. A random forest scored the same, so I chose logistic regression for interpretability.

## Robustness: cue-word masking

Perfect scores are a warning sign, so I removed the most influential words from every transcription and retrained with 5-fold CV.

| Words masked | Accuracy | Claim recall | AUC |
|---|---|---|---|
| 0 | 100% | 100% | 1.000 |
| 25 | 94.7% | 93.5% | 0.994 |
| 50 | 83.3% | 94.8% | 0.918 |
| 100 | 69.0% | 96.6% | 0.680 |
| 400 | 67.9% | 95.9% | 0.671 |

The top cue words include "my", "read", "learned", "discovered", "someone", "friend", "media" and "claim". Transcriptions follow fixed templates such as "a friend read in the media that..." (claims) and "my family is willing to bet that..." (opinions). The model is largely learning these templates.

## Limitations

- **Templated text.** The wording is far more regular than real speech. Expect real-world accuracy to be well below 100%.
- **Engagement is not available for new videos**, so the engagement model suits prioritizing already-published content, not upload-time screening.
- **One dataset, one split family.** Results come from a single dataset with cross-validation and one hold-out.
- **Labels are taken as given.** I could not verify how claim and opinion labels were assigned.

## Recommendations

1. Validate the text model on real, unstructured transcripts or a hand-labeled sample before any use in review workflows.
2. Track claim recall as the primary metric, since missed claims are the costlier error for moderation.
3. Use engagement only to prioritize videos already in circulation.
4. Treat verified status as a weak signal on its own, since its effect is explained by claim status.

## Dashboard

Not Yet Published.

## Tools

Python (pandas, SciPy, scikit-learn), Tableau, PACE workflow.
