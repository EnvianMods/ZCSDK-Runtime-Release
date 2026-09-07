# ZCSDK Runtime

The runtime half of the **Zero Company Mod SDK** by Envian Mods, for **STAR WARS: Zero Company** (Bit Reactor, Unreal Engine 5.6).
It is two [UE4SS](https://github.com/UE4SS-RE/RE-UE4SS) mods that make SDK-built content mods **enumerable in the game's own systems** — net-new items,
customization parts, class specializations and skill trees show up in the game's menus, shops, reward tables and pickers like the studio's own content.

| Folder | What it is |
|---|---|
| `ZCSDKBridge/` | a native (C++) UE4SS mod: resolves the game's non-reflected engine functions from the shipped PDB at runtime and calls them on the game thread — asset-registry injection, primary-asset rescan, per-type cache refresh, `LoadPackage`. |
| `ZCSDKLoader/` | a Lua UE4SS mod: finds every SDK mod's `<Mod>.zcsdk.lua` manifest next to the paks (`Content/Paks` and `Content/Paks/~mods`), drives the bridge for each (registry, rescan, cache refresh, optional preload) and applies the manifest's one-time-per-save item grants. |

A content mod built with the SDK tells you whether it needs the runtime (its manifest has a `registry`); plain override mods do not.

## Install

**With Zero Company Mod Command (recommended):** Settings → *ZCSDK Runtime* → Install / Update. Mod Command installs UE4SS if needed and keeps the
runtime enabled. It offers the install automatically when you add an SDK mod that needs it.

**By hand:** install UE4SS 3.0.1 for the game (`SWZeroCompany/Binaries/Win64/`), unzip the release's `ZCSDKRuntime_v<x.y>.zip` into
`SWZeroCompany/Binaries/Win64/ue4ss/Mods/` so you get `Mods/ZCSDKBridge/dlls/main.dll` and `Mods/ZCSDKLoader/Scripts/main.lua`, then create an
empty `enabled.txt` inside each of the two folders. Logs: `ue4ss/ZCSDKBridge.log` and `ue4ss/ZCSDKLoader.log` (look for `ZCSDKLoader ready`).

## Versions

`latest.json` at the root of this repository always describes the newest release (`version`, `bridge`, `loader`, download `url`).

| Runtime | ZCSDKBridge | ZCSDKLoader | Notes |
|---|---|---|---|
| 0.8 | 0.5.1 | 1.5.0 | `recruitPins[].match` — pins follow a recruit by a slot + part (recruited characters carry no definition); Class-slot pins |
| 0.7 | 0.5.1 | 1.4.0 | `recruitPins` — keeps a pre-authored recruit's specialization / weapon slots pinned across the game's re-rolls |
| 0.6 | 0.5.1 | 1.3.1 | DataRegistry refresh + pre-authored character-pool re-read (`refreshers`), idempotent |
| 0.5 | 0.4.0 | 1.2.0 | manifest `preload` (opt-in `LoadPackage` of a mod's classes after injection) |
| 0.4 | 0.4.0 | 1.1.0 | primary-asset `rescan` + per-type cache refresh (`callthis`) — customization parts and specializations enumerate |
| 0.3 | 0.3.0 | 1.0.0 | first bundled runtime: registry injection + per-save grants |

## Source

The runtime is built from the SDK repository (`tools/ue4ss-bridge`, `tools/ue4ss-loader`) and packaged with `tools/package-runtime.js`.
Content mods are built with the SDK's `zcmod-build.js`; the SDK ships them as a `.zip` that Mod Command installs like any other pak mod.
