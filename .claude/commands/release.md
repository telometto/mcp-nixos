---
allowed-tools: Bash, Read, Edit, Glob, Grep, Write, TodoWrite
description: Perform a version release with automated PyPI publishing and Docker image builds
---

# Release

Automate the release process for MCP-NixOS, including version bumping, changelog updates, tag creation, and triggering CI/CD pipelines for PyPI and Docker deployments.

## Overview

This command performs a complete release workflow:
1. Analyzes CI/CD configuration to understand the release pipeline
2. Reviews all commits since the last release
3. Determines appropriate version bump (patch/minor/major)
4. Updates version in pyproject.toml and RELEASE_NOTES.md
5. Creates and pushes git tag
6. Creates GitHub release (triggers publish workflow)
7. Verifies deployments to PyPI and Docker registries

## Key Files

### Version & Build Configuration
- `pyproject.toml` - Python package version and metadata
- `RELEASE_NOTES.md` - Comprehensive release changelog

### CI/CD Workflows
- `.github/workflows/publish.yml` - PyPI and Docker publishing on release
- `.github/workflows/ci.yml` - Continuous integration and testing
- `.github/workflows/deploy-flakehub.yml` - FlakeHub publishing

### Release Triggers
The publish workflow triggers on:
- **GitHub Release** (published) - Deploys to Test PyPI → Production PyPI → Docker registries
- **Manual Workflow Dispatch** - Docker-only builds for specific tags

## Execute

### 1. Analyze CI/CD Workflows

Read the publish workflow to understand the release pipeline:
```bash
# Review publish workflow configuration
cat .github/workflows/publish.yml

# Check CI workflow for testing and build steps
cat .github/workflows/ci.yml
```

### 2. Review Changes Since Last Release

```bash
# Get current version from pyproject.toml
grep '^version = ' pyproject.toml

# List recent version tags
git tag --list 'v*' --sort=-version:refname | head -10

# Review commits since last release (replace v1.0.1 with actual last tag)
git log v1.0.1..HEAD --oneline

# Get detailed commit log for changelog
git log v1.0.1..HEAD --pretty=format:"%h %s"
```

### 3. Update Version

Determine the appropriate version bump:
- **Patch** (x.y.Z): Bug fixes, CI/CD improvements, documentation
- **Minor** (x.Y.0): New features, backward-compatible changes
- **Major** (X.0.0): Breaking changes

Edit `pyproject.toml`:
```toml
version = "X.Y.Z"  # Update this line
```

### 4. Update Release Notes

Add new release section at the top of `RELEASE_NOTES.md`:

```markdown
# MCP-NixOS: vX.Y.Z Release Notes - [Release Title]

## Overview

Brief description of the release focus (1-2 sentences).

## Changes in vX.Y.Z

### 🚀 Major Updates / CI/CD Enhancements / Bug Fixes

- **Feature Name**: Description of change
- **Another Feature**: Description

### 🔧 Bug Fixes (if applicable)

- **Issue Description**: Fix details
- **Another Fix**: Fix details

### 🛠️ Development Experience (if applicable)

- **Improvement**: Description

### 📦 Dependencies

- List any dependency changes or "No changes from vX.Y.Z"

## Installation

```bash
# Install with pip
pip install mcp-nixos==X.Y.Z

# Install with uv
uv pip install mcp-nixos==X.Y.Z

# Install with uvx
uvx mcp-nixos==X.Y.Z
```

## Docker Images

```bash
# Pull from Docker Hub
docker pull utensils/mcp-nixos:X.Y.Z

# Pull from GitHub Container Registry
docker pull ghcr.io/utensils/mcp-nixos:X.Y.Z
```

## Migration Notes

Describe any breaking changes or migration steps. If none: "This is a drop-in replacement for vX.Y.Z with no user-facing changes."

## Contributors

- James Brink (@utensils) - [Role/Description]

---
```

### 5. Commit and Push Changes

```bash
# Commit version bump
git add pyproject.toml
git commit -m "chore: Bump version to X.Y.Z"
git push

# Commit changelog updates
git add RELEASE_NOTES.md
git commit -m "docs: Update RELEASE_NOTES.md for vX.Y.Z"
git push
```

### 6. Create and Push Version Tag

```bash
# Create annotated tag
git tag -a vX.Y.Z -m "Release vX.Y.Z: [Short description]"

# Push tag to trigger workflows
git push origin vX.Y.Z
```

### 7. Create GitHub Release

This triggers the publish workflow which handles PyPI and Docker deployments:

```bash
# Create GitHub release (triggers publish.yml workflow)
gh release create vX.Y.Z \
  --title "vX.Y.Z: [Release Title]" \
  --notes "## Overview

Brief description of the release.

## Highlights

- 🚀 Key feature or improvement
- 🔒 Security fix or important update
- 🐳 Docker/infrastructure improvement

## Installation

```bash
pip install mcp-nixos==X.Y.Z
# or
uv pip install mcp-nixos==X.Y.Z
```

## Docker Images

```bash
docker pull utensils/mcp-nixos:X.Y.Z
docker pull ghcr.io/utensils/mcp-nixos:X.Y.Z
```

See [RELEASE_NOTES.md](https://github.com/utensils/mcp-nixos/blob/main/RELEASE_NOTES.md) for full details."
```

