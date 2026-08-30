These are [GitHub reusable workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows) that can be called from any repository within the Espressif GitHub organization.

> \[!IMPORTANT\]
> Their presence here does not mean they are automatically propagated or inherited across all Espressif organization GitHub repositories
> (as with issue and pull request templates).
> These workflows need to be explicitly called. See the ["Calling from the target repository"](#calling-from-the-target-repository) section for details.

---

- [Reasoning](#reasoning)
- [Calling from the target repository](#calling-from-the-target-repository)
- [Usage in target project (calls)](#usage-in-target-project-calls)
  - [Pre-commit (PR code changes)](#pre-commit-pr-code-changes)
  - [DangerJS (PR style linter)](#dangerjs-pr-style-linter)
  - [JIRA Sync Actions (package)](#jira-sync-actions-package)
    - [Sync JIRA - Pull Requests (1/3)](#sync-jira---pull-requests-13)
    - [Sync JIRA - Issues (2/3)](#sync-jira---issues-23)
    - [Sync JIRA - Issue Comments (3/3)](#sync-jira---issue-comments-33)
- [Troubleshooting](#troubleshooting)

## Reasoning

Managing these workflows from a central place like this repository (`espressif/.github`) offers several advantages:

- **Ease of Integration**: Simplifies the process for repository administrators—only a call statement is needed to integrate the workflow.
- **Centralized Updates**: Quickly update the version of a GitHub workflow for all repositories that use it with a single change here.
- **Consistent Permissions**: Ensure that action permissions are set correctly across all repositories.

## Calling from the target repository

In the target repository, e.g., `espressif/example-repo`, the following steps are required:

1. Create a file at the usual location for GitHub Actions workflow YAML files, such as `.github/workflows/call-pre-commit.yml`.
2. Add the call to the reusable workflow from another Espressif repository.
3. Optionally, you can add parameters that will be passed to the GitHub Action.

This allows precise customization of the local action configuration without needing to modify the full action YAML file.

> \[!NOTE\]
> If the configuration ever changes in the `espressif/.github` repository (this repo), there is no need to update every repository — the
> changes are propagated automatically.

For example, if an external action version is updated (e.g., `uses: actions/setup-python@v4` to `uses: actions/setup-python@v5`), the update would automatically apply to all repositories using this workflow.

## Usage in target project (calls)

### Pre-commit (PR code changes)

```yaml
# FILE: .github/workflows/call-pre-commit.yml
---
name: Pre-commit (PR code changes)

on:
  pull_request:

jobs:
  call-pre-commit:
    uses: espressif/.github/.github/workflows/reusable-pre-commit.yml@main
```

**Optional arguments**:

| input          | description                      | type          | default       |
| -------------- | -------------------------------- | ------------- | ------------- |
| python-version | Python version the workflow uses | str           | '3.9'         |
| skip           | pre-commit hooks skipped in CI   | comma sep str | 'pip-compile' |

### DangerJS (PR style linter)

```yaml
# FILE: .github/workflows/call-dangerjs.yml
---
name: DangerJS (PR style linter)

on:
  pull_request_target:
    types: [opened, edited, reopened, synchronize]

permissions:
  pull-requests: write
  contents: write

jobs:
  call-dangerjs:
    uses: espressif/.github/.github/workflows/reusable-dangerjs.yml@master
```

**Optional arguments**:

| with:                  | description                             | type | default |
| ---------------------- | --------------------------------------- | ---- | ------- |
| `rule-commit-messages` | Enable rule for PR Lint Commit Messages | str  | 'true'  |
| `rule-description`     | Enable rule for PR Description          | str  | 'true'  |
| `rule-max-commits`     | Enable rule for PR Too Many Commits     | str  | 'true'  |

> \[!TIP\]
> The table with optional arguments is not exhaustive, it is here for reference.
> More information and config details in in project [espressif/shared-github-dangerjs](https://github.com/espressif/shared-github-dangerjs)

### JIRA Sync Actions (package)

#### Sync JIRA - Pull Requests (1/3)

```yaml
# FILE: .github/workflows/call-sync-jira-prs.yml
---
name: Sync JIRA - Pull Requests

on:
  workflow_dispatch: # Allows manual triggering of the workflow
  schedule:
    - cron: '0 * * * *' # Adjust the cron schedule as needed

concurrency:
  group: jira-issues # Ensures only one workflow in the 'jira-issues' group runs at a time (avoid sync issues)

jobs:
  call-sync-jira-pull-requests:
    uses: espressif/.github/.github/workflows/reusable-sync-jira-prs.yml@main
    with:
      jira-project: '<jira_project>' #  e.g., 'ESPTOOL' or 'IDFGH'
```

#### Sync JIRA - Issues (2/3)

```yaml
# FILE: .github/workflows/call-sync-jira-prs.yml
---
name: Sync JIRA - Issues

on: issues

concurrency:
  group: jira-issues # Ensures only one workflow in the 'jira-issues' group runs at a time (avoid sync issues)

jobs:
  call-sync-jira-pull-requests:
    uses: espressif/.github/.github/workflows/reusable-sync-jira-prs.yml@main
    with:
      jira-project: '<jira_project>' #  e.g., 'ESPTOOL' or 'IDFGH'
```

#### Sync JIRA - Issue Comments (3/3)

```yaml
# FILE: .github/workflows/call-sync-jira-prs.yml
---
name: Sync JIRA - Issue Comments

on: issue_comment

concurrency:
  group: jira-issues # Ensures only one workflow in the 'jira-issues' group runs at a time (avoid sync issues)

jobs:
  call-sync-jira-pull-requests:
    uses: espressif/.github/.github/workflows/reusable-sync-jira-prs.yml@main
    with:
      jira-project: '<jira_project>' #  e.g., 'ESPTOOL' or 'IDFGH'
```

**Optional arguments**:

| input          | description                                   | type | default  |
| -------------- | --------------------------------------------- | ---- | -------- |
| jira-component | Jira component (if used in Jira project) name | str  | 'GitHub' |

📖 More info in project: https://github.com/espressif/sync-jira-actions

> \[!TIP\]
> More information and config details in in project [espressif/sync-jira-actions](https://github.com/espressif/sync-jira-actions)

---

## Troubleshooting

If the reusable workflow is not running in the target repository, check the following:

- **Actions and Workflow Permissions**: Ensure that the target repository allows `Allow all actions and reusable workflows` (in repo `Settings -> Actions -> Actions permissions`).
- **Correct Path to Reusable Workflow**: Double-check the path in the `uses:` directive. The correct format is `espressif/.github/.github/workflows/<WORKFLOW_FILE_NAME>.yml@main` (note the two `.github` segments in the path; the first refers to the .github repository, and the second refers to the .github directory within that repository).
