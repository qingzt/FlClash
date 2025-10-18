# ARM64 Architecture Support for RPM and AppImage Packages

## Summary

This implementation adds support for building RPM and AppImage packages for ARM64 (aarch64) architecture on Linux systems.

## Changes Made

### 1. setup.dart
**File**: `setup.dart`

**Changes**:
- Removed the conditional check `if (arch == Arch.amd64)` that prevented RPM and AppImage builds on ARM64
- Enabled `appimage` and `rpm` targets for all Linux architectures (both amd64 and arm64)
- Updated `_getLinuxDependencies()` to install necessary dependencies (rpm, patchelf, libfuse2) for all architectures
- Download architecture-specific `appimagetool` binary (x86_64 for amd64, aarch64 for arm64)

**Lines modified**: 372-397, 469-473

### 2. flutter_distributor Submodule
**Submodule**: `plugins/flutter_distributor`
**Branch**: FlClash
**New commit**: 6a6165f

Three files were modified in the flutter_distributor submodule to add dynamic architecture detection:

#### a. app_package_maker_appimage.dart
**File**: `packages/flutter_app_packager/lib/src/makers/appimage/app_package_maker_appimage.dart`

**Changes**:
- Changed hardcoded `'ARCH': 'x86_64'` environment variable to use dynamic `_getArchitecture()` function
- Added `_getArchitecture()` helper function that detects system architecture using `uname -m`
  - Returns `'aarch64'` for ARM64 systems
  - Returns `'x86_64'` for AMD64 systems

#### b. app_package_maker_rpm.dart
**File**: `packages/flutter_app_packager/lib/src/makers/rpm/app_package_maker_rpm.dart`

**Changes**:
- Changed default architecture fallback from hardcoded `'x86_64'` to dynamic `_getArchitecture()`
- Added `_getArchitecture()` helper function with same logic as above
- This ensures RPM packages are built for the correct architecture directory structure

#### c. make_rpm_config.dart
**File**: `packages/flutter_app_packager/lib/src/makers/rpm/make_rpm_config.dart`

**Changes**:
- Changed RPM BuildArch specification from hardcoded `'x86_64'` to dynamic `_getArchitecture()`
- Added `_getArchitecture()` helper function with same logic as above
- This ensures the RPM package metadata correctly identifies its architecture

## How It Works

### Architecture Detection
All three modified files use the same architecture detection logic:

```dart
String _getArchitecture() {
  final result = Process.runSync('uname', ['-m']);
  if ('${result.stdout}'.trim() == 'aarch64') {
    return 'aarch64';
  } else {
    return 'x86_64';
  }
}
```

This matches the approach already used in the DEB package maker (`make_deb_config.dart`), ensuring consistency across all Linux package formats.

### Build Process
When building on an ARM64 system:
1. `setup.dart` detects the architecture and downloads `appimagetool-aarch64.AppImage`
2. Flutter distributor builds the application for `linux-arm64` platform
3. AppImage maker uses `ARCH=aarch64` environment variable when running appimagetool
4. RPM maker creates packages in the `RPMS/aarch64/` directory with `BuildArch: aarch64` metadata

When building on an AMD64 system:
1. `setup.dart` downloads `appimagetool-x86_64.AppImage`
2. Flutter distributor builds for `linux-x64` platform
3. AppImage maker uses `ARCH=x86_64` environment variable
4. RPM maker creates packages in `RPMS/x86_64/` directory with `BuildArch: x86_64` metadata

## Testing

To test ARM64 builds, use an ARM64 Linux system (or GitHub Actions ubuntu-24.04-arm runner):

```bash
# Build Linux ARM64 packages
dart setup.dart linux --arch arm64

# This will now produce:
# - dist/FlClash-*-linux-arm64.deb
# - dist/FlClash-*-linux-arm64.AppImage
# - dist/FlClash-*-linux-arm64.rpm
```

## CI/CD Integration

The existing GitHub Actions workflow already includes ARM64 Linux builds:

```yaml
- platform: linux
  os: ubuntu-24.04-arm
  arch: arm64
```

With these changes, this workflow will now successfully build all three Linux package formats (deb, AppImage, rpm) for ARM64 architecture.

## Dependencies

The following dependencies are required for building on ARM64:
- `rpm` - RPM package builder
- `patchelf` - Tool for modifying rpath in shared libraries
- `libfuse2` - Required for AppImage runtime
- `appimagetool-aarch64.AppImage` - AppImage packaging tool for ARM64

All dependencies are automatically installed by `_getLinuxDependencies()` in setup.dart.

## Backwards Compatibility

These changes maintain full backwards compatibility with existing AMD64 builds:
- AMD64 builds continue to work exactly as before
- No changes to package naming or output structure
- Existing build commands and workflows remain unchanged

## Files Modified

1. `setup.dart` - Main build script
2. `FLUTTER_DISTRIBUTOR_CHANGES.md` - Documentation for submodule changes
3. `flutter_distributor_arm64_support.patch` - Patch file for reference
4. `plugins/flutter_distributor` - Submodule updated to commit 6a6165f
   - `packages/flutter_app_packager/lib/src/makers/appimage/app_package_maker_appimage.dart`
   - `packages/flutter_app_packager/lib/src/makers/rpm/app_package_maker_rpm.dart`
   - `packages/flutter_app_packager/lib/src/makers/rpm/make_rpm_config.dart`

## References

- AppImageKit releases: https://github.com/AppImage/AppImageKit/releases
- RPM packaging documentation: https://rpm-packaging-guide.github.io/
- Flutter distributor: https://github.com/chen08209/flutter_distributor (FlClash branch)
