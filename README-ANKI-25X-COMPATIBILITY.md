# Anki 25.x compatibility patch

This branch contains a small compatibility patch for controller handling on
newer Anki 25.x builds, including Anki 25.09.x with Python 3.13, Qt 6.9, and
QtWebEngine.

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

## Installing this branch manually

1. Download or clone this branch.
2. Zip the contents of the `contanki/` directory as an `.ankiaddon` package.
3. In Anki, open `Tools -> Add-ons -> Install from file...`.
4. Select the generated `.ankiaddon` file.
5. Restart Anki before testing a controller.

Example packaging command from the repository root:

```sh
cd contanki
zip -r ../contanki-anki25x-compat.ankiaddon . -x '__pycache__/*'
```

## Verification notes

This patch was verified against Anki 25.09.4 on macOS with an 8BitDo Micro
controller. The add-on loaded cleanly under Anki's bundled Python 3.13, and the
controller path no longer crashed when the controller connected.
