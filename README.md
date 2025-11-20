# antigravity-nix

> Auto-updating Nix Flake for [Google Antigravity](https://antigravity.google) - The AI-Native IDE for Autonomous Development

[![Update Antigravity](https://github.com/jacopone/antigravity-nix/actions/workflows/update.yml/badge.svg)](https://github.com/jacopone/antigravity-nix/actions/workflows/update.yml)
[![Flake Check](https://img.shields.io/badge/flake-check%20passing-success)](https://github.com/jacopone/antigravity-nix)
[![NixOS](https://img.shields.io/badge/NixOS-ready-blue?logo=nixos)](https://nixos.org)

## ✨ Features

- 🧠 **True AI Agents**: Multi-file reasoning, architectural planning, and autonomous execution
- 🚀 **Smart Auto-Updates**: Browser-based version detection with zero manual intervention
- 🎯 **NixOS Optimized**: FHS environment with system Chrome integration
- 📦 **Multi-Platform**: Linux (x86_64, aarch64) and macOS (x86_64, aarch64)
- ⚡ **Lightning Fast**: New versions within 48 hours, auto-merge when tests pass
- 🔐 **Production Ready**: Automated hash verification and comprehensive build testing
- 🎨 **Seamless Integration**: First-class NixOS and Home Manager support with overlays

## 🎯 Antigravity vs Traditional AI IDEs

Google Antigravity represents a paradigm shift from AI **assistants** to AI **agents**. While tools like Cursor provide intelligent code completion and chat interfaces, Antigravity operates as an autonomous development partner:

### **Antigravity (Agentic)**
- 🧠 **Autonomous Reasoning**: Agents independently analyze codebases, plan multi-step changes, and execute complex refactorings
- 🏗️ **Architectural Intelligence**: Understands system design, makes holistic decisions across modules
- 🔄 **Self-Directed Execution**: Can implement entire features from natural language specifications
- 🐛 **Intelligent Debugging**: Analyzes logs, traces, and system behavior to root-cause issues
- 📚 **Adaptive Learning**: Learns your team's patterns, conventions, and architectural preferences
- 🎯 **Goal-Oriented**: Given high-level objectives, breaks them into tasks and executes autonomously

### **Traditional AI IDEs (Assistant-Based)**
- 💬 Chat-based code suggestions and explanations
- ⌨️ Context-aware autocomplete and inline generation
- 🔍 File-by-file code understanding
- ❓ Requires explicit prompts and guidance for each step
- 📝 Primarily focused on writing code snippets

**Think of it this way**: Traditional AI IDEs are like a smart autocomplete that can write functions. Antigravity is like having a senior developer who can architect, implement, and debug entire features while you focus on product direction.

### The NixOS Challenge

Running binary-distributed IDEs like Antigravity on NixOS can be challenging due to:
- Non-standard filesystem layout breaking hardcoded paths
- Missing system libraries that aren't in the Nix store
- Complex runtime dependencies that require manual patching

### Our Solution

This flake uses **FHS Environment** (`buildFHSEnv`) to provide Antigravity with a standard Linux filesystem layout within an isolated container. This means:

- ✅ **Zero configuration** - Works out of the box on NixOS
- ✅ **All dependencies included** - No manual library hunting
- ✅ **Maintains purity** - Isolated from the rest of your system
- ✅ **Automatic updates** - Stay current with Google's latest releases

## Installation

### Try without installing

```bash
nix run github:jacopone/antigravity-nix
```

### NixOS (Flakes)

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

After installation, launch Antigravity from your application menu or run:

```bash
antigravity
```

Open a specific project:

```bash
antigravity /path/to/project
```

## Version Management

This flake automatically tracks the latest stable version of Google Antigravity.

### Using Releases

We provide tagged releases for version stability:

```nix
# Latest release (recommended)
inputs.antigravity-nix.url = "github:jacopone/antigravity-nix";

# Specific version
inputs.antigravity-nix.url = "github:jacopone/antigravity-nix/v1.11.2-6251250307170304";
```

View all releases: https://github.com/jacopone/antigravity-nix/releases

### Updating

To manually update your flake:

```bash
# Update the flake lock
nix flake update antigravity-nix

# Rebuild your system
sudo nixos-rebuild switch --flake .
```

## 🔧 How It Works

This flake implements a sophisticated auto-update system with browser automation:

1. **🕐 3x Weekly Checks**: GitHub Actions runs Monday, Wednesday, Friday at 9:00 UTC
2. **🌐 Smart Version Detection**:
   - Uses Playwright with system Chrome for JavaScript-rendered page scraping
   - NixOS-optimized with automatic Chrome path detection (`/run/current-system/sw/bin/google-chrome-stable`)
   - Separates logs (stderr) from version output (stdout) for clean parsing
3. **🔒 Cryptographic Verification**: Downloads and verifies SHA256 hashes for all platforms
4. **✅ Automated Testing**: Builds package and runs comprehensive flake checks
5. **🔄 Intelligent PR Management**: Creates PRs with detailed changelogs, auto-merges when tests pass
6. **🏷️ Release Tagging**: Automatically creates GitHub releases for version pinning
7. **🐚 FHS Isolation**: Provides standard Linux environment within Nix's purity model
8. **🔗 Chrome Integration**: Bundles Chrome wrapper for Antigravity's browser automation features

### Recent Improvements

**✨ November 2025 Update**: Complete rewrite of version detection system
- **Fixed browser scraping** to properly separate logs from version output
- **Added NixOS system Chrome integration** with automatic path detection
- **Improved error handling** and workflow reliability
- **Added comprehensive documentation** (CLAUDE.md) for contributors
- **Zero false positives** on version checks
- **Enhanced automation** with npm-based Playwright dependency management

## Comparison with Other Approaches

| Method | Update Speed | FHS Support | Reliability | Platforms |
|--------|-------------|-------------|-------------|-----------|
| **antigravity-nix** | 3x weekly | ✅ Built-in | Automated testing | Linux, macOS |
| Manual binary | Immediate | ❌ Manual setup | Self-managed | Linux only |
| Custom derivation | Varies | ❌ Complex patching | Community | Linux, macOS |

## Requirements

- NixOS or Nix package manager
- Nix Flakes enabled
- `allowUnfree = true` in your Nix configuration (Antigravity is proprietary software)

### Enabling Unfree Packages

Add to your `configuration.nix`:

```nix
nixpkgs.config.allowUnfree = true;
```

Or for Nix flakes, add to your `flake.nix`:

```nix
nixpkgs = import inputs.nixpkgs {
  inherit system;
  config.allowUnfree = true;
};
```

## Troubleshooting

### Antigravity won't start

**Problem**: Application fails to launch or shows permission errors

**Solution**: Ensure unfree packages are enabled:

```bash
nix-instantiate --eval -E '(import <nixpkgs> {}).config.allowUnfree'
```

Should return `true`. If not, add `nixpkgs.config.allowUnfree = true;` to your configuration.

### Build fails with hash mismatch

**Problem**: `hash mismatch` error during build

**Solution**: The upstream binary may have changed. Update with:

```bash
./scripts/update-version.sh
```

Or wait for the next automatic update (runs 3x weekly).

### Missing libraries error

**Problem**: Error messages about missing `.so` files

**Solution**: The FHS environment should provide all necessary libraries. If you still encounter issues:

1. Check your NixOS version: `nixos-version`
2. Try rebuilding: `nix build .#default --rebuild`
3. Open an issue with:
   - Full error message
   - Output of `antigravity --version` (if it partially works)
   - Your system architecture

### Application crashes or freezes

**Problem**: Antigravity crashes during operation

**Solution**:
1. Check system resources (RAM, disk space)
2. Look for relevant logs in `~/.config/antigravity/logs/`
3. Try removing cache: `rm -rf ~/.cache/antigravity/`
4. Report to [Google Antigravity support](https://antigravity.google/support)

## Project Structure

```
antigravity-nix/
├── flake.nix              # Main flake configuration with overlay
├── package.nix            # Package derivation with FHS environment
├── scripts/
│   └── update-version.sh  # Auto-update script
├── .github/
│   └── workflows/
│       ├── update.yml     # Auto-update workflow (3x weekly)
│       ├── release.yml    # Automatic release tagging
│       └── cleanup-branches.yml  # Branch cleanup automation
└── README.md
```

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Test your changes with `nix build`
4. Submit a pull request

Areas where contributions are especially appreciated:
- Additional platform support
- Build optimization
- Documentation improvements
- Bug reports and fixes

## License

MIT License - see [LICENSE](LICENSE) for details.

Google Antigravity itself is proprietary software by Google LLC.

## Maintainers

- [@jacopone](https://github.com/jacopone)

## Related Projects

- [code-cursor-nix](https://github.com/jacopone/code-cursor-nix) - Auto-updating Cursor AI editor with browser automation
- [claude-code-nix](https://github.com/sadjow/claude-code-nix) - Auto-updating Claude Code CLI
- [nixpkgs](https://github.com/NixOS/nixpkgs) - Official Nix packages collection

## Inspired By

This project follows the auto-updating patterns established by [code-cursor-nix](https://github.com/jacopone/code-cursor-nix) and [claude-code-nix](https://github.com/sadjow/claude-code-nix), providing the same seamless experience for Google Antigravity users on NixOS.

## Disclaimer

This is an unofficial package. Google Antigravity is a trademark of Google LLC. This flake is not affiliated with or endorsed by Google.
