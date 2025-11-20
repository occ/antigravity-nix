# antigravity-nix

Auto-updating Nix Flake packaging for Google Antigravity.

[![Update Antigravity](https://github.com/jacopone/antigravity-nix/actions/workflows/update.yml/badge.svg)](https://github.com/jacopone/antigravity-nix/actions/workflows/update.yml)
[![Flake Check](https://img.shields.io/badge/flake-check%20passing-success)](https://github.com/jacopone/antigravity-nix)
[![NixOS](https://img.shields.io/badge/NixOS-ready-blue?logo=nixos)](https://nixos.org)

## Overview

This flake provides Google Antigravity for NixOS systems with:

- **Automated updates**: Browser-based version detection with 3x weekly checks
- **FHS environment**: Standard Linux filesystem layout via `buildFHSEnv`
- **Multi-platform support**: Linux (x86_64, aarch64) and macOS (x86_64, aarch64)
- **Chrome integration**: Bundled wrapper for system Chrome with user profile support
- **Chromium fallback**: Automatically uses Chromium on `aarch64-linux` where Google Chrome isn't available
- **Version pinning**: Tagged releases for reproducible builds
- **Zero configuration**: All dependencies included

## Installation

### Quick Start

```bash
nix run github:jacopone/antigravity-nix
```

### NixOS Configuration

Add to your `flake.nix`:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    antigravity-nix = {
      url = "github:jacopone/antigravity-nix";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { self, nixpkgs, antigravity-nix, ... }: {
    nixosConfigurations.your-hostname = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [
        {
          environment.systemPackages = [
            antigravity-nix.packages.x86_64-linux.default
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
    antigravity-nix = {
      url = "github:jacopone/antigravity-nix";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { self, nixpkgs, home-manager, antigravity-nix, ... }: {
    homeConfigurations.your-user = home-manager.lib.homeManagerConfiguration {
      pkgs = nixpkgs.legacyPackages.x86_64-linux;
      modules = [
        {
          home.packages = [
            antigravity-nix.packages.x86_64-linux.default
          ];
        }
      ];
    };
  };
}
```

### Using the Overlay

```nix
{
  nixpkgs.overlays = [
    inputs.antigravity-nix.overlays.default
  ];

  environment.systemPackages = with pkgs; [
    google-antigravity
  ];
}
```

## Usage

Launch from application menu or command line:

```bash
antigravity
```

Open a specific project:

```bash
antigravity /path/to/project
```

## Version Management

### Pinning Versions

Use tagged releases for stability:

```nix
# Latest release (recommended)
inputs.antigravity-nix.url = "github:jacopone/antigravity-nix";

# Specific version
inputs.antigravity-nix.url = "github:jacopone/antigravity-nix/v1.11.2-6251250307170304";
```

View all releases: https://github.com/jacopone/antigravity-nix/releases

### Updating

```bash
# Update flake lock
nix flake update antigravity-nix

# Rebuild system
sudo nixos-rebuild switch --flake .
```

## Implementation Details

### Packaging Approach

Antigravity is distributed as a binary that expects a standard Linux filesystem layout. NixOS uses a non-standard structure (`/nix/store`), requiring special handling:

1. **antigravity-unwrapped**: Extracts upstream tarball without modification
2. **FHS Environment**: Wraps binary in isolated container with standard paths and all required libraries

### Auto-Update System

The flake implements automated version tracking:

1. **Scheduled checks**: GitHub Actions runs Monday, Wednesday, Friday at 9:00 UTC
2. **Browser automation**: Playwright scrapes version from JavaScript-rendered download page
3. **Hash verification**: Downloads and verifies SHA256 hashes for all platforms
4. **Build testing**: Validates package builds successfully before creating PR
5. **Auto-merge**: Merges PR when tests pass
6. **Release tagging**: Creates GitHub releases for version pinning

### Chrome Integration

Creates a Chrome wrapper that:
- Forces use of user's Chrome profile (`~/.config/google-chrome`)
- Preserves installed extensions
- Sets `CHROME_BIN` and `CHROME_PATH` environment variables

## Requirements

- NixOS or Nix package manager with flakes enabled
- `allowUnfree = true` in Nix configuration (Antigravity is proprietary software)
- System browser:
  - `x86_64-linux`: `google-chrome-stable`
  - `aarch64-linux`: `chromium`

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

### Missing Libraries

The FHS environment provides all necessary libraries. If issues persist:

1. Check NixOS version: `nixos-version`
2. Rebuild: `nix build .#default --rebuild`
3. Open issue with error details and system architecture

## Project Structure

```
antigravity-nix/
├── flake.nix              # Main flake configuration with overlay
├── package.nix            # Package derivation with FHS environment
├── scripts/
│   ├── scrape-version.js  # Playwright-based version scraper
│   ├── check-version.sh   # Quick version comparison
│   └── update-version.sh  # Full update process
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
3. Test with `nix build` and `nix flake check`
4. Submit pull request

## License

MIT License - see [LICENSE](LICENSE) for details.

Google Antigravity is proprietary software by Google LLC.

## Related Projects

- [code-cursor-nix](https://github.com/jacopone/code-cursor-nix) - Auto-updating Cursor AI editor
- [claude-code-nix](https://github.com/sadjow/claude-code-nix) - Auto-updating Claude Code CLI
- [nixpkgs](https://github.com/NixOS/nixpkgs) - Official Nix packages collection

## Disclaimer

This is an unofficial package. Google Antigravity is a trademark of Google LLC. This flake is not affiliated with or endorsed by Google.
