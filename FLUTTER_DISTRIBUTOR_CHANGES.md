# Flutter Distributor Changes for ARM64 Support

This document describes the changes required in the `plugins/flutter_distributor` submodule to enable ARM64 architecture support for RPM and AppImage packages.

## Overview

The flutter_distributor plugin needs to be updated to dynamically detect the system architecture (aarch64 for ARM64, x86_64 for AMD64) instead of hardcoding x86_64.

## Changes Required

The patch file `flutter_distributor_arm64_support.patch` contains all the necessary changes. To apply:

```bash
cd plugins/flutter_distributor
git apply ../../flutter_distributor_arm64_support.patch
```

## Modified Files

1. **packages/flutter_app_packager/lib/src/makers/appimage/app_package_maker_appimage.dart**
   - Changed hardcoded `'ARCH': 'x86_64'` to use dynamic `_getArchitecture()` function
   - Added `_getArchitecture()` helper function to detect architecture

2. **packages/flutter_app_packager/lib/src/makers/rpm/app_package_maker_rpm.dart**
   - Changed default architecture from `'x86_64'` to use dynamic `_getArchitecture()` function
   - Added `_getArchitecture()` helper function to detect architecture

3. **packages/flutter_app_packager/lib/src/makers/rpm/make_rpm_config.dart**
   - Changed default buildArch from `'x86_64'` to use dynamic `_getArchitecture()` function
   - Added `_getArchitecture()` helper function to detect architecture

## Architecture Detection Logic

The `_getArchitecture()` function uses `uname -m` to detect the system architecture:
- Returns `'aarch64'` for ARM64 systems
- Returns `'x86_64'` for AMD64 systems

This matches the existing logic already used in the DEB package maker.

## Alternative: Manual Application

If the patch doesn't apply cleanly, manually add the following function to each of the three files listed above:

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

And replace hardcoded `'x86_64'` or `'ARCH': 'x86_64'` with calls to `_getArchitecture()`.
