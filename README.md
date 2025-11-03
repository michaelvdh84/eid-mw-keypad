# eid-mw — Keypad bug fix (Issue #236)

This README summarizes the changes introduced by Pull Request #237 which fixes the keypad / virtual keyboard issue described in Issue #236. It includes a short explanation of the bug, the root cause, the technical changes made, how to test the fix locally, the registry keys that enable the keypad, and visual before/after screenshots included in the PR.

PR: https://github.com/Fedict/eid-mw/pull/237  
Issue: https://github.com/Fedict/eid-mw/issues/236  
Author: michaelvdh84  
Branch (head): fix/keypad-askpin-bitmap-bug (from michaelvdh84/eid-mw-keypad)

---

## Summary

We provide citizen kiosks that allow users to print documents. On some Windows kiosks the virtual keyboard (TabTip.exe) did not reliably appear when selecting the PIN field. Additionally, when enabling the virtual keypad via a registry DWORD, the keypad showed no bitmaps and the numeric buttons were effectively invisible.

This PR replaces the bitmap-based keypad buttons with standard labeled buttons and stabilizes the input handling, which fixes the visibility and input reliability issues.

Registry keys added to enable the virtual keypad:
- For the current user:
  - HKEY_CURRENT_USER\Software\BEID\configuretool\use_virtual_keypad = 1 (DWORD)
- For the whole machine:
  - HKEY_LOCAL_MACHINE\Software\BEID\configuretool\use_virtual_keypad = 1 (DWORD)

Either key can be created depending on whether you want the setting for the current user or for the entire machine.

---

## Root cause

- The keypad implementation relied on bitmap images for buttons; on some Windows environments the bitmaps were not available or not loaded correctly, causing the buttons to appear invisible.
- Input/focus handling allowed race conditions where very fast key presses could be dropped or the PIN input could lose focus.
- Immediate validation/reset of the input during rapid input could clear characters before queued UI events were processed.

---

## What changed (technical)

- Replaced bitmap-based keypad buttons with standard button controls that use simple text labels (0–9, backspace, confirm). This avoids invisible buttons when bitmaps fail to load.
- Centralized keypad input handlers to avoid duplicate handlers and race conditions.
- Adjusted focus management so the PIN input retains focus during continuous keypad use.
- Introduced a short debounce/coalescing step to group very fast events and avoid dropped keystrokes.
- Changed validation to be deterministic (trigger only when expected length is reached or on explicit confirmation).
- Added/updated tests to cover fast keypad input and focus behavior.
- Added debug logging to make future diagnosis easier.

---

## Files modified (high level)

- UI component responsible for the PIN dialog / keypad (replaced bitmap-button rendering with labeled standard buttons)
- Input / validation module for PIN handling
- Tests covering keypad input / focus scenarios

(See PR #237 for the exact file list and diffs.)

---

## Visuals — before / after

Before — keypad buttons invisible (bitmap-based)  
![Before (bitmap buttons invisible)](https://github.com/user-attachments/assets/1613cf9d-8130-479a-9fd8-a443ded434b4)

After — keypad using standard labeled buttons  
![After (standard labeled buttons)](https://github.com/user-attachments/assets/b091f375-554a-462c-989a-abf159ad3e4c)

---

## How to test locally

1. Fetch the branch containing the fix and check it out:
   - git fetch origin pull/237/head:fix/keypad-askpin-bitmap-bug
   - git checkout fix/keypad-askpin-bitmap-bug
   - Or add the fork as a remote and checkout michaelvdh84/fix/keypad-askpin-bitmap-bug

2. Build and run the application using your normal local steps:
   - Example: ./gradlew build && ./gradlew run
   - Or the project-specific commands for your environment.

3. Manual test scenarios:
   - Open the PIN entry dialog and select the PIN field; verify the Windows virtual keyboard (TabTip.exe) appears as expected on kiosk Windows installations.
   - Use the on-screen keypad and press keys rapidly (0–9, backspace) to ensure all presses are registered.
   - Try switching focus (mouse clicks outside/in) while typing on the keypad to ensure the input does not lose keypresses or reset unexpectedly.
   - Confirm validation happens only when expected (full PIN length or explicit confirm).

4. Run automated tests:
   - Run the project test suite (e.g., ./gradlew test, npm test, pytest, etc., depending on project stack).
   - Verify the new/updated tests for keypad input pass.

---

## Impact and compatibility

- Backwards-compatible: no public API changes.
- Improves reliability on kiosks and Windows devices that previously showed invisible keypad buttons.
- Reduces likelihood of lost keystrokes during rapid keypad entry.

---

## Notes and follow-up

If you still observe keypad issues on specific kiosk hardware or Windows builds:
- Open a new issue with details: OS version, kiosk hardware, driver info, exact reproduction steps, and screenshots.
- Enable debug logs included in this PR to capture event traces that will help diagnose remaining edge cases.

---
