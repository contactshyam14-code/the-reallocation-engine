# CHANGE-BRIEF — "Network, don't apply" for an Engineering Management graduate on 12-month OPT

## Executive summary

This is the plan and the predictions for a small recipe and prototype, written before any prototype code existed. It is worth reading next to the worked run, because the point of a prediction is to be compared with what actually happened.

The plan: take companies whose public H-1B record shows they have sponsored project-management or production-management titles, check whether each one has a matching job open right now, and split them three ways — apply now (a matching role is open), network first (they sponsor this kind of role but nothing is open, so an informational conversation comes before any application), or skip. Every value is labelled as a record, the person's own input, or a model judgment, and the recipe stops at two human checkpoints: a visa-path sign-off before any application is tailored, and an identity check for companies whose sponsorship record may belong to a different, similarly named company.

## Who wrote this, and when

- **Drafted by Claude (AI) on 2026-10-01 at Shyam's request**, after reading the repository's rules and exploring the shipped data, and **before any prototype code was written**. Shyam reviews and edits it; edits go under *Revisions* at the bottom. The original text stays — revisions are added, not overwritten.
- Section 6 lists things already **observed** during data exploration. They are not predictions and are kept separate on purpose.

## 1. Career situation

| Item | Value | Label |
|---|---|---|
| Persona | "Shyam" — a stand-in modelled on the author's situation; no résumé, contacts, or real email (uses `shyam@example.com`) | your-input |
| Degree | MS Engineering Management | your-input |
| Status | F-1, 12-month post-completion OPT; no STEM OPT extension claimed | your-input |
| OPT end date | 2026-12-31 (month from the author; the day is an assumption — replace it with the exact date on the EAD card) | your-input |
| Target occupations | SOC 13-1082 Project Management Specialists; SOC 11-3051 Industrial Production Managers | your-input |
| Needs sponsorship | Yes — an employer must be willing to sponsor an H-1B | your-input |

On the run date (2026-10-01) about 91 days remain on the OPT card. With no STEM extension, the next H-1B cap registration (expected around March 2027 under the usual annual cycle — not a record in this repository) falls **after** the OPT end date. Whether anything bridges that gap — a cap-exempt employer, the degree's STEM eligibility, or another status — is a question for the school's DSO or an immigration attorney. The recipe flags it and stops; it does not answer it.

**Engine layers used:** 80 Days to Stay (sponsorship history and funding), Job-Ops (posting liveness), The Cognitive Pivot (SOC titles only — no wages, no ability levels).

## 2. What is reused, and what is new

**Reused, unmodified:**

| What | Path | Used for |
|---|---|---|
| 80 Days company table | `data/80-days-to-stay/80-days-csv/mapped_student_employment_targets_v3.csv` | H-1B approvals/denials/rate, sponsored job titles, funding columns |
| SEC Form D samples | `data/sec/form-d/processed/sample/companies-sec-2025q2-d.sample.json` … `companies-sec-2026q1-d.sample.json` | funding record join (4 quarters × first 50 filers) |
| BLS/O*NET compact file | `data/bls/compact/soc_occupation_compact.csv` | confirm the two SOC codes exist; official titles and sample alternate titles |
| Role scorer | `scripts/score/role-scorer.mjs` (CLI, called as-is) | the Apply / Consider / Skip decision and its audit trace |
| Board scanner | `scripts/ats/scan.mjs --dry-run` with `REALLOCATION_ENGINE_PORTALS` | board-level evidence: "matching role open" vs "board fetched, nothing matching" vs "could not check" |
| Posting checker | `scripts/ats/check-liveness.mjs` | posting-level liveness when a specific URL exists |

**New (in my namespace only):**

