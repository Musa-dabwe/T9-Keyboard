# Dependency Audit Report - T9 Keyboard

This report summarizes the audit of dependencies for the T9 Keyboard project as of April 3, 2026.

## Summary Table

| Library Name | Declared Version | Latest Stable Version | Status | Recommended Action |
| :--- | :--- | :--- | :--- | :--- |
| **Android Gradle Plugin (AGP)** | 8.13.2 | 9.1.0 | ✅ Up to date | **Updated to 8.13.2.** Resolved Kotlin 2.3.20 / D8 metadata mismatch. |
| **Kotlin** | 2.1.0 | 2.3.20 | ⚠️ Outdated | **Updated to 2.3.20.** |
| **AndroidX Core KTX** | 1.17.0 | 1.18.0 | ⚠️ Outdated | **Updated to 1.18.0.** |
| **AppCompat** | 1.7.1 | 1.7.1 | ✅ Up to date | None. |
| **Material** | 1.13.0 | 1.13.0 | ✅ Up to date | None. |
| **ConstraintLayout** | 2.2.1 | 2.2.1 | ✅ Up to date | None. |
| **RecyclerView** | 1.3.2 | 1.4.0 | ⚠️ Outdated | **Updated to 1.4.0.** |
| **Emoji2** | 1.4.0 | 1.6.0 | ⚠️ Outdated | **Updated to 1.6.0.** |
| **Coroutines** | 1.10.1 | 1.10.2 | ⚠️ Outdated | **Updated to 1.10.2.** |
| **JUnit** | 4.13.2 | 6.0.3 | 🔍 Major version jump / ✅ Up to date (4.x) | **Requires manual review** for JUnit 6. Kept at 4.13.2 for maintenance branch stability. |
| **Mockito Core** | 5.11.0 | 5.23.0 | ⚠️ Outdated | **Updated to 5.23.0.** |
| **Mockito-Kotlin** | 5.2.1 | 6.3.0 | ❌ Unused / 🗑️ Removed | **Removed.** Verified no usage in all test files. |

## Additional Observations

### Preview / Experimental Tools
- **compileSdk = 36**: This is a preview SDK (Android 16). The project targets it intentionally.
- **AGP 8.13.2**: This version was chosen to ensure compatibility with Kotlin 2.3 metadata format for the D8/R8 compiler.

### ProGuard / R8 Rules
- Updated `app/proguard-rules.pro` with keep rules for:
    - **AOSP Dictionary Assets**: Preservation of `AospDictionary` and `AospBigrams` classes.
    - **EmojiCompat**: Preservation of `androidx.emoji2` internal and widget classes to ensure runtime emoji rendering.
    - **LearnedDictionary**: Preservation of the class to ensure `SharedPreferences` serialization remains intact.
    - **FontUtils**: Preservation of class responsible for asset-based font loading.

### Verified Unused Dependencies
- `org.mockito.kotlin:mockito-kotlin`: No direct imports or Kotlin-specific Mockito extensions were found in the `app/src/test` directory.

## Build Verification
All "Safe Updates" have been applied and verified.
- **Build Status**: ✅ Success
- **Verification Command**: `./gradlew assembleDebug`
- **Result**: `BUILD SUCCESSFUL`
