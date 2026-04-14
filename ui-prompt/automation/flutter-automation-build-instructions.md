# Flutter Setup, Pubspec, and Build Instructions for Mobile Automation

This document is reusable for humans and coding tools (Cursor, GitHub Copilot, and similar agents).
It covers Windows and macOS setup, pubspec automation dependencies, and APK build commands.

## 1) Prerequisites

- Flutter SDK (stable channel)
- Android Studio
- Android SDK + Platform Tools + Build Tools
- Java JDK 17 (recommended)
- Git installed

## 2) Install Flutter

### Windows

1. Download Flutter SDK zip from the official Flutter website.
2. Extract to a directory such as `C:\tools\flutter`.
3. Add Flutter to PATH:
   - Add `C:\tools\flutter\bin` in System Environment Variables.
4. Open a new PowerShell window and run:

```powershell
flutter --version
```

### macOS

1. Download Flutter SDK zip from the official Flutter website.
2. Extract to a directory such as `$HOME/development/flutter`.
3. Add Flutter to PATH in your shell profile (`~/.zshrc` or `~/.bash_profile`):

```bash
export PATH="$HOME/development/flutter/bin:$PATH"
```

4. Reload shell and verify:

```bash
flutter --version
```

## 3) Android Toolchain Setup (Windows and macOS)

1. Install Android Studio.
2. In SDK Manager, install:
   - Android SDK
   - Android SDK Platform-Tools
   - Android SDK Build-Tools
   - Android Emulator (optional)
3. Accept Android licenses:

```bash
flutter doctor --android-licenses
```

4. Validate everything:

```bash
flutter doctor -v
```

## 4) Open Project and Sync Dependencies

Project path example:

- Windows: `C:\path\to\your\project\inventory-management-app-main`
- macOS: `/path/to/your/project/inventory-management-app-main`

Run in project root:

```bash
flutter clean
flutter pub get
```

## 5) Pubspec Setup for Mobile Automation

Use or keep these entries in `pubspec.yaml`.

### Required in dependencies

```yaml
dependencies:
  flutter:
    sdk: flutter
```

### Recommended in dev_dependencies

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter
  test: ^1.26.2
```

What they are used for:

- `flutter_test`: unit and widget tests
- `integration_test`: end-to-end style tests on emulator/device
- `test`: Dart test tools and helpers

## 6) Build APK for Automation

Run from Flutter project root.

### Debug APK (primary for automation)

Use this as the default build for Appium/Flutter automation because it is faster to build and easier to debug.

```bash
flutter build apk --debug
```

Output:

- `build/app/outputs/flutter-apk/app-debug.apk`

Recommended automation capability value:

- `build/app/outputs/flutter-apk/app-debug.apk`

### Release APK

```bash
flutter build apk --release
```

Output:

- `build/app/outputs/flutter-apk/app-release.apk`

### Split APK by ABI

```bash
flutter build apk --release --split-per-abi
```

Typical outputs:

- `build/app/outputs/flutter-apk/app-armeabi-v7a-release.apk`
- `build/app/outputs/flutter-apk/app-arm64-v8a-release.apk`
- `build/app/outputs/flutter-apk/app-x86_64-release.apk`

## 7) Verify APK Artifact

### Windows (PowerShell)

```powershell
Get-ChildItem .\build\app\outputs\flutter-apk\*.apk | Select-Object Name, Length, LastWriteTime
```

### macOS (Terminal)

```bash
ls -lh build/app/outputs/flutter-apk/*.apk
```

## 8) Optional iOS Build Check (macOS only)

If your automation also needs iOS artifacts:

```bash
flutter build ios --debug --no-codesign
```

## 9) Reusable Agent Prompt

```text
Set up Flutter mobile automation for this project.
1. Validate Flutter and Android toolchain.
2. Ensure flutter_test, integration_test, and test are in dev_dependencies.
3. Run flutter clean and flutter pub get.
4. Build debug APK.
5. Return relative APK path and build result.
```

## 10) Success Criteria

- `flutter --version` works
- `flutter doctor -v` has no blocking issues
- `pubspec.yaml` includes automation dependencies
- `flutter pub get` succeeds
- APK exists in `build/app/outputs/flutter-apk/`
