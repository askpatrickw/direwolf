# Release Process

This document describes how to create a new release of Direwolf with pre-built binaries for Windows and macOS.

## Overview

The release process is automated using GitHub Actions. When you push a version tag, the workflow automatically:
1. Builds binaries for Windows (x86_64), macOS (x86_64), and macOS (arm64/Apple Silicon)
2. Creates distributable ZIP packages
3. Uploads the packages to the GitHub release page

## Creating a Release

### 1. Prepare the Release

Before creating a release tag, ensure:
- All desired changes are merged to the master or dev branch
- Version numbers are updated in `CMakeLists.txt`
- `CHANGES.md` is updated with release notes
- All tests pass on the target branch

### 2. Create and Push a Tag

```bash
# Create an annotated tag
git tag -a v1.8.1 -m "Release version 1.8.1"

# Or for a pre-release (beta, alpha, rc)
git tag -a v1.8.1-beta1 -m "Release version 1.8.1 beta 1"

# Push the tag to trigger the release workflow
git push origin v1.8.1
```

### 3. Automated Build Process

Once the tag is pushed, the GitHub Actions workflow (`.github/workflows/release.yml`) will:

1. **Build for Windows x86_64**:
   - Uses MinGW compiler on windows-latest runner
   - Creates `direwolf-{version}_x86_64.zip`

2. **Build for macOS x86_64** (Intel Macs):
   - Uses macOS-13 runner
   - Installs dependencies via Homebrew
   - Creates `direwolf-{version}_x86_64.zip`

3. **Build for macOS arm64** (Apple Silicon):
   - Uses macos-latest runner (M-series)
   - Installs dependencies via Homebrew
   - Creates `direwolf-{version}_arm64.zip`

4. **Upload to GitHub Release**:
   - All ZIP files are automatically uploaded to the release
   - Pre-release tags (containing 'beta', 'alpha', or 'rc') are marked as pre-releases
   - Regular version tags are marked as full releases

### 4. Add Release Notes

After the automated build completes:
1. Go to the [Releases page](https://github.com/wb2osz/direwolf/releases)
2. Find your new release
3. Click "Edit release"
4. Add or update the release description with:
   - New features
   - Bug fixes
   - Breaking changes
   - Upgrade instructions
   - Platform-specific notes

## Manual Release Trigger

You can also trigger the release workflow manually:

1. Go to Actions → Release Build
2. Click "Run workflow"
3. Enter the tag name (e.g., `v1.8.1`)
4. Click "Run workflow"

This is useful for:
- Rebuilding a release if the automated build failed
- Creating a release for an existing tag
- Testing the release process

## Version Tag Formats

The workflow is triggered by tags matching these patterns:
- `v*` (e.g., `v1.8`, `v1.8.0`, `v1.8.1-beta1`)
- `[0-9]+.[0-9]+*` (e.g., `1.8`, `1.8.0`, `1.8.1-beta1`)

## Troubleshooting

### Build Fails

If the automated build fails:
1. Check the Actions tab for error logs
2. Fix the issue in the code
3. Delete the tag: `git tag -d v1.8.1 && git push origin :refs/tags/v1.8.1`
4. Create a new tag with the fix
5. Push the tag again

### Missing Dependencies

macOS builds require these Homebrew packages:
- cmake
- portaudio
- hamlib
- gpsd
- hidapi

Windows builds require:
- MinGW toolchain (provided by the runner)

### Package Not Created

If `make package` fails:
- Verify CMakeLists.txt has proper CPack configuration
- Check that CMAKE_BUILD_TYPE is set to Release
- Review build logs for packaging errors

## Platform-Specific Notes

### macOS
- **x86_64**: For Intel Macs (2020 and earlier)
- **arm64**: For Apple Silicon Macs (M1, M2, M3, etc.)
- Users can download the appropriate version for their hardware
- The arm64 version will also run on Intel Macs via Rosetta 2

### Windows
- **x86_64**: For 64-bit Windows systems
- Includes all necessary DLLs in the ZIP package
- Users should unzip and run directly

## Continuous Integration

The regular CI workflow (`.github/workflows/ci.yml`) runs on every push and pull request to:
- Verify builds work on all platforms
- Run unit tests
- Catch issues before release

The release workflow only runs on tags to avoid unnecessary builds and to ensure release artifacts are only created for official releases.
