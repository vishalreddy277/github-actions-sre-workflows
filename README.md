# github-actions-sre-workflows

Reusable GitHub Actions workflows I keep copying between projects. Instead of copying the YAML every time and inevitably having them drift out of sync, I centralise them here and call them as reusable workflows.

---

## Available workflows

| Workflow | What it does |
|----------|-------------|
| `terraform-plan-apply.yml` | Plan on PR, apply on merge. Posts plan output as PR comment. |
| `docker-build-push.yml` | Builds and pushes to ECR with proper tagging (sha + latest + semver) |
| `eks-deploy.yml` | Updates K8s deployment image tag and waits for rollout |
| `datadog-deployment-marker.yml` | Creates a Datadog deployment event so you can correlate deploys with metrics |
| `python-lint-test.yml` | Ruff + pytest with coverage report |

---

## Usage

In your project's workflow file:

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    uses: vishalreddy277/github-actions-sre-workflows/.github/workflows/docker-build-push.yml@main
    with:
      image_name: my-service
      aws_region: us-east-1
    secrets:
      aws_role_arn: ${{ secrets.AWS_ROLE_ARN }}

  deploy:
    needs: build
    uses: vishalreddy277/github-actions-sre-workflows/.github/workflows/eks-deploy.yml@main
    with:
      cluster_name: my-cluster
      namespace: production
      deployment_name: my-service
      image_tag: ${{ needs.build.outputs.image_tag }}
    secrets:
      aws_role_arn: ${{ secrets.AWS_ROLE_ARN }}
```

---

## Why reusable workflows instead of composite actions?

Composite actions are great for steps but they run in the calling workflow's runner. Reusable workflows are full jobs — they can have their own permissions, environments, and secrets handling. For deploy workflows that need to assume different IAM roles per environment, reusable workflows are the right call.
