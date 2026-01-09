# Trigger Workflow

## Overview

The `trigger.yml` workflow is a reusable workflow that enables controlled triggering of deployment workflows from other workflows. It validates the specified git reference (branch or tag) and then triggers the deployment workflow using the specified parameters.

## Purpose

This workflow serves as a gateway for initiating eployments, ensuring that:

- Only valid git references (branches or tags) are used for deployments
- The deployment workflow path is properly specified
- The deployment process is initiated through a standardized, validated mechanism

## Key Features

- **Reference Validation**: Automatically validates that the specified workflow reference is either a valid branch or tag before proceeding
- **Flexible Reference Selection**: Defaults to the current reference if not specified
- **Runner Selection**: Configurable runner for job execution
- **Deployment Workflow Triggering**: Uses the custom deployment-trigger action to initiate deployments

## Input Parameters

| Parameter | Description | Required | Default | Type |
|-----------|-------------|----------|---------|------|
| `workflow-ref` | The reference (tag or branch) to be used for workflow run | No | `${{ github.ref_name }}` | string |
| `workflow-path` | The path to the workflow file | Yes | N/A | string |
| `build-version` | The version to be used for the release.  | No  | `''` (empty string) | string |
| `github-job-runs-on` | The runner to use for the job (e.g., ubuntu-latest) | No | `ubuntu-latest` | string |

## Permissions

This workflow requires the following permissions:

- **actions**: `write` - Required to trigger other workflows
- **contents**: `write` - Required to access repository contents and git references

## Workflow Steps

1. **Checkout**: Checks out the repository code
2. **Validate Reference**: Validates that the specified `workflow-ref` is either:
   - A valid remote branch (in origin)
   - A valid tag
   - If validation fails, the workflow exits with an error and displays available branches and tags
3. **Trigger Deployment**: Uses the `lordlive/custom-action/trigger` action to trigger the production deployment workflow

## Usage Example

```yaml
name: Deploy
on:
  workflow_dispatch:
    inputs:
      deployment-ref:
        description: 'Git reference to deploy (branch or tag)'
        required: true
        type: string

jobs:
  trigger-deployment:
    uses: lordlive/custom-action/trigger@v1.0.8
    with:
      workflow-ref: ${{ inputs.deployment-ref }}
      workflow-path: '.github/workflows/production-deploy.yml'
      github-job-runs-on: 'ubuntu-latest'
```

## Error Handling

If the specified `workflow-ref` is invalid, the workflow will:

1. Display an error message indicating the reference is neither a valid branch nor tag
2. List up to 20 available branches
3. List up to 20 available tags
4. Exit with a failure status

## Dependencies

This workflow depends on:

- **actions/checkout@v5**: For repository checkout
- **lordlive/custom-action/trigger@v1.0.8**: Custom action for triggering deployment workflows

## Notes

- The workflow runs on the specified runner (default: `ubuntu-latest`)
- All tags and branches are fetched before validation to ensure accuracy
- The workflow is designed to be called by other workflows (via `workflow_call`)
- The reference validation step outputs the reference type (branch or tag) for logging purposes
