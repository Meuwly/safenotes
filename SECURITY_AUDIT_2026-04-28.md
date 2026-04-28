# Security & Update Audit (2026-04-28)

This project appears to have been maintained around Flutter 3.19 / Dart 3.3 era and needs a structured refresh.

## Scope checked

- `pubspec.yaml`
- `pubspec.lock`
- Android build and manifest files
- iOS Podfile and plist settings
- Repo hygiene for potential secret leakage controls

> Note: runtime dependency CVE lookup could not be completed in this environment because package/advisory APIs are blocked (HTTP 403), and `flutter`/`dart` CLIs are unavailable.

## High-priority security findings

Status in this branch:
- ✅ Legacy Android storage permissions and `requestLegacyExternalStorage` removed.
- ✅ APK v1 signing disabled.
- ✅ iOS document sharing keys removed from `Info.plist`.
- ✅ Podfile now pins iOS minimum platform to `13.0`.

1. **Legacy external storage model was enabled on Android (now corrected)**
   - `android:requestLegacyExternalStorage="true"` was set.
   - Broad storage permissions were requested: `READ_EXTERNAL_STORAGE` and `WRITE_EXTERNAL_STORAGE`.
   - Risk: larger data exposure surface and weaker scoped-storage isolation on modern Android.
   - Recommendation: migrate backup/import/export to Storage Access Framework (`ACTION_OPEN_DOCUMENT`, `ACTION_CREATE_DOCUMENT`) and remove legacy/broad storage permissions.

2. **APK v1 signing was enabled (now corrected)**
   - `v1SigningEnabled true` was present while `minSdkVersion` is 25.
   - Risk: unnecessary compatibility mode and weaker signing scheme than modern v2/v3/v4 requirements.
   - Recommendation: disable v1 signing for release builds.

3. **iOS file sharing was enabled (now corrected)**
   - `UIFileSharingEnabled` and `LSSupportsOpeningDocumentsInPlace` were enabled.
   - Risk: app documents can be exposed through Finder/iTunes workflows, which may conflict with the app's secure-note posture.
   - Recommendation: keep enabled only if this export path is explicitly required and documented; otherwise disable.

## Upgrade debt that affects security posture

1. **Very old Dart lower bound and pinned Flutter toolchain**
   - Dart SDK constraint still allows `>=2.12.0`.
   - Flutter is pinned to `3.19.3` in `pubspec.yaml`.
   - Recommendation: move to a current stable Flutter/Dart baseline, then run full dependency and platform migration.

2. **Dependency refresh needed**
   - Lockfile shows many package versions tied to older ecosystems.
   - Recommendation (when toolchain is available):
     - `flutter pub outdated`
     - `flutter pub upgrade --major-versions`
     - `flutter pub audit` (or equivalent CVE scan)
     - full regression testing on Android + iOS.

3. **Android/iOS build stack modernization**
   - Gradle wrapper is 7.4 (old for current Android builds).
   - iOS Podfile does not explicitly set a modern minimum platform.
   - Recommendation: update Gradle/AGP/Kotlin and set explicit iOS minimum target supported by current Flutter plugins.

## Immediate action plan (order)

1. Remove legacy/broad storage permissions and adopt scoped document APIs.
2. Disable v1 signing.
3. Review/disable iOS file-sharing flags unless strictly needed.
4. Upgrade Flutter + Dart baseline.
5. Refresh dependencies and run CVE audit with live tooling.
6. Run end-to-end tests and a fresh release build in CI.

## Commands executed during this audit

- `rg --files`
- `sed -n '1,220p' pubspec.yaml`
- `sed -n '1,240p' pubspec.lock`
- `sed -n '1,220p' android/build.gradle`
- `sed -n '1,260p' android/app/build.gradle`
- `sed -n '1,220p' android/gradle/wrapper/gradle-wrapper.properties`
- `sed -n '1,220p' android/gradle.properties`
- `sed -n '1,220p' ios/Podfile`
- `sed -n '1,220p' ios/Runner/Info.plist`
- `sed -n '1,240p' android/app/src/main/AndroidManifest.xml`
- `flutter --version` (failed: command unavailable)
- `dart --version` (failed: command unavailable)
- `curl -I https://pub.dev` (403 in this environment)
