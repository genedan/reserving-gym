# reserving-gym

Feature benchmarking for the `chainladder` (chainladder-python) package.

## Goal

Solve actuarial exam problems in `problems/` and use the experience to tell the
user how `chainladder-python` can be improved. Prefer `chainladder` for the
solutions. Where it can't do what a problem needs, solve it with pandas/numpy
and record the gap (see "Findings log").

- Problems are mostly CAS Exam 7 (Estimation of Policy Liabilities, Insurance Company
  Valuation, and ERM), organized as `problems/exam_7/<year>/<year>_<n>.ipynb`.
  There is also an Exam 5 problem (`problems/exam_5/2019_1.ipynb`, exposures,
  from Werner & Modlin ch. 4). The 2019 syllabus PDF covers all exams, and the
  user adds readings to `papers/` as needed (`Werner_Modlin_Ratemaking.pdf`,
  `Friedland_estimating.pdf` were added for Exam 5).
- The 2019 syllabus is the source of truth for what is in scope
  (`papers/2019syllabus.pdf`, Exam 7 section starts at printed page Exam 7-1).
- Use the readings in `papers/` as syllabus material.
- Packages may be installed as needed (`uv add <pkg>`).
- **Do not look up answers to the problems on the internet** (no searching for
  solutions, exam solutions, study-manual write-ups, flashcards, or the
  original exam text). The only knowledge source for solving a problem is the
  syllabus papers in `papers/` (plus the problem text itself). Web access is
  fine only for fetching the syllabus readings themselves.

## Environment

- Python project managed with `uv` (`pyproject.toml`, `uv.lock`, `.venv/`).
  Run code with `uv run ...`.
- Dependencies: `chainladder`, `pandas`, `ipykernel`.
- The user works in PyCharm. Notebooks are rendered by PyCharm's Markdown engine.

## Notebook conventions

- Standard Markdown has no `a. b. c.` lists. For lettered question parts, use
  `<ol type="a"><li>...</li></ol>` in markdown cells. (The existing
  `2019_5.ipynb` currently uses `* a. (1 point) ...` bullets, as edited by the
  user. Follow whatever the user last did in a notebook.)
- Not yet confirmed that PyCharm renders `<ol type="a">` correctly.
- Each notebook: markdown cell with the question text and point values, then
  code cells with the solution, then a final markdown section titled
  "chainladder-python gaps" listing what the package doesn't cover for that
  problem and a suggested new feature. Also add notable gaps to the Findings
  log below.
- After the gaps section, add a **"Sketch"** section: a runnable prototype of
  the suggested feature, written in the notebook (not in the package), as done
  in `problems/exam_7/2019/2019_2.ipynb` (`HurlimannCLR`). Include a markdown
  intro, the prototype class with a sklearn-style `fit(X, sample_weight=...)`
  and trailing-underscore attributes, a cell that uses it on the problem's
  data with asserts against the hand solution, and a note on extensions not
  prototyped. Name estimators after the author/method, as chainladder does
  (`ClarkLDF`, `MackChainladder`, `HurlimannCLR`). Accept the problem's own
  triangle form (incremental or cumulative) where the method allows.
  Layout in 2019_2.ipynb: gaps first, then the sketch.

## Papers (`papers/`)

The Exam 7 readings in this folder are publicly available. The CAS "7_*" files come from the 2021 text-reference set,
not 2019, so wording may differ slightly from what 2019 candidates had.

- `Patrik.pdf` is all of Chapter 7; only pp. 434-464 (reinsurance loss
  reserving) are assigned. `Taylor.pdf`: Chapters 1-3 only. Teng & Perkins:
  exclude Section 5. Feldblum discussion: Sections 1-2 only.
- Known errata not downloaded: Venter Table 3 (p. 823), Verrall eq. 5.4 (p. 63).
- Meyers 2015 is the original Monograph 1 (superseded by Monograph 8, but the
  2019 syllabus uses the original).

