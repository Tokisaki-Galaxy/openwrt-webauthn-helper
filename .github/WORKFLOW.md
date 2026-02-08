# Automated Release Workflow

This document describes the automated release workflow for openwrt-webauthn-helper.

## Prerequisites

### Required: Personal Access Token (PAT)

To enable automatic workflow triggering, you need to create a Personal Access Token (PAT):

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate a new token with the following permissions:
   - `repo` (Full control of private repositories)
   - `workflow` (Update GitHub Action workflows)
3. Copy the token
4. Go to your repository Settings → Secrets and variables → Actions
5. Create a new repository secret named `PAT` and paste the token

**Why is this needed?** GitHub's default `GITHUB_TOKEN` cannot trigger additional workflow runs for security reasons. The PAT allows the sync workflow to trigger the build workflow automatically.

**Fallback**: If no PAT is configured, the workflow will fall back to `GITHUB_TOKEN`, but automatic build triggering may not work. In this case, you'll need to manually trigger the build workflow.

## Overview

The repository automatically monitors the upstream [webauthn-helper](https://github.com/Tokisaki-Galaxy/webauthn-helper) repository and creates synchronized releases with ipk/apk packages for multiple OpenWrt architectures.

## Workflow Architecture

### 1. Sync Upstream Release (Primary Workflow)

**File**: `.github/workflows/sync-upstream-release.yml`

**Triggers**:
- Repository dispatch: Event type `sync-upstream` (for upstream repository notifications)
- Manual: via workflow_dispatch

**Process**:
1. Fetches latest release tag from `Tokisaki-Galaxy/webauthn-helper`
2. Compares with current repository's latest release tag
3. If tags differ:
   - Updates Makefile using `update.sh` script
   - Commits and pushes changes
   - Triggers build workflow with:
     - `upstream_tag`: Git tag (e.g., "v1.0.0")
     - `upstream_body`: Release description (base64 encoded)

### 2. Auto Compile with OpenWrt SDK (Build Workflow)

**File**: `.github/workflows/Auto compile with openwrt sdk.yml`

**Triggers**:
- `repository_dispatch` with type `upstream-release` (from Sync workflow)
- Manual: via workflow_dispatch

**Process**:
1. **Check Version**
   - Reads version from Makefile
   - Determines if release creation is needed

2. **Prepare Release**
   - If triggered by upstream release: Uses upstream release description
   - Otherwise: Uses default description
   - Adds build status badges and platform notes

3. **Build Packages**
   - Compiles for 50+ OpenWrt architectures
   - Supports both ipk (stable) and apk (snapshot) formats
   - Matrix includes: x86_64, aarch64, arm, mips, mipsel, riscv64, etc.

4. **Upload to Release**
   - Creates GitHub release with synchronized description
   - Uploads compiled packages as release assets

### 3. Version Check (Legacy Workflow)

**File**: `.github/workflows/version check.yml`

**Triggers**:
- Repository dispatch: Event type `sync-upstream` (for upstream repository notifications)
- Manual: via workflow_dispatch

**Process**:
- Similar to Sync Upstream Release workflow
- Maintained for backward compatibility
- Can be disabled once the new workflow is verified

## Release Description Format

When a new release is created, the description includes:

1. **Upstream Release Description** (if available)
   - Synchronized from the upstream repository
   - Preserves all formatting and content

2. **Separator** (`---`)

3. **Build Information**
   - Download counter badge
   - Build status badge
   - Platform build progress notes (in English and Chinese)

Example:
```markdown
[Upstream release description content here - varies by release]

---

![](https://img.shields.io/github/downloads/Tokisaki-Galaxy/openwrt-webauthn-helper/v1.0.0/total?style=flat-square) ![](https://github.com/Tokisaki-Galaxy/openwrt-webauthn-helper/actions/workflows/Auto%20compile%20with%20openwrt%20sdk.yml/badge.svg)

> ⚠️ **Note**: Packages for other platforms are currently being built. Files will be added to this release as they complete.
> ⚠️ **注意**: 其他平台的软件包正在构建中，完成后将陆续添加到本页面
```

**Note**: The upstream release description (if available) appears above the separator. If triggered manually without upstream metadata, only the build information section is included.

## Supported Architectures

### IPK Format (OpenWrt 24.10.2)
- x86_64
- ARM: cortex-a5, cortex-a7, cortex-a8, cortex-a9, cortex-a15, arm926ej-s, arm1176jzf-s, xscale, fa526
- AArch64: cortex-a53, cortex-a72, cortex-a76, generic
- MIPS: 24kc, 4kec, mips32
- MIPSel: 24kc, 24kc_24kf, 74kc, mips32
- RISC-V: riscv64

### APK Format (OpenWrt Snapshot)
- Same architecture support as IPK
- Built from latest OpenWrt snapshot releases
- Uses newer toolchain (GCC 14.3.0)

## Triggering Workflows

### Manual Trigger

To manually trigger a release sync:

1. Go to Actions tab
2. Select "Sync Upstream Release" or "Version check (Legacy)" workflow
3. Click "Run workflow"
4. Wait for the workflow to complete
5. The build workflow will be triggered automatically if a new version is detected

### Upstream Repository Notification

To trigger the workflow from the upstream repository or another source, use the GitHub API to send a `repository_dispatch` event:

```bash
curl -X POST \
  -H "Authorization: token YOUR_PAT_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/Tokisaki-Galaxy/openwrt-webauthn-helper/dispatches \
  -d '{"event_type":"sync-upstream"}'
```

Replace `YOUR_PAT_TOKEN` with a personal access token that has `repo` scope.

This allows the upstream repository to proactively notify this repository when a new release is published, saving GitHub Actions resources by avoiding scheduled cron jobs.

## Troubleshooting

### Workflow Not Triggering

- Check if the upstream repository has new releases
- Verify that GITHUB_TOKEN has proper permissions
- Review workflow logs in the Actions tab

### Build Failures

- Check individual job logs for specific platform failures
- Some architectures may fail due to SDK availability
- Failed jobs won't prevent other platforms from building

### Release Description Issues

- If description is not synced, check base64 encoding in workflow logs
- Verify that upstream release has a body/description
- Check `UPSTREAM_BODY` environment variable in build workflow

## Maintenance

### Updating the Workflow

1. Modify workflow files in `.github/workflows/`
2. Test manually using workflow_dispatch
3. Monitor first automated run
4. Adjust cron schedule if needed

### Adding New Architectures

Edit the matrix in `Auto compile with openwrt sdk.yml`:
```yaml
- platform: new_arch
  url_sdk: https://downloads.openwrt.org/.../sdk.tar.zst
  ver: "ipk" or "apk"
```

## Security Considerations

- Workflows use `GITHUB_TOKEN` with minimal required permissions
- All upstream data is validated before use
- Base64 encoding prevents injection attacks in release descriptions
- Update scripts validate checksums from upstream releases
