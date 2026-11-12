# Changelog (stable line)

## v72.1 stable (build 136)
The version that ran without problems (v72), with these changes only:

- **Works on Age of the Ring updates (10.01).** v72 showed a red light on AOTR 10.01: its check for a newer
  game.dat compared 10 places in the game's import table, which Windows fills in when the game starts, so every
  update looked changed. The import table is not compared any more (539 places are; on AOTR 10.01 the v72 check
  found no other difference). And an update that really changes some places now turns off only the parts that use
  them - not everything.
- **Fewer antivirus signals.** The starter no longer asks Windows for the debug privilege (it was never needed) and
  asks only for the rights loading the DLL needs. Both files now carry their name, version and maker.
- **The crash log** no longer writes Windows' own memory probes (IsBadStringPtr and friends) as crashes.
- **Small PCs** (4 GB): ground textures out of view are kept only when the PC has the memory for them (off under
  3.5 GB, 48-160 MB by memory), and the menu's read-ahead of earlier battles' files takes at most a third of the
  free memory and stops when under 1.2 GB is free.
- **The learned list** (files of earlier battles) is read newest first and trimmed instead of stopping at 512 KB.
- **The starter**: started from the source tree (PLAY.bat) it always loads the DLL it built; it is never silent when
  nothing opens - it says why. The package carries LICENSE and NOTICE.

Left out on purpose (they are in the development line, still proving themselves): the newer logic shortcuts
(command buttons, object modules, flag masks, id map), the particle spread, the render task read-ahead, the
working-set floor and every diagnostics tool.

## v72 (build 122)
- Age of the Ring 10 recognised; later game.dat updates checked by anchors.

## v71 (build 121)
- One-click install, tray ring (gold / yellow / red), autostart, desktop shortcut, single instance.
- Status light in the game (F7), refresh after every fifth match, English and Uzbek guides.

## v70 (build 120)
- Release build for the AOTR team: English, no tests, nothing on screen but the status light.
