# YouTube Video Script: Flutter Build & Upload - iOS & Android from Terminal

**Video Title:** How to Build and Upload Flutter Apps to TestFlight & Google Play from Terminal | Complete Guide 2025

**Video Length:** ~15-20 minutes

**Target Audience:** Flutter developers who want to automate app deployment

---

## [INTRO - 0:00-0:30]

**[Screen: Channel intro animation]**

**You:**
> "Hey everyone! Today I'm going to show you how to build and upload your Flutter apps to TestFlight for iOS and Google Play Console for Android - all from the terminal. No more clicking through Xcode or Android Studio. Let's automate this process!"

**[Screen: Show terminal window]**

---

## [OVERVIEW - 0:30-1:30]

**[Screen: Split screen - iOS logo left, Android logo right]**

**You:**
> "By the end of this video, you'll know how to:
> - Build production-ready iOS IPA files
> - Upload to TestFlight using command line
> - Build Android App Bundles
> - Upload to Google Play Console
> - And I'll give you ready-to-use shell scripts to automate everything"

**[Screen: Show terminal with commands queued up]**

> "Everything we do today works on macOS. For Android builds, you can also do this on Linux or Windows with some modifications. Let's get started!"

---

## [CHAPTER 1: iOS SETUP - 1:30-5:00]

**[Screen: App Store Connect website]**

### Part 1: Getting App Store Connect API Key (1:30-2:30)

**You:**
> "First, we need to set up authentication with App Store Connect. We'll use API keys instead of your Apple ID password - this is more secure and works great for automation."

**[Screen: Navigate through App Store Connect]**

**Steps to show:**
1. Open https://appstoreconnect.apple.com
2. Click "Users and Access" → "Keys" → "App Store Connect API"
3. Click the `+` button
4. Name it "Terminal Upload" or similar
5. Select "Developer" access
6. Click "Generate"
7. Download the `.p8` file

**You:**
> "Important! You can only download this key file ONCE. If you lose it, you'll have to create a new one. Also note down the Key ID and Issuer ID - we'll need these."

**[Screen: Terminal window]**

### Part 2: Installing the API Key (2:30-3:00)

**You:**
> "Now let's install this key where Xcode tools can find it."

**[Type in terminal]**
```bash
# Create the directory
mkdir -p ~/.appstoreconnect/private_keys

# Copy the key (replace with your actual path)
cp ~/Downloads/AuthKey_ABC123XYZ.p8 ~/.appstoreconnect/private_keys/
```

**You:**
> "That's it! The key is now installed."

### Part 3: Flutter Project Setup (3:00-5:00)

**[Screen: Show pubspec.yaml in editor]**

**You:**
> "Before building, we need to update our version number. Open your pubspec.yaml file."

**[Show file]**
```yaml
version: 2.6.8+28
```

**You:**
> "This format is MAJOR.MINOR.PATCH plus BUILD NUMBER. For each new upload, you must increment either the version or build number. Let's change this to:"

```yaml
version: 2.6.9+29
```

**You:**
> "I incremented both the patch version and build number. Save the file."

---

## [CHAPTER 2: BUILDING iOS - 5:00-7:30]

**[Screen: Terminal in Flutter project directory]**

**You:**
> "Now for the fun part - building! It's just three commands."

### Command 1: Clean (5:00-5:30)

**[Type in terminal]**
```bash
flutter clean
```

**You:**
> "First, clean any old build artifacts. This prevents weird caching issues."

**[Wait for completion]**

### Command 2: Get Dependencies (5:30-6:00)

**[Type in terminal]**
```bash
flutter pub get
```

**You:**
> "Next, fetch all dependencies. This ensures we have the latest packages."

**[Wait for completion]**

### Command 3: Build IPA (6:00-7:30)

**[Type in terminal]**
```bash
flutter build ipa --release
```

**You:**
> "And now we build! This command creates a production-ready IPA file. It usually takes 1-2 minutes."

