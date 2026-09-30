# ZCSDK Runtime

The ZCSDK Runtime is the runtime half of the **Zero Company Mod SDK** by Envian Mods, for **STAR WARS: Zero Company** (Bit Reactor, Unreal Engine 5.6).

It is two [UE4SS](https://github.com/UE4SS-RE/RE-UE4SS) mods. Together they let content mods built with the SDK (new weapons, armour, classes, abilities) appear in the game's own menus, shops, reward tables and pickers, alongside the game's own content.

| Folder | What it is |
|---|---|
| `ZCSDKBridge/` | A native (C++) UE4SS mod. It finds the game functions that UE4SS can't reach, using the game's shipped PDB, and calls them on the game thread: asset-registry injection, primary-asset rescan, per-type cache refresh and `LoadPackage`. |
| `ZCSDKLoader/` | A Lua UE4SS mod. It finds every SDK mod's `<Mod>.zcsdk.lua` manifest, drives the bridge for each one, keeps recruit pins, and applies the manifest's once-per-save item grants. |

An SDK-built mod tells you on its own page whether it needs the runtime. Plain override mods do not.

**Tested on:** Steam, game build **197649**, **UE4SS 3.0.1**.

---

## Install

### 1. UE4SS 3.0.1 (required, once)

Download **UE4SS v3.0.1** from its official release page: <https://github.com/UE4SS-RE/RE-UE4SS/releases/tag/v3.0.1>. The asset is `UE4SS_v3.0.1.zip`. Do not use the `zDEV` build.

Extract it into:

```
...\Star Wars Zero Company\SWZeroCompany\Binaries\Win64\
```

That puts `dwmapi.dll` and a `ue4ss\` folder next to `SWZeroCompany.exe`. Other UE4SS versions are not supported. A different version is the most common reason nothing loads.

### 2a. With Zero Company Mod Command (recommended)

Go to Settings → **ZCSDK Runtime** → Install / Update. Mod Command installs UE4SS if needed, puts every file in place (including the signatures) and keeps the runtime switched on. It also offers the install on its own when you add an SDK mod that needs it.

### 2b. By hand: one extract

Download **`ZCSDKRuntime_v0.12_manual.zip`**. It is the one with `_manual` in its name. The other zip on the release is for Mod Command.

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
   ├─ UE4SS_Signatures\               (this runtime: 6 .lua files for game build 197649)
   │   ├─ ConsoleManager.lua
   │   ├─ FName_Constructor.lua
   │   ├─ FName_ToString.lua
   │   ├─ GUObjectHashTables.lua
   │   ├─ ProcessLocalScriptFunction.lua
   │   └─ StaticConstructObject.lua
   └─ Mods\
      ├─ ZCSDKBridge\
      │   ├─ dlls\main.dll
      │   ├─ modinfo.json
      │   └─ enabled.txt
      └─ ZCSDKLoader\
          ├─ Scripts\main.lua
          ├─ modinfo.json
          └─ enabled.txt
```

If Windows asks whether to replace files, answer **Yes**. The only files replaced are the runtime's own and the six signature files.

**About `enabled.txt`.** UE4SS starts a mod folder only when it holds an `enabled.txt`, or when `ue4ss\Mods\mods.txt` has a `Name : 1` line for it. Both runtime folders ship an empty `enabled.txt`, so nothing else is needed. To switch one part off by hand, rename its `enabled.txt` (for example to `enabled.txt.off`). If your `mods.txt` has `ZCSDKBridge : 0` or `ZCSDKLoader : 0`, change it to `1` or delete that line.

**About `UE4SS_Signatures\`.** UE4SS's own pattern scans can't find some of this game's functions, and every game update moves them. These six files are generated from the game's own PDB for build 197649. They belong **next to** `Mods\`, not inside it. If they are missing or out of date, UE4SS stops with `AOB scans could not be completed`, and then no Lua mod runs at all.

**Game Pass / Microsoft Store.** This build is not tested there. The runtime is built and tested on the Steam edition only.

### Updating from 0.11

- **Mod Command:** Settings → ZCSDK Runtime → Update.
- **By hand:** extract `ZCSDKRuntime_v0.12_manual.zip` into `Win64\` as above and replace everything.
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
- `ue4ss\Mods\ZCSDKLoader\Scripts\main.lua`
- `ue4ss\Mods\ZCSDKLoader\enabled.txt`
- `ue4ss\UE4SS_Signatures\StaticConstructObject.lua` (and the five others)

**The log.** Start the game, reach the main menu, then open `ue4ss\UE4SS.log`. `ue4ss\ZCSDKLoader.log` carries the same lines. Look for these, in this order:

```
UE4SS - v3.0.1 Beta ...
Starting C++ mod 'ZCSDKBridge'           (or: Mod 'ZCSDKBridge' has enabled.txt, starting mod.)
Starting Lua mod 'ZCSDKLoader'
[ZCSDKLoader] ZCSDKLoader v1.8.29 loaded (win64=...); waiting for the main-menu world
[ZCSDKLoader] pump: running on the game thread (1.8.29; ...)
[ZCSDKLoader] === ZCSDKLoader start ===
[ZCSDKLoader] manifest: .../<YourMod>.zcsdk.lua  (<YourMod> v<x.y.z> ...)      (one line per SDK mod installed)
[ZCSDKLoader] N manifest(s), M grant(s)
[ZCSDKLoader] === ZCSDKLoader ready; ... ===
```

`ZCSDKLoader v1.8.x loaded` followed by `ZCSDKLoader ready` means the runtime works.

**The Armoury row fix (ROWFIX, 1.8.29).** In the hub, `ZCSDKLoader.log` says ONE of these once:

```
ROWFIX skipped: ZCUnlocked is installed and enabled (ue4ss/Mods/ZCUnlocked via mods.txt) and heals the Armoury rows itself (...)
ROWFIX: ZCUnlocked is not installed          (then, on the Armoury weapon page: ROWFIX armed / backed off / disarmed)
```

Neither ever repeats every few seconds, and there should be no `pump: task rowfix took … ms` line with ZCUnlocked installed. `0 manifest(s)` means the runtime works but no SDK mod was found (see Troubleshooting). The bridge writes its own log to `ue4ss\ZCSDKBridge.log`.

---

## Troubleshooting

| What you see | Likely cause | Fix |
|---|---|---|
| New weapons / armour / classes are missing, and the log has **no** `ZCSDKLoader` line | The runtime is in the wrong folder or nested one level too deep, e.g. `Win64\ue4ss\Mods\ZCSDKRuntime_v0.12\ue4ss\Mods\…`, `Win64\Mods\…`, or `SWZeroCompany\Mods\ZCSDKLoader`. | Extract `…_manual.zip` into `Binaries\Win64\` itself, so `Win64\ue4ss\Mods\ZCSDKLoader\Scripts\main.lua` exists. |
| `ZCSDKLoader` is listed but never `loaded` | No `enabled.txt`, or `mods.txt` says `ZCSDKLoader : 0` | Add the empty `enabled.txt`, or set the line to `: 1`. |
| `ZCSDKLoader ready` but `0 manifest(s)` | The content mod is not installed where its page says. SDK mods live in `SWZeroCompany\Mods\<Mod>\` or in `Content\Paks\~mods\` with their `.zcsdk.lua`. | Reinstall the content mod per its page. |
| **No** UE4SS mod works (not just this one), or `AOB scans could not be completed` | `UE4SS_Signatures\` is missing, out of date, or sitting **inside** `Mods\`; or the UE4SS version is not 3.0.1 | Put the six files in `ue4ss\UE4SS_Signatures\`, install UE4SS 3.0.1, and re-download the runtime after a game update. |
| Another UE4SS mod broke after installing the runtime | Usually the signature folder in the wrong place, or a second UE4SS copy (for example an old `ue4ss\` inside `Win64\Mods\`) | As above. Keep exactly one UE4SS install in `Win64\`. |
| Short hitch every ~5 s in the hub (log: `pump: task rowfix took 60-90 ms in one frame`) | Loader 1.8.24–1.8.28's Armoury row fix (ROWFIX) walked the game's whole object list on every pass, even with the Armoury closed | **Fixed in 1.8.29 (this release):** no class lookup, one lookup per pass with back-off, and it stands down entirely while ZCUnlocked's own row healers (`vm-blank` / `listfix`) are on. The log says `ROWFIX skipped: ZCUnlocked is installed and enabled …` once. |
| Blank rows in the Armoury weapon list with ZCUnlocked installed and 1.8.29 | ROWFIX stood down for ZCUnlocked (`ROWFIX skipped` in the log), but ZCUnlocked's healers did not fix those rows | Create `ue4ss\Mods\ZCSDKLoader\settings.ini` with `[ROWFIX]` / `Enabled = on` to run both. `Enabled = off` switches the row fix off entirely. |
| The game crashes at start after a game update | The signatures are for an older build | Wait for the runtime release for the new build, or use Mod Command. |

Please send **both** `ue4ss\UE4SS.log` and `ue4ss\ZCSDKLoader.log` with any report.

---

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
- The ~5 s hub stutter from the Armoury row fix is gone:
  - it makes one lookup per pass instead of a class lookup on every pass;
  - it backs off when there is nothing to heal: 5 → 30 s while the Armoury weapon page is closed, 2 → 8 s on the page after 3 clean passes (a stuck row brings back 1 s passes);
  - closing the page or leaving the hub disarms it;
  - it stands down for the session, with one log line, while ZCUnlocked's row healers (`vm-blank` / `listfix`) are on.
- A new optional `ue4ss\Mods\ZCSDKLoader\settings.ini` with `[ROWFIX] Enabled = auto | on | off`.

**Signatures**
- For game build 197649 (regenerated 2026-09-28).

## Versions

`latest.json` at the root of this repository always describes the newest release: `version`, `bridge`, `loader`, and the download `url` of the Mod Command zip.

| Runtime | ZCSDKBridge | ZCSDKLoader | Notes |
|---|---|---|---|
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
