# Failure Rate Analysis for `nlohmann/json` and `eclipse-score/inc_nlohmann_json` CI Workflows

## Introduction

GitHub’s “failure rate” counts all non‑successful job runs (including cancelled jobs and infrastructure/tooling problems), so it should not be interpreted as a direct measure of regressions in the JSON libraries. Throughout this analysis we distinguish between:

- **Test‑result failures/regressions:** unit or integration test failures that indicate a behavioural issue in the library.
- **CI/environment/infrastructure failures:** runner, tooling or network issues (e.g., coverage publishing, label synchronization).

## Methodology

For jobs with non‑zero failure rates we inspected the logs of failed workflow runs to determine whether failures originated from failing unit/integration tests (test‑result failures/regressions) or from CI/environment/tooling steps (CI/environment/infrastructure failures). Only the former are treated as behaviour‑related evidence.


## `nlohmann/json` – Ubuntu CI

### Scope and data source
- **Repository:** `nlohmann/json`
- **Workflow:** `.github/workflows/ubuntu.yml`
- **Date range:** 2025‑01‑11 – 2025‑04‑11 (3 months before the v3.12.0 release)
- **Filter:** `workflow_file_name: ubuntu.yml`

### Jobs with the highest reported failure rates

| Job (short name)                           | Failure rate | Avg. run time | Job runs |
|:-------------------------------------------|-------------:|--------------:|---------:|
| `ci_test_coverage_clang`                   | **50 %**     | 5m 33s        |        6 |
| `ci_static_analysis_clang (ci_clang_tidy)` | **14 %**     | 15m 36s       |      247 |
| `ci_test_coverage`                         | **13 %**     | 9m 19s        |      248 |
| `ci_test_clang`                            | **12 %**     | 6m 12s        |       25 |
| `ci_test_documentation`                    | 4 %          | 59 s          |      232 |
| `ci_static_analysis_clang (ci_test_clang)` | 4 %          | 6m 41s        |      214 |
| `ci_test_gcc`                              | 4 %          | 6m 27s        |      245 |
| `ci_cmake_options`                         | 3 %          | 6m 01s        |      248 |
| `ci_test_standards_clang`                  | 3 %          | 2m 50s        |      222 |
| `ci_test_compiler_gcc`                     | 3 %          | 3m 28s        |      227 |

