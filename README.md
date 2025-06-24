


***THIS IS AN UNOFFICIAL BUILD SCRIPT!***

If you run into an issue with this build script, make an issue here. Don't bug Anthropic about it - they already have enough on their plates.

# Claude Desktop for Linux

This project was inspired by [k3d3's claude-desktop-linux-flake](https://github.com/k3d3/claude-desktop-linux-flake) and their [Reddit post](https://www.reddit.com/r/ClaudeAI/comments/1hgsmpq/i_successfully_ran_claude_desktop_natively_on/) about running Claude Desktop natively on Linux. Their work provided valuable insights into the application's structure and the native bindings implementation.

And now by me, via [Aaddrick's claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian), modified to work for Fedora 41.

Supports MCP!
![image](https://github.com/user-attachments/assets/93080028-6f71-48bd-8e59-5149d148cd45)

Location of the MCP-configuration file is: ~/.config/Claude/claude_desktop_config.json

Supports the Ctrl+Alt+Space popup!
![image](https://github.com/user-attachments/assets/1deb4604-4c06-4e4b-b63f-7f6ef9ef28c1)

Supports the Tray menu! (Screenshot of running on KDE)
![image](https://github.com/user-attachments/assets/ba209824-8afb-437c-a944-b53fd9ecd559)

# Installation Options

## 1. Fedora Package (New!)

For Fedora-based distributions you can build and install Claude Desktop using the provided build script (updated):

```bash
sudo dnf install rpm-build
# Clone this repository
git clone https://github.com/bsneed/claude-desktop-fedora.git
cd claude-desktop-fedora

# Install the package
# Check what file was actually created
ls build/electron-app/x86_64/

# Install using the correct filename (example - your version may differ)
sudo dnf install build/electron-app/x86_64/claude-desktop-0.10.14-1.fc40.x86_64.rpm

# Download standalone Electron
# The installed Claude Desktop will have GTK conflicts with your system Electron. Download a clean Electron binary
cd /tmp
wget https://github.com/electron/electron/releases/download/v31.0.2/electron-v31.0.2-linux-x64.zip

# Extract it
unzip electron-v31.0.2-linux-x64.zip

# Create directory and move all files
sudo mkdir -p /opt/electron
sudo cp -r /tmp/* /opt/electron/
sudo chmod +x /opt/electron/electron

# Fix the Claude Desktop Script The default script will try to use your system Electron. Replace it with one that uses the standalone version:

# Backup the original script
sudo cp /usr/bin/claude-desktop /usr/bin/claude-desktop.backup

# Create the fixed script
sudo tee /usr/bin/claude-desktop << 'EOF'
#!/usr/bin/bash
LOG_FILE="$HOME/claude-desktop-launcher.log"

# Set environment to avoid GTK conflicts
export GDK_BACKEND=x11
export GTK_USE_PORTAL=0
export ELECTRON_DISABLE_SECURITY_WARNINGS=true

# Use the standalone electron installation
/opt/electron/electron /usr/lib64/claude-desktop/app.asar --ozone-platform-hint=auto --enable-logging=file --log-file=$LOG_FILE --log-level=INFO --disable-gpu-sandbox --no-sandbox "$@"
EOF

# Make it executable
sudo chmod +x /usr/bin/claude-desktop

# Launch Claude Desktop
claude-desktop

```

Installation video here : https://www.youtube.com/watch?v=RsmBSwU_TcU

Requirements:
- Fedora 41 Linux distribution
- Node.js >= 12.0.0 and npm
- Root/sudo access for dependency installation

## 2. Debian Package (New!)

For Debian users, please refer to [Aaddrick's claude-desktop-debian](https://github.com/aaddrick/claude-desktop-debian) repository.  Their implementation is specifically designed for Debian and provides the original build script that inspired THIS project.

## 3. NixOS Implementation

For NixOS users, please refer to [k3d3's claude-desktop-linux-flake](https://github.com/k3d3/claude-desktop-linux-flake) repository. Their implementation is specifically designed for NixOS and provides the original Nix flake that inspired this project.

# How it works

Claude Desktop is an Electron application packaged as a Windows executable. Our build script performs several key operations to make it work on Linux:

1. Downloads and extracts the Windows installer
2. Unpacks the app.asar archive containing the application code
3. Replaces the Windows-specific native module with a Linux-compatible implementation
4. Repackages everything into a proper RPM package

The process works because Claude Desktop is largely cross-platform, with only one platform-specific component that needs replacement.

## The Native Module Challenge

The only platform-specific component is a native Node.js module called `claude-native-bindings`. This module provides system-level functionality like:

- Keyboard input handling
- Window management
- System tray integration
- Monitor information

Our build script replaces this Windows-specific module with a Linux-compatible implementation that:

1. Provides the same API surface to maintain compatibility
2. Implements keyboard handling using the correct key codes from the reference implementation
3. Stubs out unnecessary Windows-specific functionality
4. Maintains critical features like the Ctrl+Alt+Space popup and system tray

The replacement module is carefully designed to match the original API while providing Linux-native functionality where needed. This approach allows the rest of the application to run unmodified, believing it's still running on Windows.

## Build Process Details

> Note: The build script was generated by Claude (Anthropic) to help create a Linux-compatible version of Claude Desktop.

The build script (`build-fedora.sh`) handles the entire process:

1. Checks for a Fedora-based system and required dependencies
2. Downloads the official Windows installer
3. Extracts the application resources
4. Processes icons for Linux desktop integration
5. Unpacks and modifies the app.asar:
   - Replaces the native module with our Linux version
   - Updates keyboard key mappings
   - Preserves all other functionality
6. Creates a proper RPM package with:
   - Desktop entry for application menus
   - System-wide icon integration
   - Proper dependency management
   - Post-install configuration

## Updating the Build Script

When a new version of Claude Desktop is released, simply update the `CLAUDE_DOWNLOAD_URL` constant at the top of `build-deb.sh` to point to the new installer. The script will handle everything else automatically.

# License

The build scripts in this repository, are dual-licensed under the terms of the MIT license and the Apache License (Version 2.0).

See [LICENSE-MIT](LICENSE-MIT) and [LICENSE-APACHE](LICENSE-APACHE) for details.

The Claude Desktop application, not included in this repository, is likely covered by [Anthropic's Consumer Terms](https://www.anthropic.com/legal/consumer-terms).

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any
additional terms or conditions.
# claude-desktop-fedora