**[Show time-lapse or speed up footage while build runs]**

**[When complete, show output]**
```
✓ Built build/ios/archive/Runner.xcarchive (190.0MB)
✓ Built IPA to build/ios/ipa (25.1MB)
```

**You:**
> "Perfect! Our IPA file is ready at build/ios/ipa. It's about 25 megabytes."

---

## [CHAPTER 3: UPLOADING TO TESTFLIGHT - 7:30-9:30]

**[Screen: Terminal]**

**You:**
> "Now let's upload this to TestFlight. We'll use the altool command that comes with Xcode."

**[Type in terminal - but PAUSE before executing]**
```bash
xcrun altool --upload-app \
  --type ios \
  --file build/ios/ipa/*.ipa \
  --apiKey ABC123XYZ \
  --apiIssuer YOUR_ISSUER_ID
```

**You:**
> "Let me break this down:
> - `xcrun altool` is Apple's upload tool
> - `--type ios` specifies iOS app
> - `--file` points to our IPA
> - `--apiKey` is that Key ID we noted earlier
> - `--apiIssuer` is the Issuer ID UUID
>
> Replace these with your actual keys!"

**[Execute command]**

**[Show upload progress - speed up if needed]**

**[Show success message]**
```
UPLOAD SUCCEEDED with no errors
Delivery UUID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Transferred 24209525 bytes in 2.030 seconds
```

**You:**
> "Boom! Upload successful in just 2 seconds! Now we wait about 5-10 minutes for Apple to process the build, then we can submit it for testing in App Store Connect."

**[Screen: Quick cut to App Store Connect showing the new build]**

**You:**
> "And here it is - version 2.6.9, build 29, ready for testing!"

---

## [CHAPTER 4: ANDROID SETUP - 9:30-11:30]

**[Screen: Android Studio or File Manager]**

### Part 1: Creating Keystore (9:30-10:30)

**You:**
> "For Android, we need a signing key. If you already have one, skip ahead. If not, let's create one."

**[Screen: Terminal]**

**[Type in terminal]**
```bash
keytool -genkey -v \
  -keystore ~/my-release-key.jks \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias my-key-alias
```

**You:**
> "This creates a keystore valid for 10,000 days - about 27 years. You'll be prompted for passwords and info. SAVE THESE PASSWORDS - you can't recover them!"

**[Show prompts being filled in - can skip through quickly]**

### Part 2: Configure Signing (10:30-11:30)

**[Screen: Show android/key.properties file]**

**You:**
> "Create a file called `key.properties` in your android folder with these settings:"

**[Show file content]**
```properties
storePassword=YOUR_PASSWORD
keyPassword=YOUR_PASSWORD
keyAlias=my-key-alias
storeFile=/Users/yourname/my-release-key.jks
```

**You:**
> "Replace with your actual passwords and path. Make sure this file is in .gitignore - never commit passwords to Git!"

**[Quick check of .gitignore]**

---

## [CHAPTER 5: BUILDING ANDROID - 11:30-13:00]

**[Screen: Terminal in Flutter project]**

**You:**
> "Android build is even simpler than iOS. Same three-step process."

### Commands (11:30-13:00)

**[Type and execute]**
```bash
flutter clean
flutter pub get
flutter build appbundle --release
```

**You:**
> "Notice we use `appbundle` instead of `apk`. App Bundles are Google's recommended format - they're smaller and optimize for each device."

**[Show build completion]**
```
✓ Built build/app/outputs/bundle/release/app-release.aab (15.2MB)
```

**You:**
> "Done! Our AAB file is only 15 megabytes. That's because App Bundles don't include resources for every device - Google Play generates optimized APKs for each user."

---

## [CHAPTER 6: UPLOADING TO GOOGLE PLAY - 13:00-15:00]

**[Screen: Browser - Google Play Console]**

### Manual Method (13:00-14:00)

**You:**
> "For Android, I'll show you both manual and automated methods. First, the manual way:"

