# Installation Guide

This guide covers multiple ways to install KlipperSearch on your Klipper printer.

## Prerequisites

- A working Klipper installation with Mainsail
- SSH access to your Raspberry Pi (or other host)
- Basic familiarity with the Linux command line

## Installation Methods

Choose the method that best fits your situation:

| Method | Best For | Difficulty |
|--------|----------|------------|
| [Method 1: Build on Pi](#method-1-build-on-raspberry-pi) | Most users | Medium |
| [Method 2: Build locally, copy to Pi](#method-2-build-locally-and-copy) | Users with slow Pi or Mac/Linux dev machine | Easy |
| [Method 3: Pre-built release](#method-3-pre-built-release) | Quickest installation | Easy |

---

## Method 1: Build on Raspberry Pi

Build directly on your printer's Raspberry Pi.

### Step 1: Install Node.js

SSH into your Raspberry Pi:

```bash
ssh pi@YOUR_PRINTER_IP
```

Install Node.js 20:

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -
sudo apt-get install -y nodejs
```

Verify installation:

```bash
node --version   # Should show v20.x.x
npm --version    # Should show 10.x.x
```

### Step 2: Backup Existing Mainsail

```bash
# Find where Mainsail is installed
ls -la ~/mainsail/

# Create a backup
cp -r ~/mainsail ~/mainsail.backup
```

### Step 3: Clone and Build

```bash
# Clone the repository
cd ~
git clone -b develop https://github.com/mikehatch/mainsail.git mainsail-search
cd mainsail-search

# Install dependencies (this may take a few minutes on Pi)
npm ci

# Build the project (this may take 2-5 minutes on Pi)
npm run build
```

### Step 4: Deploy

```bash
# Copy built files to Mainsail directory
cp -r dist/* ~/mainsail/
```

### Step 5: Clear Browser Cache

Open your Mainsail URL in a browser and do a hard refresh:
- **Windows/Linux**: `Ctrl+Shift+R`
- **Mac**: `Cmd+Shift+R`

Or clear your browser cache manually.

---

## Method 2: Build Locally and Copy

Build on your development machine (Mac/Linux/Windows with WSL) and copy to the Pi.

### Step 1: Build on Your Computer

```bash
# Clone the repository
git clone -b develop https://github.com/mikehatch/mainsail.git
cd mainsail

# Install dependencies
npm ci

# Build
npm run build
```

### Step 2: Copy to Raspberry Pi

```bash
# Replace PRINTER_IP with your printer's IP address
scp -r dist/* pi@PRINTER_IP:~/mainsail/
```

If your Mainsail is in a different location:

```bash
# First, find where Mainsail is installed
ssh pi@PRINTER_IP "find /home -name 'mainsail' -type d 2>/dev/null"

# Then copy to that location
scp -r dist/* pi@PRINTER_IP:/path/to/mainsail/
```

### Step 3: Clear Browser Cache

Hard refresh your browser (`Ctrl+Shift+R` or `Cmd+Shift+R`).

---

## Method 3: Pre-built Release

Download a pre-built release (if available).

### Step 1: Download Release

```bash
ssh pi@PRINTER_IP

# Download the latest release
cd ~
wget https://github.com/mikehatch/mainsail/releases/latest/download/mainsail.zip

# Extract
unzip mainsail.zip -d mainsail-new
```

### Step 2: Deploy

```bash
# Backup existing installation
cp -r ~/mainsail ~/mainsail.backup

# Copy new files
cp -r ~/mainsail-new/* ~/mainsail/
```

### Step 3: Clear Browser Cache

Hard refresh your browser.

---

## Verification

After installation, verify the search feature is working:

1. Open Mainsail in your browser
2. Look for the **magnifying glass icon** in the top navigation bar (next to the emergency stop button)
3. Click it or press `Ctrl+Shift+F` to open the search dialog
4. Type a search term (e.g., "stepper" or "extruder")
5. Click a result to open the file at that line

---

## Troubleshooting

### Search icon doesn't appear

- Make sure Klipper is connected (the icon only shows when Klipper is connected)
- Clear your browser cache and do a hard refresh
- Check browser console for JavaScript errors (F12 > Console)

### npm ci fails with memory errors

The Raspberry Pi may run out of memory during installation. Try:

```bash
# Increase swap space temporarily
sudo dphys-swapfile swapoff
sudo sed -i 's/CONF_SWAPSIZE=.*/CONF_SWAPSIZE=2048/' /etc/dphys-swapfile
sudo dphys-swapfile setup
sudo dphys-swapfile swapon

# Then retry npm ci
npm ci

# Restore original swap size after build
sudo sed -i 's/CONF_SWAPSIZE=.*/CONF_SWAPSIZE=100/' /etc/dphys-swapfile
sudo dphys-swapfile setup
```

### Build fails

Make sure you have Node.js 20 or higher:

```bash
node --version
```

If not, reinstall Node.js using the instructions in Method 1.

### Search returns no results

- Make sure you're connected to a printer with config files
- Check that the config files are in the `config` root directory in Moonraker
- Try searching for a term you know exists (like "stepper")

### Old Mainsail version still showing

Your browser is caching the old version. Try:

1. Hard refresh: `Ctrl+Shift+R` (Windows/Linux) or `Cmd+Shift+R` (Mac)
2. Clear browser cache completely
3. Try incognito/private browsing mode
4. Check the version in Mainsail's about dialog (should be different from official)

---

## Updating

To update to the latest version:

```bash
cd ~/mainsail-search  # or wherever you cloned the repo

# Pull latest changes
git pull origin develop

# Rebuild
npm ci
npm run build

# Deploy
cp -r dist/* ~/mainsail/
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

To find your Mainsail installation:

```bash
# Check nginx config
cat /etc/nginx/sites-enabled/mainsail 2>/dev/null | grep root

# Or search for it
find /home -name "mainsail" -type d 2>/dev/null
```

---

## Support

If you encounter issues:

1. Check the [Troubleshooting](#troubleshooting) section above
2. Open an issue at [github.com/mikehatch/mainsail/issues](https://github.com/mikehatch/mainsail/issues)
3. Include:
   - Your Raspberry Pi model
   - Node.js version (`node --version`)
   - Any error messages from the build or browser console
