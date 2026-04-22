# IaC GitHub Actions Pipeline

GitHub Actions workflows for running OpenTofu/Terraform safely in CI/CD.

## What's included

| Workflow | Trigger | What it does |
|---------|---------|-------------|
| `tofu-pipeline.yml` | Pull Request | Lint → Security scan → Plan (per changed stack) |
| `merge-gatekeeper.yml` | Pull Request to main/master | Blocks merge until all required checks pass |

## Structure

```
iac-github-actions-pipeline/
├── .github/
│   └── workflows/
│       ├── tofu-pipeline.yml       # Main IaC pipeline
│       └── merge-gatekeeper.yml    # PR merge protection
└── docs/
    └── setup.md
```

## Prerequisites

- GitHub repository with your Terraform/OpenTofu code
- AWS OIDC role for GitHub Actions (keyless auth — no stored AWS credentials)
- `tflint` config file in your repo (optional but recommended)

## Setup

### 1. Create the AWS OIDC role

GitHub Actions needs to assume an AWS role to run Terraform. Follow the [AWS OIDC guide](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services) or use the `aws-multi-account-landing-zone` project's `management/iam` configuration.

### 2. Add GitHub secrets and variables

In your repository Settings → Secrets and Variables:

**Variables (not secret — visible in logs):**

| Name | Value |
|------|-------|
| `GH_APP_ID` | Your GitHub App ID (for token generation) |

**Secrets (encrypted):**

| Name | Value |
|------|-------|
| `GH_APP_PRIVATE_KEY` | Private key for your GitHub App |
| `AWS_OIDC_ROLE_ARN` | ARN of the AWS role GitHub Actions assumes |

### 3. Copy the workflows

Copy `.github/workflows/` into your IaC repository's `.github/workflows/` folder. Update the `with:` inputs in `tofu-pipeline.yml` to match your setup.

### 4. Configure branch protection

In your repository Settings → Branches → main:
- Enable "Require status checks to pass before merging"
- Add `merge-gatekeeper` as a required check

## How the pipeline works

```
PR opened/updated
       │
       ▼
  tflint (lint all changed .tf files)
       │
       ▼
  trivy (security scan — fails on HIGH/CRITICAL misconfigs)
       │
       ▼
  tofu plan (for each changed stack)
       │        └── posts plan output as PR comment
       ▼
  merge-gatekeeper (waits for all checks to pass)
       │
       ▼
  Merge → main
       │
       ▼
  tofu apply (triggered separately or via Spacelift)
```