**[Navigate through Play Console]**

**Steps to show:**
1. Go to play.google.com/console
2. Select your app
3. Release → Testing → Internal testing
4. Create new release
5. Upload AAB file
6. Add release notes
7. Review and rollout

**You:**
> "Pretty straightforward! But doing this every time gets tedious. Let's automate it."

### Automated Method with Fastlane (14:00-15:00)

**[Screen: Terminal]**

**You:**
> "We'll use Fastlane - a popular automation tool for mobile deployments."

**[Type]**
```bash
# Install fastlane
gem install fastlane

# Initialize fastlane in Android folder
cd android
fastlane init
```

**[Show quick setup]**

**You:**
> "You'll need to set up a Google Play service account - I've linked a detailed guide in the description. Once configured, uploading is just one command:"

**[Type]**
```bash
fastlane internal
```

**You:**
> "That's it! Fastlane handles the upload, release notes, and rollout automatically."

---

## [CHAPTER 7: AUTOMATION SCRIPTS - 15:00-17:00]

**[Screen: Text editor with shell script]**

**You:**
> "Let's wrap this all up into reusable scripts. I'll create one for iOS and one for Android."

### iOS Script (15:00-16:00)

**[Show creating upload-ios.sh]**
```bash
#!/bin/bash
set -e

# Configuration
API_KEY="YOUR_API_KEY"
ISSUER_ID="YOUR_ISSUER_ID"

echo "Building iOS release..."
flutter clean
flutter pub get
flutter build ipa --release

echo "Uploading to TestFlight..."
xcrun altool --upload-app --type ios \
  --file build/ios/ipa/*.ipa \
  --apiKey "$API_KEY" \
  --apiIssuer "$ISSUER_ID"

echo "✅ Upload complete!"
```

**[Make executable]**
```bash
chmod +x upload-ios.sh
```

**You:**
> "Now uploading to TestFlight is just: `./upload-ios.sh`. Replace the API key and Issuer ID with your actual values."

### Android Script (16:00-17:00)

**[Show creating upload-android.sh]**
```bash
#!/bin/bash
set -e

echo "Building Android release..."
flutter clean
flutter pub get
flutter build appbundle --release

echo "Uploading to Play Console..."
cd android
fastlane internal

echo "✅ Upload complete!"
```

**[Make executable]**
```bash
chmod +x upload-android.sh
```

**You:**
> "Same deal - run `./upload-android.sh` and you're done! Both scripts include error handling, so they'll stop if anything fails."

---

## [TROUBLESHOOTING - 17:00-18:30]

**[Screen: Show common errors]**

**You:**
> "Let me quickly cover the most common issues you might encounter:"

### iOS Issues (17:00-17:45)

**[Show error message on screen]**

**Issue 1:**
```
Error: Invalid Pre-Release Train. The train version 'X.X.X' is closed
```

**You:**
> "This means you forgot to increment your version number. Go to pubspec.yaml and bump the version."

**Issue 2:**
```
Error: Failed to load AuthKey file
```

**You:**
> "Your API key isn't in the right place. Make sure it's in ~/.appstoreconnect/private_keys/ and the filename matches your Key ID."

### Android Issues (17:45-18:30)

**Issue 1:**
```
Error: Keystore not found
```

**You:**
> "Check your key.properties file - the storeFile path must be absolute, not relative."

**Issue 2:**
```
Error: Version code has already been used
```

**You:**
> "Same as iOS - increment your build number in pubspec.yaml."

---

## [BONUS TIPS - 18:30-19:30]

**[Screen: Terminal with git commands]**

**You:**
> "A few pro tips before we finish:

### Tip 1: Git Tags

**[Type]**
```bash
git tag v2.6.9
git push --tags
```

**You:**
> "Always tag your releases in Git. Makes it easy to find the exact code for each version."

### Tip 2: Changelog

**You:**
> "Keep a CHANGELOG.md file documenting what changed in each version. Your future self will thank you."

