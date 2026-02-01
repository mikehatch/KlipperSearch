# KlipperSearch

A global configuration search feature for [Mainsail](https://github.com/mainsail-crew/mainsail), the popular web interface for [Klipper](https://www.klipper3d.org/) 3D printer firmware.

## The Problem

Klipper configurations can span multiple files - `printer.cfg`, macro files, hardware configs, and more. Finding a specific setting like `rotation_distance` or `pressure_advance` means manually opening each file and searching, which is tedious and time-consuming.

## The Solution

KlipperSearch adds a global search feature to Mainsail that:

- Searches across **all** `.cfg` and `.conf` files in your config directory
- Shows results with **filename, line number, and matching text**
- **Opens the file directly** at the matching line when you click a result
- **Caches file contents** for fast subsequent searches

![Search Dialog](docs/search-demo.png)

## Features

- **Search Icon** in the top navigation bar
- **Keyboard Shortcut**: `Ctrl+Shift+F` (or `Cmd+Shift+F` on Mac)
- **Real-time Search**: Results appear as you type (minimum 2 characters)
- **Highlighted Matches**: Search terms are highlighted in results
- **Direct Navigation**: Click a result to open the file in the editor at that exact line
- **File Caching**: Files are cached for faster repeated searches

## Installation

```bash
# On your Raspberry Pi
ssh pi@YOUR_PRINTER_IP

# Backup your current Mainsail
cp -r ~/mainsail ~/mainsail.backup

# Download and install KlipperSearch
wget https://github.com/mikehatch/KlipperSearch/releases/latest/download/mainsail.zip
unzip mainsail.zip -d mainsail-new
sudo cp -r mainsail-new/* ~/mainsail/
```

Then clear your browser cache (`Ctrl+Shift+R` or `Cmd+Shift+R`).

For detailed instructions, troubleshooting, and reverting to official Mainsail, see [INSTALL.md](INSTALL.md).

## Usage

1. **Open the search dialog** by clicking the magnifying glass icon in the top bar, or press `Ctrl+Shift+F`

2. **Type your search query** (minimum 2 characters)

3. **Browse results** showing filename, line number, and matching text

4. **Click a result** to open the file in the editor at that line

## How It Works

KlipperSearch uses a client-side search approach:

1. Fetches the list of config files from Moonraker's file API
2. Downloads and caches file contents
3. Performs text search in the browser
4. Navigates to results using Mainsail's built-in editor

This approach requires no backend modifications - it works with any existing Moonraker installation.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      Mainsail UI                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │           ConfigSearchDialog.vue                 │   │
│  │  - Search input with debouncing                  │   │
│  │  - Results list with highlighting                │   │
│  │  - Keyboard navigation                           │   │
│  └─────────────────────────────────────────────────┘   │
│                         │                               │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Vuex Search Store                   │   │
│  │  - File list retrieval                          │   │
│  │  - Content fetching & caching                   │   │
│  │  - Search logic                                 │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                    Moonraker API                        │
│  GET /server/files/list?root=config                     │
│  GET /server/files/config/{filename}                    │
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                   Config Files                          │
│  printer.cfg, macros.cfg, hardware/*.cfg, etc.          │
└─────────────────────────────────────────────────────────┘
```

## Development

Want to contribute or set up a development environment? See [DEVELOPMENT.md](DEVELOPMENT.md) for:

- File structure and architecture details
- Development environment setup
- Testing with virtual or physical printers
- Code quality guidelines
- Contribution workflow

## License

This project inherits the [GNU General Public License v3.0](LICENSE) from Mainsail.

## Acknowledgments

- [Mainsail](https://github.com/mainsail-crew/mainsail) - The excellent Klipper web interface this builds upon
- [Moonraker](https://github.com/Arksine/moonraker) - The API server that makes this possible
- [Klipper](https://www.klipper3d.org/) - The 3D printer firmware

## Related Projects

- [mainsail-crew/mainsail](https://github.com/mainsail-crew/mainsail) - Original Mainsail project
- [Arksine/moonraker](https://github.com/Arksine/moonraker) - Moonraker API server
- [Klipper3d/klipper](https://github.com/Klipper3d/klipper) - Klipper firmware
