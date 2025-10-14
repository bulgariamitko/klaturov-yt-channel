# Flutter Testing Workflow: Automated Scripts

Technical guide for setting up automated Flutter testing workflow using three bash scripts: app runner, phone connection monitor, and cache cleanup.

---

## Overview

This workflow uses three bash scripts to streamline Flutter development and testing:

1. **`run_flutter_app.sh`** - Automated Flutter app launcher with device selection and DevTools
2. **`phone_connection_monitor.sh`** - ADB connection monitor for Android devices
3. **`cleanup_flutter_cache.sh`** - Clean build artifacts and free disk space

---

## Prerequisites

### Required Tools

- Flutter SDK (stable channel)
- ADB (Android Debug Bridge)
- scrcpy (for Android screen mirroring): `brew install scrcpy` (macOS)
- bash shell

### Project Structure

All Flutter projects must be in the same parent directory where the scripts are located:

```
/flutter/
├── run_flutter_app.sh
├── phone_connection_monitor.sh
├── cleanup_flutter_cache.sh
├── project-1/
│   └── pubspec.yaml
├── project-2/
│   └── pubspec.yaml
└── project-3/
    └── pubspec.yaml
```

**Important:** Scripts scan subdirectories for Flutter projects. Keep all projects in the same folder tree.

---

## Script 1: Flutter App Runner

### Purpose

Automates the process of running Flutter apps with automatic:
- Project selection from available Flutter projects
- Device detection and selection
- scrcpy screen mirroring for Android devices
- DevTools automatic launching in Chrome

### Setup

1. Download the script to your Flutter projects directory:
   ```bash
   cd /path/to/your/flutter/projects
   curl -O https://your-repo.com/run_flutter_app.sh
   chmod +x run_flutter_app.sh
   ```

2. Ensure all Flutter projects are subdirectories of this location

### Usage

**Interactive Mode:**
```bash
./run_flutter_app.sh
```

**With Debug Flag:**
```bash
./run_flutter_app.sh --debug
```

**Quick Selection (non-interactive):**
```bash
# Select app 3 and device 1
./run_flutter_app.sh 3 1

# Select app 3, device 1, with debug mode
./run_flutter_app.sh 3 1 --debug

# Alternative syntax
./run_flutter_app.sh --3 --1
```

**Help:**
```bash
./run_flutter_app.sh --help
```

### Features

**Automatic Project Detection:**
- Scans all subdirectories for `pubspec.yaml` files
- Filters out non-Flutter projects (keys, build, .git, etc.)
- Numbered list for easy selection

**Device Management:**
- Detects all connected devices (physical + emulators)
- Parses Flutter device output
- Supports both USB and wireless ADB connections
- Handles device IDs with colons (IP:port format)

**Android Screen Mirroring:**
- Automatically starts scrcpy for physical Android devices
- Skips scrcpy for emulators
- Window title: "Flutter App Mirror"
- Cleanup on exit (Ctrl+C)

**DevTools Auto-Launch:**
- Monitors Flutter output for DevTools URL
- Opens in Chrome automatically
- Falls back to default browser if Chrome not found
- 3-second delay for stability

**Cleanup:**
- Kills scrcpy on exit
- Handles Ctrl+C gracefully
- Cleans up temporary files

### Examples

**Example 1: Interactive Mode**
```bash
$ ./run_flutter_app.sh

🚀 Flutter App Runner
====================

📱 Step 1: Select Flutter Project
Available Flutter projects:
  1) my_app
  2) another_app
  3) test_project
Select project (1-3): 2

Selected project: another_app
Project directory: /flutter/another_app

📲 Step 2: Select Target Device
Getting available devices...
Available devices:
  1) iPhone 15 Pro (ios-simulator)
  2) M2103K19PG (mobile) (android-arm64)
Select device (1-2): 2

Selected device: M2103K19PG (mobile) (android-arm64)

🖥️  Step 3: Starting Android Screen Mirroring
Starting scrcpy for Android device mirroring...
scrcpy started (PID: 12345)

🏃 Step 4: Running Flutter App
Command: flutter run -d 192.168.1.251:5555
Mode: default mode

✅ Setup Complete!
App: another_app
Device: M2103K19PG (mobile)
Screen mirroring: Active

Starting app... DevTools will open automatically when available
Press Ctrl+C to stop everything and cleanup
```

