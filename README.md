# code-cursor-nix

> Auto-updating Nix Flake for [Cursor](https://cursor.com) - The AI Code Editor with Full Browser Automation Support

[![Update Cursor](https://github.com/jacopone/code-cursor-nix/actions/workflows/update.yml/badge.svg)](https://github.com/jacopone/code-cursor-nix/actions/workflows/update.yml)

## Features

- 🌐 **Browser Automation Ready**: Chrome bundled in FHS environment for seamless browser testing
- 🚀 **Auto-updating**: Checks for new Cursor releases 3x weekly via GitHub Actions
- 📦 **Multi-platform**: Supports Linux (x86_64, aarch64) and macOS (x86_64, aarch64)
- ⚡ **Fast**: New versions available within 48 hours of official release
- 🔐 **Reliable**: Automatic hash verification and build testing
- 🤖 **Automated PRs**: Creates and auto-merges PRs when tests pass
- 🔧 **Zero Configuration**: Browser automation works out of the box on NixOS

## 🎯 Browser Automation Support (Linux Only)

This flake includes **full browser automation support** for Cursor on NixOS Linux, solving a common pain point for developers who use browser testing tools like Playwright, Puppeteer, or Selenium.

**Note:** Browser automation support is currently only available on Linux (x86_64-linux, aarch64-linux). macOS users will need to install Chrome separately.

### The Problem on NixOS

On standard NixOS installations, browser automation typically fails because:
- Browsers aren't available in the isolated environment
- Path resolution issues prevent Cursor from finding Chrome/Chromium
- Complex manual configuration is usually required

### Our Solution (Linux)

On Linux systems, we've bundled Google Chrome directly in the FHS (Filesystem Hierarchy Standard) environment, making browser automation **work out of the box** with zero configuration needed.

### What This Enables (Linux Only)

- ✅ **Playwright** browser testing works immediately
- ✅ **Puppeteer** automation runs without setup
- ✅ **Selenium** WebDriver functions properly
- ✅ **Web scraping** tools work seamlessly
- ✅ **AI coding assistants** can interact with browsers for testing
- ✅ **Screenshot testing** and visual regression testing
- ✅ **End-to-end testing** frameworks run without issues

### Technical Implementation (Linux)

The Chrome integration is achieved through:
1. **FHS Environment**: Chrome is included in the AppImage's FHS sandbox (Linux only)
2. **Environment Variables**: `CHROME_BIN` and `CHROME_PATH` are automatically set
3. **Path Resolution**: Chrome is accessible at standard locations expected by automation tools
4. **No System Pollution**: Chrome remains isolated within Cursor's environment

### Example Use Cases

```javascript
// Playwright - works immediately
const { chromium } = require('playwright');
const browser = await chromium.launch(); // Just works!

// Puppeteer - no configuration needed
const puppeteer = require('puppeteer');
const browser = await puppeteer.launch(); // Finds Chrome automatically

// Selenium - standard setup works
const { Builder } = require('selenium-webdriver');
const driver = await new Builder().forBrowser('chrome').build();
```

## Installation

### NixOS (Flakes)

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

### Try without installing

```bash
nix run github:jacopone/code-cursor-nix
```

## Usage

After installation, launch Cursor from your application menu or run:

```bash
cursor
```

## Version Management

This flake automatically tracks the latest stable version of Cursor.

### Using Releases

We provide tagged releases for version stability:

```nix
# Latest release (recommended)
inputs.code-cursor-nix.url = "github:jacopone/code-cursor-nix";

# Specific version
inputs.code-cursor-nix.url = "github:jacopone/code-cursor-nix/v2.0.34";
```

View all releases: https://github.com/jacopone/code-cursor-nix/releases

### Updating

To manually update your flake:

```bash
# Update the flake lock
nix flake update code-cursor-nix

# Rebuild your system
sudo nixos-rebuild switch --flake .
```

## How It Works

1. **3x weekly checks**: GitHub Actions runs Monday, Wednesday, Friday at 9:00 UTC
2. **Version detection**: Queries Cursor's official API: `https://api2.cursor.sh/updates/api/download/stable/{platform}/cursor`
3. **Multi-platform support**: Downloads and verifies hashes for all supported platforms
4. **Automated testing**: Builds the package on Linux x86_64 and runs flake checks
5. **Pull requests**: Creates PR with updates, auto-merges if tests pass
6. **Release tagging**: Automatically creates GitHub releases for version tracking
7. **Browser integration**: Chrome is bundled in FHS environment for seamless browser automation
8. **Branch cleanup**: Automatically deletes merged branches to keep repository clean

## Manual Updates

To manually trigger an update:

```bash
# Clone the repository
git clone https://github.com/jacopone/code-cursor-nix
cd code-cursor-nix

# Run the update script
./scripts/update-version.sh

# Test the build
nix build .#cursor
```

## Comparison with Other Approaches

| Method | Update Speed | Browser Automation | Reliability | Platforms |
|--------|-------------|-------------------|-------------|-----------|
| **code-cursor-nix** | 3x weekly | ✅ **Chrome on Linux** | Automated testing | Linux, macOS |
| nixpkgs (code-cursor) | Days to weeks | ❌ Manual setup needed | Manual review | Linux, macOS |
| omarcresp/cursor-flake | Manual updates | ❌ Not included | Community-driven | Linux, macOS |
| Direct AppImage | Immediate | ❌ Complex configuration | Manual | Linux only |

## Why This Matters for NixOS Users

### The NixOS Browser Testing Challenge

NixOS users have historically faced significant challenges with browser automation:
- Standard browser testing tutorials don't work on NixOS
- Requires extensive configuration and workarounds
- Often involves impure solutions that break reproducibility
- AI coding assistants struggle to help with browser-based tasks

### This Flake Solves It (Linux)

With this flake, NixOS Linux users get the same seamless browser automation experience as users on other platforms:
- **No manual Chrome/Chromium installation required** (Linux only)
- **No complex shell environments or nix-shell configurations**
- **AI assistants in Cursor can now effectively help with browser testing**
- **Maintains NixOS purity** - Chrome is isolated within the FHS environment

**macOS users:** You'll need to install Chrome separately for browser automation support.

### Perfect for AI-Assisted Development (Linux)

On Linux, Cursor's AI capabilities can fully leverage browser automation:
- Generate and run Playwright tests directly
- Debug web applications with AI assistance
- Create end-to-end test suites with AI guidance
- Perform web scraping tasks through natural language

## Troubleshooting

### Browser automation not working (Linux only)

**Problem**: Playwright/Puppeteer tests fail with "browser not found" errors

**Solution**: The Chrome binary should be automatically available in the FHS environment on Linux systems. Verify:

```bash
# Inside a shell with cursor available
echo $CHROME_BIN
echo $CHROME_PATH
```

Both should point to the Chrome binary. If not, the FHS environment may not be properly configured.

**macOS users**: Browser automation requires separate Chrome installation. This feature is Linux-only.

### Cursor won't start on NixOS

**Problem**: Application fails to launch or shows library errors

**Solution**: This usually indicates the FHS environment is not working correctly:

1. Try rebuilding: `nix build .#cursor --rebuild`
2. Check if AppImage tools are available: `nix-shell -p appimage-run`
3. Open an issue with your NixOS version and error message

### Update script fails

**Problem**: `./scripts/update-version.sh` fails with API errors

**Solution**: Cursor's API may be temporarily unavailable or rate-limiting:

1. Wait a few minutes and retry
2. Check https://cursor.com/changelog for latest version
3. The automatic workflow will retry on the next scheduled run

### Hash mismatch during build

**Problem**: Build fails with "hash mismatch" error

**Solution**: Cursor may have updated the binary without changing the version:

```bash
# Manually update hashes
./scripts/update-version.sh
```

Or wait for the next automatic update (runs 3x weekly).

## Project Structure

```
code-cursor-nix/
├── flake.nix              # Main flake configuration with overlay
├── package.nix            # Package derivation with FHS + Chrome
├── scripts/
│   └── update-version.sh  # Auto-update script
├── .github/
│   └── workflows/
│       ├── update.yml     # Auto-update workflow (3x weekly)
│       ├── release.yml    # Automatic release tagging
│       └── cleanup-branches.yml  # Branch cleanup automation
└── README.md
```

## Maintainers

- [@jacopone](https://github.com/jacopone)

## Related Projects

- [antigravity-nix](https://github.com/jacopone/antigravity-nix) - Auto-updating Google Antigravity agentic IDE
- [claude-code-nix](https://github.com/sadjow/claude-code-nix) - Auto-updating Claude Code CLI
- [nixpkgs](https://github.com/NixOS/nixpkgs) - Official Nix packages collection

## Inspired By

This project is inspired by [sadjow/claude-code-nix](https://github.com/sadjow/claude-code-nix), which provides similar auto-updating functionality for Claude Code.

## Contributing

Contributions are welcome! Feel free to:

- Report issues
- Submit pull requests
- Improve documentation
- Add platform support

## License

MIT License - see [LICENSE](LICENSE) for details.

Cursor itself is proprietary software. This repository only provides a Nix packaging wrapper.

## Maintainers

- [@jacopone](https://github.com/jacopone)

## See Also

- [Cursor Official Site](https://cursor.com)
- [Cursor Changelog](https://www.cursor.com/changelog)
- [claude-code-nix](https://github.com/sadjow/claude-code-nix) - Similar auto-updating flake for Claude Code
