# PathReview contribution journal

Student: Blayne Montaque (`blayne-montaque`)

Course: CodePath AI201 / Howard University CSCI 485

Recorded: November 7, 2026 (America/New_York)

Setup completed and final Unit 7 audit recorded: November 8, 2026 (America/New_York).

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
| Can the behavior be reproduced reliably? | The detector takes a plain list of paths and returns a `ToolResult`; no GitHub request, LLM, database, or vector store is needed for the focused behavior. `test_node_modules_excluded` and `test_build_directory_excluded` already carry strict xfail markers naming #57. Selection was based on source/test inspection; the later full-suite setup baseline is recorded below. No separate Unit 8 reproduction artifact or commit was created. |
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
| [#1 — Duplicate embeddings generated when re-ingesting the same repository](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/1) | `tier-2` | `_check_skip()` in `ingestion/pipeline.py` queries a string and catches exceptions; the issue points to `core/models/ingested_source.py`. The issue estimates 4–6 hours. Database/session and ingestion-state behavior increase integration effort and regression risk, especially with the Docker blocker present at selection time. No focused pipeline unit-test file was found in `tests/unit/`. |

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
- [x] Frontend accessible at `http://localhost:5173`: HTTP 200, HTML contains `PathReview` and `/@vite/client`; verified again after `make run` on November 8.
- [x] Docker-backed development setup and `make setup` completed successfully using the documented local compatibility adjustments below.
- [x] Backend verified: `/docs` and `/openapi.json` return HTTP 200, with API title `PathReview API`. Seeded login and authenticated profile/review reads return HTTP 200.
- [x] All three Compose services are healthy; direct database, Redis, and Chroma probes pass.
- [ ] The existing `/health` route returns HTTP 200: it still returns 503 due to upstream issues #61 and #62, despite successful independent dependency probes. Those unrelated defects were not changed.

The first November 7 frontend check used standalone Vite and did not establish full setup. The November 8 checks instead followed a successful `make setup` and actual `make run`, including database migrations and seed data. Verification used HTTP, existing tests, a production build, and direct service queries; no browser-rendered screenshot or live-LLM/end-to-end review-generation evidence is claimed.

| Prerequisite | Actual result |
| --- | --- |
| Git | `git --version`: 2.49.0.windows.1; meets 2.39 minimum. |
| Git Bash | `C:\Program Files\Git\bin\bash.exe`, GNU Bash 5.2.37; used for the documented Docker/Make command attempts. |
| Python | Installed Python 3.11.9 for the current user at `C:\Users\Blayne\AppData\Local\Programs\Python\Python311\python.exe` following explicit approval. Recreated this project's `.venv` with it and verified both its version and `sys.base_prefix`. The earlier Python 3.13 environment is backed up outside the fork in `../.pathreview-setup/venv-python313-before311`. Project dependencies are installed; `pip check` passes. Default Python remains 3.10; global PATH, launcher, file associations, and unrelated environments were not changed. |
| Node.js | `node --version`: v22.14.0; meets 18 minimum. |
| npm | `npm.cmd --version` and Git Bash `npm --version`: 10.9.2; meets 9 minimum. PowerShell's `npm.ps1` was blocked by execution policy; `npm.cmd` works without changing that policy. |
| Docker / Compose | Docker Desktop 4.98.0 installed per-user at `%LOCALAPPDATA%\Programs\DockerDesktop`, using the WSL 2 backend. CLI and server report Docker 29.9.2; Compose reports v5.6.1. `docker info` succeeds. |
| WSL | Installed WSL 3.0.3; `wsl --version` reports 3.0.3.0 and kernel 6.18.54.1-1. After the student's restart, `wsl --status` reports default version 2 and `wsl --list --verbose` shows `docker-desktop` Running with VERSION 2. |
| GNU Make | Installed the guide's `GnuWin32.Make` package and verified GNU Make 3.81 at `C:\Program Files (x86)\GnuWin32\bin\make.exe`. The project command wrapper adds it to PATH only for its own process. |
| Memory | Read-only Windows hardware query reports approximately 61.6 GiB RAM; exceeds 8 GB minimum. |
| Disk space | Read-only Windows query reports approximately 679.6 GiB free on D:, exceeding the 20 GB minimum. |

**Initial commands and observed results — November 7, before prerequisite installation:**

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
- G19 (PR link) remains blank, appropriate for Unit 7. H19 was initially left blank while the comment was pending; after the student posted the comment, readback verified H19 is now `Claimed`. Row 19 was rechecked on November 8 and still contains the correct identity, issue, title, tier, URL, and status. No duplicate row was added.
- Neighboring rows, formula outputs, and validation were preserved. Connector readback verified values and formatting metadata; native visual fit was not verified because browser automation was unavailable.

**Issue-claim status:** VERIFIED — the student posted the claim comment as `blayne-montaque`.

Comment permalink: https://github.com/codepath/pathreview-ai201-fa26-howard/issues/57#issuecomment-6714471166

GitHub readback confirms the comment identifies Blayne Montaque from Howard's AI201 cohort and describes the intent to work on vendored/build-output exclusion with regression and edge-case tests. The earlier connector request failed with HTTP 403 and browser fallback was unavailable; those failed attempts were superseded by the student's verified post. No assignee change is claimed or required.

### Historical setup checkpoint — November 7 restart

On November 7, the student explicitly authorized WSL 2, Docker Desktop, GNU Make, required dependencies, and administrator elevation, requested Python 3.11, and instructed that setup stop if a Windows restart was needed. That authorization supersedes the earlier pending installation permission.

Completed and verified:

1. `winget install --id Python.Python.3.11 --exact --source winget --scope user --silent --accept-source-agreements --accept-package-agreements --disable-interactivity --override '/quiet /norestart InstallAllUsers=0 PrependPath=0 AppendPath=0 Include_launcher=0 Include_test=0 AssociateFiles=0 Shortcuts=0'` succeeded with Python 3.11.9. Installer hash verification succeeded. The options preserve existing Python defaults and launcher behavior.
2. After verifying that both paths were within the Project 7 workspace and that the backup did not exist, moved only this fork's `.venv` to `../.pathreview-setup/venv-python313-before311`. Created the replacement using the installed Python 3.11 interpreter with `-m venv`; `.venv/Scripts/python.exe` reports 3.11.9 and the Python311 base installation.
3. `winget install --id GnuWin32.Make --exact --source winget --silent --accept-source-agreements --accept-package-agreements --disable-interactivity --override '/VERYSILENT /SUPPRESSMSGBOXES /NORESTART /SP-'` succeeded. Installer hash verification succeeded; the installed executable reports GNU Make 3.81.
4. Ran `wsl.exe --install --no-distribution --web-download` with administrator elevation. This installed WSL 3.0.3 without an additional Linux distribution and enabled the required Virtual Machine Platform component. The installer exited 0 but explicitly reported: **"Changes will not be effective until the system is rebooted."** The feature query changed from disabled (state 0) to enable-pending (state 2). This is a required restart, not proof that WSL can run workloads yet.

That setup attempt stopped at the restart message. No automatic Windows restart was requested or performed. Docker installation, application setup, and baseline checks were deferred until the student restarted Windows and authorized resuming. This checkpoint is preserved as history; the successful follow-up is recorded below.

The Docker Desktop 4.98.0 download, started while WSL was installing, completed successfully. The official installer is staged at `../.pathreview-setup/DockerDesktopInstaller.exe`; its SHA-256 matches winget's published value `cd6273d9eaa4f07fa7593fc6813d37e1004381cebc3f26047a2554fa5db24388`, and its Authenticode signature is valid with Docker Inc. as signer. It was executed successfully after the restart using `install --user --quiet --accept-license --backend=wsl-2` (exit 0).

Logs are outside the fork in `../.pathreview-setup/`, including `python311-winget.log`, `make-winget.log`, `wsl-install.log`, `wsl-install.exitcode`, and before/after virtualization-feature JSON. Only `JOURNAL.md` is intended for this follow-up commit. No existing source, tests, issue markers, or dependency manifests were changed.

Installation options were checked against the [Python Windows installation guide](https://docs.python.org/3.11/using/windows.html), [Microsoft WSL command reference](https://learn.microsoft.com/en-us/windows/wsl/basic-commands), and [Docker Desktop Windows guide](https://docs.docker.com/desktop/setup/install/windows-install/).

### Completed setup and baseline — November 8

The existing branch and Python 3.11 environment were reused. Docker Desktop was started with its WSL 2 engine, and the existing named volumes were retained. On resuming after the overnight pause Docker was no longer running; it was restarted before the final successful setup. An earlier retry while Docker was down failed to connect to PostgreSQL and is not counted as successful.

**Local compatibility adjustments (outside the fork):**

- The unmodified `chromadb/chroma:0.4.22` image runs `pip install --force-reinstall --no-cache-dir chroma-hnswlib` on startup. That installed NumPy 2.2.6 and crashed on the removed `np.float_` alias. The local constraints file keeps NumPy below 2 and Chroma's original HNSW dependency at 0.7.3.
- The image also lacks `curl`, which the repository's health probe invokes. A local Compose override uses Python's standard-library HTTP client to probe the same `/api/v1/heartbeat` endpoint. It does not replace a real probe with unconditional success.
- The initial `make setup` applied migrations and committed seed data, then failed while printing a Unicode checkmark through Windows CP1252. Setting `PYTHONUTF8=1` and `PYTHONIOENCODING=utf-8` for the process resolved this. No database reset was needed; the successful retry detected the existing seed users.
- Repository source, `docker-compose.yml`, Makefile, `.env`, dependency manifests, tests, and xfail markers were preserved. A before/after SHA-256 comparison confirmed `.env` was unchanged. `git status` was clean before editing this journal.

Local `../.pathreview-setup/chroma-constraints.txt`:

```text
numpy<2
chroma-hnswlib==0.7.3
```

Local `../.pathreview-setup/compose.compat.yml` (volume source resolves relative to the repository's base Compose file):

```yaml
services:
  vector-db:
    environment:
      PIP_CONSTRAINT: /pathreview-constraints.txt
    volumes:
      - ../.pathreview-setup/chroma-constraints.txt:/pathreview-constraints.txt:ro
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8000/api/v1/heartbeat', timeout=4).read()"]
```

The successful commands used Git Bash and process-local tool selection, equivalent to the following from the repository root:

```bash
export PATH="/c/Users/Blayne/AppData/Local/Programs/Python/Python311:/c/Program Files (x86)/GnuWin32/bin:/c/Users/Blayne/AppData/Local/Programs/DockerDesktop/resources/bin:$PATH"
export PYTHONUTF8=1 PYTHONIOENCODING=utf-8
docker compose -f docker-compose.yml -f ../.pathreview-setup/compose.compat.yml up -d --wait --wait-timeout 180
make setup
make run
```

The existing `../.pathreview-setup/project-command.sh` wrapper applies these PATH/UTF-8 settings for a single command and changes into the fork. It can be invoked from the fork with `bash ../.pathreview-setup/project-command.sh make run`. Docker Desktop must be running first, and the Compose override must be included when recreating the services. These files are local setup support; their full relevant configuration is documented here so the pushed journal remains reproducible. No global shell profile or default Python was changed.

**Verified runtime evidence:**

| Check | Actual result |
| --- | --- |
| `docker info` and WSL distro list | Engine 29.9.2 reachable; `docker-desktop` Running, WSL version 2. |
| Compose startup with local override | Exit 0; PostgreSQL, Redis, and Chroma all healthy. |
| `make setup` with Python 3.11/UTF-8 | Exit 0; dependency install, pre-commit hook, migrations, seed check, and frontend installation complete. |
| PostgreSQL | Direct `SELECT 1` returns 1; 3 users, 3 profiles, 6 reviews; Alembic revision `002`. |
| Redis | Direct client ping using the configured Redis URL returns true. |
| Chroma | `http://localhost:8001/api/v1/heartbeat` returns HTTP 200. |
| `make run` | Vite and Uvicorn started; backend log records application startup complete. Servers were left running. |
| Frontend | `http://localhost:5173` returns HTTP 200 with PathReview HTML and the Vite client. |
| Backend docs/schema | `/docs` and `/openapi.json` return HTTP 200; schema title is `PathReview API`. |
| Authentication/data reads | Seeded account login returns HTTP 200 and a token; authenticated GET of its profile and `/reviews` return HTTP 200. No token was printed or committed. An exploratory GET `/profiles` returned 405 because that collection route supports POST; the actual documented GET `/profiles/{profile_id}` succeeded. |
| Existing health endpoint | `/health` returns 503. Logs show the raw SQL error described by [#61](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/61) and missing `redis_host` described by [#62](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/62). Independent dependency probes above pass. These issues remain untouched. |

**Baseline checks (existing, unmodified code):**

| Command | Actual result |
| --- | --- |
| `make lint` | PASS, exit 0. |
| `.venv/Scripts/black --check .` | PASS, exit 0; 110 files would be unchanged. Used the non-writing check rather than `make format`, which writes files. |
| `make typecheck` | PASS, exit 0; no issues in 76 source files. Together with the preceding checks, this covers the checks behind `make check` without rewriting source. |
| `make test-unit` | PASS, exit 0; **375 passed, 53 xfailed, 2 warnings**, 9.95 seconds. Expected-failure markers, including #57, remain unchanged. Existing Pydantic deprecation and async-mock warnings are recorded in the log. |
| `make test-integration` | No tests collected: pytest exit 5, Make exit 2. `tests/integration/` contains only `__init__.py`; upstream CI explicitly accepts pytest exit 5 for this empty suite. This is not claimed as passing integration coverage. |
| `npm --prefix frontend test -- --run` | **17 passed, 1 timed out** at the default 5,000 ms: `ProfileForm > validates portfolio URL character limit`, which types 501 characters. |
| `npm --prefix frontend test -- --run --testTimeout=30000` | PASS, exit 0; **18 passed**, 2 test files, 9.60 seconds. This was a diagnostic command-line timeout override only; no test configuration was edited. The default-timeout failure remains disclosed. |
| `npm --prefix frontend run build` | PASS, exit 0; TypeScript and Vite build completed. Vite warned that the shared API module's mixed static/dynamic imports do not create a separate chunk. |
| `.venv/Scripts/python.exe -m pip check` | PASS; no broken requirements. |

`npm install` reported 17 dependency advisories (1 low, 6 moderate, 8 high, 2 critical). No `npm audit fix`, forced dependency upgrade, unrelated code repair, or source formatting was performed. No remote CI run or all-green PR status is claimed.

Key installed versions: Python 3.11.9, FastAPI 0.162.2, SQLAlchemy 2.1.8, Uvicorn 0.54.0, pytest 9.2.0, Ruff 0.17.4, Black 26.10.1, mypy 2.4.1, Vite 5.4.21, and Vitest 1.6.1. The host Chroma Python package resolved to 1.5.9; the Compose server remains the specified 0.4.22. Live vector ingestion and LLM review generation were not exercised, so heartbeat success is not represented as full client/server workflow validation.

Evidence logs and the installed Python package snapshot remain local in `../.pathreview-setup/`: `make-setup-final.log` (exit 0), `compose-resume.log`, `make-run.stdout.log`, `make-run.stderr.log`, `baseline-*.log` with corresponding exit-code files, and `python311-installed.txt`. Logs and environments are not included in the Git commit.

**Remaining student action:** submit the existing working branch URL in the course portal. No portal submission was performed. The local baseline limitations above remain documented for future units; fixing them is not part of this Unit 7 run.

**Unit 7 audit:**
- [x] Existing fork/history preserved; correct origin and upstream verified.
- [x] Live Tier 1/Tier 2 comparison and relevant source/test inspection completed.
- [x] Exact open issue, title, tier, selection reasoning, and baseline documented.
- [x] Correct working branch created from main; required journal is at the repository root.
- [x] Required tools installed, Python 3.11 environment used, and actual setup/startup verified.
- [x] Frontend and backend accessibility confirmed; service health independently verified.
- [x] Baseline checks run and limitations recorded without altering seeded exercises.
- [x] Howard ledger identity/issue entry written and verified.
- [x] GitHub claim comment posted by the student and verified by readback.
- [x] Docker services, `make setup`, `make run`, and backend verified with documented local setup adjustments.
- [ ] Course portal submission completed by student.

All technical Unit 7 requirements in the supplied assignment are complete; the course-portal submission remains the student's action. This audit is against the supplied requirements, not a claimed numerical grade or an inaccessible detailed portal rubric. No issue-specific reproduction artifact/commit, `PLAN.md`, fix, test edits, PR, or Unit 8 work was performed. AI assistance was used for inspection, selection analysis, environment setup, baseline verification, ledger registration, and this evidence-based journal; the student posted the GitHub claim comment.
