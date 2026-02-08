# Testing the Automated Release Workflow

This guide explains how to test the automated release workflow.

## Prerequisites

- Repository admin/write access
- GitHub Actions enabled
- Understanding of GitHub Actions workflows
- **Recommended**: Personal Access Token (PAT) configured as `PAT` secret
  - Without PAT, automatic workflow triggering may not work
  - See WORKFLOW.md for PAT setup instructions

## Method 1: Manual Trigger (Recommended for Testing)

### Test the Sync Workflow

1. Go to GitHub Actions tab
2. Select "Sync Upstream Release" workflow
3. Click "Run workflow" button
4. Select the branch (usually `main`)
5. Click "Run workflow"

**Expected Behavior**:
- If upstream has a newer version than the Makefile:
  - Updates the Makefile
  - Commits the changes
  - Triggers the build workflow automatically
- If versions match:
  - Logs "No new release"
  - Exits without further action

### Test the Build Workflow

1. Go to GitHub Actions tab
2. Select "Auto compile with openwrt sdk" workflow
3. Click "Run workflow" button
4. Select the branch
5. Click "Run workflow"

**Expected Behavior**:
- Checks current version from Makefile
- Compares with last release tag
- If different, creates a new release and builds packages
- If same, skips release creation

## Method 2: Wait for Scheduled Run

The sync workflow runs automatically every 6 hours:
- 00:00 UTC
- 06:00 UTC
- 12:00 UTC
- 18:00 UTC

Check the Actions tab after these times to see the workflow runs.

## Method 3: Create a Test Release (Advanced)

To fully test the upstream synchronization:

### Option A: Using a Fork

1. Fork the upstream webauthn-helper repository
2. Update the workflow to point to your fork temporarily:
   ```yaml
   upstream_release=$(curl -s https://api.github.com/repos/YOUR_USERNAME/webauthn-helper/releases/latest)
   ```
3. Create a test release in your fork
4. Trigger the sync workflow
5. Verify it picks up the test release

### Option B: Simulating Workflow Locally

You can simulate the workflow logic locally to verify behavior:

```bash
# Check upstream release
upstream_release=$(curl -s https://api.github.com/repos/Tokisaki-Galaxy/webauthn-helper/releases/latest)
upstream_tag=$(echo "$upstream_release" | jq -r '.tag_name')
upstream_version=$(echo "$upstream_tag" | sed 's/^v//')

echo "Upstream version: $upstream_version"

# Check current version
current_version=$(grep "^RUST_WEBAUTHN_HELPER_VERSION:=" openwrt-webauthn-helper/Makefile | cut -d'=' -f2)
echo "Current version: $current_version"

# Compare
if [ "$upstream_version" != "$current_version" ]; then
  echo "✓ New version detected - workflow would trigger"
else
  echo "✓ Already up to date - workflow would skip"
fi
```

This will show you what would happen without actually running the workflow.

## Verification Checklist

After running the workflow, verify:

### Sync Workflow
- [ ] Workflow completed successfully
- [ ] If update detected: Makefile was updated
- [ ] If update detected: Commit was created
- [ ] If update detected: Build workflow was triggered
- [ ] Workflow logs show correct version numbers

### Build Workflow
- [ ] All required jobs started
- [ ] Release was created (if version changed)
- [ ] Release description includes upstream content
- [ ] Release description includes build badges
- [ ] Release description includes platform notes
- [ ] Build jobs are running for all architectures

### Release Output
- [ ] Release tag matches upstream version (e.g., v1.0.0)
- [ ] Release name matches tag
- [ ] Release description has upstream content
- [ ] Release description has separator (`---`)
- [ ] Release description has build information
- [ ] Packages are being uploaded as they complete

## Common Issues and Solutions

### Issue: Build workflow not triggered automatically

**Cause**: GitHub's `GITHUB_TOKEN` cannot trigger additional workflows for security reasons.

**Solution**: 
1. Create a Personal Access Token (PAT) with `repo` and `workflow` permissions
2. Add it as a repository secret named `PAT`
3. The workflow will automatically use it: `token: ${{ secrets.PAT || secrets.GITHUB_TOKEN }}`
4. See WORKFLOW.md for detailed PAT setup instructions

**Workaround**: Manually trigger the build workflow after sync completes

### Issue: Workflow doesn't trigger automatically

**Solution**: 
- Check if the cron schedule is correct
- Verify Actions are enabled for the repository
- Check workflow permissions in repository settings

### Issue: Build workflow not triggered by sync workflow

**Solution**:
- Verify `peter-evans/repository-dispatch@v3` action is working
- Check that `GITHUB_TOKEN` has correct permissions
- Review workflow logs for dispatch errors

### Issue: Release description not synced from upstream

**Solution**:
- Check if `UPSTREAM_BODY` environment variable is set
- Verify base64 encoding/decoding in workflow logs
- Check if upstream release has a body/description

### Issue: Some platform builds fail

**Solution**:
- This is expected for some platforms due to SDK availability
- Check individual job logs for details
- Failed jobs won't prevent other platforms from building
- Packages for successful platforms will still be uploaded

## Debugging

### Enable Debug Logging

Add to workflow file (temporarily):

```yaml
env:
  ACTIONS_STEP_DEBUG: true
  ACTIONS_RUNNER_DEBUG: true
```

### Check Workflow Logs

1. Go to Actions tab
2. Click on the workflow run
3. Click on individual jobs
4. Expand steps to see detailed logs

### Useful Log Checks

Look for these in the logs:

**Sync Workflow**:
```
Upstream tag: v1.0.0
Upstream version: 1.0.0
Current version: 1.0.0
New release detected: 1.0.0  # or "No new release"
```

**Build Workflow**:
```
Using upstream release description  # or "Using default release description"
```

### Manual Version Check

Check current Makefile version:
```bash
grep "^RUST_WEBAUTHN_HELPER_VERSION:=" openwrt-webauthn-helper/Makefile
```

Check upstream version:
```bash
curl -s https://api.github.com/repos/Tokisaki-Galaxy/webauthn-helper/releases/latest | jq -r '.tag_name'
```

## Testing New Versions

When the upstream releases a new version (e.g., v1.0.1):

1. Within 6 hours, the sync workflow should detect it
2. Makefile will be updated automatically
3. A commit will be created
4. Build workflow will be triggered
5. A new release will be created with packages

Monitor the process:
1. Check Actions tab for workflow runs
2. Check Commits for the version update
3. Check Releases for the new release
4. Verify release description matches upstream

## Rollback Procedure

If something goes wrong:

1. Go to the commit history
2. Find the last working version
3. Revert the problematic commit:
   ```bash
   git revert <commit-hash>
   git push
   ```
4. Fix the workflow files
5. Test again

## Success Criteria

The workflow is working correctly when:

1. ✅ New upstream releases are detected within 6 hours
2. ✅ Makefile is updated automatically
3. ✅ Build workflow triggers automatically
4. ✅ Release is created with synced description
5. ✅ Packages are built for all platforms
6. ✅ No manual intervention needed

## Next Steps

Once verified:

1. Monitor the first few automated runs
2. Adjust cron schedule if needed
3. Consider disabling the legacy "Version check" workflow
4. Update documentation if needed
