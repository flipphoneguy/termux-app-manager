# tam — Termux App Manager

A single-file bash wrapper around `pm`, `am`, `cmd package`, `cmd appops`,
`cmd deviceidle`, and the Intent Firewall for managing Android apps from
rooted Termux.

## Requirements

- Rooted Android (passwordless `sudo` / `tsu`)
- `pm`, `am` (always present); `numfmt` for size formatting
- `fzf` recommended for nicer pickers (numbered fallback otherwise)

## Install

```sh
chmod +x tam
ln -s "$PWD/tam" "$PREFIX/bin/tam"
```

## Usage

`tam` with no args opens an interactive menu. Otherwise pass a subcommand —
`tam help` lists every one. Most accept a partial package name; if more than
one matches you get an `fzf` picker.

```sh
tam list -3                  # third-party packages
tam info termux              # partial → picker if ambiguous
tam open settings            # pick an activity to launch
tam ifw-block com.foo        # pick a component, write IFW rule
tam extract com.termux ~/apks
```

## Features

### Packages

- `list [-3 | -s | -d | -e | -all]` — third-party / system / disabled / enabled
- `search <partial>` — print matching package names
- `info <pkg>` — version, enabled state, appId, targetSdk, APK path, launcher, IFW
- `path <pkg>` — APK path(s)

### Launching

- `launch <pkg>` — default MAIN/LAUNCHER activity
- `open <pkg>` — pick an activity to start
- `start <pkg/component>` — direct launch of any component
- `settings <pkg>` — open the OS app-info screen

### Components

- `components <pkg> [kind]` — kind = `activity` | `service` | `receiver` | `provider` | `all`
- Aliases: `activities`, `services`, `receivers`, `providers`
- `enable-c <pkg/comp>` / `disable-c <pkg/comp>` — toggle a single component

### Lifecycle / state

- `enable` / `disable` (uses `disable-user --user 0`)
- `hide` / `unhide` / `suspend [true|false]`
- `force-stop` / `kill` / `clear` (wipes data)
- `uninstall <pkg> [-k]` (-k keeps data) / `install <apk>`

### Extraction

- `extract <pkg> [dir]` — pulls base + split APKs into `dir` (default: cwd)

### Permissions

- `perms <pkg>` — pretty list, grouped: runtime / install (auto) / declared-only
- `perm <pkg>` — interactive picker, toggles state, loops until quit
- `grant  <pkg> [perm]` — picker over denied if perm omitted
- `revoke <pkg> [perm]` — picker over granted if perm omitted

### Accessibility (`a11y` = "accessibility")

Manages `secure enabled_accessibility_services` and the master
`accessibility_enabled` flag.

- `a11y` — list enabled and available services
- `a11y on` / `a11y off` — master switch
- `a11y enable [pkg/comp]` / `a11y disable [pkg/comp]` (picker if omitted)
- `a11y toggle` — pick any service, flip its state
- `a11y <pkg>` — pick & toggle services declared by `<pkg>`

### App-Ops

For permissions that `pm grant` can't toggle — `SYSTEM_ALERT_WINDOW`,
`MANAGE_EXTERNAL_STORAGE`, `PACKAGE_USAGE_STATS`, `WRITE_SETTINGS`,
`REQUEST_INSTALL_PACKAGES`, etc.

- `appops <pkg>` — list current op states with color (allow/ignore/deny/default)
- `appops <pkg> pick` — pick op → pick mode
- `appops <pkg> <op> [mode]` — set one op directly (mode picker if omitted)

### Battery optimisation

Wraps `cmd deviceidle whitelist`. Three categories exist; only the `user`
list is toggleable (Android settings UI mirrors this set).

- `batt` — list user-whitelisted (battery-exempt) packages
- `batt all` — full whitelist (system + system-excidle + user)
- `batt status <pkg>` — which list(s) `<pkg>` is in
- `batt add <pkg>` / `batt remove [pkg]` (picker if omitted) / `batt toggle <pkg>`

### Size breakdown

- `size <pkg>` — APK(s), code dir, internal data, external data, OBB, total

Kept out of `info` because it walks data directories — slow on large apps.

### Intent Firewall

Writes `/data/system/ifw/<pkg>.xml`, one file per package, idempotently merged.
Auto-detects activity / service / broadcast kind.

- `ifw-block   <pkg> [comp]` — picker if comp omitted
- `ifw-unblock <pkg> [comp]` — picker over current rules
- `ifw-list    [pkg]` — show rules (or list rule files)
- `ifw-clear   <pkg>` — delete all rules for pkg

## Compatibility

Targeted at modern stock Android. Tested on Android 14. Caveats per feature:

| Feature | Minimum | Notes |
|---|---|---|
| `pm grant` / `revoke`, runtime perms parser | Android 6 | dumpsys section names stable on 8+ |
| `pm suspend` | Android 9 | older just errors out |
| `appops` | Android 9 (`cmd appops`) | falls back to bare `appops` binary on older |
| `launch` / `a11y` (uses `cmd package query-…`) | Android 10 | dumpsys-resolver-table fallback for `launch` |
| IFW (`/data/system/ifw/`) | Android 4.4+ | **some ROMs strip the dir** (a few Chinese OEMs, GrapheneOS) |
| `pm hide` / `disable-user` / `suspend` | varies | OEM policy (Knox, MIUI) can refuse even with root |

`du -sb`, `numfmt`, `paste`, `fzf` are Termux coreutils — always available
in Termux, not Android-side.

## Notes

- All privileged calls go through `sudo` — no in-place root checks beyond that.
- `NO_COLOR=1` disables colored output.
- Most subcommands accept a partial package name and prompt via `fzf`
  (numbered menu fallback) when more than one matches; an exact match
  short-circuits the picker.
- Interactive menu groups: a11y stays under **permissions**; battery
  optimisation and size breakdown are top-level entries.
