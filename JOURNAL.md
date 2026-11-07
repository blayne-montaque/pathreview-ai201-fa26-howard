# PathReview contribution journal

Student: Blayne Montaque (`blayne-montaque`)

Course: CodePath AI201 / Howard University CSCI 485

Recorded: November 7, 2026 (America/New_York)

## Unit 7 — Issue selection

**Issue link:** https://github.com/codepath/pathreview-ai201-fa26-howard/issues/57

**Issue title:** Tech detector counts vendored and build-output files, skewing language detection

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3 — verified GitHub label: `tier-1`.

**Problem summary:**
The agent subsystem's `TechDetector` includes root-level dependency and generated files when identifying a repository's technology stack. Its `_should_skip_file()` patterns contain leading slashes, so repository-relative paths such as `node_modules/lib/index.js` and `build/bundle.js` escape exclusion, while equivalent nested paths can match. The issue's supplied example therefore reports JavaScript for Python source accompanied by vendored or bundled JavaScript; the expected result is Python with those excluded paths contributing no technologies. Filtering should apply consistently before both extension and configuration-file detection. Source inspection also shows that `primary_language` currently comes from alphabetically sorted detected languages, not file-frequency counting, so changing that separate selection policy is outside the intended issue scope.

**Issue selection reasoning:**
Issue #57 is bounded but substantive: it affects a real analysis result, has two explicit existing regression tests, and supports useful path-boundary and false-positive checks. The reasoning below uses the suitability dimensions in the assignment and master spec. The linked course checklist and detailed grading page could not be retrieved, so this is not represented as a verbatim review of those inaccessible pages or a numerical grade prediction.

| Suitability question | Evidence and assessment |
| --- | --- |
| Is the problem and expected behavior clear? | The issue supplies an eight-path input and expected Python output. Source inspection identifies the leading-slash mismatch in `_should_skip_file()`. |
| Can the behavior be reproduced reliably? | The detector takes a plain list of paths and returns a `ToolResult`; no GitHub request, LLM, database, or vector store is needed for the focused behavior. `test_node_modules_excluded` and `test_build_directory_excluded` already carry strict xfail markers naming #57. This is source/test evidence of reproducibility, not a claim that a local reproduction was run during Unit 7. |
| Are relevant code and tests available? | `agent/tools/tech_detector.py`, `agent/tools/base.py`, and `tests/unit/test_tech_detector.py` exist. `_detect_tech()` uses the same filtered list for language extensions and configuration indicators. |
| Is the effort feasible? | Estimated by this review: roughly 3–5 hours for later investigation, a focused change, meaningful edge-case tests, and checks. This is an estimate, not tracked work or an issue-provided estimate. |
| What risks need attention? | Root versus nested paths, exact directory components versus similar names, excluded configuration files, all-excluded input, and preservation of ordinary source files. Windows separator behavior needs an explicit decision based on the actual caller contract. Preserve the result schema and unrelated language-selection behavior. |
| Is the issue available? | Before registering: GitHub showed open, no assignees, and no comments; the full Howard username/issue columns contained no #57 or `blayne-montaque`; Issue Catalog showed zero claims. The repository's seven returned open/closed PRs contained no #57 title, body, or branch reference. Availability is a dated observation, not exclusive ownership. |
| Can this become a clean PR? | A later contribution should be concentrated in the detector and its unit tests. Only #57's xfail markers should be removed when its fix actually passes. No implementation or marker removal occurred in this run. |

Candidate comparison based on live GitHub issues, ledger entries, and inspected source/tests:

