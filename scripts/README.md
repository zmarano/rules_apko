# Apko Update Scripts

This directory contains scripts for managing apko dependency updates.

## Scripts

### `mirror_apko.sh`
Script for mirroring apko releases with rollback capability, and optional PR creation.

**Usage:**
```bash
# Update without creating PR (backward compatible)
./scripts/mirror_apko.sh [repository] [tool]

# Update and create PR (requires gh CLI)
./scripts/mirror_apko.sh chainguard-dev/apko apko true
```

## Automation

### GitHub Actions Workflow
The `.github/workflows/update-apko.yml` workflow runs daily at 10:00 UTC to:

1. Check for new apko releases
2. Compare with current version
3. Run the mirror script if update is needed
4. Create PR with update
5. Run basic validation tests

**Manual trigger:**
```bash
# Trigger via GitHub CLI
gh workflow run update-apko.yml

# Trigger with force update
gh workflow run update-apko.yml -f force_update=true
```

**Workflow features:**
- Prevents duplicate PRs for the same version
- Comments on existing PRs if they're still current
- Provides detailed summary in workflow output
- Validates file updates after running script

## Supporting Files

### `filter_apko.jq`
JSON transformation filter that processes GitHub Releases API data into the structure needed for Bazel version files.

**Purpose:**
- Transforms GitHub API JSON response into structured version data
- Filters out non-tarball assets (keeps only `.tar.gz` files)
- Extracts platform identifiers from filenames
- Creates nested JSON with version keys and platform mappings

**Input Example:**
```json
[{
  "tag_name": "v0.29.9",
  "assets": [{
    "name": "apko_0.29.9_darwin_amd64.tar.gz",
    "browser_download_url": "https://github.com/chainguard-dev/apko/releases/download/v0.29.9/apko_0.29.9_darwin_amd64.tar.gz"
  }]
}]
```

**Output Example:**
```json
{
  "v0.29.9": {
    "darwin_amd64": "apko_0.29.9_darwin_amd64.tar.gz",
    "linux_amd64": "apko_0.29.9_linux_amd64.tar.gz"
  }
}
```

**Integration:**
The mirror script uses this filter with `jq -f filter_apko.jq` to process GitHub API responses, then replaces the filenames with SHA256 integrity hashes from the checksums.txt file.

## Requirements

- `jq` - JSON processing
- `xxd` - Binary data processing
- `curl` - HTTP requests
- `gh` (optional) - GitHub CLI for PR creation
- Git repository with clean working directory
