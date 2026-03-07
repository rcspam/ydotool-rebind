# ydotool-rebind

A wrapper for `ydotool` that translates keyboard input from non-QWERTY layouts to QWERTY, allowing proper text input with AZERTY, QWERTZ, and other keyboard layouts.

This is a fork of [david-vct/ydotool-rebind](https://github.com/david-vct/ydotool-rebind), which originally supported French AZERTY only. This fork adds multi-layout support (German, Belgian, Italian, Spanish) and can select the right layout from your system configuration. Other layouts can be [easily added](#adding-a-new-layout).

> **Note:** If you're starting a new project, consider [dotool](https://git.sr.ht/~geb/dotool) which natively supports keyboard layouts via `DOTOOL_XKB_LAYOUT`. ydotool-rebind is intended for users who already have `ydotool` in their workflow.

## What is this?

`ydotool` is a Linux keyboard/mouse automation tool that internally uses QWERTY layout regardless of your system keyboard layout. This wrapper automatically translates input to QWERTY before passing it to `ydotool`.

**Example (French AZERTY):**

- Input: `ydotool type "Bonjour"`
- Without wrapper: Types `Vonjout`
- With wrapper: Types `Bonjour`

## Supported layouts

| Layout | Description | Key differences |
|--------|-------------|-----------------|
| `fr`   | French AZERTY | a/q, z/w, m swaps, accents |
| `de`   | German QWERTZ | z/y swap, umlauts, sharp s |
| `be`   | Belgian AZERTY | Similar to FR, different number row |
| `it`   | Italian | QWERTY-based, accented vowels on special keys |
| `es`   | Spanish | QWERTY-based, ñ, ç, ¡/¿, dead keys for accents |
| `us`   | US QWERTY | Passthrough (no translation) |

> Only the `fr` layout has been tested on real hardware. Other layouts (de, be, it, es) were built from XKB symbol files and need community testing. Contributions and bug reports welcome!

Missing your layout? You can [add it yourself](#adding-a-new-layout) — it only takes a few lines.

## Installation

**Requirements:** `ydotool` installed, Bash 4.0+, root access

### Debian/Ubuntu (.deb)

Download the latest `.deb` from the [releases page](https://github.com/rcspam/ydotool-rebind/releases):

```bash
sudo dpkg -i ydotool-rebind_2.0.0_all.deb
```

### Other distributions (tar.gz)

Download the latest `tar.gz` from the [releases page](https://github.com/rcspam/ydotool-rebind/releases):

```bash
tar xzf ydotool-rebind-2.0.0.tar.gz
cd ydotool-rebind-2.0.0
sudo ./install.sh
```

### Uninstall

```bash
# .deb
sudo dpkg -r ydotool-rebind

# tar.gz
sudo ./uninstall.sh
```

## Configuration

The layout is detected automatically by cascade:

1. `YDOTOOL_LAYOUT` environment variable
2. `/etc/ydotool-rebind/config` file (`LAYOUT=fr`)
3. `setxkbmap` auto-detection (X11)
4. `localectl` auto-detection (systemd)
5. Fallback: `fr`

To change the default layout:

```bash
# Edit config file
sudo nano /etc/ydotool-rebind/config
# Set: LAYOUT=de

# Or use environment variable
YDOTOOL_LAYOUT=de ydotool type "Hallo Welt"
```

## Usage

After installation, use `ydotool` normally:

```bash
# Types correctly with your keyboard layout
ydotool type "Bonjour, ça va ?"

# File mode
ydotool type -f /path/to/file.txt

# Other commands work as usual
ydotool key Return
ydotool mousemove 100 100
```

## How it works

1. Wrapper intercepts all `ydotool` commands
2. For `type` commands: loads the keyboard layout mapping and translates text
3. Passes translated result to real `ydotool`

## Supported characters

- French accents: e, e, e, a, u, c (circumflex, diaeresis)
- German umlauts: a, o, u, ss
- Ligatures: ae, oe
- All layout-specific symbols and key positions

## Adding a new layout

You can add support for any keyboard layout by creating a layout file in `layouts/`:

1. Create `layouts/xx.sh` (where `xx` is the XKB layout code)
2. Define a `KEYMAP` array mapping each character to its QWERTY equivalent
3. Only map characters that differ from QWERTY — unmapped characters pass through unchanged
4. Use `/usr/share/X11/xkb/symbols/xx` as reference for key positions

Example for a minimal layout:

```bash
#!/bin/bash
# Example layout: only map what differs from QWERTY
KEYMAP=(
    ['z']='y'     # if z and y are swapped
    ['y']='z'
    ['ñ']=';'     # special character at QWERTY ; position
)
```

For dead key accents, use multi-character values. The value is sent directly to ydotool without re-translation:

```bash
    ['â']='[q'    # dead_circumflex (QWERTY [) + a (QWERTY q on AZERTY)
```

Test with: `YDOTOOL_LAYOUT=xx ydotool type "test text"`

After installation, copy your layout to `/etc/ydotool-rebind/layouts/`.

Pull requests for new layouts are welcome!

## Debug

```bash
DEBUG=1 ydotool type "test"
# Log: /tmp/ydotool-translate-debug.log
```

## License

MIT License - see [LICENSE](LICENSE)