| Candidate | Actual tier | Scope, reproduction, effort, and tradeoff |
| --- | --- | --- |
| [#57 — Tech detector counts vendored and build-output files, skewing language detection](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/57) | `tier-1` | Selected. Deterministic path-list input, existing strict xfail tests, one principal module, low coupling, and worthwhile boundary coverage. No other claim found. |
| [#69 — Output parser crashes on a top-level JSON array fallback](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/69) | `tier-1` | `rag/generator/output_parser.py` calls `.items()` on parsed JSON; `tests/unit/test_output_parser.py::TestOutputParser::test_json_array_fallback` covers the array case. The issue estimates 2–4 hours. Bounded and unclaimed in the checked ledger/comments, but defining fallback semantics across raw and fenced output adds contract questions. |
| [#24 — Injection detection misses role-switch lines with whitespace before the colon](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/24) | `tier-2` | `safety/prompt_defense.py` requires an adjacent colon; `test_whitespace_variations_detected` already exists in `tests/unit/test_prompt_defense.py`. The issue estimates 4–6 hours. Deterministic and unclaimed in the checked ledger/comments, but regex changes require careful false-positive and whitespace-boundary coverage in safety behavior. |
| [#1 — Duplicate embeddings generated when re-ingesting the same repository](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/1) | `tier-2` | `_check_skip()` in `ingestion/pipeline.py` queries a string and catches exceptions; the issue points to `core/models/ingested_source.py`. The issue estimates 4–6 hours. Database/session and ingestion-state behavior increase integration effort and regression risk, especially with the current Docker blocker. No focused pipeline unit-test file was found in `tests/unit/`. |

Issues #54, #56, #60, #62, #64, #65, and #67 already had other students' ledger entries; #62 and #72 also had open PRs. They were not selected. The ledger explicitly permits shared issues, but spreading out avoids unnecessary overlap.

**Branch name:** `fix/57-tech-detector-exclude-generated`

**Branch URL:** https://github.com/blayne-montaque/pathreview-ai201-fa26-howard/tree/fix/57-tech-detector-exclude-generated

**Repository baseline and conventions:**
The existing fork was cloned into the empty Project 7 workspace; no new GitHub repository was created. Local `main`, `origin/main`, and fetched `upstream/main` all resolved to `14578ac9708dcbd94c3258430baddeae621c7e40`. The working branch was created from that `main` with `git switch -c fix/57-tech-detector-exclude-generated main`. Its name follows `docs/CONTRIBUTING.md`; the Unit 7 journal commit uses `docs(agent): record Unit 7 issue selection and setup`.

```text
origin   https://github.com/blayne-montaque/pathreview-ai201-fa26-howard.git
upstream https://github.com/codepath/pathreview-ai201-fa26-howard.git
```

Read `README.md`, `docs/SETUP.md`, `docs/CONTRIBUTING.md`, `docs/ARCHITECTURE.md`, `docs/adr/003-agent-orchestration.md`, the Makefile, configuration, and PR template. ADR-003 places the detector within the plan-execute agent. Generic `codepath/pathreview` URLs in setup documentation were not used in place of the Howard repository. No files or history in Projects 1–6 were changed.

**Setup confirmation:**
- [x] Frontend accessible at `http://localhost:5173`: HTTP 200, HTML contains `PathReview` and `/@vite/client`.
- [ ] Full application setup verified: blocked by missing Docker/Compose, WSL, and GNU Make.
- [ ] Backend verified: requests to `http://localhost:8000/docs` and `/health` failed to connect.

The frontend was started independently with Vite after installing locked dependencies; this does **not** establish successful `make setup`, `make run`, API startup, database migrations, seed data, or end-to-end functionality. Browser-rendered verification was unavailable because Computer Use stopped when it could not confidently determine the current browser URL. No screenshot evidence is claimed.

| Prerequisite | Actual result |
| --- | --- |
| Git | `git --version`: 2.49.0.windows.1; meets 2.39 minimum. |
| Git Bash | `C:\Program Files\Git\bin\bash.exe`, GNU Bash 5.2.37; used for the documented Docker/Make command attempts. |
| Python | Default `python --version`: 3.10.0, below requirement. Found and verified `C:\Users\Blayne\AppData\Local\Programs\Python\Python313\python.exe`: 3.13.2. Created the ignored project `.venv` with this interpreter; `.venv\Scripts\python.exe --version` confirms 3.13.2. Full project dependencies are not installed and their compatibility is not yet verified. |
| Node.js | `node --version`: v22.14.0; meets 18 minimum. |
| npm | `npm.cmd --version` and Git Bash `npm --version`: 10.9.2; meets 9 minimum. PowerShell's `npm.ps1` was blocked by execution policy; `npm.cmd` works without changing that policy. |
| Docker / Compose | Commands not found on PATH or at the standard Docker Desktop installation location. No Docker services started. |
| WSL | `wsl --status`: Windows Subsystem for Linux is not installed. |
| GNU Make | Not found on PATH or in the checked GnuWin32/MSYS installation paths. |
| Memory | Read-only Windows hardware query reports approximately 61.6 GiB RAM; exceeds 8 GB minimum. |
| Disk space | Read-only Windows query reports approximately 679.6 GiB free on D:, exceeding the 20 GB minimum. |

**Commands and observed results:**

```text
git clone https://github.com/blayne-montaque/pathreview-ai201-fa26-howard.git pathreview-ai201-fa26-howard
  Initial sandbox network attempt failed; approved network retry succeeded.
git remote add upstream https://github.com/codepath/pathreview-ai201-fa26-howard.git
git fetch upstream
  Succeeded; fork and upstream main are identical at the baseline above.
```

No `.env` existed in the new clone. It was copied from `.env.example` only after an existence check. `.env`, `.venv`, and `frontend/node_modules` are ignored by Git, and `.env` is not tracked. No credentials were printed or committed. The copied example uses `LLM_PROVIDER=mock`; `core/config.py` supports this no-key configuration. The README's OpenRouter guidance differs from the example (tracked separately as #75), so no fabricated API key or unrelated source change was introduced. Live AI behavior remains unverified.

The following were actually attempted through Git Bash from the repository root:

```text
docker --version          -> docker: command not found
docker compose version    -> docker: command not found
docker compose up -d      -> docker: command not found
docker compose ps         -> docker: command not found
make --version            -> make: command not found
make setup                -> make: command not found
make run                  -> make: command not found
```

Independent setup completed:

```powershell
# Repository root; explicit existing Python 3.13 interpreter
& 'C:\Users\Blayne\AppData\Local\Programs\Python\Python313\python.exe' -m venv .venv
.\.venv\Scripts\python.exe --version
# Python 3.13.2

# frontend directory
npm.cmd ci --no-audit --no-fund
# Exit 0; 324 packages installed, lockfile unchanged.
```

A hidden Node process was launched from `frontend/` with arguments `node_modules/vite/bin/vite.js --host 127.0.0.1 --port 5173 --strictPort`. Vite 5.4.21 reported ready; its stdout/stderr logs are outside the fork in the Project 7 workspace. HTTP checks used `Invoke-WebRequest -UseBasicParsing -TimeoutSec 10` against the frontend and both backend endpoints above.

**Cohort ledger:** [x] Issue added to the correct Howard cohort ledger and read back.

- Ledger: https://docs.google.com/spreadsheets/d/1-39i16vRXrVwGKUHwxWfg2w_UAgsmxUYaR1oJ2djH8w/edit#gid=739942384
- Before the edit, a bounded read of `Howard!A1:C1000` showed no existing username entry or #57 entry. `Howard!A19:H19` was checked for values and validation immediately before writing.
- Wrote only `Howard!A19:C19`: `Blayne Montaque`, `blayne-montaque`, numeric `57`.
- Readback verified those values and the automatically populated title, tier `1`, and correct issue URL in D19:F19. The Issue Catalog's #57 claim count changed from 0 to 1 because it counts registered rows.
- G19 (PR link) and H19 (Status) remain blank. H19 allows `Claimed`, `In progress`, `PR open (Unit 9)`, `Complete (Unit 10)`, or `Dropped / switched`; it was deliberately not set to `Claimed` because the issue comment has not been posted.
- Neighboring rows, formula outputs, and validation were preserved. Connector readback verified values and formatting metadata; native visual fit was not verified because browser automation was unavailable.

**Issue-claim status:** PENDING MANUAL POST — not claimed on GitHub.

The authenticated GitHub profile is `blayne-montaque`. The authorized comment request returned HTTP 403, `Resource not accessible by integration`. A subsequent comments read still returned an empty list. Browser fallback stopped before interaction because it could not determine the current browser URL confidently. No GitHub comment or issue assignment was made.

Exact comment to post on issue #57:

> I'm Blayne Montaque (@blayne-montaque) from Howard AI201 / CSCI 485, and I'm working on this issue for the PathReview project. I'll focus on excluding vendored and build-output paths from technology detection and validating the behavior with the existing tests plus focused edge cases.

After posting, verify the comment permalink and set only `Howard!H19` to `Claimed`; the ledger identity/issue entry is already complete.

**Remaining Unit 7 setup actions:**
1. Install/enable WSL 2 and Docker Desktop, then use the WSL 2 backend; a Windows administrator prompt or restart may be required. Install GNU Make using the setup guide's `winget install GnuWin32.Make` and expose its bin directory to Git Bash. System configuration was not changed automatically: the master spec explicitly says to avoid doing so without student approval.
2. In Git Bash, select the existing Python 3.13 installation for the session (the default is still 3.10), or activate the created environment before setup. Preserve the existing `.env`.
3. From the repository root, run `docker compose up -d`, confirm all configured services with `docker compose ps` (PostgreSQL, Redis, and Chroma), and then run `make setup`. Resolve actual dependency/platform failures if any; none are preemptively marked successful here.
4. Stop the standalone Vite process before `make run` so port 5173 is available. Verify the frontend, `http://localhost:8000/docs`, and backing services; record actual results in this Unit 7 entry and commit/push the update.
5. Post the claim comment, update H19, and submit the working branch URL through the course portal. No portal submission was performed.

**Unit 7 audit:**
- [x] Existing fork/history preserved; correct origin and upstream verified.
- [x] Live Tier 1/Tier 2 comparison and relevant source/test inspection completed.
- [x] Exact open issue, title, tier, selection reasoning, and baseline documented.
- [x] Correct working branch created from main; required journal is at the repository root.
- [x] Frontend HTTP accessibility confirmed; environment blockers recorded honestly.
- [x] Howard ledger identity/issue entry written and verified.
- [ ] GitHub claim comment posted and verified.
- [ ] Docker services, `make setup`, `make run`, and backend verified.
- [ ] Course portal submission completed by student.

Unit 7 remains incomplete until the claim and full setup blockers are resolved. No issue-specific reproduction run/commit, `PLAN.md`, fix, test edits, PR, or Unit 8 work was performed. AI assistance was used for inspection, selection analysis, setup attempts, ledger registration, and this evidence-based journal.
