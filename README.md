# TerraVision GitHub Action

Generate cloud architecture diagrams from Terraform code automatically in your CI/CD pipeline.

## Prerequisites

This action requires **Terraform** (or OpenTofu) to be available on the runner's PATH. Set it up before calling this action:

```yaml
- uses: hashicorp/setup-terraform@v3
```

If your Terraform code accesses cloud resources during `init`/`plan`, configure credentials before this action:

```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789012:role/my-role
```

## Usage

### Basic

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: hashicorp/setup-terraform@v3

  - uses: patrickchugh/terravision-action@v1
    with:
      source: ./infrastructure
```

### With Options

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: hashicorp/setup-terraform@v3
    with:
      terraform_version: 1.7.0

  - uses: patrickchugh/terravision-action@v1
    with:
      source: ./terraform
      outfile: docs/architecture
      format: both
      varfile: environments/prod.tfvars
      annotate: terravision.yml
```

### Full Workflow Example

```yaml
name: Update Architecture Diagrams

on:
  push:
    branches: [main]
    paths: ['**.tf', '**.tfvars']

jobs:
  generate-diagrams:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3

      - uses: patrickchugh/terravision-action@v1
        id: diagrams
        with:
          source: ./infrastructure
          outfile: docs/architecture
          format: both

      - name: Commit Diagrams
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add docs/architecture.*
          git commit -m "Update architecture diagrams [skip ci]" || exit 0
          git push
```

### Multi-Environment

```yaml
jobs:
  diagrams:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [dev, staging, prod]
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3

      - uses: patrickchugh/terravision-action@v1
        with:
          source: ./terraform
          outfile: docs/architecture-${{ matrix.environment }}
          varfile: environments/${{ matrix.environment }}.tfvars
          format: svg
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `source` | Path to Terraform source directory | Yes | `.` |
| `outfile` | Output file path (without extension) | No | `architecture` |
| `format` | Output format: `png`, `svg`, or `both` | No | `png` |
| `varfile` | Path to `.tfvars` file | No | |
| `annotate` | Path to `terravision.yml` annotation file | No | |
| `extra-args` | Additional arguments for `terravision draw` | No | |

## Outputs

| Output | Description |
|--------|-------------|
| `diagram-files` | Comma-separated list of generated file paths |

## Docker Alternative

If you prefer a fully self-contained setup (no prerequisites needed), use the TerraVision Docker image directly:

```yaml
- name: Generate Diagram
  uses: docker://patrickchugh/terravision:latest
  with:
    args: draw --source ./infrastructure --outfile architecture --format png
```

This bundles Graphviz and OpenTofu inside the container, so no separate setup steps are needed. However, it won't use your runner's Terraform version or cloud credentials unless you pass them explicitly.

## License

MIT
