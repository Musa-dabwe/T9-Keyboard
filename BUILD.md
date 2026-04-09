# Build and Deployment Guide - T9 Keyboard

This guide provides instructions for building, testing, and deploying the T9 Keyboard project.

## Prerequisites

To build this project, you need:
- **Java Development Kit (JDK) 17**: Ensure `JAVA_HOME` is set to a JDK 17+ installation.
- **Android SDK**: The `ANDROID_HOME` or `ANDROID_SDK_ROOT` environment variable must point to your Android SDK directory.
- **Android Gradle Plugin (AGP)**: The project currently uses AGP 8.13.0 (experimental stable).
- **Compile SDK**: 36 (Android 16 preview).
- **Target SDK**: 34.

## Building the Project

All build commands should be run from the root directory using the Gradle wrapper (`./gradlew`).

### Debug Build
To create a clean debug APK:
```bash
./gradlew clean assembleDebug
```
The output APK will be located at:
`app/build/outputs/apk/debug/app-debug.apk`

### Release Build
To create a clean release APK:
```bash
./gradlew clean assembleRelease
```
The output APK will be located at:
`app/build/outputs/apk/release/app-release.apk`

---

## Signing the Release APK

The release build requires a valid signing configuration. The project expects a `keystore.properties` file in the root directory to store signing credentials.

### `keystore.properties` Format
Create a file named `keystore.properties` in the root with the following keys:
```properties
storeFile=../your-keystore-file.jks
storePassword=your_store_password
keyAlias=your_key_alias
keyPassword=your_key_password
```
> **IMPORTANT**: Never hardcode credentials in `build.gradle.kts`. The `keystore.properties` and any `.jks` or `.keystore` files are automatically ignored by Git and must not be committed to the repository.

---

## Testing

To run all unit tests:
```bash
./gradlew test
```
Test reports are generated in:
`app/build/reports/tests/testDebugUnitTest/index.html`

---

## Installation

To install the debug APK directly to a connected device or emulator:
```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

---

## ProGuard / R8 Rules

The project includes custom ProGuard rules (`app/proguard-rules.pro`) to ensure functionality after code shrinking and obfuscation:

1.  **AOSP Dictionary Assets**:
    - *Rule*: `-keep class com.musa.t9keyboard.AospDictionary { *; }`
    - *Reason*: Prevents stripping of dictionary loading logic and ensures that AOSP dictionary data can be parsed correctly.
2.  **EmojiCompat**:
    - *Rule*: `-keep class androidx.emoji2.text.** { *; }`
    - *Reason*: Ensures the `androidx.emoji2` library can initialize and render system emojis at runtime without class stripping.
3.  **LearnedDictionary Serialization**:
    - *Rule*: `-keep class com.musa.t9keyboard.LearnedDictionary { *; }`
    - *Reason*: Protects the class used for `SharedPreferences` serialization of user-learned words and frequencies.
4.  **FontUtils Asset Loading**:
    - *Rule*: `-keep class com.musa.t9keyboard.FontUtils { *; }`
    - *Reason*: Ensures that dynamic font and typeface loading from the `assets/` directory remains functional.
