# GitHub Workflow Templates

This repository holds reusable GitHub Actions workflow templates (starting with Gemini-CLI templates) that other repositories can call. New automations and templates should be added here so they can be centrally maintained and consumed.

## Included templates

- [.github/workflows/gemini-dispatch.yml](.github/workflows/gemini-dispatch.yml) — central dispatcher that routes requests and comments to the appropriate Gemini workflows.
- [.github/workflows/gemini-invoke.yml](.github/workflows/gemini-invoke.yml) — generic invoke workflow for ad-hoc Gemini CLI tasks.
- [.github/workflows/gemini-review.yml](.github/workflows/gemini-review.yml) — reusable pull request review workflow.
- [.github/workflows/gemini-triage.yml](.github/workflows/gemini-triage.yml) — reusable issue triage workflow for single issues.
- [.github/workflows/gemini-scheduled-triage.yml](.github/workflows/gemini-scheduled-triage.yml) — scheduled batch triage for unlabeled issues.
- [.gitignore](.gitignore)

## Usage

There are two common ways to consume these templates:

- From another repository (recommended for central maintenance):

  - Replace `owner/repo` and `ref` with your organization/repo and branch or tag:
  ```yaml
  uses: 'owner/repo/.github/workflows/gemini-triage.yml@main'

- From within this repository (useful for testing or local composition):
  uses: './.github/workflows/gemini-review.yml'
  
  Most workflows are declared as reusable workflows (workflow_call). Check individual workflow inputs/outputs and required permissions in the workflow files above before calling them.

**Secrets & permissions**
Templates assume callers will provide the necessary secrets and variables (for example: GEMINI_API_KEY, GOOGLE_API_KEY, optional GitHub App credentials).
When calling a reusable workflow from another repo, set secrets: inherit or explicitly map secrets as required.
Workflows follow least-privilege principles; review the permissions: blocks in each workflow file to confirm they match your use case.
Contributing
Add new templates under .github/workflows/.
Follow the naming convention gemini-*.yml for Gemini-related templates.
Document required inputs, outputs, and any secrets in the top-level comment of the workflow file.
Update this README to include newly added templates.

**Security notes**
Do not hardcode secrets or tokens in workflow files. Use GitHub Secrets or GitHub App tokens.
Reusable workflows may mint short-lived GitHub App tokens — review minting behavior before enabling in production repositories.
The templates include explicit safeguards around untrusted inputs (see prompts and env handling in the workflows). Keep those safeguards intact when making changes.

**Files in this repository**
.gitignore
.github/workflows/gemini-dispatch.yml
.github/workflows/gemini-invoke.yml
.github/workflows/gemini-review.yml
.github/workflows/gemini-triage.yml
.github/workflows/gemini-scheduled-triage.yml

Example caller workflow:

```yaml
# filepath: .github/workflows/example-call-gemini-triage.yml
name: Example  Call Gemini Triage

on:
  workflow_dispatch:
    inputs:
      issue_number:
        description: 'Issue number to triage (leave blank to let workflow pick recent issues)'
        required: false
        default: ''

permissions:
  contents: read
  issues: write

jobs:
  call-gemini-triage:
    # Replace OWNER/REPO and ref (e.g., main or a tag) with your templates repo
    uses: 'OWNER/REPO/.github/workflows/gemini-triage.yml@main'
    with:
      # Map inputs expected by the reusable gemini-triage workflow
      issue_number: ${{ github.event.inputs.issue_number }}
      labels: 'triage'
    secrets:
      # Pass required secrets from the calling repository
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

