These are [GitHub reusable workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows) that can be called from any repository within the Espressif GitHub organization.

ℹ️ Their **presence here does not mean they are automatically propagated or inherited across all Espressif organization GitHub repositories** (as with issue and pull request templates). These workflows **need to be explicitly called**. See the ["Calling from the target repository"](#calling-from-the-target-repository) section for details.

---

- [Reasoning](#reasoning)
- [Calling from the target repository](#calling-from-the-target-repository)
- [Custom Parameters and Configuration](#custom-parameters-and-configuration)
- [Troubleshooting](#troubleshooting)

## Reasoning

Managing these workflows from a central place like this repository (`espressif/.github`) offers several advantages:

- **Ease of Integration**: Simplifies the process for repository administrators—only a call statement is needed to integrate the workflow.
- **Centralized Updates**: Quickly update the version of a GitHub workflow for all repositories that use it with a single change here.
- **Consistent Permissions**: Ensure that action permissions are set correctly across all repositories.

## Calling from the target repository

In the target repository, e.g., `espressif/example-repo`, the following steps are required:

1. Create a file at the usual location for GitHub Actions workflow YAML files, such as `.github/workflows/call-pre-commit.yml`.
2. Add the following content to that file:

```yaml
# REPO: https://github.com/espressif/example-repo
# FILE: .github/workflows/call-pre-commit.yml
---
name: Pre-commit (PR code changes)

on:
  pull_request:

jobs:
  call-pre-commit:
  uses: espressif/.github/.github/workflows/reusable-pre-commit.yml@main
```

This will enable the reusable GitHub workflow in the target repository (`espressif/example-repo`).

If the configuration ever changes in the `espressif/.github` repository (this repo), there is no need to make changes in every repository — it is propagated automatically. For example, if an external action version is updated (e.g., `uses: actions/setup-python@v4` to `uses: actions/setup-python@v5`), it would automatically apply to all repositories using this workflow.

## Custom Parameters and Configuration

The reusable workflow can be called with custom parameters, such as:

```yaml
# REPO: https://github.com/espressif/example-repo
# FILE: .github/workflows/call-pre-commit.yml
---
name: Pre-commit checks

on:
  pull_request:

jobs:
  call-pre-commit:
    uses: espressif/.github/.github/workflows/reusable-pre-commit.yml@main
    with:
      python-version: '3.12' # Optionally override the Python version
      skip: 'codespell,mdformat' # Optionally skip specific pre-commit hooks in CI
```

This allows precise customization of the local action configuration without needing to delve into the full action YAML file.

## Troubleshooting

If the reusable workflow is not running in the target repository, check the following:

- **Actions and Workflow Permissions**: Ensure that the target repository allows `Allow all actions and reusable workflows` (in repo `Settings -> Actions -> Actions permissions`).
- **Correct Path to Reusable Workflow**: Double-check the path in the `uses:` directive. The correct format is `espressif/.github/.github/workflows/<WORKFLOW_FILE_NAME>.yml@main` (note the two `.github` segments in the path; the first refers to the .github repository, and the second refers to the .github directory within that repository).

---
