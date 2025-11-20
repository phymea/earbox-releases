# Release Process

Centralized release workflow for Earbox Analysis Pipeline and AWS Runner.

## Quick Start

### 1. Tag Repositories

```bash
# List tags newest first 
git tag --sort=-version:refname
# List remote tags
git ls-remote --tags origin
# Tag analysis repo
cd earbox-analysis
git tag v1.2.3
git push origin v1.2.3

# Tag runner repo
cd earbox-aws-runner
git tag v1.1.2
git push origin v1.1.2
```

### 2. Run Workflow

1. Go to `earbox-releases` → **Actions** → **Prepare Release**
2. Click **"Run workflow"**
3. Fill inputs:
   - **`release_version`** (required): Release tag you want to create (e.g., `v1.3.5-rc1`)
   - **`analysis_version`** (required): Tag from `earbox-analysis` repo (e.g., `v1.2.3`)
   - **`runner_version`** (required): Tag from `earbox-aws-runner` repo (e.g., `v1.1.2`)
   - **`skip_docker`** (optional): Skip Docker builds for runner-only releases (default: false)

### 3. Review & Publish

1. Go to **Releases** → **Drafts**
2. Review changelog, notes, artifacts
3. Click **"Publish release"**
4. Uncheck **"Set as a pre-release"** if stable release

## What Gets Created

- **Draft + Pre-release**: Private until you publish
- **Docker Images**: Built from analysis repo (unless skipped)
  - `ghcr.io/phymea/earbox:v{VERSION}-cpu`
  - `ghcr.io/phymea/earbox:v{VERSION}-gpu`
  - `ghcr.io/phymea/earbox:v{VERSION}-gpu-weights`
- **Changelog**: Combined from both repos (commits since previous tags)
- **Runner Package**: `earbox-runner-{VERSION}.tar.gz` and `.zip`

## Notes

- All releases start as **drafts** (private) - review before publishing
- Changelog automatically includes commits from both repos
- Tag cannot be changed after release creation - delete and recreate if wrong
- Docker images use release version (without `v` prefix)

## Secrets Required

All secrets must be stored in the `earbox-releases` repository.

- `PUBLIC_RELEASES_PAT`: For cross-repo operations (required)
- `ANALYSIS_REPO_PAT`: If analysis repo is private (optional)
- `RUNNER_REPO_PAT`: If runner repo is private (optional)

### Setup Secrets

**1. Create Personal Access Tokens:**

GitHub CLI doesn't have a command to create PATs. Create them via web interface:
Go to https://github.com/settings/tokens → "Generate new token (classic)" and create:

- **PUBLIC_RELEASES_PAT** (required):
  - Name: `earbox-releases-workflow`
  - Expiration: Choose (e.g., 90 days or no expiration)
  - Scopes: `repo` (full control)
  
- **ANALYSIS_REPO_PAT** (if analysis repo is private):
  - Name: `earbox-analysis-checkout`
  - Expiration: Choose
  - Scopes: `repo` (full control)
  
- **RUNNER_REPO_PAT** (if runner repo is private):
  - Name: `earbox-runner-checkout`
  - Expiration: Choose
  - Scopes: `repo` (full control)

**Important:** Copy each token immediately after creation - you won't see it again!

**2. Add Secrets to Repository:**

```bash
# Add PUBLIC_RELEASES_PAT (required)
gh secret set PUBLIC_RELEASES_PAT -R phymea/earbox-releases
# Paste the token when prompted

# Add ANALYSIS_REPO_PAT (if analysis repo is private)
gh secret set ANALYSIS_REPO_PAT -R phymea/earbox-releases

# Add RUNNER_REPO_PAT (if runner repo is private)
gh secret set RUNNER_REPO_PAT -R phymea/earbox-releases
```

**Verify secrets are set:**
```bash
gh secret list -R phymea/earbox-releases
```
