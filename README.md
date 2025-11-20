# code-cursor-nix

Auto-updating Nix Flake packaging for Cursor.

[![Update Cursor](https://github.com/jacopone/code-cursor-nix/actions/workflows/update.yml/badge.svg)](https://github.com/jacopone/code-cursor-nix/actions/workflows/update.yml)

## Overview

This flake provides Cursor for NixOS systems with:

- **Automated updates**: API-based version detection with 3x weekly checks
- **Multi-platform support**: Linux (x86_64, aarch64) and macOS (x86_64, aarch64)
- **Chrome integration (Linux only)**: Bundled browser for testing frameworks
- **FHS environment**: Standard Linux filesystem layout via AppImage extraction
- **Version pinning**: Tagged releases for reproducible builds
- **Zero configuration**: Browser automation works out of the box on Linux

## Installation

### Quick Start

```bash
nix run github:jacopone/code-cursor-nix
```

### NixOS Configuration

Add to your `flake.nix`:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    code-cursor-nix.url = "github:jacopone/code-cursor-nix";
  };

  outputs = { self, nixpkgs, code-cursor-nix, ... }: {
    nixosConfigurations.your-hostname = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [
        {
          environment.systemPackages = [
            code-cursor-nix.packages.x86_64-linux.cursor
          ];
        }
      ];
    };
  };
}
```

### Home Manager

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    home-manager.url = "github:nix-community/home-manager";
    code-cursor-nix.url = "github:jacopone/code-cursor-nix";
  };

  outputs = { self, nixpkgs, home-manager, code-cursor-nix, ... }: {
    homeConfigurations.your-user = home-manager.lib.homeManagerConfiguration {
      pkgs = nixpkgs.legacyPackages.x86_64-linux;
      modules = [
        {
          home.packages = [
            code-cursor-nix.packages.x86_64-linux.cursor
          ];
        }
      ];
    };
  };
}
```

## Usage

Launch from application menu or command line:

```bash
cursor
```

## Version Management

### Pinning Versions

Use tagged releases for stability:

```nix
# Latest release (recommended)
inputs.code-cursor-nix.url = "github:jacopone/code-cursor-nix";

# Specific version
inputs.code-cursor-nix.url = "github:jacopone/code-cursor-nix/v2.0.34";
```

View all releases: https://github.com/jacopone/code-cursor-nix/releases

### Updating

```bash
# Update flake lock
nix flake update code-cursor-nix

# Rebuild system
sudo nixos-rebuild switch --flake .
```

## Implementation Details

### Packaging Approach

Cursor is distributed as an AppImage that expects a standard Linux filesystem layout. The packaging process:

1. **AppImage extraction**: Extracts upstream AppImage without modification
2. **FHS Environment**: Wraps binary in isolated container with standard paths and required libraries
3. **Chrome integration (Linux)**: Bundles Google Chrome for browser automation frameworks

### Auto-Update System

The flake implements automated version tracking:

1. **Scheduled checks**: GitHub Actions runs Monday, Wednesday, Friday at 9:00 UTC
2. **API-based detection**: Queries Cursor's official API at `https://api2.cursor.sh/updates/api/download/stable/{platform}/cursor`
3. **Hash verification**: Downloads and verifies hashes for all platforms
4. **Build testing**: Validates package builds successfully before creating PR
5. **Auto-merge**: Merges PR when tests pass
6. **Release tagging**: Creates GitHub releases for version pinning

### Browser Automation (Linux Only)

On Linux systems, Chrome is bundled in the FHS environment for testing frameworks:

- Sets `CHROME_BIN` and `CHROME_PATH` environment variables
- Enables Playwright, Puppeteer, and Selenium without additional configuration
- Chrome remains isolated within Cursor's environment

**Note**: Browser automation support is Linux-only. macOS users need to install Chrome separately.

**Example usage:**

```javascript
// Playwright - works immediately
const { chromium } = require('playwright');
const browser = await chromium.launch();

// Puppeteer - no configuration needed
const puppeteer = require('puppeteer');
const browser = await puppeteer.launch();
```

## Requirements

- NixOS or Nix package manager with flakes enabled
- `allowUnfree = true` in Nix configuration (Cursor is proprietary software)

### Enabling Unfree Packages

**NixOS Configuration** (`configuration.nix`):

```nix
nixpkgs.config.allowUnfree = true;
```

**Flakes** (`flake.nix`):

```nix
nixpkgs = import inputs.nixpkgs {
  inherit system;
  config.allowUnfree = true;
};
```

## Troubleshooting

### Hash Mismatch Error

Upstream binary changed. Update with:

```bash
./scripts/update-version.sh
```

Or wait for automatic update (runs 3x weekly).

### Application Won't Start

Verify unfree packages are enabled:

```bash
nix-instantiate --eval -E '(import <nixpkgs> {}).config.allowUnfree'
```

Should return `true`.

### Browser Automation Not Working (Linux)

Verify Chrome environment variables are set:

```bash
echo $CHROME_BIN
echo $CHROME_PATH
```

Both should point to the Chrome binary. If empty, the FHS environment may not be configured correctly.

**macOS users**: Browser automation requires separate Chrome installation.

### Update Script Fails

Cursor's API may be temporarily unavailable or rate-limiting:

1. Wait a few minutes and retry
2. Check https://cursor.com/changelog for latest version
3. Automatic workflow will retry on next scheduled run

## Manual Updates

To manually trigger an update:

```bash
# Clone repository
git clone https://github.com/jacopone/code-cursor-nix
cd code-cursor-nix

# Run update script
./scripts/update-version.sh

# Test build
nix build .#cursor
```

## Project Structure

```
code-cursor-nix/
├── flake.nix              # Main flake configuration with overlay
├── package.nix            # Package derivation with FHS + Chrome
├── scripts/
│   └── update-version.sh  # Auto-update script
└── .github/
    └── workflows/
        ├── update.yml     # Auto-update workflow (3x weekly)
        ├── release.yml    # Automatic release tagging
        └── cleanup-branches.yml  # Branch cleanup
```

## Contributing

Contributions welcome. Please:

1. Fork repository
2. Create feature branch
3. Test with `nix build .#cursor` and `nix flake check`
4. Submit pull request

## License

MIT License - see [LICENSE](LICENSE) for details.

Cursor is proprietary software. This repository provides Nix packaging only.

## Related Projects

- [antigravity-nix](https://github.com/jacopone/antigravity-nix) - Auto-updating Google Antigravity
- [claude-code-nix](https://github.com/sadjow/claude-code-nix) - Auto-updating Claude Code CLI
- [nixpkgs](https://github.com/NixOS/nixpkgs) - Official Nix packages collection
