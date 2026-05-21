# Plumber GitHub Action

Analyze your GitHub Actions workflows and repository settings for CI/CD compliance and supply-chain risks.

Plumber checks branch protection rules, workflow permissions, secret handling, pinned actions, and more — then produces a compliance score, a [SARIF](https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning) report for GitHub Code Scanning, and a [Pipeline Bill of Materials](https://getplumber.io/docs/pbom) (PBOM).

## Usage

### Minimal

```yaml
jobs:
  compliance:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write   # required for SARIF upload
    steps:
      - uses: actions/checkout@v4
      - uses: plumber-org/gh-action@v1
```

### With a custom threshold

```yaml
- uses: plumber-org/gh-action@v1
  with:
    threshold: 90       # fail if compliance drops below 90%
    score: true         # print letter score + points breakdown
```

### Pin a specific version

```yaml
- uses: plumber-org/gh-action@v1
  with:
    version: v0.3.10
```

### Soft-fail (report only, never block)

```yaml
- uses: plumber-org/gh-action@v1
  with:
    soft-fail: true
```

### Scan a remote repository (no checkout needed)

```yaml
- uses: plumber-org/gh-action@v1
  with:
    project: my-org/my-other-repo
    github-token: ${{ secrets.ORG_READ_TOKEN }}
```

### Use step outputs

```yaml
- uses: plumber-org/gh-action@v1
  id: plumber
  with:
    threshold: 80

- run: |
    echo "Compliance: ${{ steps.plumber.outputs.compliance }}%"
    echo "Passed:     ${{ steps.plumber.outputs.passed }}"
```

## Inputs

### Installation

| Input | Default | Description |
|---|---|---|
| `version` | `latest` | Plumber release tag to install (e.g. `v0.3.10`). |
| `verify-attestation` | `true` | Verify the binary's build-provenance attestation via sigstore/SLSA. Disable for air-gapped or GHES setups. |

### Authentication

| Input | Default | Description |
|---|---|---|
| `github-token` | `github.token` | Token for GitHub API access and SARIF upload. Needs `security-events:write` for SARIF uploads. |

### Project scope

| Input | Default | Description |
|---|---|---|
| `project` | _(current repo)_ | `owner/repo` to scan remotely — no checkout required. |
| `github-url` | _(github.com)_ | GitHub Enterprise Server host (e.g. `ghes.example.com`). |

### Compliance

| Input | Default | Description |
|---|---|---|
| `threshold` | `100` | Minimum compliance percentage required to pass (0–100). |
| `config-file` | _(auto-detect)_ | Path to a `.plumber.yaml` config file (relative to repo root). |
| `controls` | — | Run only these controls (comma-separated). Mutually exclusive with `skip-controls`. |
| `skip-controls` | — | Skip these controls (comma-separated). Mutually exclusive with `controls`. |
| `fail-warnings` | `false` | Treat configuration warnings (unknown keys) as errors. |

### Scoring & failure

| Input | Default | Description |
|---|---|---|
| `score` | `true` | Print the letter score, points and full breakdown in the log and job summary. |
| `soft-fail` | `false` | Do not fail the job when compliance is below the threshold. Runtime errors (exit 2) still fail. |

### Report outputs

| Input | Default | Description |
|---|---|---|
| `output` | `plumber-report.json` | Path for the JSON compliance report. Leave empty to skip. |
| `pbom` | `plumber-pbom.json` | Path for the Pipeline Bill of Materials (PBOM). Leave empty to skip. |
| `pbom-cyclonedx` | `plumber-cyclonedx-sbom.json` | Path for the CycloneDX SBOM. Leave empty to skip. |
| `sarif` | `plumber.sarif` | Path for the SARIF 2.1.0 report. Leave empty to skip. |

### Artifact upload

| Input | Default | Description |
|---|---|---|
| `upload-sarif` | `true` | Upload the SARIF report to GitHub Code Scanning. Requires `security-events:write`. |
| `upload-artifacts` | `true` | Upload JSON report, PBOM and CycloneDX SBOM as a workflow artifact. |
| `artifact-name` | `plumber-compliance` | Name of the uploaded artifact bundle. |
| `artifact-retention-days` | `30` | Artifact retention in days. |

## Outputs

| Output | Description |
|---|---|
| `compliance` | Overall compliance percentage (e.g. `85`). |
| `passed` | `true` when compliance is at or above the threshold. |
| `report` | Path to the JSON compliance report. |
| `sarif` | Path to the SARIF report. |

## Permissions

```yaml
permissions:
  contents: read         # always required
  security-events: write # required for upload-sarif: true (default)
```

For scanning a private remote repository, supply a token with `repo` scope via `github-token`.

## Configuration file

Plumber auto-detects `.plumber.yaml` in the repository root. To use a custom path:

```yaml
- uses: plumber-org/gh-action@v1
  with:
    config-file: "configs/plumber.yaml"
```

Config file priority:

1. `config-file` input (if set)
2. `.plumber.yaml` in the repo root (if present)
3. Built-in defaults

See the [Plumber documentation](https://getplumber.io/docs) for the full configuration reference and available controls.

## Security

Plumber binaries are downloaded from [GitHub Releases](https://github.com/getplumber/plumber/releases) and verified in two layers:

1. **Checksum** — the binary is checked against `checksums.txt` published alongside each release (guards transit integrity).
2. **Build provenance** — the binary's digest is verified against its sigstore/SLSA attestation, tying it to the `getplumber/plumber` release workflow (guards against a re-pointed or compromised tag).

To disable attestation verification (e.g. air-gapped environments):

```yaml
- uses: plumber-org/gh-action@v1
  with:
    verify-attestation: false
```

## License

[Mozilla Public License 2.0](LICENSE)