## Reporting

- Tell the user about anything that can't be found, is missing, or is
  incomplete (e.g. missing problem inputs) instead of guessing silently.
- Don't fabricate missing exam inputs; label assumed values clearly.

## Findings log (chainladder gaps)

Add new entries here as problems are solved.

- `ClarkLDF` (v0.10.1) exposes `omega_`, `theta_`, `elr_`, `scale_`,
  `pct_unreported_`, `norm_resid_` but no parameter covariance matrix, no
  process/parameter variance, and no standard error or CoV of reserves. Clark's
  variance calculations (the basis of 2019 Exam 7 Q5) are not available.
  Covariance/process-risk code in the package exists only for Mack.
- Audit of `ClarkLDF` vs Clark sections 2-4 using the paper's own triangle
  (Mack 1993 data, section 4): LDF-method omega/theta/sigma^2 match the paper
  (1.43434/48.621/65,030 vs 1.434294/48.6249/65,029; Weibull also matches). The paper triangle equals the `genins` sample except one cell (2001, 48 months: 2,218,270 vs 2,182,708 in the paper), so use `genins` in notebooks and, to match the paper's printed figures, set that cell to the paper's value (`tri.values[0, 0, 0, 3] = 2_182_708` on a copy).
  Cape Cod omega/theta/ELR match (1.44758/48.024/59.78%), and the 240-month
  truncated reserve matches to 0.005% when computed paper-style. Bugs/gaps
  found: (1) Cape Cod `scale_` is wrong, 53,822 vs paper 61,577, because
  `incremental_fits_` uses latest-diagonal ultimates, not ELR x premium (this
  also affects `norm_resid_`; fully documented with a reproduction at the
  bottom of `2019_5.ipynb`); (2) no covariance/variance outputs (sections
  2.3-2.4, 4.3); (3) no calendar-year development (4.3.2) or discounted
  reserves (4.3.3, App. C); (4) no Appendix B exposure-period adjustment
  (fixed mid-period age offsets); (5) optimizer uses numerical gradients,
  not Appendix A derivatives; (6) LDF-method truncated-at-240 reserve via
  TailClark is 29.20M vs paper 28.99M (0.7% off, cause not investigated).
  Check script: scratchpad `clark_check.py` (not in repo).
- `cl.Benktander` needs the a priori loss ratio as a user input and uses an
  LDF-based pattern. No premium-weighted loss-ratio payout pattern, no
  Huerlimann individual/collective loss ratio reserves, no optimal credibility
  weights (`Z = p/(p+sqrt(p))`), no MSE (Exam 7 2019 Q2).

## Status / open items

- `problems/exam_7/2019/2019_5.ipynb` is solved (Clark section 4.3.1; total CV
  12.1%, by hand with pandas because `ClarkLDF` has no variance output). Only
  the ELR variance is needed, so the earlier "missing growth function" concern
  was wrong. It is the only problem so far.
- `problems/exam_7/2019/2019_2.ipynb` is solved (Huerlimann: R_GB 1,751, R_c 1,786; the question's "2008" labels don't match the 2016-2018 data, so AY 2008 was read as the latest year, 2018).
- `problems/exam_7/2019/2019_3.ipynb` is solved (Huerlimann Neuhaus reserve total 254; part b: collective ELR assumption fails, 12-month loss ratios 40/50/70%).
- `problems/exam_7/2019/2019_6.ipynb` is solved (Clark Cape Cod with given omega/theta: ELR 60.4%, AY2018 fitted incremental 382.2, normalized residual -0.080, AY2016 ultimate 6,316 using truncation average age 114 (G(120) would give 6,415)). `ClarkLDF` cannot take fixed omega/theta/scale or a truncation age.
- Need more problems from the user.
- `.gitignore` covers `.venv/`, `.idea/`, `local/` and `CLAUDE.local.md` (personal files).
