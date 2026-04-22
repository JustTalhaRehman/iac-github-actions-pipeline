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