**Example 2: Quick Launch**
```bash
$ ./run_flutter_app.sh 2 2 --debug
🚀 Flutter App Runner
====================
Selected project: another_app
Selected device: M2103K19PG (mobile) (android-arm64)
Starting app in debug mode...
```

### Troubleshooting

**"No Flutter projects found"**
- Ensure Flutter projects are in subdirectories
- Check that `pubspec.yaml` contains `flutter:` entry
- Verify script is in the parent directory of projects

**"No devices found"**
- Connect a device or start an emulator
- Run `flutter devices` to verify detection
- Check USB connection or wireless ADB

**"scrcpy not found"**
- Install scrcpy: `brew install scrcpy` (macOS)
- Or continue without screen mirroring (optional feature)

**DevTools doesn't open automatically**
- Manually open the URL shown in console
- Check Chrome installation
- Verify browser permissions

---

## Script 2: Phone Connection Monitor

### Purpose

Monitors ADB connection to Android device and automatically attempts reconnection when connection is lost. Useful for wireless debugging.

### Setup

```bash
cd /path/to/your/scripts
curl -O https://your-repo.com/phone_connection_monitor.sh
chmod +x phone_connection_monitor.sh
```

### Usage

**With Default IP (192.168.1.251):**
```bash
./phone_connection_monitor.sh
```

**With Custom IP:**
```bash
./phone_connection_monitor.sh 192.168.1.100
```

**Help:**
```bash
./phone_connection_monitor.sh --help
```

### Features

**Connection Monitoring:**
- Checks ADB connection every 5 seconds
- Verifies device in `adb devices` output
- Tests network connectivity (ping)
- Auto-reconnect attempts

**Status Reporting:**
- Green: Connected via ADB
- Yellow: Reachable on network, not connected via ADB
- Red: Not reachable on network

**Auto-Reconnect:**
- Attempts `adb connect` when network is reachable
- 2-second wait between attempts
- Continuous monitoring until connected

**Diagnostic Scripts:**
- Runs system checks while disconnected
- Rotates through 5 different checks:
  - System info (uname -a)
  - Disk space (df -h)
  - Memory usage (free -h / vm_stat)
  - Network interfaces (ip addr / ifconfig)
  - Running processes (ps aux)

### Example Output

```bash
$ ./phone_connection_monitor.sh 192.168.1.251

Starting phone connection monitor...
Target phone IP: 192.168.1.251
Target port: 5555
Check interval: 5s

Press Ctrl+C to stop monitoring
========================================
Phone not reachable on network (192.168.1.251)
Phone disconnected. Running script #1
→ Running system info check...
Darwin MacBook-Pro.local 25.0.0 Darwin Kernel Version 25.0.0
----------------------------------------
Phone reachable on network (192.168.1.251) but not connected via ADB
Attempting to connect to 192.168.1.251:5555...
Phone disconnected. Running script #2
→ Checking disk space...
----------------------------------------
✓ Phone connected via ADB!
ADB devices output:
List of devices attached
192.168.1.251:5555    device

Connection restored. Exiting monitor.
```

### Use Cases

**Wireless ADB Development:**
- Monitor unstable WiFi connections
- Auto-reconnect when network drops
- Continuous development without cable

**Remote Device Testing:**
- Monitor devices on local network
- Track connection stability
- Automatic recovery from disconnections

### Troubleshooting

