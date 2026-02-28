# HP LaserJet Professional M1217nfw MFP - Mac Scanner Driver

[HP never released scanner drivers for this MFP beyond macOS Mojave (10.14)](https://h30434.www3.hp.com/t5/Printer-Setup-Software-Drivers/my-new-Mac-OS-will-not-connect-to-HP-LaserJet-M1217-nfw-MFP/m-p/8960975/highlight/true#M304071). From what I found online, [it seems people have given up trying to use it](https://www.reddit.com/r/printers/comments/qc72gb/hp_m1217nfw_scanner_issues/). But I was able to install the legacy scanner drivers from a backup archive onto a newer Mac. These instructions describe how. I currently have it working with MacOS Tahoe. YMMV.

## Prerequisites

- If your Mac has Apple Silicon, **Rosetta 2** must be installed (the driver binary is Intel x86_64 only):
  ```bash
  # Check if Rosetta 2 is installed
  test -d /Library/Apple/usr/share/rosetta && echo "Installed" || echo "Not installed"

  # Install if needed
  softwareupdate --install-rosetta --agree-to-license
  ```
- The printer portion of the MFP should be set up first (it works with Apple's built-in AirPrint support).

## Driver Archive Contents

The file `HP_M1217_scanner_drivers.tar.gz` contains:

| Path | Purpose |
|------|---------|
| `Library/Image Capture/Devices/HP M1130_M1210 Scanner.app` | Main scanner driver (Image Capture/TWAIN). Discovers the scanner via Bonjour on port 8290 and also supports USB. |
| `Library/Printers/hp/laserjet/M1130_1210series/` | Raster print bundle and LaserJet utility app. |
| `Library/Printers/Icons/Hewlett_Packard HP LaserJet Professional M1217nfw MFP.icns` | Device icon. |

## Installation

**Important:** Place `HP_M1217_scanner_drivers.tar.gz` in your home directory (`~`) before running the commands below. The extraction command expects it there.

```bash
# 1. Extract driver files to the correct system locations
sudo tar xzf ~/HP_M1217_scanner_drivers.tar.gz -C /

# 2. Remove quarantine flag so Gatekeeper doesn't block the old driver
sudo xattr -rd com.apple.quarantine "/Library/Image Capture/Devices/HP M1130_M1210 Scanner.app"

# 3. Reboot (or log out and back in)
```

## Verify

After rebooting, open **Image Capture** (or **Preview > File > Import from Scanner**). The scanner should appear as an available device.

## Troubleshooting

- **Scanner not detected**: Make sure the MFP is on the same network and reachable. The scanner driver uses Bonjour (`_printer._tcp.`) to discover the device on port 8290.
- **Driver won't load**: Check Console.app for errors. If the ICADevices framework has changed in your macOS version, the driver may be incompatible.
- **SIP blocking file placement**: If `/Library/Image Capture/Devices/` is protected, boot into Recovery Mode (hold Command+R on Intel, or hold power button on Apple Silicon) and disable SIP with `csrutil disable`, install the files, then re-enable SIP with `csrutil enable`.

## Origin

With Claude Code doing the real work, I extracted these drivers from a 2016 MacBook Pro running macOS Monterey (12.7.6). The scanner app bundle was originally built against the macOS 10.7 SDK and signed on August 28, 2017.
