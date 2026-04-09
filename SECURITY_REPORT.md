# Security Audit Report - T9 Keyboard (THREAT-001)

This report details the security and privacy audit of the T9 Keyboard IME codebase.

## Finding 1: Sensitive Data Leakage in Learned Dictionary (THREAT-001)
*   **Rating**: High
*   **Description**: The keyboard was previously learning and storing words typed into sensitive fields, such as password inputs, numeric-only fields (often used for PINs), and phone number fields. This data was persisted in the `learned_words` SharedPreferences file, potentially exposing sensitive user credentials.
*   **Recommended Fix**: Implement a suppression mechanism that checks the `EditorInfo.inputType` of the active field. Suppress both word learning and suggestion generation if the field is a password, numeric, or phone type.
*   **Status**: ✅ **Fixed**. Implemented `T9Utils.isInputTypeSensitive` and integrated checks in `T9InputMethodService` and `SuggestionEngine`.

## Finding 2: Excessive Data Retention for Learned Words (THREAT-002)
*   **Rating**: Medium
*   **Description**: Words added to the `LearnedDictionary` with a frequency greater than 1 were stored indefinitely. While a recency decay multiplier was applied for ranking, the raw words remained on disk. This poses a long-term privacy risk if a user types sensitive unique information multiple times (e.g., specific addresses or account numbers).
*   **Recommended Fix**: Implement a hard expiration limit for all entries in the learned dictionary.
*   **Status**: ✅ **Fixed**. Updated `LearnedDictionary.cleanup` to enforce a hard 180-day (6-month) expiration for all words, regardless of frequency.

## Finding 3: Contacts Permission Usage Audit (THREAT-003)
*   **Rating**: Low
*   **Description**: `ContactsDictionary` requires the `READ_CONTACTS` permission. While it correctly checks for permission at runtime, the tokenization logic could potentially ingest sensitive contact info if not carefully scoped.
*   **Analysis**: The current implementation only reads `DISPLAY_NAME_PRIMARY` and splits it into individual word tokens. It limits tokens to 2000 and filters out non-alpha words. This is a safe and localized usage of contact data for improving multi-tap/XT9 suggestions.
*   **Recommended Fix**: Ensure the user can easily toggle this feature off in settings.
*   **Status**: ✅ **Verified**. The feature is opt-in and can be disabled in the Settings UI, which clears the memory buffer.

## Finding 4: External Intent Exposure (THREAT-004)
*   **Rating**: Low
*   **Description**: Potential for internal data exposure via implicit intents.
*   **Analysis**: A review of `T9InputMethodService` and `MainActivity` shows that internal transitions use explicit intents (targeting `SettingsActivity.class`). The only semi-implicit intent is for the system clipboard manager, which is standard and doesn't expose internal private data.
*   **Status**: ✅ **Safe**.

## Finding 5: Telemetry File Permissions (CrashLogger) (THREAT-005)
*   **Rating**: Low
*   **Description**: The `CrashLogger` writes error logs to `t9_errors.log` in `filesDir`. There was a concern that these logs might be world-readable.
*   **Analysis**: `CrashLogger` uses the standard `File` API within the app's internal storage (`context.filesDir`). On Android, this directory is private to the application (UID-based isolation), which is equivalent to `MODE_PRIVATE`. No `MODE_WORLD_READABLE` or `MODE_WORLD_WRITEABLE` flags are used.
*   **Status**: ✅ **Verified**. File permissions are secure.

## Finding 6: Stale Learned Word Persistence (THREAT-006)
*   **Rating**: Medium
*   **Description**: The decay logic in `LearnedDictionary` was not enforcing expiration at read-time, and the forgetting mechanism only targeted low-frequency words. This allowed stale learned words (even highly frequent ones) to persist and be suggested indefinitely.
*   **Recommended Fix**: Enforce 180-day hard expiration at both read-time (filtering in suggestions) and write-time (cleanup).
*   **Status**: ✅ **Fixed**. Updated `LearnedDictionary.getSuggestions`, `getNextWordSuggestions`, and `cleanup` to enforce the 180-day limit.

---
*Audit performed by Jules on April 3, 2026.*
