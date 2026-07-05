# Differentiating Data Anomalies from Intentional Reposts

During our analysis of the "Revolving Door" hiring phenomenon, one of the primary challenges was ensuring that the data we interpreted as "recurrent hiring" was not just a technical glitch, a platform bug, or an accidental double-click by a recruiter. 

Here is a breakdown of the analytical filtering and logic we applied to isolate **intentional organizational behavior** from **data anomalies**:

## 1. Exclusion of Immediate / Same-Day Duplicates
We calculated the chronological gap (`days_since_last_post`) between consecutive job postings for the exact same role at the same company. We strictly ignored all instances where the time gap was less than 30 days.

*Why do this?* If a role is posted two, three, or four times on the exact same day or within the same week, it is overwhelmingly likely to be a scraper duplication bug, an ingestion anomaly from the API, or an accidental double-post by a human recruiter. By entirely removing these short-interval duplicates from the "Revolving Door" dataset, we eliminated standard platform bugs.

## 2. Targeting the "ATS Sweet Spot" (30 to 60 Days)
Our core metric was scoped strictly to a 30-to-60-day reposting window.

*Why do this?* Modern Applicant Tracking Systems (ATS) and job board algorithms inherently penalize "stale" job listings. Many recruiters have a standard operating procedure (or use automated scripts) to close and re-open unfulfilled roles out of a 30-day or 60-day cycle to bump the posting back to the top of search results. 
Finding exact interval spikes around the 30-day and 60-day marks is the classic signature of an automated, systemic internal process rather than an organic, sporadic hiring need (like a standard replacement). This proves intentionality.

## 3. Strict Dimensional Matching
We enforced a strict grouping by both `company_name` and an *exact string match* on `job_title`. 

*Why do this?* This ensured we weren't conflating "Senior Engineer" with "Junior Engineer" at the same company. We only flagged a role if the company explicitly posted the identical title again, which points to a localized churn or artificial inflation event rather than broad, diverse corporate expansion.

## 4. Role-Tier Classification — Separating Signal from Noise

After applying the above filters, we added a critical classification step: **categorizing each role by its expected turnover profile** before ranking results.

Roles are classified into two tiers:

| Tier | Examples | Treatment |
|------|----------|-----------|
| **Expected Churn** | Cashiers, grocery clerks, visual merchandisers, behavior technicians, warehouse associates, shift leads | Filtered *out* of the revolving-door signal |
| **Knowledge-Worker+** | Software engineers, business development managers, product managers, directors, analysts, account executives | **This is the signal** — revolving-door patterns here indicate genuine dysfunction |

*Why do this?* Without this step, the top of any revolving-door ranking is dominated by companies like Kroger (cashiers), Williams-Sonoma (visual merchandisers), and Acorn Health (behavior technicians) — all roles where constant reposting is operationally normal. A grocery chain needing to hire cashiers is not a revolving-door signal; it's just what it takes to staff a grocery chain.

The value is in surfacing **roles that should NOT be high-turnover** — knowledge workers, specialized roles, mid-career positions — where a tight reposting cadence actually points at something meaningful: culture issues, ghost jobs, or growth theater. The classification uses a keyword-based heuristic to identify inherently high-turnover roles and exclude them from the signal.

## 5. Anomaly Report Format — Grouped by Category

Data anomalies (records with missing critical fields) are reported as a **categorized report** rather than a flat list of `_id` values. Each category includes:

- **A count** of affected records
- **A description** of the likely root cause (LLM parsing failure, scraper issue, date format problem, etc.)
- **Example IDs** for engineering to use as starting points for investigation

This format allows engineering to immediately route each category to the appropriate subsystem owner rather than sifting through an undifferentiated list.

---

By stripping out short-term technical noise, classifying roles by their expected turnover profile, and structuring anomaly reports for engineering consumption, we arrived at a highly confident signal of intentional, likely problematic, corporate hiring behavior — specifically in the roles where such patterns matter most.
