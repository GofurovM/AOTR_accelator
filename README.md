# AOTR Accelerator - stable

A performance DLL for **Age of the Ring** (BFME2: Rise of the Witch-king 2.02). It is loaded into the running game
at start-up. It makes the game's own work cheaper or moves it off the game's main thread, so big battles stutter
less and stay closer to the game's 30 fps cap.

- **Multiplayer-safe.** The simulation is untouched: the logic rate, frame cap and game rules stay as they are.
  The few parts that answer logic questions faster prove themselves against the engine's own code (see
  [NOTES.md](NOTES.md)). Other players do **not** need the DLL.
- **Nothing on disk changes.** game.dat and the game folder are never modified, so the AotR launcher's checks
  still pass. Everything happens in memory, for that session only.
- **Age of the Ring only.** The starter loads the DLL into AOTR's game.dat and no other game.
- **Graceful fallback.** Every part checks the exact game bytes it patches first. On an unknown build, the parts
  tied to fixed addresses stay off.

## The stable version

This repository holds the **stable** line: the version that ran without problems (v72), with fixes only and a few
additions that cannot change the game (listed in [CHANGELOG.md](CHANGELOG.md)). New and experimental work is done
in a separate development line and comes here only after it has proved itself in real games. No diagnostics, test
panels or measurement tools are part of what the players get.

**Download for players:** build with `PACKAGE.bat` (or take the zip from a release) - unzip, run
`AOTR_ACCELERATOR.exe`, answer *Yes* once, then start Age of the Ring as usual. A small light in the top right
corner of the game shows that it runs (green), that one part switched itself off (yellow), or that it is not
running (red). F7 hides it. *Uninstall* is in the menu of the ring near the clock.

## The continuation of the first version

