# Automated Release Workflow

This document describes the automated release workflow for openwrt-webauthn-helper.

## Overview

The repository automatically monitors the upstream [webauthn-helper](https://github.com/Tokisaki-Galaxy/webauthn-helper) repository and creates synchronized releases with ipk/apk packages for multiple OpenWrt architectures.

## Workflow Architecture

### 1. Sync Upstream Release (Primary Workflow)

**File**: `.github/workflows/sync-upstream-release.yml`

**Triggers**:
- Scheduled: Every 6 hours (`0 */6 * * *`)
- Manual: via workflow_dispatch

**Process**:
1. **Check Upstream Release**
   - Fetches latest release from `Tokisaki-Galaxy/webauthn-helper`
   - Compares upstream version with current Makefile version
   - Extracts release description (body) from upstream

2. **Update Version** (if new release detected)
   - Updates Makefile using `update.sh` script
   - Commits and pushes changes
   - Triggers build workflow via `repository_dispatch` with:
     - `upstream_version`: Version number (e.g., "1.0.0")
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
- Scheduled: Weekly on Friday at 5:00 UTC
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
![](https://img.shields.io/github/downloads/Tokisaki-Galaxy/webauthn-helper/v1.0.0/total?style=flat-square)

---

![](https://img.shields.io/github/downloads/Tokisaki-Galaxy/openwrt-webauthn-helper/v1.0.0/total?style=flat-square) ![](https://github.com/Tokisaki-Galaxy/openwrt-webauthn-helper/actions/workflows/Auto%20compile%20with%20openwrt%20sdk.yml/badge.svg)

> ⚠️ **Note**: Packages for other platforms are currently being built. Files will be added to this release as they complete.
> ⚠️ **注意**: 其他平台的软件包正在构建中，完成后将陆续添加到本页面
```

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

## Manual Trigger

To manually trigger a release sync:

1. Go to Actions tab
2. Select "Sync Upstream Release" workflow
3. Click "Run workflow"
4. Wait for the workflow to complete
5. The build workflow will be triggered automatically if a new version is detected

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