### Tip 3: Version Consistency

**You:**
> "Use the same version number for both iOS and Android. It's confusing for users if they're on different versions."

### Tip 4: Backup Keys

**You:**
> "Store your signing keys, passwords, and API keys in a secure password manager. If you lose your Android keystore, you can NEVER update your app again!"

---

## [RECAP - 19:30-20:00]

**[Screen: Show bullet points]**

**You:**
> "Let's recap what we covered:
> - ✅ Set up App Store Connect API keys
> - ✅ Built iOS IPA files from terminal
> - ✅ Uploaded to TestFlight with one command
> - ✅ Created Android signing keys
> - ✅ Built App Bundles for Play Store
> - ✅ Automated uploads with Fastlane
> - ✅ Created reusable shell scripts
>
> All of this documentation is available in the description - including the full step-by-step guide and these shell scripts."

---

## [OUTRO - 20:00-20:30]

**[Screen: Channel end screen]**

**You:**
> "That's it for today! If this video helped you, give it a thumbs up and subscribe for more Flutter development content. Drop your questions in the comments - I read every single one.
>
> Next week I'll show you how to set up CI/CD pipelines with GitHub Actions to automate this even further. See you then!"

**[Show end screen with subscribe button and next video recommendation]**

---

## VIDEO DESCRIPTION

```
🚀 Learn how to build and upload Flutter apps to TestFlight (iOS) and Google Play Console (Android) directly from the terminal!

⏱️ TIMESTAMPS:
0:00 - Introduction
0:30 - Overview
1:30 - iOS Setup (App Store Connect API)
5:00 - Building iOS IPA
7:30 - Uploading to TestFlight
9:30 - Android Setup (Keystore Creation)
11:30 - Building Android AAB
13:00 - Uploading to Google Play
15:00 - Automation Scripts
17:00 - Troubleshooting Common Errors
18:30 - Pro Tips
19:30 - Recap
20:00 - Outro

📚 RESOURCES:
📄 Full Written Guide: [Link to README.md]
📜 Shell Scripts: [Link to scripts]
🔗 Fastlane Setup Guide: https://docs.fastlane.tools
🔗 App Store Connect: https://appstoreconnect.apple.com
🔗 Google Play Console: https://play.google.com/console

💻 COMMANDS USED:

iOS:
flutter clean
flutter pub get
flutter build ipa --release
xcrun altool --upload-app --type ios --file build/ios/ipa/*.ipa --apiKey YOUR_KEY --apiIssuer YOUR_ISSUER

Android:
flutter clean
flutter pub get
flutter build appbundle --release
fastlane internal

🔧 TOOLS MENTIONED:
- Flutter SDK
- Xcode Command Line Tools
- Fastlane
- App Store Connect API
- Google Play Upload API

❓ QUESTIONS? Drop them in the comments!

👍 If this helped you, please like and subscribe!

#flutter #mobiledev #ios #android #testflight #googleplay #appdevelopment #coding #tutorial
```

---

## FILMING NOTES

### B-Roll Suggestions:
- Terminal window with commands running
- App Store Connect interface
- Google Play Console interface
- File explorer showing build outputs
- Code editor showing version updates

### Graphics to Prepare:
- Intro animation with channel logo
- Chapter title cards (iOS Setup, Building, etc.)
- Error message overlays
- Pro tips callout boxes
- End screen with subscribe button

### Screen Recording Settings:
- 1920x1080 resolution
- Show terminal with large font (18-20pt)
- Use a clean terminal theme (dark background)
- Enable cursor highlighting
- Record at 30fps

### Audio Notes:
- Use pop filter to reduce plosives
- Record in quiet room
- Speak clearly and pace yourself
- Pause between sections for easier editing
- Add background music at low volume (~15%)

---

**Script Version:** 1.0
**Last Updated:** October 2025
**Estimated Edit Time:** 3-4 hours
**Target Upload Date:** [Your date]
