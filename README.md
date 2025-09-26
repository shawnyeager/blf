# blf - Blue Light Filter

A fast, hardware-level blue light filter control for external monitors using DDC/CI.

## Features

- **Hardware control** - Uses your monitor's built-in blue light filter (not software gamma)
- **Fast operation** - Optimized DDC/CI commands with caching
- **Persistent state** - Remembers your settings between sessions
- **Universal compatibility** - Works with any Linux desktop environment
- **Portable** - Easy configuration for different monitor models

## Requirements

- `ddcutil` - DDC/CI communication tool
- External monitor with DDC/CI blue light filter support
- `libnotify` (optional) - For desktop notifications

### Installation

```bash
# Install ddcutil
sudo pacman -S ddcutil         # Arch Linux
sudo apt install ddcutil       # Debian/Ubuntu

# Add user to i2c group (may be required)
sudo usermod -a -G i2c $USER
sudo modprobe i2c-dev

# Install blf
curl -O https://raw.githubusercontent.com/shawnyeager/blf/main/blf
chmod +x blf
sudo mv blf /usr/local/bin/
```

## Configuration

Before using, configure your monitor's VCP code:

1. **Find your monitor's blue light VCP code:**
   ```bash
   ddcutil capabilities --verbose
   ```
   Look for manufacturer-specific codes (usually 0xE0-0xFF range)

2. **Reference resources for VCP codes:**
   - **ddcutil VCP feature codes**: https://www.ddcutil.com/vcpinfo_output/
   - **ArchWiki DDC section**: https://wiki.archlinux.org/title/Backlight#DDC/CI_(external_monitors)

3. **Edit the script configuration:**
   ```bash
   # Open blf in your editor and modify these variables:
   VCP_CODE="0xE6"    # Change to your monitor's code
   I2C_BUS="17"       # Optional: set your I2C bus for faster access
   INVERT_VALUES=1    # Set to 0 if your monitor logic is inverted
   ```

4. **Find your I2C bus (optional but recommended):**
   ```bash
   ddcutil detect
   ```

## Usage

```bash
blf 5           # Set blue light filter to level 5
blf toggle      # Toggle between off (0) and max (10)
blf up          # Increase blue light filtering by 1 level
blf down        # Decrease blue light filtering by 1 level
blf status      # Show current level
blf sync        # Sync with monitor (if manually changed via OSD)
```

**Blue Light Levels:**
- `0` = Blue light filter OFF (normal screen colors)
- `10` = Blue light filter at MAX (warmest colors, most blue light removed)

## Hyprland Integration

Add these bindings to your `~/.config/hypr/hyprland.conf`:

### Basic Controls
```bash
# Blue Light Filter Controls
bindld = SHIFT, XF86AudioPrev, Decrease blue light filtering, exec, blf down
bindld = SHIFT, XF86AudioPlay, Toggle blue light on/off, exec, blf toggle  
bindld = SHIFT, XF86AudioNext, Increase blue light filtering, exec, blf up

# Quick Presets
bindld = CTRL SHIFT, XF86AudioPrev, Blue light filter off, exec, blf 0
bindld = CTRL SHIFT, XF86AudioPlay, Medium blue light filtering, exec, blf 5
bindld = CTRL SHIFT, XF86AudioNext, Maximum blue light filtering, exec, blf 10
```

### Alternative Key Bindings
If your F7/F8/F9 are mapped differently:

```bash
# Using F10/F11/F12
bindld = SHIFT, F10, Decrease blue light filtering, exec, blf down
bindld = SHIFT, F11, Toggle blue light on/off, exec, blf toggle
bindld = SHIFT, F12, Increase blue light filtering, exec, blf up

# Using Super key combinations
bindld = SUPER, F7, Decrease blue light filtering, exec, blf down
bindld = SUPER, F8, Toggle blue light on/off, exec, blf toggle
bindld = SUPER, F9, Increase blue light filtering, exec, blf up
```

## Supported Monitors

Successfully tested on:
- Asus ProArt PA27JCV (VCP code: 0xE6)

*Add your monitor model and VCP code via pull request!*

## Troubleshooting

### Monitor not detected
```bash
# Check if ddcutil can see your monitor
ddcutil detect

# Try with sudo if permission issues
sudo ddcutil detect

# Ensure i2c modules are loaded
sudo modprobe i2c-dev
```

### Blue light control not working
```bash
# Find the correct VCP code for your monitor
ddcutil capabilities --verbose

# Test different codes manually
ddcutil setvcp 0xE6 50  # Try different codes: 0xE0, 0xE1, etc.
```

### Keybindings not working in Hyprland
```bash
# Check what keys your F7/F8/F9 actually produce
wev  # Install with: sudo pacman -S wev (Arch) or sudo apt install wev (Ubuntu)

# Use the actual keysym names shown by wev
bindld = SHIFT, YourActualKeysym, Description, exec, blf toggle
```

### Performance issues
```bash
# Set your I2C bus number in the script for faster access
ddcutil detect  # Find your bus number
# Edit blf script: I2C_BUS="17"  # Use your bus number
```

## How it Works

1. **Hardware Control**: Uses DDC/CI protocol to communicate directly with your monitor's blue light filter
2. **Smart Caching**: Maintains persistent state to avoid slow DDC reads on every operation  
3. **Optimized Commands**: Uses `--noverify` and `--sleep-multiplier` for faster response
4. **Universal Design**: Configurable VCP codes and scaling for different monitor models

## Contributing

1. Fork the repository
2. Test with your monitor model
3. Add your monitor's VCP code to the supported list
4. Submit a pull request

## License

MIT License - See LICENSE file for details.

---

**Why hardware control?** Software blue light filters (like `redshift`, `wlsunset`) apply color correction at the GPU level. Hardware control uses your monitor's built-in blue light filter, providing better color accuracy and per-monitor control in multi-display setups.
