# Calyntro Release Notes — April 12, 2026
## Version 0.3.5
---

## What's New

### Repository Overview
A new **Repository Info** panel shows a high-level summary of your repository at a glance: total files tracked, file type distribution, and key statistics. This gives engineering managers a quick entry point before diving into detailed analysis views.

### Regexp-Based File Exclusion
The import configuration now supports **regular expression patterns** under `excluded_patterns`. This allows you to exclude test files, generated code, or vendor directories from all analyses — not just by path prefix or file extension.

Example patterns you can now use:
```yaml
excluded_patterns:
  - "_test\\.(cpp|cc|py|go|java|ts|js)$"
  - "/(tests?|spec|__tests__)/"
  - "\\.generated\\."
```

Files matching these patterns are excluded at import time and will not appear in any analysis view, hotspot ranking, or warning list.

---

## Accuracy Improvements

### Hotspot Analysis — Sharper Signal

**Percentile-based severity classification**  
Previously, hotspot warnings used a threshold relative to the single highest-scoring file. In large codebases where all top files cluster within a narrow score range (e.g. MongoDB's `db` module), this caused nearly every file to be classified as "Critical" — making the signal meaningless.

Warnings now use percentile-based thresholds:
- **Critical** — top 5% of files by hotspot score
- **High** — top 5–15% of files by hotspot score

This typically reduces the number of critical warnings from dozens to 2–3 genuinely actionable files.

**Consistent hotspot formula across all views**  
The hotspot score shown in the **File Details** drawer now uses the same formula as the main **Hotspot Analysis** view (`weighted churn × cognitive complexity`). Previously the two views could show contradictory scores for the same file.

**Complexity assessed at the correct point in time**  
When analysing a specific time window, the complexity signal now reflects the state of each file *at the end of that window*, not the most recent state ever. This matters when files were heavily refactored after the analysis period.

### Knowledge Risk (Silo Ratio) — Cumulative Instead of Window-Based

The **Knowledge Risk** KPI card, dashboard sparkline, and **File Details** silo indicator now all use cumulative ownership — counting every commit ever made to a file up to the selected end date.

Previously, silo ratio was calculated only within the active filter window. A file with 90 historical commits from Alice and 2 recent commits from Bob would incorrectly show Bob as the sole owner when filtering to the past month.

This change affects three places:
- Dashboard KPI card "Knowledge Risk"
- Dashboard trend sparkline
- File Details → Silo Risk indicator

### Trend Sparklines — No More End-of-Period Drop

**Complexity** and **Lines of Code** sparklines no longer drop off at the end of the observation period. Previously, the last bucket would appear artificially low because it was based on incomplete data (a partial month). Both metrics are now computed as cumulative snapshots rather than per-bucket activity, which eliminates the visual cliff.

**Commits** is still a flow metric (events per bucket) but the last partial bucket is now normalized to a daily rate, so the final data point is comparable to earlier buckets.

### KPI Cards and Sparklines — Consistent Values

The **Knowledge Risk** KPI card and its corresponding sparkline now use the same calculation basis and comparison window (current value vs. 30 days ago). Previously the card and the sparkline could show contradictory trend directions for the same metric.

### File Details — Corrected Metrics

| Metric | Before | After |
|---|---|---|
| **Churn Rate** | Showed commit count (e.g. "148") | Shows lines changed (e.g. "12,450") |
| **File Age** | Days between first and last commit *in the filter window* | Days from the file's very first commit to the selected end date |
| **Silo Risk** | Based on commits in the filter window only | Cumulative — based on all commits up to end date |

---

## Interface Improvements

- **Module filter resets automatically** when switching the analysis scope between File, Module, and Team views, preventing stale filter state from carrying over.
- **Code Map (Code City)** — the risk colour scale is no longer dominated by a single outlier module. Scores are clipped at the 95th percentile before normalising, so the full colour range is used across all modules even when one module has dramatically higher activity than the rest.
- **Code Map legend** now explicitly labels which part of the risk score is all-time (complexity) and which is window-scoped (churn), making it easier to interpret what changes when you adjust the date filter.
- **Hotspot Table** — metric labels and units are now fully translated in both English and German.
