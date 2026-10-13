# CI error categorization on auto-injected steps

This repository is a Harness CI fixture for validating custom error categorization on steps injected by the CI stage, rather than authored directly in pipeline YAML.

## Fixture

- `.harness/errors.yml` maps `ERRCAT_AUTO_CACHE_FAILURE`, `NoSuchBucket`, and `AccessDenied` to `APPLICATION_FAILURE / DEPENDENCY_ISSUE`.
- The rule emits the message `errors.yml matched an auto-injected CI cache step.`
- `cache-input.txt` is a normal cache input fixture.

## Harness setup

- Pipeline: [CI Auto-Injected Step Error Categorization](https://app.harness.io/ng/account/gCoPSwHxS7ipOgx2iA9tOQ/all/orgs/Ansibler/projects/Automate/pipelines/ci_auto_injected_error_categorization/pipeline-studio)
- Validated execution: [Build 2](https://app.harness.io/ng/account/gCoPSwHxS7ipOgx2iA9tOQ/all/orgs/Ansibler/projects/Automate/pipelines/ci_auto_injected_error_categorization/deployments/TkNM8B1USPa1k1gfj-m0KQ/pipeline)
- Codebase clone is enabled so the engine can discover `.harness/errors.yml` in the workspace.
- Cache Intelligence uses `pull-push` with a deliberately missing path named `ERRCAT_AUTO_CACHE_FAILURE_missing_path`.
- The missing path forces both injected cache operations to fail. Their built-in `Ignore Failure` strategy allows the stage to finish and exposes both results in one execution.

## Validated result

| Step | Injection | Result | Categorization evidence |
| --- | --- | --- | --- |
| Clone codebase | Automatic | Success | Makes the repository rule file available to later steps |
| Restore Cache From Harness | Automatic, before user steps | Failure ignored | Details show `errors.yml matched an auto-injected CI cache step.` |
| Verify Rules and Keep Stage Successful | Authored Run step | Success | Confirms `.harness/errors.yml` exists in the workspace |
| Save Cache to Harness | Automatic, after user steps | Failure ignored | Details show `errors.yml matched an auto-injected CI cache step.` |

The test confirms that auto-injected Harness cache restore and save steps use repository-based custom error categorization.

Because these steps apply `Ignore Failure`, the top-level execution summary reports `SUCCESS (FAILURE IGNORED)` and does not include their failure details. Open each injected child step in the execution UI to see the categorized message.

## Boundary

This fixture does not prove repository-based categorization for an auto-injected clone failure. Clone runs before the repository is available, so its own failure cannot consume `.harness/errors.yml` from that repository. That case needs a rule source available before clone, such as a platform-provided rule or an explicitly configured external rules path.
