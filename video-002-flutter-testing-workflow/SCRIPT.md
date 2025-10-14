# Video Script: Flutter Testing Workflow with Automated Scripts

## Video Title
"My Flutter Testing Workflow: 3 Bash Scripts That Save Hours Every Week"

## Video Description
Learn my complete Flutter testing workflow using 3 powerful bash scripts: automated app runner, phone connection monitor, and cache cleanup tool. Perfect for developers managing multiple Flutter projects.

**Timestamps:**
0:00 - Introduction
1:30 - Overview of the 3 Scripts
3:45 - Script 1: Flutter App Runner Demo
12:20 - Script 2: Phone Connection Monitor
17:30 - Script 3: Cache Cleanup Tool
22:00 - Combined Workflow Demo
26:30 - Installation & Setup
30:00 - Tips & Best Practices
33:00 - Conclusion

---

## Introduction (0:00 - 1:30)

**[Screen: Intro animation/channel logo]**

Hey everyone, Dimitar here! Today I'm sharing my complete Flutter testing workflow that I use daily to manage multiple Flutter projects efficiently.

**[Screen: Show terminal with scripts running]**

If you're like me and work on multiple Flutter projects, you've probably experienced these pain points:
- Manually selecting projects and devices every time
- Typing long flutter run commands
- Dealing with dropped Android wireless connections
- Running out of disk space from build cache

**[Screen: Show disk space before/after]**

I've automated all of this with 3 simple bash scripts that have literally saved me hours every week.

**[Screen: List the 3 scripts]**

Today, I'll show you:
1. An automated Flutter app runner with scrcpy and DevTools
2. A phone connection monitor for wireless debugging
3. A cache cleanup tool that frees gigabytes of space

Let's dive in!

---

## Overview of the 3 Scripts (1:30 - 3:45)

**[Screen: File structure view]**

Here's my setup. All my Flutter projects live in one parent directory:

```
/flutter/
├── run_flutter_app.sh          ← App runner
├── phone_connection_monitor.sh  ← Connection monitor
├── cleanup_flutter_cache.sh     ← Cache cleanup
├── project-1/
├── project-2/
└── project-3/
```

**[Highlight each script]**

This structure is important - the scripts need to be in the parent directory because they scan subdirectories for Flutter projects.

**[Screen: Quick demo montage]**

Let me show you what each script does in action, then we'll dive deep into each one.

**[Show 5-second clips of each script]**

1. App runner: One command, automatically selects project, device, starts scrcpy, opens DevTools
2. Phone monitor: Watches your wireless ADB connection, auto-reconnects when dropped
3. Cache cleanup: One command, frees gigabytes of disk space from all projects

**[Screen: Benefits list]**

Why I love this workflow:
- No more typing long commands
- Automatic screen mirroring for Android
- DevTools opens automatically
- Never lose wireless connection
- Keep disk space under control

Alright, let's start with the app runner!

---

## Script 1: Flutter App Runner Demo (3:45 - 12:20)

**[Screen: Terminal full screen]**

This is the script I use most - probably 10-20 times a day.

### Basic Usage (4:00 - 6:00)

**[Type command]**

```bash
./run_flutter_app.sh
```

**[Screen: Script output]**

Look at this! It automatically:
1. Scans for all Flutter projects
2. Shows me a numbered list

**[Show numbered list on screen]**

```
Available Flutter projects:
  1) my_app
  2) another_app
  3) test_project
```

Let me select project 2...

**[Type: 2]**

**[Screen: Device selection]**

Now it detects all my devices:

```
Available devices:
  1) iPhone 15 Pro (ios-simulator)
  2) My Android Phone (android-arm64)
```

I'll select my physical Android device...

**[Type: 2]**

**[Screen: scrcpy starting]**

Watch this - it automatically starts scrcpy for screen mirroring!

**[Screen: Split view - terminal on left, scrcpy mirror on right]**

