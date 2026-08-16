<p align="center">
  <img src="https://img.shields.io/github/v/release/xinspark/dsh-better-session-title?style=flat&amp;label=release&amp;color=08C" alt="GitHub release">
  <img src="https://img.shields.io/badge/plugin-UI%20enhancement-47848F?style=flat" alt="UI enhancement plugin">
  <img src="https://img.shields.io/badge/DSH%20Web%20GUI-Windows%20%7C%20macOS%20%7C%20Linux-4493F8?style=flat" alt="Windows | macOS | Linux">
  <img src="https://img.shields.io/badge/license-MIT-2EA44F?style=flat" alt="MIT License">
</p>

<p align="center"><sub><a href="README.md">简体中文</a> · English</sub></p>

<h3 align="center">Replace the DSH topbar session title with a two-level "Workspace / Session" breadcrumb — switch right from the topbar</h3>

<p align="center"><a href="#install"><ins><code>dsh plugin --profile web add github:xinspark/dsh-better-session-title#v1.0.0</code></ins></a></p>

Better Session Title is a **session-topbar enhancement plugin** for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) — pure UI enhancement, no new capabilities, no system source changes. DSH's native topbar shows only the session title (an ancestry breadcrumb) in the center, while workspace/session switching lives in the left sidebar; when the sidebar is collapsed, every switch costs extra clicks. This plugin puts the switching entry right into the topbar:

- **Workspace dropdown** — all workspaces with their session counts, one-click switch, plus "Add workspace" (native directory picker) and delete.
- **Session dropdown** — all sessions of the current workspace with relative timestamps, one-click switch, plus "New session".

All inline row actions reuse native DSH capabilities: rename workspace/session, delete workspace, fork session, archive session — identical to the native sidebar behavior.

Works on **Windows · macOS · Linux** (DSH Web GUI) — plain JavaScript, no native dependencies, no build step for distribution.

## Install

