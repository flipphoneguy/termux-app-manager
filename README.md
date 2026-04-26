# tam — Termux App Manager

A single-file bash wrapper around `pm`, `am`, `cmd package`, and the Intent
Firewall for managing Android apps from rooted Termux.

## Requirements

- Rooted Android (passwordless `sudo` / `tsu`)
- `pm`, `am` (always present); `fzf` recommended for nicer pickers

## Install

```sh
chmod +x tam
ln -s "$PWD/tam" "$PREFIX/bin/tam"
```

## Usage

Run `tam` with no args for an interactive menu, or pass a subcommand:

```sh
tam list -3                  # third-party packages
tam info termux              # partial name → picker if ambiguous
tam open settings            # pick an activity to launch
tam ifw-block com.foo        # pick a component, write IFW rule
tam extract com.termux ~/apks
```

`tam help` prints every subcommand. Most take a partial package name and
prompt via `fzf` (or a numbered menu fallback) when more than one matches.

## What it covers

- Listing / searching packages, showing path, version, enabled state, launcher
- Launching the default activity or any chosen activity / component
- Listing activities, services, receivers, providers
- Enable / disable / hide / suspend at package or component granularity
- Force-stop, kill, clear data, uninstall (with `-k` to keep data), install
- Extracting base + split APKs to a directory
- Listing / granting / revoking runtime permissions
- Intent Firewall: block / unblock / list / clear rules in
  `/data/system/ifw/<pkg>.xml` (auto-detects activity / service / broadcast)

## Notes

- All privileged calls go through `sudo` — no in-place root checks beyond that.
- IFW writes one file per package and merges rules idempotently.
- `NO_COLOR=1` disables colored output.
