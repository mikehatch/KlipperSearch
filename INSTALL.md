# Installation Guide

This guide will help you install KlipperSearch on your Klipper printer.

## Prerequisites

- A working Klipper installation with Mainsail
- SSH access to your Raspberry Pi (or other host)
- Basic familiarity with the Linux command line

---

## Installation

### Step 1: Backup Existing Mainsail

Before installing, create a backup of your current Mainsail installation:

```bash
ssh pi@YOUR_PRINTER_IP
cp -r ~/mainsail ~/mainsail.backup
```

### Step 2: Download and Extract

Download the latest release and extract it:

```bash
cd ~
wget https://github.com/mikehatch/KlipperSearch/releases/latest/download/mainsail.zip
unzip mainsail.zip -d mainsail-new
```

### Step 3: Deploy

Copy the new files to your Mainsail directory:

```bash
sudo cp -r mainsail-new/* ~/mainsail/
```

**If your Mainsail is in a different location**, find it first:

```bash
# Check nginx config for the path
cat /etc/nginx/sites-enabled/mainsail 2>/dev/null | grep root

# Or search for it
find /home -name "mainsail" -type d 2>/dev/null

# Then copy to that location
sudo cp -r mainsail-new/* /path/to/mainsail/
```

### Step 4: Clear Browser Cache

Open your Mainsail URL in a browser and do a hard refresh:
- **Windows/Linux**: `Ctrl+Shift+R`
- **Mac**: `Cmd+Shift+R`

Or clear your browser cache manually.

---

## Verification

After installation, verify the search feature is working:

1. Open Mainsail in your browser
2. Look for the **magnifying glass icon** 🔍 in the top navigation bar (next to the emergency stop button)
3. Click it or press `Ctrl+Shift+F` to open the search dialog
4. Type a search term (e.g., "stepper" or "extruder")
5. Click a result to open the file at that line

---

## Troubleshooting

### Search icon doesn't appear

- Make sure Klipper is connected (the icon only shows when Klipper is connected)
- Clear your browser cache and do a hard refresh
- Check browser console for JavaScript errors (F12 > Console)

### Search returns no results

- Make sure you're connected to a printer with config files
- Check that the config files are in the `config` root directory in Moonraker
- Try searching for a term you know exists (like "stepper")

### Old Mainsail version still showing

Your browser is caching the old version. Try:

1. Hard refresh: `Ctrl+Shift+R` (Windows/Linux) or `Cmd+Shift+R` (Mac)
2. Clear browser cache completely
3. Try incognito/private browsing mode
4. Check the version in Mainsail's about dialog

### wget or unzip not found

Install the missing tools:

```bash
sudo apt-get update
sudo apt-get install -y wget unzip
```

---

## Updating

To update to the latest version, repeat the installation steps:

```bash
# Download latest release
cd ~
wget https://github.com/mikehatch/KlipperSearch/releases/latest/download/mainsail.zip
unzip -o mainsail.zip -d mainsail-new

# Deploy
sudo cp -r mainsail-new/* ~/mainsail/
```

Then clear your browser cache.

---

## Reverting to Official Mainsail

If you want to go back to the official Mainsail:

### Option 1: Restore from backup

```bash
rm -rf ~/mainsail
mv ~/mainsail.backup ~/mainsail
```

### Option 2: Reinstall via KIAUH

```bash
cd ~/kiauh
./kiauh.sh
# Select "Install" > "Mainsail"
```

---

## Common Installation Paths

| OS/Distribution | Typical Mainsail Path |
|-----------------|----------------------|
| MainsailOS | `/home/pi/mainsail/` |
| FluiddPi (with Mainsail) | `/home/pi/mainsail/` |
| Manual Install | Varies - check your nginx config |
| Docker | Mounted volume - check docker-compose.yml |

---

## Building from Source

If you prefer to build from source instead of using the pre-built release, see the [Development Guide](DEVELOPMENT.md) for detailed build instructions.

---

## Support

If you encounter issues:

1. Check the [Troubleshooting](#troubleshooting) section above
2. Open an issue at [github.com/mikehatch/KlipperSearch/issues](https://github.com/mikehatch/KlipperSearch/issues)
3. Include:
   - Your Raspberry Pi model
   - Browser and version
   - Any error messages from the browser console (F12)