See? My phone screen appears in a window. This is perfect for recording demos or testing without looking down at your phone.

**[Screen: Flutter building]**

And now Flutter is building and running...

**[Screen: DevTools opening in Chrome]**

**[Highlight the DevTools tab opening]**

And there! DevTools opened automatically in Chrome. No manual URL copying, no searching for the link - it just opens.

**[Screen: Full setup - terminal, scrcpy, DevTools]**

This is my complete testing setup:
- Left: Terminal with Flutter logs
- Center: scrcpy mirror showing the app
- Right: DevTools for debugging

All from one command!

### Quick Selection Mode (6:00 - 7:30)

**[Screen: Back to terminal]**

But it gets better. Once you know your project and device numbers, you can skip the interactive mode:

**[Type command]**

```bash
./run_flutter_app.sh 3 1
```

**[Show fast execution]**

Boom! Project 3, Device 1, instantly running. No questions asked.

**[Screen: Add --debug flag]**

Want debug mode with hot reload?

```bash
./run_flutter_app.sh 3 1 --debug
```

**[Screen: Show help]**

There's also a help command:

```bash
./run_flutter_app.sh --help
```

**[Show help output]**

All the options are documented here.

### Key Features Explanation (7:30 - 10:00)

**[Screen: Show project scanning code]**

Let me explain how this works under the hood.

**Feature 1: Smart Project Detection**

**[Highlight the find command]**

The script uses `find` to scan for all `pubspec.yaml` files in subdirectories, then filters out non-Flutter projects like `.git`, `build`, etc.

**[Screen: Show device parsing]**

**Feature 2: Device Management**

The script parses `flutter devices` output and handles all device types:
- iOS simulators
- Android emulators
- Physical devices (USB)
- Wireless ADB devices (IP:port)

**[Screen: Show scrcpy code]**

**Feature 3: Automatic scrcpy**

For physical Android devices, it automatically starts scrcpy. For emulators or iOS, it skips this step.

**[Screen: Show DevTools detection code]**

**Feature 4: DevTools Auto-Launch**

The script monitors Flutter's output for the DevTools URL, then opens it in Chrome after a 3-second delay for stability.

### Cleanup on Exit (10:00 - 11:00)

**[Screen: Terminal with running app]**

Here's something important - when you press Ctrl+C...

**[Press Ctrl+C]**

**[Show cleanup messages]**

```
Cleaning up...
Stopped scrcpy
```

The script automatically kills scrcpy and cleans up. No orphaned processes.

### Real-World Example (11:00 - 12:20)

**[Screen: Show typical development session]**

Here's how I use this in real development:

```bash
# Morning: Start working on client app
./run_flutter_app.sh 5 2

# Make changes, test, Ctrl+C when done

# Afternoon: Switch to personal project
./run_flutter_app.sh 2 1 --debug

# Make changes with hot reload, test

# Evening: Test on iOS simulator
./run_flutter_app.sh 2 3
```

**[Screen: Time saved calculation]**

Without this script:
- Select project path: 10 seconds
- Type flutter run command: 5 seconds
- Start scrcpy manually: 10 seconds
- Find and open DevTools URL: 15 seconds
- Total: 40 seconds per run

With this script: 2 seconds for quick selection mode

**[Show calculation]**
If I run apps 15 times a day: 40 seconds × 15 = 10 minutes saved daily
That's 50 minutes per week, 3+ hours per month!

---

## Script 2: Phone Connection Monitor (12:20 - 17:30)

**[Screen: New terminal window]**

Alright, let's talk about wireless ADB debugging.

### The Problem (12:40 - 13:30)

**[Screen: Show wireless ADB setup]**

I love wireless debugging - no cables, freedom to move around. But there's one huge problem...

**[Screen: Show disconnection]**

The connection drops. A lot. WiFi hiccups, phone goes to sleep, network changes - boom, disconnected.