All remaining jobs are at or below ≈3 %, many near 0 % (see GitHub [insights](https://github.com/nlohmann/json/actions/metrics/performance?dateRangeType=DATE_RANGE_TYPE_CUSTOM&sort=failureRate&tab=jobs&filters=workflow_file_name%3Aubuntu.yml&range=1736553600000-1744329600000)).

### Interpretation of the higher failure rates

- **`ci_test_coverage_clang`:** This job was a short‑lived experiment that only ran six times in the selected window. Several of these runs were part of the “Fix coverage job” pull request where the coverage script was intentionally broken and iteratively fixed. The failures therefore stem from coverage tooling/CI configuration issues rather than from failing unit tests or misbehaviour in the JSON library.

- **`ci_static_analysis_clang (ci_clang_tidy)`:** Most failures of this job come from pull requests deliberately changing static analysis or CI tooling. PR [#4654](https://github.com/nlohmann/json/pull/4654) (“Fix ~basic_json causing std::terminate”), PR [#4663](https://github.com/nlohmann/json/pull/4663) (“Add clang‑tidy plugin to convert implicit conversions to explicit ones”) and the change referenced in PR [#4701](https://github.com/nlohmann/json/pull/4701) (“Suppress clang-analyzer-webkit.NoUncountedMemberChecker”) show multiple failed runs while clang‑tidy/analyzer configuration was being tuned. Because the job treats every clang‑tidy or analyzer diagnostic as a hard error, each stricter check initially makes the job fail until warnings are fixed or suppressed. The ~14 % failure rate thus reflects intentional tightening and maintenance of the static‑analysis pipeline plus normal PR iteration, not unexplained misbehaviour in the library.

- **`ci_test_coverage`:** Failures of this job in the analysed period are explained by intentional CI maintenance rather than unstable tests. A significant portion comes from PR [#4595](https://github.com/nlohmann/json/pull/4595), where the coverage workflow was updated and repeatedly executed on the `fix-coverage` branch; intermediate runs failed until `gcov`, `lcov` and the Coveralls uploader were configured correctly. Additional failures stem from PR [#4709](https://github.com/nlohmann/json/pull/4709), which upgraded the minimum CMake version and introduced new CMake and OpenSSL configurations in CI; some runs failed until the revised toolchain setup stabilised. All changes were merged only after `ci_test_coverage` succeeded, and the underlying unit tests remained consistently green. The elevated failure rate therefore reflects CI/tooling evolution rather than regressions in the JSON library.

- **`ci_test_clang`:** This job existed as its own standalone entry in the Ubuntu workflow until PR [#4560](https://github.com/nlohmann/json/pull/4560) (“Clean up and document project files”) reorganised the workflow and moved `ci_test_clang` into a matrix of the `ci_static_analysis_clang` job. GitHub Actions always uses the workflow file from the commit on which a PR is based, so PRs that branch off before #4560 and are never rebased still run the old workflow. Newer PRs run the new workflow where `ci_test_clang` is just a matrix target. Thus `ci_test_clang` appears in the metrics in different ways even though it refers to the same underlying CMake target. During this period it ran 25 times and failed three times (12 %). All failures occurred on PR branches while Clang‑related code or CI configuration was being adjusted and were resolved before merging. As a result, a few legitimate work‑in‑progress failures translate into a relatively high percentage without indicating persistent instability of the Clang test setup.

### Conclusion

In the selected three‑month window before the v3.12.0 release, the Ubuntu CI for `nlohmann/json` shows a small set of jobs with noticeably higher failure rates. In every case these failures can be traced back to intentional CI/tooling work or strict analysis settings rather than to undetected misbehaviour of the library itself. All high failure rates occurred on PR branches, were visible to developers and resolved before merging. The majority of other jobs (including `ci_test_gcc`, `ci_test_standards_*` and most compiler‑matrix entries) remain at or near 0 %. This indicates a stable CI system that reacts as intended to real issues and configuration changes, with no evidence of systematic, unexplained spikes in test failures for the Ubuntu workflow.

## `eclipse-score/inc_nlohmann_json` – Parent‑Workflow/Ubuntu CI

### Scope and data source
- **Repository:** `eclipse-score/inc_nlohmann_json`
- **Workflow:** `Parent Workflow` (top‑level CI workflow)
- **Date range:** last 90 days from 2025‑12‑08
- **Filter:** `workflow_file_name: parent-workflow.yml`

### Jobs with the highest reported failure rates

| Job (short name)                                  | Failure rate | Avg. run time | Job runs |
|:--------------------------------------------------|-------------:|--------------:|---------:|
| `Run Ubuntu Workflow / publish_test_data_success` | **5 %**      | ≈ 25 s        |      38  |
| `Run Labeler Workflow / clone_missing_labels`     | **3 %**      | ≈ 6 s         |      38  |

All other Parent‑Workflow jobs in the last 90 days report a failure rate of 0 % over 38 runs each. See GitHub [insights](https://github.com/eclipse-score/inc_nlohmann_json/actions/metrics/performance?dateRangeType=DATE_RANGE_TYPE_LAST_90_DAYS&sort=failureRate&tab=jobs&filters=workflow_file_name%3Aparent-workflow.yml) for the complete data.

### Interpretation of the higher failure rates

- **`publish_test_data_success`:** This job is the final step of the Ubuntu child workflow that publishes the persistent test‑result database (`TSF/data_storage/MemoryEfficientTestResultData*.db`) back to the `save_historical_data` branch. The observed 5.26 % failure rate corresponds to two failed runs out of 38. These failures are limited to the publishing step (e.g., `git push`/branch/permission issues) rather than to execution of the tests themselves.

- **`clone_missing_labels`:** This job belongs to the separate “Labeler Workflow” and synchronises GitHub issue/PR labels with the organisation defaults. Its 2.63 % failure rate corresponds to one failed run out of 38. These failures relate to repository/label management and not to building or testing the JSON library.

### Conclusion

Over the three‑month period examined, all test‑related jobs in the Parent Workflow have 0 % failures, while the small non‑zero failure rates are confined to meta‑jobs that handle publishing of historical test data and label synchronisation. This indicates a stable CI setup for `inc_nlohmann_json`, with the only reported failures occurring in infrastructure‑side integration steps rather than in the core test pipeline.
