# Changes Made

## 1. Migrated from flutter_distributor submodule to fastforge

### What changed:
- Removed the `plugins/flutter_distributor` git submodule
- Updated `.gitmodules` to remove the flutter_distributor submodule entry
- Modified `setup.dart` to use fastforge from pub.dev instead of a local submodule

### Why:
- Fastforge (https://github.com/fastforgedev/fastforge) is the official successor to flutter_distributor
- Fastforge provides the same functionality and maintains backward compatibility via the `flutter_distributor` package
- Using the published package from pub.dev is more maintainable than maintaining a submodule

### Changes in setup.dart:
```dart
// Before:
static Future<void> getDistributor() async {
  final distributorDir = join(
    current,
    'plugins',
    'flutter_distributor',
    'packages',
    'flutter_distributor',
  );
  await exec(
    name: 'clean distributor',
    Build.getExecutable('flutter clean'),
    workingDirectory: distributorDir,
  );
  await exec(
    name: 'upgrade distributor',
    Build.getExecutable('flutter pub upgrade'),
    workingDirectory: distributorDir,
  );
  await exec(
    name: 'get distributor',
    Build.getExecutable('dart pub global activate -s path $distributorDir'),
  );
}

// After:
static Future<void> getDistributor() async {
  // Using fastforge from pub.dev (successor of flutter_distributor)
  await exec(
    name: 'get fastforge',
    Build.getExecutable('dart pub global activate flutter_distributor'),
  );
}
```

## 2. Added ARM64 support for RPM and AppImage builds

### What changed:
- Modified `setup.dart` to build RPM and AppImage packages for both amd64 and arm64 architectures
- Updated `_getLinuxDependencies()` to install build tools for both architectures

### Before:
RPM and AppImage were only built for amd64 (x86_64):
```dart
final targets = [
  'deb',
  if (arch == Arch.amd64) 'appimage',
  if (arch == Arch.amd64) 'rpm',
].join(',');
```

Dependencies were only installed for amd64:
```dart
if (arch == Arch.amd64) {
  await Build.exec(Build.getExecutable('sudo apt install -y rpm patchelf'));
  await Build.exec(Build.getExecutable('sudo apt install -y libfuse2'));
  // ... download appimagetool
}
```

### After:
RPM and AppImage are now built for both amd64 and arm64:
```dart
final targets = [
  'deb',
  'appimage',
  'rpm',
].join(',');
```

Dependencies are installed for both architectures:
```dart
// Install rpm and appimage tools for both amd64 and arm64
await Build.exec(Build.getExecutable('sudo apt install -y rpm patchelf'));
await Build.exec(Build.getExecutable('sudo apt install -y libfuse2'));

final downloadName = arch == Arch.amd64 ? 'x86_64' : 'aarch64';
await Build.exec(
  Build.getExecutable(
    'wget -O appimagetool https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-$downloadName.AppImage',
  ),
);
```

## Usage Examples

### Build Linux packages for ARM64:
```bash
# Build all Linux packages (deb, rpm, appimage) for ARM64
dart ./setup.dart linux --arch arm64 --out app

# Build only the core for ARM64
dart ./setup.dart linux --arch arm64 --out core
```

### Build Linux packages for AMD64:
```bash
# Build all Linux packages (deb, rpm, appimage) for AMD64
dart ./setup.dart linux --arch amd64 --out app
```

## Compatibility Notes

- The `flutter_distributor` command is still used in the build process, which is provided by the fastforge package for backward compatibility
- Fastforge supports both RPM and AppImage for arm64 architecture out of the box
- The build process automatically detects the architecture and downloads the appropriate appimagetool (x86_64 or aarch64)
