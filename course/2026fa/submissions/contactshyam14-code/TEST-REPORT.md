# TEST-REPORT — em-network-targets

## Executive summary

This report records how the prototype was tested: the repository's own health checks before and after the change, the real sample run, each failure case deliberately triggered, the scope of the change, and what a person still has to judge. Everything the repository checks by machine passed, and nothing that passed before the change fails after it. The one privacy-scan finding is in a file the change does not touch and was there before it. Two things are not machine-checkable and are left for the student: whether the visa path exists, and whether two flagged sponsorship records belong to the companies they name.

## Environment

- Windows 11 · Node v22.12.0 · Python 3.12.5 · Git Bash. Commands run from the repository root.
- **Outside the repository, needed on this machine only:**
  - a `python3` shim (Windows' `python3` is a Microsoft Store stub);
  - Git's `bash` on PATH for the `.sh` checks;
  - the clone checked out with LF line endings (`core.autocrlf=false`).
- **Without these, a fresh Windows clone looks broken:**
  - `verify` fails E3 on all six adapters;
  - `doctor` reports 0 of 33 recipes with frontmatter, because its parser rejects a line ending in `\r`.

  CI runs on Linux and is unaffected.
- **PowerShell drops `--out-dir`.** `npm run score -- … --out-dir …` loses the flag there, so the scorer is called as `node scripts/score/role-scorer.mjs …`.

## Toolchain baseline — before and after

**Before** = an untouched worktree of upstream `origin/main` (`015843d`), i.e. this branch before any change, same machine, same shims (`evidence/toolchain-before-*.txt`). **After** = this branch at `34b0c7a` (`evidence/toolchain-after-*.txt`).

`npm run doctor` — **identical before and after** (diff of the two outputs is empty apart from the header line). After:

```text
ENVIRONMENT (required)
  ✓ node       v22.12.0
  ✓ python3    Python 3.12.5

ENVIRONMENT (optional — features degrade without these)
  — pandoc     not found (resume/PDF rendering)
  — libreoffice not found (PDF fallback)
  ✓ playwright installed

PRIVACY (no personal data committed)
  ✓ no private/PII paths are tracked

RECIPES (33)
  with lifecycle frontmatter: 33   missing: 0
  by status: DRAFT 28 · RUNNABLE-SAMPLE 4 · RUNNABLE-LIVE  # DRAFT | SPECIFIED | RUNNABLE-SAMPLE | RUNNABLE-LIVE | VERIFIED 1
  open TODOs: 318 declared (in frontmatter) · 318 [TODO markers in bodies

SUMMARY
  environment: ✓ runnable
  recipes: 33/33 carry lifecycle frontmatter — all tracked
  next: continue
exit: 0
```

`doctor` reads only top-level `recipes/*.md`, so it **does not check this case recipe** (`recipes/cases/2026fa/…`). Its frontmatter was checked separately, using doctor's own parsing logic and a YAML parser: `todos_open: 6` matches the six `[TODO` markers in the body. Doctor also mis-reads one upstream recipe's status (the `RUNNABLE-LIVE  # …` above) because it doesn't strip trailing YAML comments. That is an upstream issue; this recipe keeps its comments on their own lines.

`npm run verify` — before:

```text
conformance: 158 files (85 md · 36 py · 30 js · 4 sh · 3 json)
✓ all conform (machine half of P4). Adequacy is still the human gate.
...
✓ manifest check passed (3 warnings)
exit: 0
```

after (the only difference is the file count — the 14 added conformance-checked files are this contribution's):

```text
$ npm run verify

> the-reallocation-engine@1.0.0 verify
> node scripts/conformance.mjs && node scripts/manifest-check.mjs

conformance: 172 files (88 md · 36 py · 32 js · 11 json · 1 yaml · 4 sh)
✓ all conform (machine half of P4). Adequacy is still the human gate.
MANIFEST CHECK — The Reallocation Engine
==========================================

WARN (3):
  W1 ignore path not in .gitignore: archive/
  W2 private path not gitignored (PII/secret risk): private/
  W2 private path not gitignored (PII/secret risk): data/ats/

✓ manifest check passed (3 warnings)
exit: 0
```

The three warnings are upstream and unchanged. The W2 lines are a pattern mismatch: `.gitignore` uses `/private/*` and `/data/ats/*`, and `git check-ignore -v private/x.txt data/ats/x.txt` confirms that files inside both folders are ignored.

Conformance on this contribution alone:

```text
$ node scripts/conformance.mjs recipes/cases/2026fa scripts/contrib/2026fa/contactshyam14-code-em-network-targets course/2026fa/submissions/contactshyam14-code logs/runs/2026fa-contactshyam14-code-1.md
conformance: 38 files (15 md · 18 json · 1 yaml · 3 js · 1 py)
✓ all conform (machine half of P4). Adequacy is still the human gate.
exit: 0
```

## Privacy scan

```text
$ node scripts/pii-scan.mjs
pii-scan: 1 finding(s) — see DATA_CONTRACT.md §Zero-Conditions

  [email] package-lock.json — <flagged address, redacted>

If a finding is a false positive (fictional data outside the sanctioned dirs),
move it under search/examples/ or resumes/ rather than allowlisting it here.
exit: 1

$ node scripts/pii-scan.mjs --diff origin/main
pii-scan: clean ✓
exit: 0
$ git diff --quiet origin/main -- package-lock.json && echo "package-lock.json unchanged vs origin/main"
package-lock.json unchanged vs origin/main
$ git grep -c "<flagged address, redacted>" origin/main -- package-lock.json
origin/main:package-lock.json:1
```

The working-tree finding is a package author's address inside an npm deprecation notice in upstream's `package-lock.json`. This branch does not modify that file. The address is redacted in this report: the first version pasted it verbatim, which re-created the finding inside this branch and failed the branch-history scan. That local commit was amended before anything was pushed (FRICTIONAL entry 16). The branch-history scan, which covers every line this branch commits, is clean.

**Other privacy checks:**
- No committed file contains a Windows user path. The search was confirmed to work by checking it against a known string first.
- Commits use the GitHub no-reply address, not a personal email.
- The persona has no résumé, contact details, or real email (`shyam@example.com`).
- Fixtures are fictional companies on `example.com`, with the phone and officer columns empty.

## Tests (offline, fixtures only, no network)

```text
$ node --test scripts/contrib/2026fa/contactshyam14-code-em-network-targets/network-targets.test.mjs
ok 1 - happy path: buckets come from the real scorer, not from this tool
ok 2 - roles.json never omits a gate, and held companies are never scored
ok 3 - every evidence value carries one of the three labels
ok 4 - record values are copied from the CSV, not computed or rounded
ok 5 - a tier explanation names only the condition that actually failed
ok 6 - liveness label follows who observed it and whether the board is confirmed
ok 7 - F1: OPT end already past → exit 3, nothing written
ok 8 - F2: SOC code with no row → exit 3, names the code, nothing written
ok 9 - F3: liveness observations that match no candidate are reported, never invented
ok 10 - F6: CSV missing a required column → exit 3, nothing written
ok 11 - a stale or failed liveness check holds the company instead of closing it
ok 12 - companies sharing an identical H-1B record are flagged for an identity check
ok 13 - BLS vocabulary: primary O*NET row by default; "all" pulls in QA titles
ok 14 - parity check is computed from the file, not hardcoded
ok 15 - out-dir outside this contribution is refused, nothing written
ok 16 - nothing checked yet → scorer is not run (no empty, NaN-rate report)
ok 17 - upstream characterization: the scorer treats a MISSING liveness gate as open
# tests 17
# pass 17
# fail 0
```

**Do the tests catch bugs?** Three bugs were injected into the prototype one at a time (`evidence/mutation-tests-2026-10-01.txt`, run against the 16-test suite of the time):

| Mutation | Tests that failed |
|---|---|
| M1: treat a missing liveness check as open | 3 |
| M2: label the tier as a record | 1 |
| M3: disable the output-folder guard | 1 |

With the guard off, M3 actually wrote five files into `data/examples/`, which broke the next clean run. Those files were moved out of the repository, and the test now uses a unique folder and cleans up after itself. Re-running M3 left no residue.

**Clean checkout:** a separate worktree of commit `a309e8c`, after `npm ci`. The tests passed, and the documented command reproduced run 03 except the `generated_at` timestamp.

## The real sample run

```text
$ node scripts/contrib/2026fa/contactshyam14-code-em-network-targets/network-targets.mjs --as-of 2026-10-01 --liveness scripts/contrib/2026fa/contactshyam14-code-em-network-targets/inputs/liveness.2026-10-01.json --out-dir course/2026fa/submissions/contactshyam14-code/runs/03-scored
✓ 57 candidates from 1557 H-1B rows (13-1082: 55, 11-3051: 2)
  network 4 · apply 1 · check-liveness 52 · skip 0
  scorer: ✓ scored 5 roles → Apply 1 · Consider 0 · Skip 4 (skip 80%)
  ! parity: all 1557 approval and denial counts are even — counts probably doubled upstream
  ! identity check needed: CONVEY INC, COVEY INC, LYNDRA THERAPEUTICS INC, LYRA THERAPEUTICS INC, SALESFORCE COM INC, SALESFORCECOM INC
  gate G4 visa-path: awaiting human sign-off
  wrote course/2026fa/submissions/contactshyam14-code/runs/03-scored/network-targets.json + network-targets.md + roles.json + role-scores.json/.md
```

**Cross-check of run 03 against the CSV** with an independent parser (Python's `csv` module): **30 of 30** values matched. Full output is in `evidence/cross-check-2026-10-01.txt`; the inputs, the scanner output, and the scorer's report are in WORKED-RUN.md.

## Each failure case, exercised on the real data

Full output: `evidence/failure-cases-2026-10-01.txt` (pasted in WORKED-RUN §5).

| Case | Command difference | Output | Exit | Written? |
|---|---|---|---|---|
| F1 OPT end already past | `--as-of 2027-01-15` | `STOP (G1): OPT end date 2026-12-31 is on or before the as-of date — refusing to score a window that has already closed` | 3 | no |
| F2 SOC code with no row | persona with 13-1028 | `STOP (G1): SOC code 13-1028 has no row in data/bls/compact/soc_occupation_compact.csv — refusing to guess a title list for it` | 3 | no |
| F3 observation for a company not in the CSV / not a candidate | `fixtures/break-liveness-unmatched.json` | `! liveness observations with no candidate: Quillfeather Assembly Works (not in the company CSV); 1LIFE HEALTHCARE INC (in the CSV but not a candidate …)` | 0 | yes; nothing invented |
| F4 board check fails | Zero Motorcycles slug in run 03 | held: `result "unchecked" is not a usable liveness result` | 0 | held, not closed |
| F5 no Form D record | every run 03 candidate | `not in shipped sample` (0 of 57 matched the 200 filings) | 0 | reported as absence from the sample |
| F6 missing CSV column | `Approval_Rate` renamed | `STOP (G1): company CSV lacks required column(s) "Approval_Rate": …` | 3 | no |
| Out-dir outside the contribution | `--out-dir data/examples/should-refuse` | `STOP (usage): --out-dir data/examples/should-refuse is outside this contribution's folders; refusing to write there` | 2 | no |
| Scorer on zero roles (prediction 3) | the scorer itself, on `[]` | `✓ scored 0 roles → Apply 0 · Consider 0 · Skip 0 (skip NaN%)` | 0 | the prototype never calls it with zero roles |

## Scope of the change

`git diff --name-only origin/main`, grouped. Every path is inside this contribution's assigned folders. No maintained file is modified, and `logs/RUN_LOG.md` is untouched.

```text
     31 course/2026fa/submissions/contactshyam14-code
      1 logs/runs/2026fa-contactshyam14-code-1.md
      1 recipes/cases/2026fa/contactshyam14-code-em-network-targets.card.md
      1 recipes/cases/2026fa/contactshyam14-code-em-network-targets.md
     15 scripts/contrib/2026fa/contactshyam14-code-em-network-targets
```

(49 files and 28,694 added lines at `34b0c7a`, before this report was added. Most of the lines are the machine-readable run logs in `runs/`. The full `git diff --stat origin/main` goes in the PR body.)

## What the gates require a person to judge

| Gate | What the person judges | State after this run |
|---|---|---|
| G2 Liveness | Is each board really the company's? Is a "nothing matching" reading right? Two of four networking targets (Genies, Senti) rest on unconfirmed boards | open for Genies and Senti |
| G4 Visa-path sign-off | With the DSO or an attorney: does any path exist past a December-2026 OPT end (cap-exempt employer, STEM eligibility, another status)? No application is tailored until then | **not signed** |
| G5 Identity | For each flagged candidate: which company owns the shared record? | none of the 5 scored companies is flagged; 6 held candidates are flagged |
| Sample-run adequacy | Has a named person read the run-03 report and found it fit for purpose? This sets the recipe's `last_gate` | **not yet** (Shyam) |
