# REFACTOR_LOG.md

## Summary of Refactoring for Modularity

The codebase was audited for files exceeding 300 lines or handling multiple responsibilities. The following major refactors were performed to improve maintainability and follow the Single Responsibility Principle.

### 1. T9InputMethodService.kt
**Original Size:** ~1309 lines
**Status:** Split into focused delegates.
**Extracted Components:**
- **EditorState.kt**: Owns the active typing state (`composingText`, `xt9DigitSequence`, `lastCommittedWord`, etc.).
- **InputConnectionManager.kt**: Wraps all `InputConnection` interactions (`commitText`, `sendDownUpKeyEvents`, `selectWord`, etc.).
- **SuggestionEngine.kt**: Manages asynchronous dictionary lookups, candidate ranking, and coroutine jobs for suggestions.
- **ViewOrchestrator.kt**: Handles view switching, dynamic height calculation, and window insets.
- **AutocorrectController.kt**: Reactive component for evaluating autocorrect triggers and tracking `AutocorrectRecord`.
- **ActionListeners.kt**: Defined `MainKeyActionListener`, `EditActionListener`, and `EmojiActionListener` interfaces to decouple views from the service.

**Reasoning:** The service was a "God Object" handling UI, business logic, and state simultaneously. Delegation clarifies the flow of data and events.

### 2. KeyboardView.kt
**Original Size:** ~436 lines
**Status:** Split into delegates.
**Extracted Components:**
- **KeyTouchHandler.kt**: Manages touch events, multi-tap timing logic, and long-press repetitions.
- **KeyLabelRenderer.kt**: Handles dynamic label updates, font application, and accent color/theming.

**Reasoning:** UI logic was tightly coupled with input timing logic. Splitting them allows for easier testing of input behavior and cleaner UI updates.

### 3. TextEditingView.kt
**Original Size:** ~373 lines
**Status:** Split into delegates.
**Extracted Components:**
- **EditKeyHandler.kt**: Handles repeating keys, long presses, and action routing for the text editing grid.

**Reasoning:** The view was handling complex touch logic for a large grid of specialized keys.

### 4. EmojiPickerView.kt
**Original Size:** ~363 lines
**Status:** Split into delegates.
**Extracted Components:**
- **EmojiAdapter.kt**: Extracted the `RecyclerView.Adapter` logic into a separate file.

**Reasoning:** The file was cluttered with adapter and view holder boilerplate, making the main view logic hard to follow.

### 5. SettingsActivity.kt
**Original Size:** ~442 lines
**Status:** Partially refactored.
**Extracted Components:**
- **SettingsDialogHelper.kt**: Extracted dialog creation logic (sliders, theme picker, info dialog).

**Reasoning:** Reduced boilerplate in the main Activity and separated dialog UI logic.

## Dependency Management
- Introduced a one-way dependency flow: **Views → Interfaces ← Service**.
- Used constructor injection for delegates.
- Zero circular dependencies introduced.
- All import statements updated.
- Fixed a `UninitializedPropertyAccessException` in `T9InputMethodService` by moving delegate initialization to `onCreate()` and adding `isViewReady` guards for lifecycle safety.
- Introduced `T9Utils.kt` to consolidate shared character-to-digit mapping logic.
- Cleaned up un-idiomatic Kotlin code (removed excessive semicolons and one-liners) for improved readability.
- Project compiles and original unit tests (excluding pre-existing failures) pass.
