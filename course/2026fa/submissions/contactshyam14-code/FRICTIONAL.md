# FRICTIONAL — what was tried, what happened, who did it

## Executive summary

This is the honest log of building the "network, don't apply" recipe: what was attempted, what was expected, what actually happened, and what changed in response, with every entry marked by who acted — Shyam (the student) or Claude (the AI assistant doing most of the hands-on work at Shyam's request). It is worth reading because several of the most useful findings came from things going wrong: the setup made the repository look broken when it wasn't, the first version of the tool counted software-testing jobs as production management, and a test of the tests leaked files into the repository. The sections marked for Shyam are his to write; Claude has not filled them in.

## Who did what

| Shyam (human) | Claude (AI) |
|---|---|
| Chose the assignment option ("network targets"); gave the situation: MS Engineering Management, 12-month OPT ending December 2026, SOC 13-1082 and 11-3051 | Explained the assignment; proposed the four example designs; asked the clarifying questions |
| Chose where the work lives (`JOB_Agent`) after the first folder failed | Cloned, installed, fixed the Windows environment outside the repo |
| Asked Claude to draft the CHANGE-BRIEF predictions for him to edit | Drafted the brief and predictions, separating observations from predictions |
| Asked "what to do" about the lifecycle status; approved the plan recommending RUNNABLE-SAMPLE with the conflict disclosed | Wrote the prototype, tests, fixtures, recipe, card, and write-ups; ran every command in this session |
| *Still to do:* re-run the tests and command; read the run-03 report; sign `last_gate` and the attestation; edit the predictions; write the sections below; ask the DSO the G4 question | Will not push, fork, or open the PR without Shyam's explicit OK |

## Log (2026-10-01, local time EDT)

Each entry: **tried → expected → happened → response → learned**, with a trace.

1. **Clone (Claude).** `git clone` into the app's session folder → expected a normal clone → `Filename too long` (Windows path limit). → Shyam picked `C:\Users\Shyam\JOB_Agent`; cloned with `core.longpaths`. *Learned:* the repo has long paths; keep it near a drive root on Windows. *Trace:* session transcript.
2. **Baseline verify failed (Claude).** `npm run verify` → expected exit 0 on a fresh clone → exit 1, three separate causes:
   - `python3` was the Microsoft Store stub;
   - `.sh` checks needed `bash` on PATH;
   - 6 × `E3 … out of sync` errors.

   → Added a `python3` shim and Git's bash to PATH, both outside the repo. The E3 errors were Windows CRLF line endings: re-checked out with LF and they vanished.

   *Learned:* the first `doctor` run reported **0 of 33 recipes with frontmatter**; on an untouched LF checkout it is 33 of 33. A one-line test shows doctor's regex rejects a line ending in `\r`. **The environment made the repo look broken twice.** *Trace:* `evidence/toolchain-before-doctor.txt` (clean), and the first saved run outside the repo.
3. **Sample scorer run (Claude).** `npm run score -- data/examples/ch11-roles.json --out-dir …` in PowerShell → expected output in my folder → npm dropped `--out-dir` and the scorer **overwrote two tracked files** in `data/examples/`. → Restored with `git checkout`; call `node scripts/score/role-scorer.mjs` directly ever since. *Learned:* the assignment's own warning applies in an unexpected way on Windows. *Trace:* WORKED-RUN §0.
4. **Data exploration (Claude), before any code.** Expected an SOC column and a usable Form D join → neither exists. The CSV has titles only, and 15 of 200 SEC samples match any company, none of them a target. Then two surprises:
   - **every one of 1,557 approval and denial counts is even**;
   - **42 groups of companies share an identical H-1B record** (CONVEY/COVEY, CAREDOX/CAREDX, BANYAN INFRASTRUCTURE/BARKING LABS).

   → Recorded as *observations*, kept separate from predictions in the CHANGE-BRIEF; built both checks into the prototype. *Trace:* `f7d5cd2`, CHANGE-BRIEF §6.
5. **The scorer's defaults (Claude, reading code).** Found that a missing liveness gate counts as 1 (open), that funding isn't a scorer term, and that a profile saying "authorized" switches sponsorship off. → The prototype never sends a role without explicit liveness, and a characterization test pins that behaviour. *Trace:* `scripts/score/role-scorer.mjs` lines 58–60 and 83–84; test 17.
6. **Run 01 (Claude).** Expected few production-manager matches (prediction 2: three or fewer) → got **10**. → Listed the matches by vocabulary source: 8 came from O*NET 11-3051.01's "Quality Assurance Manager" alternate title, i.e. software QA jobs. → Limited the BLS vocabulary to the primary O*NET row; run 02 gave 2. *Learned:* a vocabulary taken from a record is still only a vocabulary. *Trace:* `runs/01-first-pass` vs `runs/02-primary-vocab`; commit `b30d64a`.
7. **Board probing (Claude).** Guessed 33 board slugs → expected several hits → 28 were 404s; 5 boards found, and only the Greenhouse ones name their company. → Labelled Ashby/Lever board readings as model-judgment; kept a deliberately wrong slug (Zero Motorcycles) to prove that a failed check holds a company rather than closing it. *Trace:* `evidence/board-probe-2026-10-01.txt`, `evidence/scan-dry-run-2026-10-01.txt`.
8. **UTC date (Claude).** The scanner banner said **2026-10-02** at 22:57 EDT on 2026-10-01; the scorer report also uses UTC. Recording the banner date would have held every company as "dated after the as-of date". → Observations use the local date; the guard stays. *Trace:* WORKED-RUN §3.
9. **Tests passed first time (Claude).** All 16 passed on the first run → suspicious, so three bugs were injected (M1–M3) → each was caught, **but after restoring, the suite failed**. M3, with the guard disabled, had really written 5 files into `data/examples/`, and the next clean run tripped on them. → Moved them out of the repo; the out-dir test (then numbered 14) now uses a unique folder and cleans up; M3 re-run left no residue. *Learned:* run mutations in a disposable checkout. *Trace:* `a309e8c`, `evidence/mutation-tests-2026-10-01.txt`.
10. **A check that silently didn't run (Claude).** A `git grep` for user paths errored (`unable to resolve revision: --cached`), and the `|| echo none` fallback printed "none" anyway. → Noticed, re-ran correctly, and confirmed the search finds a known string before trusting "no matches". *Learned:* an error plus a default message looks exactly like a pass. *Trace:* session transcript.
11. **A user path in evidence (Claude).** The out-dir refusal printed an absolute path containing the Windows username. → It now prints a repo-relative path; evidence regenerated. *Trace:* `a0feadb`.
12. **Misleading explanation (Claude).** Run 03's report said "approvals 12 **or** rate 85.7% below the Proven bar" when only the rate failed. → It now names only the failed condition, with a test; tiers unchanged, cross-check still 30/30. *Trace:* `a0feadb`.
13. **Wrong break input (Claude).** The first F3 input named a fixture-only company, plus "Northwind", which turned out to be in the real CSV. → Grepped the real file and replaced both. *Trace:* `fixtures/break-liveness-unmatched.json`.
14. **Planning errors (Claude).**
    - The approved plan said `npm run doctor` would check the case recipe's TODO count. Doctor only reads top-level `recipes/*.md`, so the count was checked with `grep` instead (6 = 6), and the TEST-REPORT says so.
    - The recipe's G4 gate test used a `grep` that could never match the bold run-log line. It was fixed and self-tested.
15. **A number written from memory (Claude).** The first worked-run draft said run 02 had 18 Proven companies; the saved log says 15 (18 was run 01). → Corrected after re-reading every quoted count from the logs. *Trace:* WORKED-RUN §2.

16. **Pasting the privacy scan re-created its finding (Claude).** TEST-REPORT and `evidence/pii-scan-after.txt` pasted the scan output verbatim, including the flagged email address from upstream's `package-lock.json`. The final branch-history scan then reported **2 findings in my own files**. Deleting them in a later commit wouldn't help, because CI scans the whole branch history. → Redacted the address with a visible note and amended that single local commit; nothing had been pushed. The branch-history scan was clean again. *Learned:* evidence about sensitive data must not contain the sensitive data. *Trace:* TEST-REPORT §Privacy scan; `evidence/pii-scan-after.txt` header note.

## Unresolved questions

- Are the H-1B counts really doubled, and at which upstream step? This needs the USCIS export (recipe proposal 3).
- For each shared-record pair, which company owns the record?
- Do the Ashby `genies` and Lever `sentibio` boards belong to Genies Inc and Senti Biosciences?
- Is the degree STEM-designated? If so, a STEM OPT extension changes the whole timeline. **Only the DSO can answer this.**
- Should funding become a scorer term, or stay outside the composite?
- Is RUNNABLE-SAMPLE the right claim with six open proposal TODOs, or should a strict reading hold it at DRAFT?

## Shyam — in your own words *(graded; Claude has not written these)*

**What I checked myself when I re-ran it** (command, what I saw, whether it matched):

> *(write here)*

**What I accepted from Claude's work, and why:**

> *(write here)*

**What I changed or rejected** (predictions, rules in the persona, wording, the status claim):

> *(write here)*

**What I learned that I didn't expect:**

> *(write here)*
