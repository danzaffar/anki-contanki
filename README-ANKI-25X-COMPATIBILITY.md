# Anki 25.09.4 compatibility patch

This branch contains an unofficial, vibe-coded compatibility patch for
controller handling on newer Anki 25.x builds.

It was tested on:

```text
Anki 25.09.4 (d52ca669)
Python 3.13.5
Qt 6.9.1
Chromium 122
macOS
8BitDo Micro gamepad
```

The patch worked in that setup: Anki no longer crashed when the 8BitDo
controller connected, and Contanki could be used again.

This is intentionally a small patch, not a full rewrite of Contanki's
controller backend.

## What it fixes

Some users can see Anki crash or fail to initialize Contanki after connecting a
controller. One observed failure mode is a JavaScript error like:

```text
Uncaught ReferenceError: get_controller_info is not defined
```

This can happen when Contanki asks the hidden controller webview for controller
debug info before the injected script is ready. Newer Anki/QtWebEngine builds
also appear less forgiving when `navigator.getGamepads()` is unavailable,
returns sparse entries, or reports button/axis arrays that do not match the
profile assumptions.

## Changes in this branch

- Guards all JavaScript `navigator.getGamepads()` access behind a safe helper.
- Uses the `gamepadconnected` event's `event.gamepad` object when available.
- Delays and guards Python calls to `get_controller_info()`.
- Adds safe button and axis accessors in Python so controllers with mismatched
  reported button/axis counts do not trigger index errors.
- Keeps existing controller profiles and bindings unchanged.

## No-code install option

If this pull request has a downloadable `.ankiaddon` file attached in the
conversation or release where you found it, use that. That is the easiest path:

1. Download the `.ankiaddon` file.
2. Open Anki.
3. Go to `Tools -> Add-ons`.
4. Click `Install from file...`.
5. Select the downloaded `.ankiaddon` file.
6. Restart Anki completely.
7. Connect your controller only after Anki has reopened.

If Anki still shows two Contanki entries, disable the older/original one and
leave the patched one enabled.

## How to package this branch yourself

These steps are written for non-programmers. You only need to make a zip file
with the right contents and rename it to `.ankiaddon`.

### Mac

1. Open this pull request on GitHub.
2. Click the green `Code` button.
3. Click `Download ZIP`.
4. Open the downloaded zip file. It will create a folder named something like
   `anki-contanki-fix-anki-25x-gamepad-startup-crash`.
5. Open that folder.
6. Open the `contanki` folder inside it.
7. Select everything inside the `contanki` folder, not the folder itself.
8. Right-click the selected files and choose `Compress`.
9. Rename the new zip file to:

   ```text
   contanki-anki25x-compat.ankiaddon
   ```

   If macOS warns about changing the extension, choose to use `.ankiaddon`.
10. Open Anki.
11. Go to `Tools -> Add-ons`.
12. Disable or remove the old Contanki add-on if it is already installed.
13. Click `Install from file...`.
14. Pick `contanki-anki25x-compat.ankiaddon`.
15. Restart Anki completely.
16. After Anki restarts, connect your controller and test review controls.

### Windows

1. Open this pull request on GitHub.
2. Click the green `Code` button.
3. Click `Download ZIP`.
4. Extract the downloaded zip file.
5. Open the extracted folder.
6. Open the `contanki` folder inside it.
7. Select everything inside the `contanki` folder, not the folder itself.
8. Right-click the selected files and choose `Send to -> Compressed (zipped)
   folder`.
9. Rename the new zip file to:

   ```text
   contanki-anki25x-compat.ankiaddon
   ```

   If Windows hides file extensions, enable `View -> File name extensions` in
   File Explorer first.
10. Open Anki.
11. Go to `Tools -> Add-ons`.
12. Disable or remove the old Contanki add-on if it is already installed.
13. Click `Install from file...`.
14. Pick `contanki-anki25x-compat.ankiaddon`.
15. Restart Anki completely.
16. After Anki restarts, connect your controller and test review controls.

### Command-line packaging option

If you are comfortable using Terminal or PowerShell, run this from the
repository root:

```sh
cd contanki
zip -r ../contanki-anki25x-compat.ankiaddon . -x '__pycache__/*'
```

Then install the generated `.ankiaddon` file from Anki's add-on manager.

## What to do if it still crashes

1. Restart Anki with the controller disconnected.
2. Confirm only one Contanki add-on is enabled.
3. Connect the controller after Anki is fully open.
4. If it still crashes, test whether Anki opens normally when Contanki is
   disabled.
5. If reporting the issue, include:
   - Anki version
   - operating system
   - controller model
   - wired or Bluetooth connection
   - whether Anki crashes immediately on connect or only after pressing a
     button

## Verification notes

This patch was verified against Anki 25.09.4 (d52ca669) on macOS with an 8BitDo
Micro controller. The add-on loaded cleanly under Anki's bundled Python 3.13,
and the controller path no longer crashed when the controller connected.
