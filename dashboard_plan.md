# Tableau dashboard plan

**Audience:** content review and moderation stakeholders.
**Main message:** claim videos draw far more engagement than opinion videos, and author status tells a different story once claim status is accounted for.

## Data prep

Use the cleaned data (298 empty rows removed, `#` column dropped). Export a CSV from your notebook and connect it in Tableau.

Calculated fields:

- **View bin:** `IF [video_view_count] < 10000 THEN "Under 10k" ELSEIF [video_view_count] < 100000 THEN "10k-100k" ELSEIF [video_view_count] < 500000 THEN "100k-500k" ELSE "500k-1M" END`
- **Like rate:** `[video_like_count] / [video_view_count]`
- **Share rate:** `[video_share_count] / [video_view_count]`
- **Is claim:** `IF [claim_status] = "claim" THEN 1 ELSE 0 END` (its average gives the claim share)

Use the same colors throughout: one blue for claim, one orange for opinion.

## Sheets

1. **KPI tiles:** total videos (19,084), claim share (50.3%), median views for claims vs opinions.
2. **Videos by view range:** stacked bar, view bin on columns, count of videos on rows, colored by claim status. It shows opinions all sitting under 10k views.
3. **Median engagement by class:** grouped bars for views, likes, shares, downloads and comments (Measure Names / Measure Values, aggregated as median). Use a log axis, since the gaps are 100x or more.
4. **Claim share by author group:** horizontal bars of average `Is claim` by verified status and by ban status, with a reference line at 50%.
5. **Verified vs claim confounding:** grouped bars of average views by verified status, split by claim status. This is the sheet that shows the difference vanishing within each class.
6. **Views vs likes scatter:** log-log axes, colored by claim status, to show the collinearity and the class separation.
7. **Optional, top cue words:** a bar chart of the top 15 words by model weight, exported from Python as a small CSV. It supports the templated-text caveat.

## Layout (single dashboard, desktop)

```
+-----------------------------------------------------------+
| Title + one-line takeaway                                 |
+----------+----------+----------+--------------------------+
| KPI:     | KPI:     | KPI:     | Filters: claim status,   |
| videos   | claim %  | median   | verified, ban status,    |
|          |          | views    | duration range           |
+----------+----------+----------+--------------------------+
| Videos by view range     | Median engagement by class     |
+--------------------------+--------------------------------+
| Claim share by author    | Verified vs claim confounding  |
| group                    |                                |
+--------------------------+--------------------------------+
| Views vs likes scatter (or top cue words)                 |
+-----------------------------------------------------------+
```

Left-align titles, keep each chart's title as a plain-language takeaway (for example "No opinion video passes 10,000 views").

## Interactivity

- Filters for claim status, verified status, ban status and duration, applied to all sheets.
- Use "Filter" actions on the author-group chart so clicking a bar filters the rest.
- Tooltips should show counts and percentages, not just the mark's value.

## Story points (optional)

1. Data overview and balance.
2. Engagement gap between claims and opinions.
3. Author status and the confounding effect.
4. What it means for moderation.

## Publishing

Publish to Tableau Public and put the link and a screenshot in the README. Check that the dashboard reads well at typical laptop size, and that the filters do not produce empty views (for example, banned and verified together has few videos).

## Checklist

- [ ] Empty rows removed before connecting
- [ ] Consistent blue and orange for claim and opinion
- [ ] Log axes labeled as such
- [ ] Every chart title states its takeaway
- [ ] Link and screenshot added to the README