**[Screen: Show manual reconnection]**

Then you have to:
1. Notice it disconnected
2. Find the phone's IP
3. Type `adb connect IP:5555`
4. Restart your Flutter app

Annoying, right?

### The Solution (13:30 - 15:00)

**[Screen: Show phone_connection_monitor.sh]**

This script solves that. It continuously monitors your phone's ADB connection and auto-reconnects when it drops.

**[Type command]**

```bash
./phone_connection_monitor.sh 192.168.1.251
```

**[Screen: Show monitoring output]**

```
Starting phone connection monitor...
Target phone IP: 192.168.1.251
Target port: 5555
Check interval: 5s

Press Ctrl+C to stop monitoring
========================================
✓ Phone connected via ADB!
```

**[Screen: Keep script running]**

Now let me disconnect the phone...

**[Screen: Simulate disconnection - turn off phone WiFi]**

**[Show script output]**

```
Phone not reachable on network (192.168.1.251)
Phone disconnected. Running script #1
→ Running system info check...
```

See? It immediately detects the disconnection and runs diagnostic scripts while waiting.

**[Screen: Reconnect phone WiFi]**

Now I'll reconnect...

**[Show auto-reconnection]**

```
Phone reachable on network (192.168.1.251) but not connected via ADB
Attempting to connect to 192.168.1.251:5555...
✓ Phone connected via ADB!
Connection restored. Exiting monitor.
```

Boom! Auto-reconnected. The Flutter app keeps running without interruption.

### How It Works (15:00 - 16:30)

**[Screen: Show script logic]**

The script runs in a loop every 5 seconds:

1. **Check ADB connection**: Looks for device in `adb devices`
2. **If disconnected**:
   - Ping the phone IP to test network
   - If reachable: Run `adb connect`
   - If not reachable: Report and wait
3. **While waiting**: Run diagnostic scripts (disk space, memory, etc.)
4. **When connected**: Exit successfully

**[Screen: Highlight status colors]**

- **Green** ✓: Connected via ADB
- **Yellow** ⚠: Reachable on network, not connected
- **Red** ✗: Not reachable on network

### Daily Usage (16:30 - 17:30)

**[Screen: Show workflow]**

Here's how I use this:

**[Show 2 terminal windows side by side]**

**Terminal 1:**
```bash
# Start phone monitor
./phone_connection_monitor.sh 192.168.1.251
```

**Terminal 2:**
```bash
# Run Flutter app
./run_flutter_app.sh 5 2
```

**[Screen: Show working session]**

Now I can work all day without worrying about connection drops. The monitor runs in the background, and if my phone disconnects, it automatically reconnects before I even notice.

**[Screen: Show Ctrl+C cleanup]**

When I'm done for the day:
- Ctrl+C in app runner (kills app and scrcpy)
- Ctrl+C in monitor (stops monitoring)

Clean and simple!

---

## Script 3: Cache Cleanup Tool (17:30 - 22:00)

**[Screen: Show disk space warning]**

Let's talk about disk space. Flutter projects accumulate a LOT of cache files.

### The Problem (17:50 - 18:30)

**[Screen: Show large build folders]**

Each Flutter project has:
- `build/` folder: 200-500MB
- `.dart_tool/`: 50-100MB
- `ios/Pods/`: 100-200MB
- `android/.gradle/`: 50-150MB
- And more...

**[Show calculation on screen]**

One project: ~500MB-1GB of cache
10 projects: 5-10GB of cache
20 projects: 10-20GB of cache!

**[Screen: Show disk space low warning]**

All of this is regenerable. But manually deleting it from each project? Tedious.

### The Solution (18:30 - 20:00)

**[Screen: Terminal]**

One command cleans everything:

```bash
./cleanup_flutter_cache.sh
```

**[Screen: Show cleanup running]**

Watch this:

```
🧹 Starting Flutter cache cleanup in: /Users/me/flutter
================================================

🔍 Finding and deleting build folders...
  ✅ Deleted: /flutter/project1/build (234MB)
  ✅ Deleted: /flutter/project2/build (189MB)
  ✅ Deleted: /flutter/project3/build (312MB)

🔍 Finding and deleting .dart_tool folders...
  ✅ Deleted: /flutter/project1/.dart_tool (45MB)
  ✅ Deleted: /flutter/project2/.dart_tool (38MB)

🔍 Finding and deleting iOS Pods folders...
  ✅ Deleted: /flutter/project1/ios/Pods (156MB)
  ✅ Deleted: /flutter/project2/ios/Pods (142MB)

🔍 Finding and deleting Android .gradle folders...
  ✅ Deleted: /flutter/project1/android/.gradle (89MB)
```

**[Let it run through all deletions]**

**[Show final summary]**

```
================================================
🎉 Cleanup complete!
💾 Total space freed: 4.82GB (4936MB)

ℹ️  To regenerate these files when needed:
   - Run: flutter pub get
   - Run: flutter build [ios/android/web]
   - iOS: cd ios && pod install
================================================
```

**[Highlight the total]**

Almost 5GB freed! And this was just from 8 projects.

### What Gets Deleted (20:00 - 21:00)

**[Screen: Show list of deleted items]**

The script removes:

**Flutter:**
- `build/` - Build outputs
- `.dart_tool/` - Dart analyzer cache
- `.flutter-plugins*` - Plugin configs
- `.packages` - Package configs

**iOS:**
- `ios/Pods/` - CocoaPods
- `ios/DerivedData/` - Xcode cache
- `.symlinks/` - Flutter symlinks

**Android:**
- `android/.gradle/` - Gradle cache
- `android/build/` - Gradle outputs

**Cross-platform:**
- `ephemeral/` - Platform caches
- `coverage/` - Test coverage

**[Screen: Highlight safety]**

Important: It ONLY deletes regenerable files. Your source code, assets, and configs are safe.

### Regenerating Files (21:00 - 22:00)

**[Screen: Terminal]**

After cleanup, regenerate as needed:

**[Type commands]**

```bash
# Get dependencies
cd your-project
flutter pub get

# iOS pods (if needed)
cd ios
pod install

# Build (generates build folder)
flutter build ios
```

**[Screen: Show timing]**

Usually takes:
- `flutter pub get`: 10-30 seconds
- `pod install`: 1-2 minutes
- First build: 2-5 minutes

Small price to pay for gigabytes of freed space!

---

## Combined Workflow Demo (22:00 - 26:30)

**[Screen: Show all 3 terminal windows]**

Let me show you how I use all three scripts together in a real development session.

### Morning Routine (22:15 - 23:30)

**[Screen: Terminal 1]**

**Step 1: Check disk space and cleanup if needed**

```bash
df -h  # Check disk space
./cleanup_flutter_cache.sh  # If low, run cleanup
```

**[Show cleanup running fast-forward]**

**[Screen: Terminal 2]**

**Step 2: Start phone monitor**

```bash
./phone_connection_monitor.sh 192.168.1.251
```

**[Show connected status]**

**[Screen: Terminal 3]**

**Step 3: Start working on first project**

```bash
./run_flutter_app.sh 5 2 --debug
```

**[Show app launching]**

Now I have:
- Terminal 1: Available for git commands, etc.
- Terminal 2: Phone monitor (running in background)
- Terminal 3: Flutter app with hot reload
- scrcpy window: Screen mirror
- Chrome: DevTools

**[Screen: Show full workspace]**

Everything I need for productive Flutter development!

### Switching Projects (23:30 - 24:30)

**[Screen: Terminal 3]**

When I need to switch projects:

**[Press Ctrl+C]**

```
Cleaning up...
Stopped scrcpy
```

**[Type new command]**

