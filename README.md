# Wayland Assistant

Select text anywhere on Hyprland, press a key, and act on it: search the web, translate it, or ask Claude about it.

**Status:** no official release yet. Coming soon.

## Contents

- [At a glance](#at-a-glance)
- [Keys](#keys)
- [Commands](#commands)
- [Requirements](#requirements)
- [Features](#features)
  - [The popup](#the-popup)
  - [Web search](#web-search)
  - [Translate](#translate)
  - [Ask Claude](#ask-claude)
  - [Settings app](#settings-app)
- [Installing](#installing)
- [How it works](#how-it-works)
  - [Permission modes](#permission-modes)
  - [Privacy](#privacy)
  - [What the installer changes](#what-the-installer-changes)
  - [Resource use](#resource-use)
- [License](#license)

## At a glance

| | |
|---|---|
| Opens with | `Alt + T` by default, changeable in Settings |
| Opens in | About 60 ms |
| Actions | Search the web, Translate, Ask Claude, Ask Codex (coming) |
| Search engines | Brave, DuckDuckGo, Google, Startpage, or any URL |
| Translation | Google Translate (shown right in the popup) or DeepL |
| Claude models | Haiku 5.5 (default), Sonnet 5.5, Opus 5.5, Fable 5.2 |
| Chats | Not saved by default |
| Runs on | Hyprland only |

## Keys

**In the popup**

| Key | Does |
|---|---|
| `1`–`3` | Run that action (before you've typed anything) |
| `↑` `↓` or `Tab` | Move between actions |
| `Enter` | Run the highlighted action |
| `Esc` or click outside | Close |
| Typing | Adds words to the search; `[text]` stands for your selection |
| The keybind again | Moves the popup to the pointer |

**In the Claude chat**

| Key | Does |
|---|---|
| `Enter` | Send |
| `Shift + Enter` | New line |
| `Esc` | Close an open menu, then the chat |
| Click outside | Close the chat |

## Commands

| Command | Does |
|---|---|
| `wla show` | Open the popup at the pointer (what the keybinding runs) |
| `wla close` | Close the popup |
| `wla settings` | Open the Settings app |
| `wla daemon` | Start the background process without opening the popup |
| `wla stop` | Stop the background process |

Logs: `journalctl --user -t wayland-assistant`

## Requirements

- Hyprland (tested on 0.57)
- Quickshell 0.3 or newer
- `wl-clipboard`, `jq`, `curl`, `python3`, `xdg-utils`
- Optional: [Claude Code](https://docs.anthropic.com/en/docs/claude-code), for Ask Claude

## Features

### The popup

- Opens at the pointer, next to the text you selected.
- Reads your selection directly, without copying it, so your clipboard and clipboard history are never touched.
- Shows the start of your selection, so you can see what you're about to act on.
- Every action has a number key; the whole list works from the keyboard.

### Web search

- The search box starts as ` [text]` with the cursor in front, so you can type around your selection: `what does [text] mean`.
- Delete `[text]` to search only what you typed.
- Brave, DuckDuckGo, Google, Startpage, or your own URL with `%s` where the query goes.
- Opens in your default browser.

### Translate

- Google Translate can show the translation inside the popup, without opening a browser.
- DeepL, or Google in the browser, opens in a tab instead.
- Pick the language to translate into in Settings.

### Ask Claude

A chat card opens where the popup was, with your selection already quoted.

- **Model and effort:** a control in the footer switches the model (Haiku 5.5, Sonnet 5.5, Opus 5.5, Fable 5.2) and the effort level (Low to Max).
- **Permissions:** choose how much Claude may do, from *Chat only* (it can only talk) to *Allow everything*. See [Permission modes](#permission-modes).
- **Allow and Deny:** when Claude wants to use a tool, the request appears in the chat with Allow and Deny buttons.
- **Stop:** while Claude works, the send button turns into a stop button.
- **Ask again:** asking about another selection while the chat is open adds it to the same conversation. If Claude is still working, it waits its turn.
- **New chat:** the `+` button starts over.
- **Sandboxed commands:** Claude Code's command sandbox is always on.
- **Not saved by default:** chats are gone once closed. Turn saving on in Settings.
- **Stays out of the way:** floats above your windows, drags by its title row, closes when you click elsewhere.

### Settings app

A separate *Wayland Assistant Settings* app in your launcher:

- Search engine, or a custom search URL
- Translation service, where translations show, and the target language
- Whether Claude chats are saved
- The keybinding: press the keys to record it, and it's checked against your existing Hyprland binds before saving

## Installing

Install steps arrive with the first release. It will be a single script you can re-run to update, with an `--uninstall` option that removes everything it added.

## How it works

### Permission modes

| Mode | What Claude can do |
|---|---|
| Chat only | Talk. No files, commands, edits, web or connectors. |
| Ask first | Asks before every edit and command. |
| Auto-accept edits | Edits files without asking; asks before commands. |
| Plan only | Reads and plans, changes nothing. |
| Auto | A safety check approves routine actions. |
| Pre-approved only | Runs only what your Claude Code settings already allow; refuses the rest. |
| Allow everything | Runs every edit and command without asking. Commands stay sandboxed. |

*Chat only* is Wayland Assistant's own mode, not one of Claude Code's. Claude Code starts with no tools, no MCP connectors and no slash commands, and any tool request that still arrives is refused. Switching into or out of it restarts the session: a saved chat carries on where it left off, while an unsaved one starts a new conversation and tells you so.

Turning *Chat only* on while Claude is working stops the turn, withdraws any Allow and Deny buttons still on screen, and holds your next message until the tool-free session is ready, so nothing reaches the old one.

Changing the model, or switching between the other modes, takes effect immediately, mid-conversation. If Claude Code turns a change down, the menu goes back to the setting that's really in effect, with a note saying why.

### Privacy

- Your selection is read only when you press the key, and only to show it and act on it.
- Ask Claude hands your selection to the chat in a file only you can read, in your private runtime folder. The chat deletes it as soon as it's read; one left behind by a chat that failed to start is deleted after 10 minutes.
- Chats aren't saved unless you turn saving on.
- Nothing is sent anywhere except to the search engine, translation service, or Claude, when you choose that action.

### What the installer changes

- Copies the app to `~/.local/share/wayland-assistant` and links `~/.local/bin/wla`.
- Adds the keybinding to your Hyprland keybindings file, after checking nothing else uses it.
- Adds one layer rule, so Hyprland's own animation doesn't fight the popup's.
- Adds the Settings app to your launcher.
- Starts the popup's background process now and at every login.
- Reloads Hyprland.

Every line it adds to your config sits between `wayland-assistant:begin` and `:end` markers, and each file is backed up to `~/.local/state/wayland-assistant/backups/` before it's edited. Re-running it updates the app and restarts the background process on the new version. `--uninstall` stops the process and removes everything it added.

### Resource use

One small background process keeps the popup ready, so it appears about 60 ms after the keypress instead of half a second. While idle it has no window, uses no CPU, and holds about 30 MB of its own memory. Each press builds a fresh popup and throws it away on close, so nothing carries over between uses. If the background process isn't running, the next press starts it; only that press takes the full half second. The Claude chat lives only as long as its card is open, and closing it ends the Claude Code session it started. On hybrid-graphics laptops the popup goes straight to the GPU the compositor runs on, skipping a slow probe of the other one.

## License

All rights reserved. © 2026 [parasite6](https://github.com/parasite6)