### 8. Monitor CI/CD Pipeline

```bash
# Watch the publish workflow execution
gh run list --workflow=publish.yml --limit 5

# Monitor specific run (get run ID from above)
gh run watch <RUN_ID>

# Check workflow status
gh run view <RUN_ID>
```

## Verify

### 1. Verify PyPI Deployment

Test the new version is available on PyPI:

```bash
# Test with uvx (uses PyPI)
uvx mcp-nixos@X.Y.Z --help

# Should show:
# - FastMCP server banner
# - Correct version installed
# - Server starts successfully
```

### 2. Verify Docker Hub Deployment

```bash
# Pull from Docker Hub
docker pull utensils/mcp-nixos:X.Y.Z

# Verify image runs
docker run --rm utensils/mcp-nixos:X.Y.Z --help

# Check image details
docker inspect utensils/mcp-nixos:X.Y.Z | grep -A 5 "Labels"
```

### 3. Verify GitHub Container Registry Deployment

```bash
# Pull from GHCR
docker pull ghcr.io/utensils/mcp-nixos:X.Y.Z

# Verify image runs
docker run --rm ghcr.io/utensils/mcp-nixos:X.Y.Z --help

# Confirm both images have same digest
docker images --digests | grep mcp-nixos | grep X.Y.Z
```

### 4. Verify GitHub Release

```bash
# Check release was created
gh release view vX.Y.Z

# Verify release URL
echo "https://github.com/utensils/mcp-nixos/releases/tag/vX.Y.Z"
```

## Report

Provide a summary of the release:

```
## ✅ Release vX.Y.Z Complete!

Successfully performed [patch/minor/major] version bump from vA.B.C to vX.Y.Z.

### 📋 Release Summary

**Version:** vX.Y.Z
**Release URL:** https://github.com/utensils/mcp-nixos/releases/tag/vX.Y.Z
**Commits Since Last Release:** N commits

### ✅ Completed Tasks

1. ✅ Analyzed CI/CD workflows
2. ✅ Reviewed N commits since vA.B.C
3. ✅ Updated version in pyproject.toml
4. ✅ Updated RELEASE_NOTES.md
5. ✅ Committed and pushed changes
6. ✅ Created and pushed tag vX.Y.Z
7. ✅ Created GitHub release
8. ✅ Verified CI/CD pipeline success

### 🚀 Deployment Verification

- ✅ **PyPI**: https://pypi.org/project/mcp-nixos/X.Y.Z/ - Working
- ✅ **Docker Hub**: `utensils/mcp-nixos:X.Y.Z` - Working
- ✅ **GHCR**: `ghcr.io/utensils/mcp-nixos:X.Y.Z` - Working

### 📦 What Was Released

- **Version**: A.B.C → X.Y.Z ([patch/minor/major] bump)
- **Changes**: [Brief summary of changes]
- **Release Notes**: Updated in RELEASE_NOTES.md

All distribution channels verified and operational! 🚀
```

## Troubleshooting

### Publish Workflow Fails

1. Check workflow logs: `gh run view <RUN_ID> --log-failed`
2. Common issues:
   - Test PyPI rate limiting: Wait and retry
   - Docker build failures: Check Dockerfile and dependencies
   - Permissions: Verify DOCKERHUB_USERNAME, DOCKERHUB_TOKEN secrets

### PyPI Package Not Available

- Check Test PyPI first: https://test.pypi.org/project/mcp-nixos/
- Wait 2-5 minutes for CDN propagation
- Verify deploy-prod job completed successfully

### Docker Images Not Available

- Check workflow completed: `gh run view <RUN_ID>`
- Docker job runs independently and may take 5-10 minutes
- Verify docker login secrets are configured correctly

### Tag Already Exists

```bash
# Delete local tag
git tag -d vX.Y.Z

# Delete remote tag (careful!)
git push origin :refs/tags/vX.Y.Z

# Recreate tag
git tag -a vX.Y.Z -m "Release vX.Y.Z: [description]"
git push origin vX.Y.Z
```

## Notes

- **Release Types**: Only non-prerelease versions deploy to production PyPI
- **Docker Builds**: Multi-architecture builds (amd64, arm64) run automatically
- **GHCR Visibility**: Workflow attempts to set packages public automatically
- **FlakeHub**: Separate workflow publishes to FlakeHub on version tags
- **Manual Docker Builds**: Use workflow dispatch in `.github/workflows/publish.yml` for Docker-only builds

## Related Documentation

- [RELEASE_WORKFLOW.md](../../RELEASE_WORKFLOW.md) - Detailed release procedures
- [RELEASE_NOTES.md](../../RELEASE_NOTES.md) - Complete release history
- [CI/CD Workflows](../../.github/workflows/) - All GitHub Actions workflows