```bash
./run_flutter_app.sh 3 1
```

**[Show quick launch]**

2 seconds later, I'm running a different project on a different device. Phone monitor is still running in Terminal 2, so no setup needed there.

### End of Day (24:30 - 25:30)

**[Screen: All terminals]**

At the end of the day:

**Terminal 3: Stop app runner**
```
Ctrl+C
```

**Terminal 2: Stop phone monitor**
```
Ctrl+C
Monitoring stopped by user
```

**[Screen: Clean desktop]**

Everything stops cleanly. No orphaned processes, no memory leaks.

### Weekly Maintenance (25:30 - 26:30)

**[Screen: Calendar showing Friday]**

Every Friday, I run the cleanup script:

```bash
./cleanup_flutter_cache.sh
```

**[Show typical result]**

```
💾 Total space freed: 8.23GB
```

**[Screen: Disk space graph]**

This keeps my disk usage stable week over week. Without this, I'd accumulate 50-100GB of cache over a month!

---

## Installation & Setup (26:30 - 30:00)

**[Screen: GitHub repository or download links]**

Let me show you how to set this up yourself.

### Prerequisites (26:45 - 27:30)

**[Screen: Checklist]**

You'll need:
- Flutter SDK (you probably have this)
- ADB for Android
- scrcpy: `brew install scrcpy` (macOS)
- bash shell (macOS/Linux have this built-in)

### Installation Steps (27:30 - 29:00)

**[Screen: Terminal]**

**Step 1: Create your Flutter projects directory**

```bash
mkdir ~/flutter
cd ~/flutter
```

**Step 2: Download the scripts**

**[Show download/copy commands]**

```bash
# Download scripts (replace with your actual URLs or use git clone)
curl -O https://your-repo.com/run_flutter_app.sh
curl -O https://your-repo.com/phone_connection_monitor.sh
curl -O https://your-repo.com/cleanup_flutter_cache.sh
```

**Step 3: Make them executable**

```bash
chmod +x run_flutter_app.sh
chmod +x phone_connection_monitor.sh
chmod +x cleanup_flutter_cache.sh
```

**Step 4: Edit cleanup script path**

```bash
# Edit cleanup script
nano cleanup_flutter_cache.sh

# Change this line to your path:
FLUTTER_DIR="/Users/your-username/flutter"
```

**Step 5: Move your Flutter projects**

```bash
# Move existing projects into this directory
mv ~/Documents/my-flutter-app ~/flutter/
mv ~/Desktop/another-app ~/flutter/
```

**[Screen: Final structure]**

Your setup should look like:

```
~/flutter/
├── run_flutter_app.sh
├── phone_connection_monitor.sh
├── cleanup_flutter_cache.sh
├── my-flutter-app/
├── another-app/
└── third-app/
```

### Optional: Shell Aliases (29:00 - 30:00)

**[Screen: Terminal]**

Make life even easier with aliases:

```bash
# Edit your shell config
nano ~/.zshrc  # or ~/.bashrc

# Add these lines:
alias frun='cd ~/flutter && ./run_flutter_app.sh'
alias fclean='cd ~/flutter && ./cleanup_flutter_cache.sh'
alias fphone='cd ~/flutter && ./phone_connection_monitor.sh'

# Save and reload
source ~/.zshrc
```

**[Show usage]**

Now from anywhere:

```bash
frun 3 1        # Run app 3 on device 1
fclean          # Clean all projects
fphone          # Monitor phone
```

Super convenient!

---

## Tips & Best Practices (30:00 - 33:00)

**[Screen: Tips list]**

Let me share some tips I've learned using these scripts daily.

### App Runner Tips (30:15 - 31:00)

**[Screen: Show examples]**

1. **Remember your favorite combos**
   ```bash
   # I use these all the time
   ./run_flutter_app.sh 5 2  # Client app, physical phone
   ./run_flutter_app.sh 5 3  # Client app, iOS simulator
   ./run_flutter_app.sh 2 1  # Personal app, emulator
   ```

