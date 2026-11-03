# Portfolio interview guide

This guide summarizes the public repositories reviewed on 3 November 2026. It separates personal projects from team work and describes only features visible in the checked-out source and documentation. Test commands are instructions; they are not claims that every command was run for this review.

## MiniDB — personal C++20 storage-engine study

**Problem and design.** MiniDB is a persistent key-value store for learning storage internals. A custom hash table holds values in memory. Snapshots use a versioned binary format; a write-ahead log (WAL) records mutations between snapshots. Recovery loads the snapshot and replays valid WAL records newer than its checkpoint. The LRU cache speeds repeated reads, and the transaction overlay stages changes before commit. A per-database synchronization design protects library operations.

**Technologies:** C++20, CMake, CTest; no third-party runtime dependencies.

**Limits.** The documentation explicitly does not promise physical power-loss durability, cross-process coordination, MVCC, or isolation between concurrent transaction actors. Transactions are a single-process educational feature.

**Tests:** `cmake -S . -B build`, `cmake --build build`, `ctest --test-dir build --output-on-failure`. Tests cover parser, storage, WAL, recovery, transactions, cache, and concurrency. The separate [`audit/minidb-reliability-final` branch](https://github.com/Aarya801/MiniDB/tree/audit/minidb-reliability-final) adds an [audit document](https://github.com/Aarya801/MiniDB/blob/audit/minidb-reliability-final/docs/FINAL_RELIABILITY_AUDIT.md) and [Windows CI workflow](https://github.com/Aarya801/MiniDB/blob/audit/minidb-reliability-final/.github/workflows/windows-reliability.yml); it is not part of `main` and should be reviewed as a proposed PR, not described as merged.

**Likely questions**

- *Why use both a snapshot and a WAL?* The snapshot gives recovery a compact base state; the WAL captures later changes without rewriting the whole database after each mutation.
- *What does the checkpoint do?* It identifies the highest WAL sequence already represented by the snapshot, so recovery can skip those records after a save/reset interruption.
- *What does “flushed” guarantee?* The implementation flushes to the operating system. It does not call this physical power-loss durability because filesystem and device behavior can still lose buffered data.
- *Are transactions isolated?* No. The transaction overlay groups a caller's staged writes, but the project does not claim MVCC or isolation from other concurrent actors.

## ASTRA-66 — personal engineering software project

**Problem and design.** ASTRA-66 studies how a single parameter source can drive an OpenSCAD airframe, CAD-derived checks, engineering calculations, an illustrative flight simulation, data-only avionics software, and reproducible validation. Its documentation labels values by evidence class: CAD-verified, calculated, assumed, placeholder, or requiring measurement.

**Technologies:** Python, OpenSCAD, generated CAD artefacts, browser visualization, and CI validation.

**Limits and safety.** The design has not been built or flown. Propulsion and avionics inputs include placeholders or synthetic data. Simulation output is illustrative, not measured performance. It is educational software, not flight certification or flight readiness.

**Tests:** Run `python run_validation.py` as documented in [the reproducibility guide](https://github.com/Aarya801/ASTRA-66/blob/main/documentation/REPRODUCIBILITY.md). The pipeline covers software-level checks; it cannot establish physical fit, material strength, flight behavior, or hardware safety.

**Likely questions**

- *How do you keep CAD and analysis consistent?* A generated parameter file supplies shared values, while validation checks CAD-derived geometry against expected properties and downstream calculations.
- *Which results are measured?* CAD checks measure rendered model geometry. They do not measure a physical airframe; mass, fit, and hardware results still require physical tests.
- *What does simulation tell you?* It checks a simplified model against its assumptions and supports sensitivity analysis. Placeholder propulsion means its trajectory is not evidence about a real motor or flight.

## AI Travel Analyst — personal Python data project

**Problem and design.** This project cleans a supplied flight-pricing dataset, derives duration, stop, passenger, and date features, trains a CatBoost model, and presents analysis through Streamlit. A separate booking-window analysis compares historical prices across lead-time groups.

**Technologies:** Python, pandas, NumPy, scikit-learn, CatBoost, Plotly, Streamlit, pytest.

**Limits and evaluation status.** `src/modeling.py` trains on every row with a known price and saves a model; it does not implement the README's claimed 80/20 test split or compute the published metrics. The checked-in `artifacts/metrics.json` stores numbers without enough evaluation provenance to call them held-out performance. The dataset has a departure date and booking lead-time field, but no verified booking timestamp; this prevents a faithful prospective, chronological validation of what would have been known at booking time. Historical booking-window medians are descriptive and do not predict future fares.

**Tests:** `pytest -q` checks basic cleaning and booking-window output. These tests do not validate model quality. See the [test source](https://github.com/Aarya801/vit-chennai-ai-travel-analyst/blob/main/tests/test_pipeline.py). A valid model evaluation still needs a documented split, a training-only fit, a test-only score, and a baseline computed on the same split.

**Likely questions**

- *Why CatBoost?* It supports categorical features directly and can model nonlinear interactions without requiring a large one-hot matrix.
- *Can you defend the current MAE and R² as test results?* No. Those values are stored in an artifact, but the checked-in training/evaluation path does not show how the evaluation split was made. I would first implement a reproducible split and report its limitations.
- *Why not call the booking recommendation prospective?* The available data does not provide a verified timestamp for when each fare was observed, so we cannot reconstruct information available at a historical booking decision.

## J.A.R.V.I.S. — personal Windows desktop assistant

**Problem and design.** The current repository `main` contains a Python/PySide6 Windows desktop app. Its command router separates deterministic registered tools from conversational queries; a security policy constrains file operations, a permission manager gates risky actions, and a SQLite store records tool activity. AI providers include local Ollama and optional cloud support, with offline fallback behavior.

**Technologies:** Python, PySide6, SQLite, psutil, Ollama, optional cloud AI, pytest, Windows batch scripts.

**Limits.** This is a personal desktop prototype, not an unrestricted autonomous agent. Windows-specific behavior and optional integrations require testing on a compatible machine. The README's 28-test claim was not independently verified in this review because pytest and PySide6 are not installed in the local environment. The repository also retains an older Perchance browser prototype (`index.html`, `main.pjs`, and PowerShell bridge); its legacy [`docs/SETUP.md`](https://github.com/Aarya801/my-jarvis/blob/main/docs/SETUP.md) describes that older app and does not explain how to install or run the current Python desktop app.

**Tests:** The README documents `python -m pytest -v`; the current environment could not run it because required packages are absent. See [test documentation](https://github.com/Aarya801/my-jarvis/blob/main/docs/TESTING.md) and [security model](https://github.com/Aarya801/my-jarvis/blob/main/docs/SECURITY.md).

**Likely questions**

- *How are model outputs kept from becoming arbitrary commands?* Tool execution goes through a registry and security/permission checks; an LLM does not directly execute generated shell commands.
- *What requires human approval?* Medium/high-risk operations are gated by the UI's permission workflow; path containment and protected-process checks are separate policy controls.
- *What would you fix before calling it release-ready?* Bring the setup and security documentation into line with the current desktop app, then run the documented suite in a clean Windows environment with dependencies installed.

## MedInsight — personal healthcare UI concept

**Problem and design.** MedInsight proposes patient and clinician views that organize fragmented records into timelines and focused document, medication, allergy, lab, and appointment screens. The repository's ZIP contains the screen maps and written UI specifications.

**Technologies:** documentation and interface specifications only in the reviewed archive. The repository does not contain a running React app, API, clinical data model, or screenshots.

**Limits.** This is a UI concept, not a deployed or clinically validated product. It makes no diagnostic or treatment claims. Any future real-world system would need privacy, access control, auditability, interoperability, and clinical validation.

**Tests:** No software test suite is present. Review the screen-map documents and validate the proposed flows with users before implementation.

**Likely questions**

- *What is implemented?* Written problem framing and patient/doctor screen specifications, packaged in the repository ZIP; no functioning clinical application is present.
- *How would you approach the next step?* Build a small prototype with synthetic data, test navigation and accessibility, and only later assess secure integration and clinical requirements with qualified stakeholders.

## Team contributions

### DecisionOS — owned by Sunal Sharma

The repository README credits Aarya with frontend work. The checked `main` history includes Aarya's [persistence-status clarification](https://github.com/SunalSharma/DecisionOS/commit/af905dba8146c8d107999af59de38a37d7b63855). The [incident-domain profiles change](https://github.com/SunalSharma/DecisionOS/commit/354eb6b) is on the separate `feature/aarya-frontend` branch, which is 1 commit ahead and 47 behind `main`; it is not part of `main`, and no pull request was found. These are individual contributions inside a team project; they do not establish ownership of the decision engine or the complete application.

**Technologies:** React, TypeScript, Vite, Tailwind CSS, Recharts; FastAPI, Pydantic, and a Python decision engine are part of the wider project.

**Likely questions**

- *What did you contribute?* Frontend integration and experience, plus the two authored changes above. The team README gives each member's broader role.
- *What does the system do?* It simulates resource scenarios, checks constraints, scores alternatives, and presents a ranked explanation. It is a hackathon decision-support prototype, not an operational emergency-response system.

### FinSight AI — owned by PREMBISOY

The repository history contains Aarya-authored [dashboard work](https://github.com/PREMBISOY/FinSight-AI/commit/63117036320df4658c3350ee0b67debb97052e9a), [frontend integration documentation](https://github.com/PREMBISOY/FinSight-AI/commit/a241389d57d64d1af4912e989f03906dbc0c7e46), and a [contributor-record update](https://github.com/PREMBISOY/FinSight-AI/commit/6746baf4f20f78f6fc5f12e3ace846eba0dbed73). The dashboard commit is on the remote [`codex/financial-command-center` branch](https://github.com/PREMBISOY/FinSight-AI/tree/codex/financial-command-center) and is absent from `main`; its pull request is closed without merging, and the branch is 1 commit ahead and 36 behind `main`. The documented frontend covers agent status, evidence, synthesis, risk context, decision traces, and degraded-data states. The backend and financial-agent work belong to the wider team.

**Technologies:** React, TypeScript, Vite, Tailwind CSS, Recharts, and a FastAPI backend in the wider project.

**Limits.** This is a hackathon financial-research prototype, not financial advice. Data providers may be incomplete or delayed; demo fixtures are explicitly synthetic. UI presentation does not validate the correctness of financial recommendations.

**Likely questions**

- *How did the frontend handle unavailable agent data?* It has explicit degraded and unavailable presentation states rather than implying all sources always succeed.
- *What did your work not include?* I contributed to the frontend and integration documentation; I do not claim ownership of the backend agents, synthesis logic, or whole product.

## Unverified collaboration listing

The profile previously described a NodeFlow contribution at `PREMBISOY/nodeflow`. On 3 November 2026, GitHub returned 404 for that repository, and its history could not be inspected. This guide does not claim a verified contribution there. Restore the correct public repository URL or provide an accessible repository before listing it as verified experience.
