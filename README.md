# ZCSDK Runtime

The ZCSDK Runtime is the runtime half of the **Zero Company Mod SDK** by Envian Mods, for **STAR WARS: Zero Company** (Bit Reactor, Unreal Engine 5.6).

It is two [UE4SS](https://github.com/UE4SS-RE/RE-UE4SS) mods. Together they let content mods built with the SDK (new weapons, armor, classes, abilities) appear in the game's own menus, shops, reward tables and pickers, alongside the game's own content.

| Folder | What it is |
|---|---|
| `ZCSDKBridge/` | A native (C++) UE4SS mod. It finds the game functions that UE4SS can't reach, using the game's shipped PDB, and calls them on the game thread: asset-registry injection, primary-asset rescan, per-type cache refresh and `LoadPackage`. |
| `ZCSDKLoader/` | A Lua UE4SS mod. It finds every SDK mod's `<Mod>.zcsdk.lua` manifest, drives the bridge for each one, keeps recruit pins, and applies the manifest's once-per-save item grants. |

An SDK-built mod tells you on its own page whether it needs the runtime. Plain override mods do not.

**Tested on:** Steam, game build **201250** (the 2026-11-10 update), UE4SS 3.0.1 as packaged on Nexus, v1.1.0-rc3 ([UE4SS for Star Wars Zero Company](https://www.nexusmods.com/starwarszerocompany/mods/9)).

---

## Install

### 1. UE4SS (required, once, and FIRST)

Install **"UE4SS for Star Wars Zero Company"** from Nexus: <https://www.nexusmods.com/starwarszerocompany/mods/9>. This is the UE4SS build Mod Command installs and the one the runtime is tested on (it reports `UE4SS - v3.0.1 Beta` in `UE4SS.log`). Mod Command users can skip this step: Mod Command installs it for you.

Extract it into:

```
...\Star Wars Zero Company\SWZeroCompany\Binaries\Win64\
```

That puts `dwmapi.dll` and a `ue4ss\` folder next to `SWZeroCompany.exe`.

**Order: UE4SS (Nexus mod 9) first, then the runtime (step 2).** The runtime adds three signature files mod 9 does not have (`ConsoleManager.lua`, `FName_ToString.lua`, `GUObjectHashTables.lua`) and never replaces mod 9's own four. Either one can be updated at any time afterwards, and reinstalling mod 9 no longer means reinstalling the runtime.

**Coming from runtime 0.10, 0.11 or 0.12 installed by hand?** Those versions overwrote three of mod 9's files (`StaticConstructObject.lua`, `FName_Constructor.lua`, `ProcessLocalScriptFunction.lua`), which is why ZCUnlocked showed its "old UE4SS" popup. After extracting this runtime, reinstall mod 9 and let it replace all files. Mod Command does this for you when it (re)installs the runtime.

*Alternative:* the stock **UE4SS v3.0.1** from <https://github.com/UE4SS-RE/RE-UE4SS/releases/tag/v3.0.1> (`UE4SS_v3.0.1.zip`, not the `zDEV` build) is the same UE4SS version, but the runtime is not tested on it. Other UE4SS versions are not supported.

### 2a. With Zero Company Mod Command (recommended)

Go to Settings → **ZCSDK Runtime** → Install / Update. Mod Command installs UE4SS if needed, puts every file in place (including the signatures) and keeps the runtime switched on. It also offers the install on its own when you add an SDK mod that needs it.

### 2b. By hand: one extract

Download **`ZCSDKRuntime_v0.15.3_manual.zip`**. It is the one with `_manual` in its name. The other zip on the release is for Mod Command.

Extract it into the **same `Win64` folder** as step 1:

```
...\Star Wars Zero Company\SWZeroCompany\Binaries\Win64\
```

The zip holds one top-level folder, `ue4ss\`. It merges into the `ue4ss\` folder UE4SS already made, and you end up with this tree:

```
SWZeroCompany\Binaries\Win64\
├─ SWZeroCompany.exe
├─ dwmapi.dll                         (UE4SS)
└─ ue4ss\
   ├─ UE4SS.dll, UE4SS-settings.ini   (UE4SS)
   ├─ UE4SS_Signatures\               (7 .lua files: 4 from UE4SS mod 9 + 3 from this runtime)
   │   ├─ ConsoleManager.lua              (this runtime)
   │   ├─ FName_Constructor.lua           (UE4SS mod 9)
   │   ├─ FName_ToString.lua              (this runtime)
   │   ├─ GUObjectArray.lua               (UE4SS mod 9)
   │   ├─ GUObjectHashTables.lua          (this runtime)
   │   ├─ ProcessLocalScriptFunction.lua  (UE4SS mod 9)
   │   └─ StaticConstructObject.lua       (UE4SS mod 9)
   └─ Mods\
      ├─ ZCSDKBridge\
      │   ├─ dlls\main.dll
      │   ├─ symbols\symtable_….txt   (bridge 0.6.0+: the game's functions, one file per game build)
      │   ├─ modinfo.json
      │   └─ enabled.txt
      └─ ZCSDKLoader\
          ├─ Scripts\main.lua
          ├─ modinfo.json
          └─ enabled.txt
```

If Windows asks whether to replace files, answer **Yes**. The only files replaced are the runtime's own: its two mod folders and its three signature files. None of mod 9's files is in this zip.

**About `enabled.txt`.** UE4SS starts a mod folder only when it holds an `enabled.txt`, or when `ue4ss\Mods\mods.txt` has a `Name : 1` line for it. Both runtime folders ship an empty `enabled.txt`, so nothing else is needed. To switch one part off by hand, rename its `enabled.txt` (for example to `enabled.txt.off`). If your `mods.txt` has `ZCSDKBridge : 0` or `ZCSDKLoader : 0`, change it to `1` or delete that line.

**About `UE4SS_Signatures\`.** UE4SS's own pattern scans can't find some of this game's functions, and every game update moves them. The folder holds seven files: four come with UE4SS (mod 9) and three with this runtime, generated from the game's own PDB for build 201250. They belong **next to** `Mods\`, not inside it. If any are missing or out of date, UE4SS stops with `AOB scans could not be completed`, and then no Lua mod runs at all.

**Game Pass / Microsoft Store.** This build is not tested there. The runtime is built and tested on the Steam edition only.

### Linux / Steam Deck (Proton)

Install exactly as above with the `_manual` zip (Mod Command is a Windows program). Two Proton-specific points:

1. **UE4SS needs a launch option under Proton.** In Steam: Star Wars Zero Company → Properties → Launch Options:
   `WINEDLLOVERRIDES="dwmapi=n,b" %command%`. Without it Proton loads its own `dwmapi.dll` instead of UE4SS's, and no UE4SS mod
   (this runtime included) starts: there is then no `ue4ss\UE4SS.log` at all.
2. **ZCSDKBridge 0.6.0 or newer.** Older bridges (0.5.x, up to runtime 0.12.2) looked the game's functions up with Windows'
   `dbghelp`, which does not work under Proton (`UNRESOLVED … err=126` in `ue4ss\ZCSDKBridge.log`): SDK mods loaded, but none of
   their new items was listed. 0.6.0 reads them from a table that ships in `ZCSDKBridge\symbols\` (one file per game build).

`ue4ss\ZCSDKBridge.log` then says, near the top:

```
symbols: mode=auto exe stamp=… size=… pdb=… (Wine …)
symbol table: symtable_….txt matches this exe (63 symbol(s))
```

and every `resolved … (table)` line after it. `symbol table: NONE matches this exe` means the game updated after this runtime
was released: the bridge then reads the game's `SWZeroCompany.pdb` itself (`pdb reader: …` lines; a few seconds once, then
cached), and a runtime release for the new build brings the table back.

### Updating from an older runtime

After a game update, every older runtime's signature files stop matching and no Lua mod starts. Update to the newest runtime (Mod Command: Settings → ZCSDK Runtime → Update; by hand: extract `ZCSDKRuntime_v0.15.3_manual.zip` into `Win64\` and replace everything).

**Coming from 0.10, 0.11 or 0.12** (extra steps):


- **Mod Command:** Settings → ZCSDK Runtime → **Reinstall** (it installs the newest release; the button may not say Update, because the loader and bridge versions did not change). Mod Command removes the three signature files the old runtime placed and puts back the mod 9 copies it kept. If ZCUnlocked still reports an old UE4SS afterwards, reinstall UE4SS mod 9 as well.
- **By hand:** extract `ZCSDKRuntime_v0.15.3_manual.zip` into `Win64\` as above and replace everything. **Then reinstall UE4SS mod 9** (replace all files): 0.10–0.12 overwrote three of its files, and this runtime no longer carries them.
  - If you installed 0.11 by hand, check that it left no nested folder behind. `ue4ss\Mods\ZCSDKRuntime_v0.11\…` or `ue4ss\Mods\Mods\…` are leftovers; delete them.
  - Also delete any `ue4ss\Mods\UE4SS_Signatures\` folder. That was the wrong place in 0.10/0.11 by-hand installs.
  - Your content mods and their settings files are not touched.

---

## Verify

**The files.** All of these must exist, with paths relative to `SWZeroCompany\Binaries\Win64\`:

- `dwmapi.dll`
- `ue4ss\UE4SS.dll`
- `ue4ss\Mods\ZCSDKBridge\dlls\main.dll`
- `ue4ss\Mods\ZCSDKBridge\enabled.txt`
- `ue4ss\Mods\ZCSDKBridge\symbols\symtable_….txt` (bridge 0.6.0 or newer; required under Proton)
- `ue4ss\Mods\ZCSDKLoader\Scripts\main.lua`
- `ue4ss\Mods\ZCSDKLoader\enabled.txt`
- `ue4ss\UE4SS_Signatures\` with all seven files: `ConsoleManager.lua`, `FName_ToString.lua`, `GUObjectHashTables.lua` (this runtime) and `FName_Constructor.lua`, `GUObjectArray.lua`, `ProcessLocalScriptFunction.lua`, `StaticConstructObject.lua` (UE4SS mod 9)

**The log.** Start the game, reach the main menu, then open `ue4ss\UE4SS.log`. `ue4ss\ZCSDKLoader.log` carries the same lines. Look for these, in this order:

```
UE4SS - v3.0.1 Beta ...
Starting C++ mod 'ZCSDKBridge'           (or: Mod 'ZCSDKBridge' has enabled.txt, starting mod.)
Starting Lua mod 'ZCSDKLoader'
[ZCSDKLoader] ZCSDKLoader v1.8.57 loaded (win64=...); waiting for the main-menu world
[ZCSDKLoader] pump: running on the game thread (1.8.57; ...)
[ZCSDKLoader] === ZCSDKLoader start ===
[ZCSDKLoader] manifest: .../<YourMod>.zcsdk.lua  (<YourMod> v<x.y.z> ...)      (one line per SDK mod installed)
[ZCSDKLoader] N manifest(s), M grant(s)
[ZCSDKLoader] === ZCSDKLoader ready; ... ===
```

`ZCSDKLoader v1.8.x loaded` followed by `ZCSDKLoader ready` means the runtime works.

**With ZCUnlocked installed**, `ue4ss\Mods\ZCUnlocked\dlls\ZCUnlocked.log` should say `ue4ss: detected=1.1.0-rc3 required=1.1.0-rc3 verdict=ok`. `verdict=old` (for example `detected=1.0-or-partial(3-of-4-scripts)`, with ZCUnlocked's "old UE4SS" popup) means mod 9's files were overwritten by an older runtime: reinstall mod 9.

**The Armory row fix (ROWFIX, 1.8.29).** In the hub, `ZCSDKLoader.log` says ONE of these once:

```
ROWFIX skipped: ZCUnlocked is installed and enabled (ue4ss/Mods/ZCUnlocked via mods.txt) and heals the Armoury rows itself (...)
ROWFIX: ZCUnlocked is not installed          (then, on the Armoury weapon page: ROWFIX armed / backed off / disarmed)
```

Neither ever repeats every few seconds, and there should be no `pump: task rowfix took … ms` line with ZCUnlocked installed. `0 manifest(s)` means the runtime works but no SDK mod was found (see Troubleshooting). The bridge writes its own log to `ue4ss\ZCSDKBridge.log`.

---

## Troubleshooting

| What you see | Likely cause | Fix |
|---|---|---|
| New weapons / armor / classes are missing, and the log has **no** `ZCSDKLoader` line | The runtime is in the wrong folder or nested one level too deep, e.g. `Win64\ue4ss\Mods\ZCSDKRuntime_v0.12.1\ue4ss\Mods\…`, `Win64\Mods\…`, or `SWZeroCompany\Mods\ZCSDKLoader`. | Extract `…_manual.zip` into `Binaries\Win64\` itself, so `Win64\ue4ss\Mods\ZCSDKLoader\Scripts\main.lua` exists. |
| `ZCSDKLoader` is listed but never `loaded` | No `enabled.txt`, or `mods.txt` says `ZCSDKLoader : 0` | Add the empty `enabled.txt`, or set the line to `: 1`. |
| `ZCSDKLoader ready` but `0 manifest(s)` | The content mod is not installed where its page says. SDK mods live in `SWZeroCompany\Mods\<Mod>\` or in `Content\Paks\~mods\` with their `.zcsdk.lua`. | Reinstall the content mod per its page. |
| **No** UE4SS mod works (not just this one), or `AOB scans could not be completed` | `UE4SS_Signatures\` is missing, out of date, or sitting **inside** `Mods\`; or the UE4SS version is not 3.0.1 | All seven files belong in `ue4ss\UE4SS_Signatures\` (not inside `Mods\`). Reinstall UE4SS mod 9, then the runtime. After a game update, wait for both to catch up. |
| Another UE4SS mod broke after installing the runtime | Usually the signature folder in the wrong place, or a second UE4SS copy (for example an old `ue4ss\` inside `Win64\Mods\`) | As above. Keep exactly one UE4SS install in `Win64\`. |
| ZCUnlocked says UE4SS is old (`ZCUnlocked.log`: `ue4ss: detected=… verdict=old`) | Runtime 0.10–0.12 overwrote three of mod 9's signature files with copies that carry no version marker | Install this runtime (0.12.1 or newer), then reinstall UE4SS mod 9 and replace all files. In Mod Command: Settings → ZCSDK Runtime → Reinstall. |
| Short hitch every ~5 s in the hub (log: `pump: task rowfix took 60-90 ms in one frame`) | Loader 1.8.24–1.8.28's Armory row fix (ROWFIX) walked the game's whole object list on every pass, even with the Armory closed | **Fixed in 1.8.29 (since runtime 0.12):** no class lookup, one lookup per pass with back-off, and it stands down entirely while ZCUnlocked's own row healers (`vm-blank` / `listfix`) are on. The log says `ROWFIX skipped: ZCUnlocked is installed and enabled …` once. |
| Blank rows in the Armory weapon list with ZCUnlocked installed and 1.8.29 | ROWFIX stood down for ZCUnlocked (`ROWFIX skipped` in the log), but ZCUnlocked's healers did not fix those rows | Create `ue4ss\Mods\ZCSDKLoader\settings.ini` with `[ROWFIX]` / `Enabled = on` to run both. `Enabled = off` switches the row fix off entirely. |
| The game crashes at start after a game update | The signatures are for an older build | Wait for the runtime release for the new build, or use Mod Command. |
| SDK mods are missing while ZC Unlocked is installed, and `ZCUnlocked.log` says it refused this game build | ZC Unlocked turns itself off on a game build it doesn't know yet | Runtime 0.15 (loader 1.8.53) lists the SDK mods itself in that case. Update the runtime; ZC Unlocked's own features come back when its author updates it. |

Please send **both** `ue4ss\UE4SS.log` and `ue4ss\ZCSDKLoader.log` with any report.

---

## Changes: 0.15.2 → 0.15.3

| | 0.15.2 (2026-11-22) | 0.15.3 |
|---|---|---|
| ZCSDKBridge | 0.7.3 | 0.7.3 (same DLL, SHA-256 `dcd2171a…`) |
| ZCSDKLoader | 1.8.56 | **1.8.57** |

- **Camera stutter in missions fixed.** On a unit with more than 4 class abilities, the runtime searched the whole game for the ability wheel about twice a second, which made the tactical camera stutter for the whole mission. It now looks once in a while and when the wheel opens or closes.
- **Hub lag fixed.** The runtime's saber checks now run much less often in the Hub when nothing changes, and an Armory gear change no longer starts thirty seconds of quick re-checks.
- **Ability wheel with more than 4 class abilities.** Every unit's wheel gets the extra class buttons (not only one unit's), the buttons are narrowed so they overlap much less (a thin overlap of the button frames remains; a follow-up fixes it), and class buttons past the fourth no longer show the 8 / 9 / 0 keys (those keys reach other buttons).
- **If the wheel ever misbehaves:** set `Enabled = off` under `[Wheel]` in `ue4ss\Mods\ZCSDKLoader\settings.ini` and restart the game. You get the stock wheel (only the first 4 class abilities on it), and nothing else changes.

## Changes: 0.15.1 → 0.15.2

| | 0.15.1 (2026-11-12) | 0.15.2 |
|---|---|---|
| ZCSDKBridge | 0.7.3 | 0.7.3 (same DLL, SHA-256 `dcd2171a…`) |
| ZCSDKLoader | 1.8.54 | **1.8.56** |

- **SDK mods no longer depend on ZC Unlocked to show their items.** When a mod shipped a ZC Unlocked menu pack, the runtime used to leave that mod's listing to ZC Unlocked. ZC Unlocked can't add a new weapon class, so with ZC Unlocked 1.4.77 the blaster pack's Z-6 Rotary Cannon lost its own class and showed only as a DC-15LE model. The runtime now lists every mod itself; with ZC Unlocked on, the blasters also show in their own classes next to its model tiles. ZC Unlocked's extras (Bolt Color, Twin pistols, model tiles) stay as they were.
- **Pack options.** Mods can offer settings (switches and choices) that you change in their `settings.ini`, or in ZC Unlocked's menu when the mod mirrors one. `[ZCSDK] enabled = 0` in a mod's `settings.ini` hides that mod's items without uninstalling it.

## Changes: 0.15 → 0.15.1

| | 0.15 (2026-11-11) | 0.15.1 (2026-11-12) |
|---|---|---|
| ZCSDKBridge | 0.7.2 | **0.7.3** |
| ZCSDKLoader | 1.8.53 | **1.8.54** |

A security update. SDK mods work the same, on the same game build (201250) and the same signature files.

- **Mod manifests are read as data, never run.** The loader used to run each `<Mod>.zcsdk.lua` as Lua. It now reads only the values in a plain `return { ... }` table. A file with anything else in it is skipped, and `ZCSDKLoader.log` names the mod ("manifest not supported"). One broken mod no longer stops the mods after it.
- **The bridge takes only the loader's commands,** only from the current game session, and calls only the game functions the loader needs. Its debug commands are left out of the release build.
- **A reproducible DLL.** Two builds of the bridge came out byte for byte the same; its SHA-256 is `dcd2171ae195cd76f72e08a5756d73bf9c65f2e5d7a8e723cad453bded2dee1b`. Each release lists the SHA-256 of both zips, and `latest.json` carries them too.
- **A fresh log every launch.** `ZCSDKLoader.log` starts new each time; the previous one is kept as `ZCSDKLoader.prev.log`.

## Changes: 0.14.1 → 0.15

| | 0.14.1 (2026-10-17) | 0.15 (2026-11-11) |
|---|---|---|
| ZCSDKBridge | 0.6.0 | **0.7.2** |
| ZCSDKLoader | 1.8.34 | **1.8.53** |
| Signature files | 3 (build 199049) | 3 (**build 201250**) |
| Symbol tables | 197649, 199049 | 197649, 199049, **201250** |

- **Fixes mods not loading after the 2026-11-10 game update (build 201250).** The patch broke two of UE4SS's signature scans (ConsoleManager and GUObjectHashTables), so UE4SS started no Lua mods at all. This runtime ships those two files rebuilt for 201250, plus FName_ToString. Mod 9's four files are still not included.
- **SDK mods no longer wait on ZC Unlocked.** If ZC Unlocked is installed but has turned itself off for an unknown game build, the loader lists the SDK mods itself and runs the Armory row fix.
- **New gameplay tags from SDK mods are registered** when the game loads (bridge 0.7.1+), so mod abilities and items that check them behave as built.
- **Status effect visuals added by mods** show in missions (bridge 0.7.2 adds mod rows to the live status-effect table).
- Loader 1.8.35 to 1.8.53: saber customization (blade colors, fixed crystal slot counts, Dual and Staff layouts), setting-gated item grants spread over several frames, class auras and beams, the ability wheel for classes with more than four buttons, AI use of mod abilities, and many smaller fixes.

## Changes: 0.14 → 0.14.1

- v0.14 shipped an old ZCSDKBridge build by mistake, so the Proton / Steam Deck path could not resolve the game's functions. v0.14.1 ships the real bridge 0.6.0.

## Changes: 0.13 → 0.14

- Game update 199049 (2026-10-15): the three signature files rebuilt for 199049; symbol tables for 197649 and 199049; loader 1.8.34 (class auras pulse each turn).

## Changes: 0.12.2 → 0.13

- Linux / Steam Deck (Proton) support: ZCSDKBridge 0.6.0 reads the game's functions from a shipped symbol table (or its own PDB reader) instead of Windows' `dbghelp`.

## Changes: 0.12.1 → 0.12.2

| | 0.12.1 (2026-10-01) | 0.12.2 |
|---|---|---|
| ZCSDKBridge | 0.5.3 | 0.5.3 (unchanged) |
| ZCSDKLoader | 1.8.30 | **1.8.32** |
| Signature files | 3 | 3 (unchanged) |

For class mods (recruit pins):
- **Talents hold.** A recruit card's Talent is set only after the game has written the voice's talent (the loader waits until the Recruitment screen reports every card generated) and is re-checked at 8 / 30 / 60 / 120 s, re-pinned if the game overwrote it. Before, the game's voice-to-talent write could replace a mod's talent on the card.
- **Stock recruits are never taken over.** Only parts from the mod's own folder identify a character as the mod's. Before, a stock recruit whose talent or weapon happened to match a mod's pin could be given the mod's class.
- **No re-pin storm after a refresh.** The previous card set is retired on every Recruitment refresh, a write that changes nothing gives up after 3 tries, and the newest cards are pinned first (about 2-6 s after a refresh).
- **Recruited characters keep the player's choices** where a mod allows them ("roster" accept-sets now apply to recruited characters, not only to non-cards).
- **No console window at launch.** The loader no longer starts a hidden `cmd.exe` to find the game folder.

## Changes: 0.12 → 0.12.1

| | 0.12 (2026-09-30) | 0.12.1 |
|---|---|---|
| ZCSDKBridge | 0.5.3 | 0.5.3 (unchanged) |
| ZCSDKLoader | 1.8.29 | 1.8.30 (version number only, so Mod Command offers the update; no code change) |
| Signature files | 6 | **3** |

- The runtime ships only the three signature files UE4SS mod 9 lacks: `ConsoleManager.lua`, `FName_ToString.lua`, `GUObjectHashTables.lua`.
- It no longer carries `StaticConstructObject.lua`, `FName_Constructor.lua` or `ProcessLocalScriptFunction.lua`. Mod 9 ships its own copies, which work on build 197649 and carry the version marker ZCUnlocked reads. Ours overwrote them, so ZCUnlocked showed its "old UE4SS" popup.
- No gameplay change: same loader and bridge.

## Changes: 0.11 → 0.12

| | 0.11 (2026-09-29) | 0.12 |
|---|---|---|
| ZCSDKBridge | 0.5.3 | 0.5.3 (unchanged) |
| ZCSDKLoader | 1.8.26 | **1.8.29** |

**Packaging**
- A new by-hand zip, `ZCSDKRuntime_v0.12_manual.zip`, with one top-level `ue4ss\` folder. You extract it once into `Binaries\Win64\` and nothing needs moving.
- The Mod Command zip, `ZCSDKRuntime_v0.12.zip`, keeps its layout.
- Both runtime folders ship an empty `enabled.txt`.

**Loader 1.8.27**
- The recruit pin runs in slices, which removes the multi-second Den pause.
- A cheap world read: the viewport's world instead of a `FindAllOf("PlayerController")` on every call.
- The class-mod runtime:
  - `classPool` weights with a player `settings.ini`
  - `requires[]` fallbacks
  - pin re-arm on a Recruitment refresh
  - Talent pin
  - accept-sets

**Loader 1.8.28**
- No frame makes more than 2 object lookups. The 2.9 s hub frames are gone.
- Pin passes are scoped to the mod's own characters, and the hub goes dormant when nothing of the mod's is there.
- A hook-driven urgent pass, so a recruit is pinned within about 3–5 s.

**Loader 1.8.29**
- The ~5 s hub stutter from the Armory row fix is gone:
  - it makes one lookup per pass instead of a class lookup on every pass;
  - it backs off when there is nothing to heal: 5 → 30 s while the Armory weapon page is closed, 2 → 8 s on the page after 3 clean passes (a stuck row brings back 1 s passes);
  - closing the page or leaving the hub disarms it;
  - it stands down for the session, with one log line, while ZCUnlocked's row healers (`vm-blank` / `listfix`) are on.
- A new optional `ue4ss\Mods\ZCSDKLoader\settings.ini` with `[ROWFIX] Enabled = auto | on | off`.

**Signatures**
- For game build 197649 (regenerated 2026-09-28). Six files; 0.12.1 drops the three mod 9 owns.

## Versions

`latest.json` at the root of this repository always describes the newest release: `version`, `bridge`, `loader`, and the download `url` of the Mod Command zip.

| Runtime | ZCSDKBridge | ZCSDKLoader | Notes |
|---|---|---|---|
| 0.15.3 | 0.7.3 | 1.8.57 | mission camera stutter and Hub lag fixed; the ability wheel with more than 4 class abilities on every unit, much less button overlap |
| 0.15.2 | 0.7.3 | 1.8.56 | SDK mods listed by the runtime even with ZC Unlocked's menu packs (the Z-6 class is back); pack options |
| 0.15.1 | 0.7.3 | 1.8.54 | security update: manifests read as data, never run; bridge release build takes only the loader's commands; reproducible DLL (SHA-256 `dcd2171a…`); a fresh log every launch |
| 0.15 | 0.7.2 | 1.8.53 | game build 201250; mod gameplay tags registered; mod status-effect visuals; SDK mods listed when ZC Unlocked refuses the build |
| 0.14.1 | 0.6.0 | 1.8.34 | the real bridge 0.6.0 (v0.14 shipped an old dll) |
| 0.14 | 0.6.0 (old dll) | 1.8.34 | game build 199049 |
| 0.13 | 0.6.0 | 1.8.32 | Linux / Steam Deck (Proton): symbol tables, no dbghelp |
| 0.12.2 | 0.5.3 | 1.8.32 | recruit pins: talents hold, stock recruits never taken over, no re-pin storm, recruited characters keep allowed choices; no console window at launch |
| 0.12.1 | 0.5.3 | 1.8.30 | three signature files only (mod 9 keeps its own four): ZCUnlocked's "old UE4SS" popup fixed |
| 0.12 | 0.5.3 | 1.8.29 | by-hand zip with one `ue4ss\` folder; the ROWFIX hub stutter fixed; 1.8.27/1.8.28's frame budget and class-mod runtime |
| 0.11 | 0.5.3 | 1.8.26 | one game-thread pump (no Lua on UE4SS's async thread: the Den crash fix) |
| 0.10 | 0.5.3 | 1.6.0 | mods in the game's own `SWZeroCompany/Mods/<Mod>/` folder are discovered |
| 0.9 | 0.5.2 | 1.5.0 | the bridge's symbol cache: startup with many SDK mods drops from minutes to seconds |
| 0.8 | 0.5.1 | 1.5.0 | `recruitPins[].match`; Class-slot pins |
| 0.7 | 0.5.1 | 1.4.0 | `recruitPins` |
| 0.6 | 0.5.1 | 1.3.1 | DataRegistry refresh + pre-authored character-pool re-read |
| 0.5 | 0.4.0 | 1.2.0 | manifest `preload` |
| 0.4 | 0.4.0 | 1.1.0 | primary-asset `rescan` + per-type cache refresh |
| 0.3 | 0.3.0 | 1.0.0 | first bundled runtime: registry injection + per-save grants |

## Source

The runtime is built from the SDK repository (`tools/ue4ss-bridge`, `tools/ue4ss-loader`) and packaged with `tools/package-runtime.js`.