2. **Use --debug for active development**
   - Hot reload works
   - Better error messages
   - Slower startup, but worth it

3. **Keep scrcpy window visible**
   - Great for recordings
   - Good for demos
   - Useful for UI testing

### Phone Monitor Tips (31:00 - 31:45)

**[Screen: Show monitoring setup]**

1. **Run in dedicated terminal**
   - Don't close it accidentally
   - Easy to check connection status
   - Keeps your workspace organized

2. **Find your phone's IP once**
   ```bash
   # Settings > About > Status > IP Address
   # Or: adb shell ip addr show wlan0
   ```

3. **Set as default in script**
   - Edit `DEFAULT_PHONE_IP` variable
   - Then just run: `./phone_connection_monitor.sh`

### Cleanup Tips (31:45 - 32:30)

**[Screen: Show cleanup schedule]**

1. **Weekly cleanup schedule**
   - Friday afternoons work well
   - After wrapping up features
   - Before weekend backup

2. **Don't clean during active work**
   - Wait until you're done for the day
   - Not in the middle of debugging
   - Not before important builds

3. **Run flutter pub get after**
   - Especially if you open projects right away
   - Or let the app runner do it automatically

### Performance Tips (32:30 - 33:00)

**[Screen: Show performance comparisons]**

1. **Use SSD for Flutter projects**
   - Build times are 2-3x faster
   - pub get is much quicker
   - Overall better experience

2. **16GB+ RAM recommended**
   - Multiple Flutter builds use memory
   - Chrome DevTools uses memory
   - Android Studio/VS Code use memory

3. **WiFi ADB vs USB**
   - WiFi: Convenient but occasional drops
   - USB: Faster, more stable
   - Use phone monitor to handle WiFi drops

---

## Conclusion (33:00 - End)

**[Screen: Summary slide]**

Alright, let's wrap up!

**[Show all 3 scripts on screen]**

We've covered three powerful scripts:
1. **App Runner** - Automated Flutter testing with scrcpy and DevTools
2. **Phone Monitor** - Never lose wireless ADB connection again
3. **Cache Cleanup** - Free gigabytes of disk space instantly

**[Screen: Time saved calculator]**

In my case, these scripts save me:
- 10 minutes per day
- 50 minutes per week
- 3+ hours per month
- 40+ hours per year!

**[Screen: Repository/download links]**

All three scripts are available:
- Link in description
- README with full documentation
- Copy-paste ready to use

**[Screen: Final demo montage]**

This workflow has transformed how I develop Flutter apps. No more manual device selection, no more connection drops, no more disk space warnings.

**[Screen: Call to action]**

If you found this helpful:
- Give it a thumbs up
- Subscribe for more Flutter content
- Drop your questions in the comments

**[Screen: What's next preview]**

Coming next: Advanced Flutter debugging techniques with DevTools - including timeline profiling, memory analysis, and performance optimization.

**[Screen: Outro]**

Thanks for watching! Happy coding, and I'll see you in the next one!

**[End screen with subscribe button and video suggestions]**

---

## B-Roll Suggestions

Throughout the video, use these visuals:
- Terminal windows with colored output
- scrcpy mirror showing app running
- DevTools interface with live data
- Disk space before/after cleanup
- Multiple projects in file explorer
- Network connection status indicators
- Clean desktop workspace setup

## Graphics/Animations Needed

- Script workflow diagrams
- Time saved calculations
- Disk space visualization
- Network connection flow
- Terminal command animations
- File structure diagrams

## Notes for Editing

- Speed up cleanup script output (show progress but don't wait for all deletions)
- Picture-in-picture for terminal + app screen
- Highlight important terminal output
- Add timestamps in description
- Show keyboard shortcuts used
- Use consistent terminal theme (dark with good contrast)
