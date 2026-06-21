# macOS-Style Search — Linux (Zorin OS / GNOME, X11)

A simple keyboard shortcut that searches Google for whatever text you've highlighted in **any** application — PDF viewers, text editors, browsers, etc. — similar to macOS's "Search with Google" feature.

## How it works

The OS doesn't have a built-in "ask any app what text is selected" API like macOS does (Accessibility API). So this script takes a more direct approach:

1. You highlight text in any app
2. You press a keyboard shortcut
3. The script simulates a `Ctrl+C` keypress to copy your selection
4. It reads the clipboard
5. It opens your default browser with a Google search for that text

## Requirements

- **X11 session** (not Wayland) — this is required. Wayland blocks apps from simulating keystrokes into other apps for security reasons, so this script will not work there. Check your session type with:
```bash
  echo $XDG_SESSION_TYPE
```
  If it says `wayland`, log out and choose the X11/Xorg session option at the login screen (usually a gear icon near the password field) before continuing.

- `xdotool` and `xclip`:
```bash
  sudo apt update && sudo apt install -y xdotool xclip
```

- `python3` (used for proper URL-encoding of the search query — almost always already installed)

## Installation

**1. Create the script:**
```bash
nano ~/.local/bin/sys_search
```

**2. Paste in the following:**
```bash
#!/bin/bash
sleep 0.4
xdotool key --clearmodifiers ctrl+c
sleep 0.3

QUERY=$(xclip -o -selection clipboard 2>/dev/null)

if [ -n "$QUERY" ]; then
    ENCODED=$(python3 -c "import urllib.parse, sys; print(urllib.parse.quote(sys.argv[1]))" "$QUERY")
    xdg-open "https://www.google.com/search?q=${ENCODED}"
fi
```

**3. Save and exit:** `Ctrl+O`, `Enter`, `Ctrl+X`

**4. Make it executable:**
```bash
chmod +x ~/.local/bin/sys_search
```

**5. Bind it to a keyboard shortcut:**

Go to **Settings → Keyboard → View and Customise Shortcuts → Custom Shortcuts**, then add a new shortcut:

| Field | Value |
|---|---|
| Name | Search Selected Text |
| Command | `/home/<your-username>/.local/bin/sys_search` |
| Shortcut | `Super+G` (recommended — see notes below) |

> **Important:** use the full path (e.g. `/home/youruser/.local/bin/sys_search`), not `~/...` — GNOME's shortcut launcher does not expand `~`.

## Usage

1. Highlight text in any app (PDF viewer, text editor, etc.)
2. Press your shortcut key
3. Your default browser opens with a Google search for the highlighted text

## Notes on choosing a shortcut key

- **Avoid `Ctrl+Alt+F1`–`F12`** — these are reserved system-wide for switching virtual terminals (TTYs) and can freeze/switch your graphical session.
- **Avoid single `Alt+<letter>` combos** (e.g. `Alt+G`) — these often collide with menu mnemonics inside GTK/Qt apps and can cause the script to grab stale clipboard content instead of your current selection.
- **`Ctrl+G`** may conflict with "Find Next" in some apps' built-in search — test before relying on it.
- **`Super+G`** (tested and confirmed working) is a safe choice, as it isn't a standard app-level accelerator.

## Troubleshooting

- **Searches old/stale clipboard content instead of current selection:** Usually a modifier-key timing issue. Try increasing the first `sleep` value (e.g. from `0.4` to `0.6`).
- **Nothing happens at all:** Confirm you're on X11 (`echo $XDG_SESSION_TYPE`), confirm the script is executable, and confirm the Command field in the shortcut settings has the exact correct full path.
- **Works when you manually press `Ctrl+C` first but not automatically:** This is the Wayland restriction — switch to an X11 session.

## Limitations

- Only works reliably on **X11** sessions, not Wayland.
- Relies on the target app supporting standard `Ctrl+C` copy behavior — apps with non-standard or image-only text rendering (e.g. scanned PDFs without OCR) won't have copyable text to select.
- Currently hardcoded to Google search — can be adapted to other search engines or lookup tools by changing the URL.