**"Invalid IP address format"**
- Verify IP format: `192.168.1.100`
- Use `adb devices` to find device IP
- Check device WiFi settings

**Device not connecting**
- Verify device is on same network
- Check ADB over network is enabled on device
- Try USB connection first: `adb tcpip 5555`
- Ensure port 5555 is accessible

---

## Script 3: Cache Cleanup

### Purpose

Frees disk space by removing all regenerable Flutter build artifacts, dependencies, and cache files from all projects in the directory tree.

### Setup

1. Edit the script and set your Flutter directory path:
   ```bash
   FLUTTER_DIR="/Users/your-username/Dropbox/flutter"
   ```

2. Make executable:
   ```bash
   chmod +x cleanup_flutter_cache.sh
   ```

### Usage

```bash
./cleanup_flutter_cache.sh
```

**Warning:** This script deletes files that can be regenerated but requires re-running build commands afterward.

### What Gets Deleted

**Flutter Build Artifacts:**
- `build/` - All Flutter build outputs
- `.dart_tool/` - Dart analyzer cache
- `.flutter-plugins` - Plugin configuration
- `.flutter-plugins-dependencies` - Plugin dependencies
- `.packages` - Package configuration

**iOS:**
- `ios/Pods/` - CocoaPods dependencies
- `ios/DerivedData/` - Xcode build cache
- `.symlinks/` - Flutter symlinks

**macOS:**
- `macos/Pods/` - CocoaPods dependencies
- `macos/ephemeral/` - Flutter macOS cache

**Android:**
- `android/.gradle/` - Gradle cache
- `android/build/` - Gradle build outputs

**Cross-Platform:**
- `ephemeral/` - Platform-specific Flutter cache (Windows/Linux/macOS)
- `coverage/` - Test coverage reports

### Features

**Recursive Scanning:**
- Finds all Flutter projects in directory tree
- Processes multiple projects automatically
- Skips non-Flutter directories

**Size Reporting:**
- Shows size of each deleted folder
- Calculates total freed space
- Reports in MB and GB

**Safe Deletion:**
- Only deletes regenerable files
- Preserves source code and configurations
- Keeps pubspec.yaml, assets, and lib/

**Progress Display:**
- Shows what's being deleted
- Checkmarks for successful deletions
- Error messages for failed operations

### Example Output

```bash
$ ./cleanup_flutter_cache.sh

🧹 Starting Flutter cache cleanup in: /Users/me/Dropbox/flutter
================================================

🔍 Finding and deleting build folders...
  ✅ Deleted: /flutter/project1/build (234MB)
  ✅ Deleted: /flutter/project2/build (189MB)

🔍 Finding and deleting .dart_tool folders...
  ✅ Deleted: /flutter/project1/.dart_tool (45MB)
  ✅ Deleted: /flutter/project2/.dart_tool (38MB)

🔍 Finding and deleting iOS Pods folders...
  ✅ Deleted: /flutter/project1/ios/Pods (156MB)
  ✅ Deleted: /flutter/project2/ios/Pods (142MB)

🔍 Finding and deleting Android .gradle folders...
  ✅ Deleted: /flutter/project1/android/.gradle (89MB)
  ✅ Deleted: /flutter/project2/android/.gradle (76MB)

================================================
🎉 Cleanup complete!
💾 Total space freed: 2.43GB (2489MB)

ℹ️  To regenerate these files when needed:
   - Run: flutter pub get
   - Run: flutter build [ios/android/web]
   - iOS: cd ios && pod install
================================================
```

### Regenerating Files

After cleanup, regenerate as needed:

**Dependencies:**
```bash
cd your-project
flutter pub get
```

**iOS Pods:**
```bash
cd your-project/ios
pod install
```

**Build Artifacts:**
```bash
cd your-project
flutter build ios
# or
flutter build apk
# or
flutter build web
```

### When to Run

