# Brew (GD 2.207 iOS) - hook.dylib

Adds a draggable GD thumbs-up button that opens the **Brew Menu** (GD sprites + GD font) with a **Noclip** checkbox.

## Build (Windows-friendly)
1. Push this repo to GitHub. The `build-hook` Action (`.github/workflows/build.yml`) builds `hook.dylib` on a macOS runner;
   download it from the run's Artifacts. (On a Mac just run `make`.)
2. Put your own decrypted `Geometry-Dash.ipa` next to this folder (it is git-ignored, don't commit it) and run:

       python tools/build_ipa.py Geometry-Dash.ipa hook.dylib out/Brew.ipa

3. Sign + install `out/Brew.ipa` with Sideloadly / AltStore.
4. Device log should show `[Brew] loaded`, then `[Brew] UI installed`.

## Required before noclip works
Open `src/hook.mm` and set `BREW_ADDR_DESTROY_PLAYER` to the Ghidra/IDA address of
`PlayLayer::destroyPlayer(PlayerObject*, GameObject*)` for 2.207 iOS arm64 (as shown in Ghidra, e.g. `0x1001xxxxx`).
Also add Dobby and uncomment `#define BREW_USE_DOBBY` (runtime hooking needs JIT on non-jailbroken devices).
Until the address is set the UI works but the checkbox has no effect (a line is logged).

### Notes for finding the address (checked against the 2.207 binary in this repo's IPA)
* The binary is decrypted but stripped; RTTI names survive. `PlayLayer` typeinfo name string is at `0x10062e145`
  (xref it in Ghidra -> typeinfo -> vtable).
* The `explode_11.ogg` / `explodeEffect.plist` / `PlayerExplosion_%02d` strings do NOT lead to `destroyPlayer`
  here: they resolve to MenuLayer (`0x100122650`), a sound-preload routine (`0x10013b128`), MenuGameLayer
  (`0x1003943f4`) and CurrencyRewardLayer (`0x10031eabc`). Don't use them as anchors.
* Better approach: from the PlayLayer vtable, find the method that takes (PlayerObject*, GameObject*), checks the
  player's dead flag, and calls the death/reset routines.

## Layout
    .github/workflows/build.yml   CI build of hook.dylib
    Makefile                      iOS arm64 build
    src/hook.mm                   the mod
    tools/inject.py               adds LC_LOAD_DYLIB to the GD executable
    tools/build_ipa.py            repackages the IPA with hook.dylib
