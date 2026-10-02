# C-Dogs for PicoDeck

C-Dogs SDL is a classic overhead run-and-gun game — squad-based shooting, destructible
scenery and dozens of campaigns — originally written by Ronny Wester and maintained today
by Cong Xu ([cxong/cdogs-sdl](https://github.com/cxong/cdogs-sdl), GPL-2.0). This repo is
that game ported to [PicoDeck](https://github.com/PicoDeck/picodeck) on the ClockworkPi PicoCalc:
the upstream engine is vendored under `src/` and builds against a small SDL shim
(`picodeck_sdl*.{h,c}`, `sdl_shim/`) that maps SDL video, input and mixer calls onto the PicoDeck
native API. Licence is GPL-2.0 (see `COPYING`); the port keeps that licence.

The app needs the `root-filesystem` and `audio` requirements and reads its game data from
`/apps/cdogs/data/` at runtime, so the whole `data/` tree ships inside the release ZIP.

## Install

C-Dogs is on the **PicoDeck App Store** — open the Store app on your PicoCalc and install it
from there. Nothing else to do.

## Controls

On firmware with the PicoDeck gamepad (API version 9) the game reads it, so
Settings -> Controls rebinding applies. With the default bindings:

| Gamepad | Default key | In C-Dogs |
|---|---|---|
| D-pad | arrows | move / menu cursor |
| A | F4 | fire (menus: confirm) |
| B | F5 | switch weapon |
| X | Delete | grenade |
| Y | Backspace | map (menus: Backspace = back) |
| Start | F1 | Esc: pause menu (menus: back / quit) |

- The pad presses player 1's *current* C-Dogs keys, read every frame, so
  Options -> Redefine keys is followed. Start is always Esc.
- Options -> Redefine keys captures keyboard keys only, not pad buttons: to
  change what a pad button is, use Settings -> Controls in the system menu.
- A key bound to a pad button is that button only: its typed letter is not
  also sent to the game when it is one of player 1's keys (WASD on the D-pad
  does not throw grenades). Backspace stays a typed key, so it still backs out
  of menus.
- On-screen hints ("Press F4 to ...") name the key bound to the pad button.
- Enter, Esc and the letter keys the game always used (X fire, Z switch
  weapon, S grenade, A map) still work when not bound to a pad button. Older
  firmware reads only those keys and the arrows.

## Build

Needs `arm-none-eabi-gcc` (tested with 15.2) and a newlib for ARM:

```sh
make
```

This produces `main.elf` (stripped) plus `main.elf.debug`. The build is large — expect a
few minutes at `-Os`. The PicoDeck native SDK headers and linker script are vendored in
`sdk/native/`.

`data/` is tracked in this repo and is what ships to the device. If you update the vendored
`src/` tree, regenerate it with:

```sh
./prepare_data.sh
```

## Release

1. Bump `version` in `app.json`.
2. Commit the change.
3. `git tag v<version> && git push && git push --tags`

GitHub Actions builds `main.elf`, packages `app.json`, `main.elf` and `data/` into a single
ZIP and publishes it as the Release for that tag. The PicoDeck App Store re-indexes within
about 30 minutes, after which the new version shows up on-device.
