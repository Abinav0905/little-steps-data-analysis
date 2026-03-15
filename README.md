# LittleSteps– Nurse Visit Data Analysis

## Introduction

This project is a data analysis assignment for **LittleSteps**, an at-home healthcare startup. The goal is to process and analyze nurse visit data to uncover patterns in patient visit durations and nurse travel times, and to surface actionable insights for improving operational efficiency in nurse scheduling.

**Analytical approach:**
- Clean and standardize a messy real-world dataset (typost,mixed datetime formats, typos, duplicate and missing values)
- Use feature engineering to add `visit_duration_minutes` and `travel_duration_minutes` as the core features
- Perform statistical analysis (ANOVA) across service types, location zones, and individual nurses
- Extract clinical signals from free-text nurse notes
- Visualize findings and propose concrete operational recommendations


## Repository Structure

```
├── analysis.ipynb               # Main code file (includes outputs and plots inline)
├── visits.csv                   # Raw dataset
├── README.md                    # This file
└── plots/
    ├── hist_visit_duration.png
    ├── hist_travel_duration.png
    ├── bar_duration_by_service.png
    ├── bar_duration_by_location.png
    ├── box_duration_service.png
    ├── box_duration_location.png
    ├── bar_nurse_travel.png
    └── bar_notes_flags.png
```

## Setup & Running the Code

**Requirements:** Python 3.8+

```bash
pip install pandas numpy matplotlib seaborn scipy
```

Then open and run the notebook:

```bash
jupyter notebook analysis.ipynb
```

Or view it directly on GitHub — notebooks render in-browser without any setup.

---

## Key Findings & Insights

### Data Quality Issues Found
- **30 duplicate** `visit_id` records → removed
- **100 rows** with missing `visit_end_time` → dropped (duration cannot be computed)
- **100 rows** with missing `nurse_notes` → retained, filled with empty string
- **Mixed datetime formats** across both time columns: `YYYY-MM-DD HH:MM:SS`, `YYYY/MM/DD HH:MM`, `MM/DD/YYYY HH:MM`, `Month DD, YYYY HH:MMAM/PM` → all standardized
- **Typos in `service_type`**: `Pyhcisal Therapy`, `Medicatn Adminstratino`, `Wound Cae`, `General Chek-up` → corrected
- **Typos in `visit_location`**: `Wsst`, `Easst`, `Notrh`, `Soutth` → corrected
- **Outlier treatment**: IQR-based capping applied to `visit_duration_minutes` (cap rather than drop to preserve row count)

After cleaning: **876 usable records** from the original 1,000.


### 1. Average Durations (Overall)
| Metric | Value |
|---|---|
| Avg visit duration | ~66.4 mins |
| Avg travel duration | varies by nurse (see below) |

---

### 2. Visit Duration by Service Type
Wound Care and Medication Administration had the longest average durations (~68–70 mins), while General Check-ups were the shortest (~62 mins). The differences are moderate but consistent — scheduling should allocate service-specific time slots rather than a flat duration for all visit types.

---

### 3. Visit Duration by Location Zone
One-way ANOVA was used to test whether visit duration differs significantly across zones (North, South, East, West). Results suggest minimal geographic influence on duration — visit complexity (service type, patient condition) is likely a stronger driver than location.

---

### 4. Nurse Travel Duration – Top & Bottom 3
A small group of nurses consistently logs travel times well above the overall average. This likely reflects geographic mismatches — nurses being assigned to patients outside their natural zone. The bottom 3 nurses (shortest travel) are likely zone-matched or have clustered patient assignments.

---

### 5. Nurse Notes Analysis
Visits where notes mentioned "critical", "urgent", or "ASAP" tended to run longer than average. "Stable" patient visits were on the shorter end. This suggests that clinical complexity captured in free-text notes is a meaningful signal for expected visit duration — even without structured tagging.
---

### Operational Recommendations
1. **Zone-based nurse assignment** – Assign nurses to a primary geographic zone to reduce unnecessary cross-zone travel. The travel time disparity across nurses suggests this is not currently being done consistently.
2. **Service-type aware scheduling** – Use per-service average durations as the baseline for time-slot allocation rather than a flat estimate.
3. **Structured nurse notes** – Introduce a mandatory quick-select status field (e.g. Stable / Needs Follow-up / Urgent) at visit start, in addition to free-text notes. This enables real-time schedule adjustments downstream.
4. **Improve data collection** – Collect actual GPS-derived travel timestamps to replace the gap heuristic used here. Missing `visit_end_time` for 10% of records is a significant data quality gap that should be addressed at the point of entry.
---

## Assumptions & Challenges

| Item | Decision |
|---|---|
| `travel_duration_minutes` | No GPS/travel data in the dataset. Approximated as the time gap between a nurse's previous `visit_end_time` and current `visit_start_time`. First visits of each nurse set to 0. Gaps >240 mins capped as likely off-shift breaks, not actual travel. |
| Missing `visit_end_time` | Dropped — duration cannot be computed without it. Imputation would be too speculative. |
| Missing `nurse_notes` | Retained with empty string — the row is still useful for duration and travel analysis even without notes. |
| Outlier treatment | IQR-based capping (not removal) — keeps the row count intact while limiting distortion from extreme values. |
| Datetime parsing | Five distinct formats found across both time columns. A custom parser was written to handle all of them explicitly before falling back to pandas inference. |
