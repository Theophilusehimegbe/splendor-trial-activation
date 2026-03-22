# Splendor Analytics — Trial Activation Challenge

**Competition:** Splendor Analytics Data Challenge
**Grand Prize:** ₦100,000
**Author:** Theophilus

---

## Problem Statement

Only 1 in 5 organisations trialling the platform convert to paying customers.
This project defines what **Trial Activation** looks like — the specific in-app
behaviours that signal an organisation is genuinely getting value — and builds
the infrastructure to track it.

---

## Repository Structure

```
splendor-trial-activation/
│
├── README.md                        ← You are here
├── requirements.txt                 ← Python dependencies
│
├── data/
│   └── DA_task.csv                  ← Raw event log (170,526 rows)
│
├── task1_python/
│   └── Task1_analysis.ipynb         ← EDA, conversion analysis, goal definition
│
├── task2_sql/
│   └── task2_sql_models.sql         ← Staging + mart layer SQL models
│
└── task3_python/
    └── Task3_analysis.ipynb         ← Descriptive analytics & product metrics
```

---

## Dataset

| Column | Description |
|---|---|
| `ORGANIZATION_ID` | Unique identifier for each trialling organisation |
| `ACTIVITY_NAME` | Name of the in-app activity performed |
| `TIMESTAMP` | When the activity occurred |
| `CONVERTED` | Whether the organisation converted to a paying customer |
| `CONVERTED_AT` | Date of conversion (NULL if not converted) |
| `TRIAL_START` | Start date of the 30-day trial |
| `TRIAL_END` | End date of the 30-day trial |

**Raw shape:** 170,526 rows × 7 columns
**After cleaning:** 102,895 rows × 966 unique organisations
**Conversion rate:** 21.3% (206 converted, 760 not converted)

---

## Task 1 — EDA & Trial Goal Definition

**Notebook:** `task1_python/Task1_analysis.ipynb`

### Approach

1. **Data Cleaning** — Parsed date columns, removed 67,631 duplicate rows,
   derived `days_into_trial`, filtered to in-window events only.

2. **Org-Level Summary** — Collapsed 102,895 event rows into 966 rows
   (one per organisation) with activity counts, engagement metrics and
   a modules_used score.

3. **Conversion Driver Analysis** — Ran Mann-Whitney U tests across 6
   engagement metrics. All p-values > 0.05 — general engagement volume
   does not predict conversion. Shifted focus to activity-level analysis.

4. **Activity Conversion Rates** — Calculated the conversion rate for orgs
   that used each of the 28 activities vs the 21.3% baseline.

5. **Usage Gap Analysis** — Compared activity adoption rates between
   converted and non-converted orgs to identify the strongest signals.

### Defined Trial Goals

| Goal | Condition | Evidence |
|---|---|---|
| **Goal 1 — Core Scheduling Activated** | `Scheduling.Shift.Created >= 3` | 21.8% conversion rate, used by 848/966 orgs, +2.6pp usage gap |
| **Goal 2 — Scheduling Depth Reached** | `Scheduling.Template.ApplyModal.Applied >= 1` | 25.0% conversion rate, +2.4pp usage gap |
| **Goal 3 — Time & Attendance Activated** | `PunchClock.PunchedIn >= 1` | 22.7% conversion rate, +1.9pp usage gap |

**Trial Activation** = all 3 goals completed within the 30-day trial window.

### Validation Results

| Metric | Result |
|---|---|
| Goal 1 completion rate | 58.3% of orgs (563) |
| Goal 2 completion rate | 11.2% of orgs (108) |
| Goal 3 completion rate | 21.8% of orgs (211) |
| Trial Activation rate | 5.8% of orgs (56) |
| Converters who activated | 6.3% |
| Non-converters who activated | 5.7% |

---

## Task 2 — SQL Data Warehouse Models

**File:** `task2_sql/task2_sql_models.sql`
**Database:** SQL Server (T-SQL)

### Architecture

```
raw_events            ← Raw layer   — DA_task.csv loaded as-is (170,526 rows)
      ↓
stg_events            ← Staging layer — cast, deduplicated, filtered (102,895 rows)
      ↓
fct_trial_goals       ← Marts layer  — one row per org, goal1/2/3 flags (966 rows)
      ↓
fct_trial_activation  ← Marts layer  — trial_activated + activated_at (966 rows)
```

### fct_trial_goals

One row per organisation. Tracks whether each of the three Trial Goals
was completed during the trial period.

| Column | Description |
|---|---|
| `organization_id` | Unique org identifier |
| `goal1_met` | 1 if Scheduling.Shift.Created >= 3, else 0 |
| `goal2_met` | 1 if Template.ApplyModal.Applied >= 1, else 0 |
| `goal3_met` | 1 if PunchClock.PunchedIn >= 1, else 0 |
| `shifts_created` | Raw count for auditability |
| `templates_applied` | Raw count for auditability |
| `punch_ins` | Raw count for auditability |

### fct_trial_activation

One row per organisation. The final activation status table.

| Column | Description |
|---|---|
| `organization_id` | Unique org identifier |
| `trial_activated` | 1 if all 3 goals met, else 0 |
| `activated_at` | trial_end if activated, NULL otherwise |

---

## Task 3 — Descriptive Analytics & Product Metrics

**Notebook:** `task3_python/Task3_analysis.ipynb`

### Key Findings

| Metric | Value |
|---|---|
| Overall conversion rate | 21.3% |
| Trial activation rate | 5.8% |
| Activated → conversion rate | 23.2% |
| Not activated → conversion rate | 21.2% |
| Scheduling module adoption | 88.2% |
| PunchClock module adoption | 21.8% |
| Timesheets module adoption | 1.0% |
| Orgs with low stickiness (< 0.2) | 84.4% |
| March cohort conversion rate | 18.2% ↓ |

### Top Recommendations

1. **Fix the Goal 2 bottleneck** — 80% of orgs that hit Goal 1 never
   discover templates. Add an in-app nudge after the 3rd shift is created.

2. **Push PunchClock activation earlier** — Only 21.8% of orgs ever have
   a team member clock in. Build a guided "invite your team" step into onboarding.

3. **Investigate the March cohort drop** — Conversion fell from ~23% to 18.2%
   in March. Cross-reference with sales and product changes from that period.

4. **Double down on Scheduling as the anchor module** — 88.2% adoption.
   Design onboarding as Scheduling-first, then expand to other modules.

5. **Study the 95 highly sticky orgs** — They represent the product's power
   users. Their behaviour is the blueprint for ideal trial engagement.

6. **Refine Trial Goals quarterly** — As more cohorts mature, rerun the
   activity-level analysis to tighten the goal thresholds.

---

## How to Run

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/splendor-trial-activation.git
cd splendor-trial-activation

# 2. Install dependencies
pip install -r requirements.txt

# 3. Add the dataset
# Place DA_task.csv inside the data/ folder

# 4. Run the notebooks
jupyter notebook task1_python/Task1_analysis.ipynb
jupyter notebook task3_python/Task3_analysis.ipynb

# 5. For Task 2
# Open task2_sql/task2_sql_models.sql in SSMS
# Run sections in order: Setup → Raw → Staging → fct_trial_goals → fct_trial_activation
```

---

## Results Summary

| Task | Deliverable | Status |
|---|---|---|
| Task 1 | EDA + Trial Goal Definition (Python) | ✅ Complete |
| Task 2 | SQL Mart Models (fct_trial_goals + fct_trial_activation) | ✅ Complete |
| Task 3 | Descriptive Analytics + Product Metrics (Python) | ✅ Complete |
