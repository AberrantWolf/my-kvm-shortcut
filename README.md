# Dell U4021QW input switching

The monitor accepts DDC/CI input-select commands (VCP `0x60`). The keyboard is on the
monitor's KVM hub, so it only reaches the computer currently on screen. Each machine
therefore needs one hotkey: "switch to the other one".

| Input | VCP 0x60 code | Used by |
|---|---|---|
| DisplayPort-1 | `0x0f` (15) | Linux |
| HDMI-1 | `0x11` (17) | — |
| HDMI-2 | `0x12` (18) | — |
| Thunderbolt/USB-C | `0x19` (25) | Mac (assumed; check with `monitor-input get` on the Mac) |

## Linux (KDE Plasma 6, Wayland) — done

- `ddcutil` installed (its udev rule gives your session access to `/dev/i2c-*`, so no sudo is needed).
- `monitor-input` is symlinked into `~/.local/bin`.
- **Meta+Ctrl+M** → `monitor-input mac`, via `linux/monitor-to-mac.desktop` copied to `~/.local/share/applications/`
  and the `[services][monitor-to-mac.desktop]` entry in `~/.config/kglobalshortcutsrc`.
  You can change it in System Settings → Keyboard → Shortcuts → "Switch Monitor to Mac".

## macOS (Apple Silicon)

1. Install the DDC tool and the script:
   ```bash
   brew install m1ddc
   ```
   ```bash
   install -m 755 mac/monitor-input /usr/local/bin/monitor-input
   ```
2. With the Mac on screen, check the code: `monitor-input get` should print `25`.
   If it prints something else, set `MAC_INPUT` to that value in both scripts.
3. Test it: `monitor-input linux` should switch the screen to the Linux box.
4. Add a hotkey (no daemon needed):
   - Open **Shortcuts** → New Shortcut → add **Run Shell Script** with `/usr/local/bin/monitor-input linux`.
   - Name it "Switch Monitor to Linux". In the shortcut's details (ⓘ), set **Add Keyboard Shortcut**
     to **Ctrl+Cmd+M** (the same keys as Meta+Ctrl+M on Linux).
   - The first time it runs, macOS may ask to allow it. Allow it.

   Prefer Hammerspoon? Add this to `~/.hammerspoon/init.lua`:
   ```lua
   hs.hotkey.bind({"ctrl", "cmd"}, "M", function()
     hs.execute("/usr/local/bin/monitor-input linux")
   end)
   ```

Notes:
- Intel Macs: use `ddcctl -d 1 -i 15` in place of `m1ddc set input 15`.
- m1ddc can't send DDC through the built-in HDMI port on some M1/M2 Macs. The Thunderbolt/USB-C cable to the monitor works.
- GUI alternatives if you'd rather not script it: BetterDisplay or Lunar (both can switch inputs with hotkeys).
