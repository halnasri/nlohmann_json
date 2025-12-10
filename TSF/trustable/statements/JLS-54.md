---
level: 1.1
normative: true
---

In the eclipse-score/inc_nlohmann_json repository, code coverage for unit and integration tests is measured in every CI run, and a minimum coverage threshold is defined for each protected branch. If coverage for a change would fall below this threshold, the CI workflow blocks the merge until coverage is restored or the change is rejected.