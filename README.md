<div align="center">
  <img src="https://pan.samyyc.dev/s/VYmMXE" />
  <h2><strong>MVP Anthem</strong></h2>
  <h3>Round MVP anthem plugin for SwiftlyS2 with persistent player settings, permissions, and preview menu.</h3>
</div>

<p align="center">
  <img src="https://img.shields.io/badge/build-passing-brightgreen" alt="Build Status">
  <img src="https://img.shields.io/github/downloads/T3Marius/SwiftlyS2-MVP-Anthem/total" alt="Downloads">
  <img src="https://img.shields.io/github/stars/T3Marius/SwiftlyS2-MVP-Anthem?style=flat&logo=github" alt="Stars">
  <img src="https://img.shields.io/github/license/T3Marius/SwiftlyS2-MVP-Anthem" alt="License">
</p>

## Overview

MVP Anthem lets players pick a custom MVP anthem and volume, then plays it to everyone when that player gets round MVP.

It supports:

- Persistent per-player settings using Cookies.
- MP3 playback through Audio API.
- Sound event playback (`SoundEvent`) for event-name sounds.
- Per-MVP permissions by SteamID64 or permission flag.
- Random MVP mode per player.
- Localized menu/chat/html messages.

## Requirements

This plugin depends on these shared interfaces:

- `Cookies.Player.v1` / `Cookies.Player.V1` (Cookies plugin)
- `audio` (Audio plugin)

Install both plugins before MVP Anthem.

- Cookies: https://github.com/SwiftlyS2-Plugins/Cookies
- Audio: https://github.com/SwiftlyS2-Plugins/Audio

## Installation

1. Build or download release.
2. Copy `plugins/` and `data/` into your SwiftlyS2 root:
   - `game/csgo/addons/swiftlys2/plugins/`
   - `game/csgo/addons/swiftlys2/data/`
3. Make sure Cookies and Audio are installed and loaded.
4. Start/restart server.

## Default Sounds Included

Release/build already includes default MVP sounds:

- `flawless.mp3`
- `florinsalam.mp3`

If you copy both `plugins` and `data` folders from the release archive, these sounds work immediately.

## Commands

Commands come from `Main.Settings.MVPCommands`.

Default:

- `mvp` -> opens the MVP menu.

You can add multiple aliases in config.

## Config (`config.jsonc`)

The plugin reads from `Main`.

### Settings (`Main.Settings`)

| Setting | Default | Description |
| :-- | :-- | :-- |
| `MVPCommands` | `["mvp"]` | Command aliases that open menu. |
| `RemovePlayerInGameMvp` | `true` | Clears CS2 native MVP card count for MVP player. |
| `GiveRandomMVPOnFirstConnect` | `true` | On first connect only, assign a random accessible MVP. |
| `DefaultVolume` | `0.2` | Default volume stored for first connect (`0.0..1.0`). |
| `MVPMaxDuration` | `10` | Max total seconds for repeating center HTML MVP message. |
| `SoundEventFiles` | `[]` | Config field kept for compatibility/manual organization. |
| `ShakePlayerScreen` | `true` | Config field currently not used by runtime logic. |

`GiveRandomMVPOnFirstJoin` is an alias property of `GiveRandomMVPOnFirstConnect`.

### Menu (`Main.Menu`)

| Setting | Default | Description |
| :-- | :-- | :-- |
| `FreezePlayer` | `true` | Freeze player while menu is open. |
| `EnableSounds` | `true` | Enable menu navigation sounds. |
| `GradientTitleColor` | `true` | Apply gradient menu title with Swiftly helper. |
| `VolumeOptions` | `[0,10,20,40,60,80,100]` | Volume choices shown in volume submenu. |

### MVP Templates (`Main.MVPs`)

Structure:

- Category key -> dictionary of MVP items.
- MVP item key -> template object.

Template fields:

| Field | Description |
| :-- | :-- |
| `DisplayName` | Translation key for visible MVP name. |
| `Sound` | Either `.mp3` filename/path or a sound event name. |
| `EnablePreview` | Shows preview button in MVP actions menu. |
| `ShowHtml` | Shows center HTML message on round MVP. |
| `ShowChat` | Shows chat message on round MVP. |
| `Permissions` | Access list (SteamID64 or permission flags). |

## Sound Types: MP3 vs Sound Event

`Sound` behavior is automatic:

- If it ends with `.mp3` -> Audio API playback (`IAudioApi`).
- Otherwise -> `SoundEvent` emission by name.

### MP3 (Audio API)

- Recommended path: place files in `data/MVP_Anthem/`.
- In config, use relative name like `"flawless.mp3"`.
- Relative paths resolve from plugin data directory.
- Supports absolute paths too.

Example:

```jsonc
"mvp_1": {
  "DisplayName": "mvp_1.name",
  "Sound": "flawless.mp3",
  "EnablePreview": true,
  "ShowHtml": true,
  "ShowChat": true,
  "Permissions": []
}
```

### Sound Event

Use a game sound event name:

```jsonc
"mvp_ak": {
  "DisplayName": "mvp_ak.name",
  "Sound": "Weapon_AK47.Single",
  "EnablePreview": true,
  "ShowHtml": true,
  "ShowChat": true,
  "Permissions": []
}
```

## Localization and Config Keys

Config keys map directly to translation keys.

- Category dictionary key:
  - Config: `"category.public_mvp"`
  - Translation: `"category.public_mvp": "Public MVP's"`
- `DisplayName`:
  - Config: `"DisplayName": "mvp_1.name"`
  - Translation: `"mvp_1.name": "Flawless"`
- Round messages use MVP item key:
  - For item key `mvp_1`, plugin reads:
    - `mvp_1.chat`
    - `mvp_1.html`

Menu text also comes from translation keys like:

- `mvp.main_menu<title>`
- `mvp.main_menu.select_mvp<option>`
- `mvp.preview<option>`
- `volume.selected`

## Permission System

`Permissions` in each MVP template uses OR logic:

- Empty list -> everyone can use that MVP.
- Numeric string (SteamID64) -> exact `player.SteamID` match grants access.
- Non-numeric string -> permission flag checked by `Core.Permission.PlayerHasPermission`.

Menu behavior:

- Players only see MVPs they can access.
- Categories with no accessible MVPs are hidden.

## Round MVP Behavior

When a player gets round MVP:

1. Plugin resolves selected MVP.
2. If player has random mode enabled, a random accessible MVP is chosen for that round.
3. Sound is played to all valid players, including MVP player.
4. Each listener hears with their own saved volume.
5. Chat and HTML messages are sent if enabled on that MVP.
6. HTML uses `SendCenterHTML(..., MVPMaxDuration)` directly.

## Player Data (Cookies)

Per-player persisted values:

- Selected MVP key
- Selected sound path
- Volume
- `HadFirstConnect`
- `HasRandomMvp`

On first connect:

- volume initializes from `DefaultVolume`
- random MVP can be assigned if `GiveRandomMVPOnFirstConnect` is true

## Build Output

Build generates:

```text
build/
  plugins/
    MVP_Anthem/
      MVP_Anthem.dll
      resources/...
  data/
    MVP_Anthem/
      flawless.mp3
      florinsalam.mp3
```

## Release Workflow

GitHub workflow (`.github/workflows/release.yml`) creates:

- `MVP_Anthem.vx.x.x.zip`

Archive content:

- `plugins/...`
- `data/...`

So server owners can extract both folders directly into `addons/swiftlys2/`.