**Good Times to Clean:**
- Before backing up projects
- When disk space is low
- After long development sessions
- Before archiving projects
- Switching between major Flutter versions

**Don't Clean:**
- In the middle of debugging
- With unsaved work
- During active builds
- Right before deployment

### Disk Space Savings

Typical savings per project:
- Small project: 200-500MB
- Medium project: 500MB-1GB
- Large project: 1-3GB

For 10+ projects, expect 5-15GB of freed space.

---

## Workflow Integration

### Combined Workflow Example

```bash
# 1. Clean up old build artifacts
./cleanup_flutter_cache.sh

# 2. Start phone connection monitor (in separate terminal)
./phone_connection_monitor.sh 192.168.1.251

# 3. Run Flutter app with automatic setup
./run_flutter_app.sh 3 1 --debug
```

### Recommended Setup

**Directory Structure:**
```
~/flutter/
├── run_flutter_app.sh
├── phone_connection_monitor.sh
├── cleanup_flutter_cache.sh
├── project-a/
├── project-b/
└── project-c/
```

**Shell Aliases (add to ~/.zshrc or ~/.bashrc):**
```bash
alias frun='cd ~/flutter && ./run_flutter_app.sh'
alias fclean='cd ~/flutter && ./cleanup_flutter_cache.sh'
alias fphone='cd ~/flutter && ./phone_connection_monitor.sh'
```

Usage after aliases:
```bash
fclean          # Clean all projects
fphone          # Monitor phone
frun 3 1        # Run app 3 on device 1
```

---

## Tips & Best Practices

### App Runner

- Use quick selection (`./run_flutter_app.sh 3 1`) for frequent project/device combos
- Enable `--debug` for hot reload during development
- Keep scrcpy window visible for UI testing
- DevTools opens automatically - bookmark for quick access

### Phone Monitor

- Run in separate terminal window
- Use default IP if you have one device
- Let it run in background during development
- Ctrl+C to stop when done

### Cache Cleanup

- Run weekly or when low on disk space
- Run before major Flutter version upgrades
- Don't run during active development
- Always run `flutter pub get` after cleanup

### Performance

- **SSD recommended** for build performance
- **16GB+ RAM** for multiple Flutter builds
- **Wireless ADB** reduces cable wear
- **scrcpy** uses minimal CPU/bandwidth

---

## Troubleshooting

### General Issues

**Script won't run:**
```bash
chmod +x script-name.sh
```

**Permission denied:**
```bash
sudo chmod +x script-name.sh
```

**Flutter not found:**
```bash
export PATH="$PATH:$HOME/flutter/bin"
```

**ADB not found:**
```bash
export PATH="$PATH:$HOME/Library/Android/sdk/platform-tools"
```

### macOS Specific

**scrcpy not found:**
```bash
brew install scrcpy
```

**Chrome not opening DevTools:**
- Install Google Chrome
- Or script will use default browser

---

## Security Notes

- Scripts don't store sensitive data
- ADB connections are local network only
- DevTools runs on localhost
- No external network access required
- All operations are local to your machine

---

## Customization

### Modify Cleanup Script

Edit `FLUTTER_DIR` path:
```bash
FLUTTER_DIR="/your/custom/path"
```

### Modify Check Interval

In `phone_connection_monitor.sh`:
```bash
CHECK_INTERVAL=5  # Change to desired seconds
```

### Modify Default Phone IP

In `phone_connection_monitor.sh`:
```bash
DEFAULT_PHONE_IP="192.168.1.100"  # Change to your device IP
```

---

## Performance Tips

**Speed up app launches:**
- Keep projects in SSD location
- Use `--debug` only when needed (release is faster)
- Close unused apps before testing

**Reduce cleanup time:**
- Run cleanup on fewer projects at once
- Exclude specific projects by moving them temporarily

**Improve scrcpy performance:**
- Reduce device resolution in scrcpy settings
- Use USB connection for lower latency
- Close other screen recording apps