- `scripts/contrib/2026fa/contactshyam14-code-em-network-targets/network-targets.mjs` — no repository script turns 80 Days rows into scorer input, and none turns the scorer's liveness-gated Skips into a networking list. That conversion is the whole contribution.
- Two data-quality checks run on every load (count parity, shared H-1B records) — reason: both problems were found during exploration (section 6) and neither appears in the repository's existing audits.
- No change to any maintained file. No new npm script (contributions don't get one).

## 3. Gates — where the recipe stops

| Gate | Kind | Stops when | What a human needs to see to clear it |
|---|---|---|---|
| G1 Inputs | machine, halts the run | persona file invalid; OPT end on/before the as-of date; a target SOC code has no row in the BLS file; the CSV lacks a required column | the error message; nothing is written |
| G2 Liveness | gate (multiplier) | a candidate has no liveness observation → held back, never sent to the scorer | each observation's method, date, and evidence file |
| G3 Visa timeline | gate (multiplier) | computed timeline factor ≤ 0.05 | OPT end date, as-of date, hiring-lag assumptions, resulting factor |
| G4 Visa-path sign-off | human only | before any application is tailored | written DSO/attorney confirmation that a post-OPT path exists; networking may continue while this is open |
| G5 Identity check | human only | a candidate's H-1B record is shared with a differently named company | which companies share the record; the human confirms which one it belongs to |

## 4. Predicted failure cases, and how each will be checked

| # | Failure case | Expected behaviour | Check |
|---|---|---|---|
| F1 | OPT end date already past on the as-of date | refuse; exit non-zero; write nothing | test with an as-of date after the OPT end |
| F2 | Target SOC code has no row in the BLS file (e.g. a typo, 13-1028) | refuse; exit non-zero; name the missing code | test with a bad code |
| F3 | A liveness observation names a company that isn't in the CSV (e.g. "Form Energy" vs "FORM ENERGY INC") | report it as unmatched; invent nothing | test, plus the real run |
| F4 | A job board fetch fails (wrong slug, 404, timeout) | the company stays held as *unchecked* — not "no posting" | one deliberately wrong board slug in the worked run |
| F5 | A candidate has no Form D record in the shipped sample | "not in the shipped sample", never "no funding" | the real run (expected for almost every candidate) |
| F6 | CSV missing a required column | refuse; exit non-zero | test with a truncated fixture |

## 5. Predictions — what the first pass will get wrong

*(Drafted by Claude for Shyam to edit, remove, or replace with his own.)*

1. **Title-to-SOC matching will be noisy.** Software "Technical Program Manager" titles — probably filed under computer-occupation SOC codes — will dominate the project-management list, and a title like "Computer Systems Analyst (Senior IT Project Manager)" will be counted as project management even though its own wording names a different occupation. The prototype cannot see the SOC code that was actually filed.
2. **The production-manager list (11-3051) will be nearly empty** — three companies or fewer — because the sponsored-title lists in the CSV are dominated by technology roles.
3. **The first real run will hold back every company**, because no liveness observations exist yet, and the scorer will receive zero roles. I predict the scorer's summary line will print a nonsense skip rate (NaN) in that case, because it divides by the number of roles.
4. **The funding ranking will be partly wrong.** Old "Series D+" rounds attached to large public companies are probably Form D filings matched to the wrong entity, and they will sit beside genuinely recent small raises with nothing to tell them apart.

## 6. Already observed during exploration (not predictions)

Observed on 2026-10-01, before prototype code, with throwaway scripts outside the repository:

- The CSV has **no SOC column**; the only occupation signal is a truncated list of top sponsored job titles.
- **All 1,557 approval counts and all 1,557 denial counts are even numbers.** Honest counts would be roughly half odd. The absolute counts are probably doubled somewhere upstream; the approval *rate* is unaffected. Not verifiable offline.
- **42 groups covering 85 companies carry an identical H-1B record** (same approvals, denials, median salary, and title list) under different names — e.g. CONVEY INC / COVEY INC, CAREDOX INC / CAREDX INC, BANYAN INFRASTRUCTURE CORP / BARKING LABS CORP. Some are the same entity (Inc vs LLC); others look like fuzzy-match collisions.
- Only 15 of the 200 shipped SEC Form D sample records match a CSV company by normalized name; none of them is a project/production-management sponsor.
- The scorer treats a **missing gate as 1 (open)**, has **no funding term**, and gives role quality **weight 0**.
- On Windows PowerShell, `npm run score -- … --out-dir …` dropped the `--out-dir` flag and overwrote the tracked example output (restored with `git checkout`). The scorer is called directly with `node` from here on.

## Revisions

*(Shyam: add dated revisions here. Don't edit the sections above — the point is to keep the original record.)*