`dsh-better-session-title` is a **bundle** — a configuration patch layer distributed as a package (see the [official DSH plugin publishing docs](https://deepseek-harness.github.io/deepseek-harness/develop/basic/publish)). Install it into a profile with `dsh plugin`: it links the package and appends it to `dsh.profile.bundles`:

```sh
# from GitHub (primary channel; v1.0.0 is the current release tag)
dsh plugin --profile web add github:xinspark/dsh-better-session-title#v1.0.0
```

- **Consider locking a commit**: a GitHub install pulls the repository source, so later pushes don't silently change what you run. To pin the exact content, use `github:xinspark/dsh-better-session-title#<commit-sha>` (as recommended by the official docs).
- **Local directory / tarball** (development or offline):

  ```sh
  dsh plugin --profile web add ./dsh-better-session-title
  dsh plugin --profile web add ./dsh-better-session-title-1.0.0.tgz
  ```

Then start the Web UI: `dsh web` (or `dsh --profile web`). The first `add` auto-initializes the profile with `@deepseek-ai/dsh-base`.

Uninstall: `dsh plugin --profile web remove dsh-better-session-title` (removes the dependency and the composition layer).

The bundle patch (`cordis.patch.yml`) registers both the host and client halves automatically. The client half is served on demand: client changes apply after a page refresh; host changes need a restart.

## Features

| Capability | Description |
| --- | --- |
| Two-level breadcrumb | The topbar shows "Workspace / Session" as two segments, each expanding its own dropdown; auto-collapses to icons when space is tight (hover/click still works). |
| Workspace dropdown | Workspace list (session-count meta), Add workspace (native directory picker), row "more": rename / delete workspace. |
| Session dropdown | Session list of the current workspace (relative-time meta), New session, row "more": rename / fork / archive session. |
| Native-consistent interactions | The dropdown is a self-drawn layer replicating the native `Menu.module.css` token-for-token (row height 40, padding 8/10, radius 10, same shadow/border tokens); the more menu and rename/delete dialogs use the native `Menu` / `Modal` / `Button` / `Input` (autoFocus, select-all, IME-safe Enter, duplicate workspace-name check, error rows). |
| Shared lifecycle | The more menu closes with its owning dropdown (outside click / Escape / selection / switching) — no orphan menus. |
| Bilingual | Built-in zh/en dictionaries that follow the DSH language switch live. |

## Usage

After installation, open any session — the topbar center shows the "Workspace / Session" breadcrumb:

- Click the **workspace** side: the workspace dropdown opens ("Add workspace" on top).
- Click the **session** side: the session dropdown opens ("New session" on top).
- Hover a row: the "More" button appears at the row end (rename / fork / archive etc., per row type).
- Select a row to switch; click outside or press `Esc` to close the dropdown.

## Relationship with dsh-basic-right-sidebar

[dsh-basic-right-sidebar](https://github.com/xinspark/DSH-Basic-Right-Sidebar) ships this plugin as a **sub-plugin** inside its bundle: its `package.json` declares this package as a dependency and its `cordis.patch.yml` inserts both plugin rows; its "Show workspace/session breadcrumb" setting controls this plugin live via the `window.__BASRS_SETTINGS__` snapshot and the `'basrs:settings'` event (when off, this plugin hides the breadcrumb and restores the native title).

- **If you installed dsh-basic-right-sidebar, do not install this plugin separately** — two bundles inserting the same plugin row duplicate it (`insert` appends).
- If both bundles are installed, turn the breadcrumb off in dsh-basic-right-sidebar's settings and keep this plugin's topbar breadcrumb.
- This plugin can also be **installed standalone** (without the parent plugin the settings bridge defaults to enabled, same behavior as before).

## Relationship with DSH

This project is built on the official [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) and is a **community plugin** — not an official DeepSeek product. Everything it shows is DSH's own data and UI, re-arranged; no system source changes.

## Development

- **`index.js`** — host half: minimal no-op (a pure UI plugin; all logic lives in the browser half).
- **`lib/client.js`** — browser half (UMD, registered into DSH `clientModules`). Structure, top to bottom:
  1. Local utilities: style injection / timers.
  2. **Bilingual dictionaries** (`BST_ZH` / `BST_EN`) — registered into `ctx.locale`, following the language switch; `bstT()` falls back gracefully.
  3. **Self-drawn dropdown** `BstList` — styles replicate the native Menu; viewport-aware positioning; pointerdown-based outside close (no mousedown, so inline clicks are never swallowed).
  4. **Parent-plugin settings bridge** — reads the `window.__BASRS_SETTINGS__` snapshot and subscribes to `'basrs:settings'` (in `WorkspaceTitleBreadcrumb`) to toggle the breadcrumb.
  5. `WorkspaceTitleBreadcrumb` — registered into `conversation.session.header.actions` (`order: -20`, leftmost in the topbar); auto compact mode; the more menu shares the dropdown lifecycle; rename/delete use native Modals.
  6. `apply(ctx)` — style injection + slot registration; service calls mirror ui-workspace (`ctx.workspaces` / `ctx.sessions`).

Quick reference:

- **Add a row action** — add an entry to the `actions` array of `wsListItems` / `ssListItems` (`{ id, label, icon?, danger?, action }`).
- **Add copy** — add a key to the `BST_ZH` / `BST_EN` dictionaries.
- **Tweak dropdown styles** — edit the `.bst-item*` rules in `insertStyle` (tokens match the native Menu).

## Acknowledgments

Thanks to [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) and the DeepSeek AI team — the agent, models, tools, sessions, Web UI and plugin ecosystem all come from this project; to [Cordis](https://github.com/cordiverse/cordis) for the plugin foundation; and to the [Koishi.js](https://koishi.chat/) community for its long-standing plugin practices, tooling and experience.

## License

[MIT](LICENSE) — fully open source and free.

Better Session Title is a community plugin for DeepSeek Harness — **not a DeepSeek official product**.
