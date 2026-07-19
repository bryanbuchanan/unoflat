# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Uno Flat is a Zed editor theme extension — a dark theme based on Atom's One Dark. It is authored in SCSS and compiled to the Zed theme JSON format.

## Architecture

The source-of-truth is `src/theme.scss`, **not** `themes/unoflat.json`. The JSON file is a build artifact compiled from SCSS via [sassyzed](https://github.com/bryanbuchanan/sassyzed). Always edit the SCSS and re-compile; never hand-edit the JSON, since the next compile will overwrite it.

`src/theme.scss` is organized as:
1. **Semantic color variables** at the top of each theme block (`$foreground-*`, `$background-*`, named colors like `$green`/`$red`/`$orange`). Edit these to retune the palette — downstream rules reference them.
2. **`style { … }` block** mapping Zed's CSS custom properties (e.g. `--editor-background`, `--tab-active_background`) to those variables.
3. **`syntax { … }` block** mapping token types (`keyword`, `function`, `string`, etc.) to colors.
4. **`players { … }` block** for multi-cursor/collaboration colors.

The bottom of the file contains a long list of commented-out properties — these are the full set of Zed theme keys available but not currently overridden. Uncomment to customize.

`extension.toml` declares the Zed extension metadata (id, version, etc.). Bump `version` here when releasing.

## Common tasks

- **Recompile JSON after editing SCSS**: use the `sassyzed` tool (external repo) to compile `src/theme.scss` → `themes/unoflat.json`.
- **Adjust a color**: change the relevant `$variable` in `src/theme.scss`, then recompile.
- **Add a new theme key**: find the commented line at the bottom of the `style` block, uncomment it, and set the value — or add it inline near related keys.

## Publishing

Per `PUBLISH.md`: bump `version` in `extension.toml`, then fork/update the Zed `extensions` repository following <https://zed.dev/docs/extensions/developing-extensions>. The Zed extensions repo references this repo as a submodule pointing at `main`.
