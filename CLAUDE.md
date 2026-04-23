# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A GNOME Shell extension that replaces the stock battery indicator with customizable visual styles (bold, slim, plump, plain, circle, or text-only). Supports GNOME 45–50.

## Build Commands

The build system uses GNU Make with a custom `sdt` (Shutdown Timer Devel) toolchain in `sdt/build/`.

```bash
make install          # Build and install extension locally
make debug-install    # Install with debug mode (mock power state, large overlay icon)
make zip              # Build release ZIP to target/default/
make lint             # Run ESLint + Prettier checks
make format           # Auto-fix lint and formatting issues
make translations     # Extract translatable strings and update .po files
make clean            # Remove build artifacts
```

npm scripts (`npm run lint`, `npm run format`, etc.) also work and delegate to the same tools.

Debug mode activates a mock power manager (`src/modules/mock.js`) that cycles through battery states at 200ms intervals, and renders a 256×256 overlay icon for visual verification.

## Architecture

**Extension lifecycle** (`src/extension.js`):
- On enable: uses `InjectionTracker` to patch GNOME Shell's quickSettings panel, creates `BatteryDrawIcon` widgets, connects to UPower D-Bus proxy and theme/settings signals
- On update: reads battery state + user prefs, redraws icons and recolors percentage labels
- On disable: disconnects signals, removes injected widgets, restores original GNOME Shell state

**Icon rendering** (`src/modules/drawicon.js`):
- All battery shapes are drawn with Cairo 2D graphics in `BatteryDrawIcon` (a `St.DrawingArea` subclass)
- Five style implementations: `bold`, `slim`, `plump`, `plain`, `circle` — each draws the battery body, terminal nub, optional charging bolt, and optional percentage text
- `src/modules/path.js` converts text path descriptions to Cairo drawing commands

**Settings** (`src/schemas/...gschema.xml`):
- `status-style`: bold / slim / plump / plain / circle / hidden
- `show-icon-text`: 0 (none) / 1 (inside horizontal) / 2 (inside vertical)
- `icon-orientation`: vertical / horizontal
- `icon-scale`: 1.0–2.0 (width/height ratio)
- `dynamic-circle-color`, `circle-color`, `circle-empty-color`, `circle-low-color`, `circle-charge-color`: wheel colors for the circle style

**InjectionTracker** (`src/modules/sdt/injection.js`):
- Safely patches and reverts properties/methods on GNOME Shell objects
- Maintains history so nested injections unwind correctly on disable

**Debug flag**: `src/modules/sdt/util.js` exports `debugMode` — set at build time by `sdt/build/gnome-extension.mk` by rewriting that file. Do not hand-edit it.

## Translations

Source strings are extracted to `po/main.pot` via `make translations`. Per-locale `.po` files are in `po/`; compiled `.mo` files are generated into `src/locale/` during build.

## Code Style

ESLint extends GJS/Shell/extension rules from the sdt submodule, plus Prettier (single quotes, no arrow-function parens). Pre-commit hooks enforce REUSE license headers and YAML validity.