This project continues the first version, [BFME2 Accelerator](https://github.com/OhadBaehr/bfme2-accelerator)
2.0 by Ohad Baehr (MIT, see [LICENSE](LICENSE) and [NOTICE](NOTICE)). Its foundations - the render thread, the
texture workers, the file layer and most engine patches - are his work, and most of the code is still his. This
fork narrows the target to Age of the Ring 10, puts multiplayer safety first, and adds the one-click installer with
a tray icon, the in-game status light, the read-ahead of what a match can need, and the check that lets it keep
working when Age of the Ring updates its game.dat. Not affiliated with Electronic Arts, the Age of the Ring team or
Open-BFME.

> **Problems can still happen** - see [Known problems](#known-problems). If something goes wrong, every part can be
> turned off with a file (see [Switches](#switches-empty-files-beside-the-dll)), and *Uninstall* returns the game
> exactly to how it was.

## Quick start: which file does what

You need Windows and the **Visual Studio Build Tools** with "Desktop development with C++" (free). See
[BUILD.md](BUILD.md).

| Double-click | What it does |
|---|---|
| **`PLAY.bat`** | **Builds, then starts AOTR with the accelerator from `out\` (nothing is installed). Start here.** |
| `build.bat` | Only builds. The results go into `out\`. |
| `PACKAGE.bat` | Builds and makes `AOTR_accelerator.zip` for players: one click installs it. |

What the build makes, in `out\`:

| File | What it is |
|---|---|
| `aotr_start.exe` | **The starter.** It sits in the notification area (a ring), opens the AOTR launcher on request and loads the DLL into the game when it starts. Run this one, or let `PLAY.bat` run it. From the package it installs itself. |
| `bfme2_accel.dll` | **The accelerator itself.** It is never run by hand; the starter loads it into game.dat. It must stay next to `aotr_start.exe`. |
| `bfme2_accel.log` | Written by the DLL while the game runs: what was installed, and any problem. |

In the package made by `PACKAGE.bat`, the starter is named `AOTR_ACCELERATOR.exe`. The package also holds the
players' guides, `README_EN.txt` and `QOLLANMA_UZ.txt` (Uzbek); they are kept in `guide/`.

**From the package (one click):** unpack `AOTR_accelerator.zip` and run `AOTR_ACCELERATOR.exe`, then answer *Yes*
to the administrator question once. It then:
1. copies itself and the DLL to `%ProgramData%\AOTR Accelerator`;
2. makes a desktop shortcut "AOTR Accelerator";
3. registers a Task Scheduler task that starts it hidden at every logon, with the rights the game needs and no
   prompt each time.

From then on the player just opens the AOTR launcher. The starter waits in the notification area and loads the
DLL when AOTR's game.dat starts. A second start, or the shortcut, never runs a second copy: it asks the running
one to open the AOTR launcher. Running a newer package exe updates the installed copy. *Uninstall* is in the
ring's menu.

While the game runs, the starter:
- switches Windows to the *High performance* power plan and *Best performance* slider, and puts both back when
  the game closes (an empty `POWER_OFF` file in the install folder leaves them alone);
- opens the game's files once when the AOTR launcher appears, so the virus scanner has already looked at them
  (`WARM_OFF` turns this off);
- colours its ring from the DLL's `aotr_status.txt`: gold when all is well, yellow when a part switched itself
  off, red when the DLL is not running. Red is the only way to see that an antivirus blocked the DLL. A balloon
  says why, in English and Uzbek.

## What it does (release build)

The only thing on the screen is a small light in the top right corner of the picture. It is green when
everything runs, yellow when a part switched itself off after a self-check, and red when the main part (the
render thread) is not running. F7 hides and shows it, and the choice is remembered. The DLL writes a short
`bfme2_accel.log` beside itself. It records what was
installed at start-up and, after that, only problems: a part that switched itself off, the freeze guard, a crash.

| Area | What changes |
|---|---|
| Drawing | Direct3D 9 calls run in order on their own thread (DXVK only). Texture decoding runs on worker threads. Ground patch textures are built on all cores. Shadow mesh preparation uses fast exact algorithms. The render list sort is replaced with an exact one. The minimap and shroud go through the render queue. |
| Files | The engine's unbuffered reads are served from read-ahead windows, and saves use a write buffer. Files of earlier sessions, and the files this match's factions can use (found from the mod's INI files), are read into the Windows file cache ahead of time. |
| Logic (exact) | The logic step's six engine calls run unchanged and in order; only where the frames fall between them changes. Several hot lookups are memoised or indexed, and each proves itself against the engine's own code. |
| System | The game's thread gets a physical core to itself. Windows power throttling (EcoQoS) is off for the game. |
| Long sessions | After every fifth match, back in the menu, the DLL lets go of its own short-cut tables and parked ground textures. The game's loaded art, the file cache and the learned list stay, so later matches stay fast. After 2.5 hours in one session the starter suggests restarting the game. |

[NOTES.md](NOTES.md) lists each part with its file, its multiplayer argument and the file name that turns it off.

## Switches (empty files beside the DLL)

`VANILLA` keeps the DLL idle for that session. `SLICER_OFF`, `TERRAINKEEP_OFF`, `RADARHOOKS_OFF`, `PRELOAD_OFF`,
`CORES_OFF`, `PARTFAST_OFF`, `AMFIND_OFF`, `EQUIV_OFF`, `FILTER_OFF`, `AUDIOLIMIT_OFF`, `PSFIND_OFF`, `PSREMOVE_OFF`,
`PEVICT_OFF`, `W3DLOCK_OFF` and `PXBATCH_OFF` each turn off one part. Use them to bisect a problem.

## Measured (the author's laptop, AOTR, DXVK)

- **Large 4v4 battles:** 28–30 fps average (median 29–30). The first builds measured about 23 fps average in
  the same kind of battle.
- **Multiplayer:** 2.5 hours in network games (5 matches) ran at 29.2 fps and 4.9 logic steps a second. The
  self-checks ran 208 million times with 0 differences.
- **The heaviest battles** still dip to about 18–22 fps. That is the cost of the logic and the drawing together
  on the game's single main thread.

## Known problems

- **Dark stretched shapes at full graphics.** In big battles, dark columns stretched upward from units and flat dark
  "walls" near trees and buildings have been seen for a moment. The cause is still being looked for in the
  development line. If you see it, note the time and send `bfme2_accel.log`.
- **Antivirus warnings.** Some antivirus programs give a generic or heuristic "trojan" warning, because the starter
  loads the DLL into the game with the standard Windows calls that trainers and overlays also use. The source is
  here for anyone to read. v72.1 removed the strongest signals (see CHANGELOG).
- **DXVK is needed for the render thread.** With Windows' own Direct3D 9 the render thread is not used and the
  light is red. Turn DXVK on in the AOTR launcher's graphics settings.
- **Age of the Ring updates.** The DLL checks every place in game.dat it uses. An update that changes some of them
  turns off only the parts that use those places; a big update that changes the engine's core turns the light red
  until a new version knows it.
- **The heaviest battles still dip to about 18-22 fps** - the engine's own logic and drawing on its single main
  thread.
- **Laptops slow themselves down** on battery, on a loose charger or when hot; the game stutters then whatever the
  DLL does. Keep the charger plugged in firmly.
- **Rare broken geometry after a long session.** Seen once, in the fifth match of a 2.5-hour session, on one PC.
  The DLL clears its own tables after every fifth match, and the starter suggests a restart after 2.5 hours. If you
  see it, send `bfme2_accel.log` and DXVK's `game_d3d9.log` (in the `rotwk` folder), then try with `SLICER_OFF`,
  then `RADARHOOKS_OFF`, then `TERRAINKEEP_OFF`.
- **32-bit address space.** The game is a 32-bit process (4 GB) and Age of the Ring alone uses 2-2.5 GB of it, so
  nothing is loaded into the game ahead of time - read-ahead goes only to the Windows file cache.
