# Better Session Title (dsh-better-session-title)

[English](README.en.md) · [简体中文](README.md)

![npm version](https://img.shields.io/npm/v/dsh-better-session-title) ![npm downloads](https://img.shields.io/npm/dw/dsh-better-session-title) ![license](https://img.shields.io/npm/l/dsh-better-session-title)

> A session topbar enhancement plugin for **DeepSeek Harness**: replaces the native session title in the topbar with a two-level "Workspace / Session" breadcrumb, letting you switch workspaces and sessions right from the topbar — **pure UI enhancement, no new capabilities, no system source changes**.

Works on **Windows · macOS · Linux** (DSH Web GUI) — plain JavaScript, no native dependencies.

```sh
dsh plugin --profile web add dsh-better-session-title
```

**Quick Start** · [Why This Plugin](#why-this-plugin) · [Features](#features) · [Installation](#installation) · [Usage](#usage) · [Relationship with basrs-sidebar](#relationship-with-basrs-sidebar) · [Development](#development) · [License](#license)

---

## Why This Plugin

DSH's native topbar shows the session title (an ancestry breadcrumb) in the center, while workspace/session switching lives in the left sidebar — when the sidebar is collapsed, switching costs extra clicks. Better Session Title puts the switching entry right into the topbar:

- **Workspace dropdown** — all workspaces with their session counts, one-click switch, plus "Add workspace" (native directory picker) and delete.
- **Session dropdown** — all sessions of the current workspace with relative timestamps, one-click switch, plus "New session".

All inline row actions reuse native DSH capabilities: rename workspace/session, delete workspace, fork session, archive session — identical to the native sidebar behavior.

## Features

**Two-level breadcrumb** — the topbar shows "Workspace / Session" as two segments, each expanding its own dropdown list; auto-collapses to icons when space is tight (hover/click still works).

**Workspace dropdown** — workspace list (session-count meta), Add workspace (native directory picker), row "more": rename / delete workspace.

**Session dropdown** — session list of the current workspace (relative-time meta), New session, row "more": rename / fork / archive session.

**Native-consistent interaction details**:

- Meta info sits at the row end and is covered by the "More" button on hover; touch devices keep the button visible and hide the meta.
- The dropdown is a self-drawn layer whose styles replicate native `Menu.module.css` token-for-token (row height 40, padding 8/10, radius 10, same shadow/border tokens).
- The more menu and rename/delete dialogs use the native `Menu` / `Modal` / `Button` / `Input` (autoFocus, select-all, IME-safe Enter handling, duplicate workspace-name check, error rows).
- The more menu shares its lifecycle with the owning dropdown: when the dropdown closes (outside click / Escape / selection / switching), the menu closes with it — no orphan menus.
- Built-in zh/en dictionaries that follow the DSH language switch live.

## Installation

`dsh-better-session-title` is a **bundle** — a configuration layer distributed as an npm package (see the [official plugin docs](https://deepseek-harness.github.io/deepseek-harness/develop/basic/publish)). Install it into a profile with `dsh plugin`: it links the package and appends it to `dsh.profile.bundles`:

```sh
# from the npm registry
dsh plugin --profile web add dsh-better-session-title

# or a local directory / packed tarball
dsh plugin --profile web add ./dsh-better-session-title
dsh plugin --profile web add ./dsh-better-session-title-1.0.0.tgz

# or straight from git (pure JS package, no build step)
dsh plugin --profile web add github:xinspark/dsh-better-session-title#<sha>
```

Then start the Web UI: `dsh web` (or `dsh --profile web`). The first `add` auto-initializes the profile with `@deepseek-ai/dsh-base`.

Uninstall: `dsh plugin --profile web remove dsh-better-session-title` (removes the dependency and the composition layer).

The bundle patch (`cordis.patch.yml`) registers both the host and client halves automatically. The client half is served on demand: client changes apply after a page refresh; host changes need a restart.

## Usage

After installation, open any session — the topbar center shows the "Workspace / Session" breadcrumb:

- Click the **workspace** side: the workspace dropdown opens ("Add workspace" on top).
- Click the **session** side: the session dropdown opens ("New session" on top).
- Hover a row: the "More" button appears at the row end (rename / fork / archive etc., per row type).
- Select a row to switch; click outside or press `Esc` to close the dropdown.

## Relationship with basrs-sidebar

[basrs-sidebar](https://github.com/xinspark/DSH-Basic-Right-Sidebar) also ships a workspace/session breadcrumb for the session topbar. **Both plugins register the same topbar slot — use only one of them.** If both are installed, turn off the breadcrumb in basrs-sidebar's settings and keep this plugin's topbar breadcrumb.

## Development

- **`index.js`** — host half: minimal no-op (a pure UI plugin; all logic lives in the browser half).
- **`lib/client.js`** — browser half (UMD, registered into DSH `clientModules`). Structure, top to bottom:
  1. Local utilities: style injection / timers.
  2. **Bilingual dictionaries** (`BST_ZH` / `BST_EN`) — registered into `ctx.locale`, following the language switch; `bstT()` falls back gracefully.
  3. **Self-drawn dropdown** `BstList` — styles replicate the native Menu; viewport-aware positioning; pointerdown-based outside close (no mousedown, so inline clicks are never swallowed).
  4. `WorkspaceTitleBreadcrumb` — registered into `conversation.session.header.actions` (`order: -20`, leftmost in the topbar); auto compact mode; the more menu shares the dropdown lifecycle; rename/delete use native Modals.
  5. `apply(ctx)` — style injection + slot registration; service calls mirror ui-workspace (`ctx.workspaces` / `ctx.sessions`).

Quick reference:

- **Add a row action** — add an entry to the `actions` array of `wsListItems` / `ssListItems` (`{ id, label, icon?, danger?, action }`).
- **Add copy** — add a key to the `BST_ZH` / `BST_EN` dictionaries.
- **Tweak dropdown styles** — edit the `.bst-item*` rules in `insertStyle` (tokens match the native Menu).

## License

[MIT](LICENSE) — fully open source and free.

Better Session Title is a community plugin for DeepSeek Harness — **not a DeepSeek official product**.
