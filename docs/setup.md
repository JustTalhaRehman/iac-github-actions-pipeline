# Setup Guide

## AWS OIDC Configuration

GitHub Actions authenticates to AWS via OIDC — no long-lived credentials stored in GitHub secrets.

### Create the trust policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:*"
        }
      }
    }
  ]
}
```

Replace `YOUR_ACCOUNT_ID`, `YOUR_ORG`, and `YOUR_REPO` with your values.

### Scope the role

Start broad (AdministratorAccess) for simplicity, then tighten to exactly what Terraform needs once you know your resource scope.

## GitHub App vs PAT

The pipeline uses a GitHub App for authentication instead of a personal access token. This is better because:

- App tokens are short-lived (1 hour)
- App permissions are scoped to specific repositories
- No risk of a developer leaving and revoking a PAT

To create a GitHub App:
1. Go to Settings → Developer settings → GitHub Apps → New
2. Grant: Contents (read), Pull requests (write), Checks (read/write)
3. Install the app on your repository
4. Store the App ID as `GH_APP_ID` variable and private key as `GH_APP_PRIVATE_KEY` secret

## tflint configuration

Create `.tflint.hcl` in your repo root:

```hcl
plugin "aws" {
  enabled = true
  version = "0.32.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

rule "terraform_required_version" {
  enabled = true
}

rule "terraform_required_providers" {
  enabled = true
}
```

## Adding more stacks to the plan matrix

In `tofu-pipeline.yml`, update the `matrix.stack` list:

```yaml
matrix:
  stack:
    - aws/production/vpc
    - aws/production/eks/cluster
    - aws/staging/vpc
    - aws/staging/eks/cluster
```

Or make it dynamic by detecting which stacks changed using `git diff --name-only`.

## Apply strategy

This repo only handles planning in CI. For applies, you have two options:

1. **Spacelift** — use the `spacelift-iac-automation` project, which auto-applies on merge
2. **Manual apply job** — add an `apply` job that runs on `push` to `main` with `environment: production` protection rules
