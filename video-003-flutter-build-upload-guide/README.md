# Flutter Build & Upload Guide - iOS & Android

Complete step-by-step guide for building and uploading Flutter apps to TestFlight (iOS) and Google Play Console (Android) directly from the terminal.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [iOS Build & Upload to TestFlight](#ios-build--upload-to-testflight)
3. [Android Build & Upload to Google Play Console](#android-build--upload-to-google-play-console)
4. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### General Requirements
- Flutter SDK installed and configured
- Active Apple Developer Account ($99/year)
- Active Google Play Developer Account ($25 one-time)
- Git installed (for version control)

### iOS Requirements
- macOS computer (required for iOS builds)
- Xcode installed from App Store
- Valid Apple Developer signing certificate
- App Store Connect API Key (recommended)

### Android Requirements
- Android Studio installed (or Android SDK tools)
- Valid signing keystore file
- Google Play Console service account JSON key (recommended)

---

## iOS Build & Upload to TestFlight

### Step 1: Prepare App Store Connect API Key

1. **Download API Key from App Store Connect:**
   - Go to https://appstoreconnect.apple.com
   - Navigate to: Users and Access → Keys → App Store Connect API
   - Click the `+` button to create a new key
   - Download the `.p8` file (e.g., `AuthKey_ABC123XYZ.p8`)
   - **Note down**: API Key ID (e.g., `ABC123XYZ`) and Issuer ID (UUID format)

2. **Install API Key:**
   ```bash
   # Create the directory
   mkdir -p ~/.appstoreconnect/private_keys

   # Copy your .p8 file there
   cp /path/to/AuthKey_ABC123XYZ.p8 ~/.appstoreconnect/private_keys/
   ```

### Step 2: Update Version Number

Edit `pubspec.yaml`:
```bash
# Open in your preferred editor
nano pubspec.yaml

# Update the version line (increment version or build number)
# From: version: 2.6.8+28
# To:   version: 2.6.9+29
```

**Version Format:** `MAJOR.MINOR.PATCH+BUILD_NUMBER`
- Increment `PATCH` for bug fixes (2.6.8 → 2.6.9)
- Increment `MINOR` for new features (2.6.9 → 2.7.0)
- Increment `MAJOR` for breaking changes (2.7.0 → 3.0.0)
- Always increment `BUILD_NUMBER` for each upload

### Step 3: Clean Previous Build

```bash
flutter clean
flutter pub get
```

### Step 4: Build iOS Release IPA

```bash
flutter build ipa --release
```

**What happens:**
- Creates release archive at `build/ios/archive/Runner.xcarchive`
- Generates `.ipa` file at `build/ios/ipa/*.ipa`
- Build time: ~1-2 minutes
- File size: ~20-30 MB (varies by app)

**Expected Output:**
```
✓ Built build/ios/archive/Runner.xcarchive (190.0MB)
✓ Built IPA to build/ios/ipa (25.1MB)
```

### Step 5: Upload to TestFlight

**Method 1: Command Line (Recommended)**
```bash
xcrun altool --upload-app \
  --type ios \
  --file build/ios/ipa/*.ipa \
  --apiKey YOUR_API_KEY_ID \
  --apiIssuer YOUR_ISSUER_ID
```

**Replace:**
- `YOUR_API_KEY_ID` with your API Key (e.g., `MW4J27BT88`)
- `YOUR_ISSUER_ID` with your Issuer UUID (e.g., `6167cdf0-4e7f-4998-bfc7-44c636e911f3`)

**Expected Output:**
```
UPLOAD SUCCEEDED with no errors
Delivery UUID: c51a5963-e53a-4fdc-83b3-d17138bc5bc8
Transferred 24209525 bytes in 2.030 seconds (11.9MB/s)
```

**Method 2: Apple Transporter App (GUI)**
```bash
# Open Transporter app
open -a "Transporter"

# Open IPA folder in Finder
open build/ios/ipa/

# Drag .ipa file into Transporter window
# Click "Deliver" button
```

### Step 6: Submit for TestFlight Testing

1. Wait 5-10 minutes for Apple to process the build
2. Go to https://appstoreconnect.apple.com
3. Navigate to: **My Apps** → **Your App** → **TestFlight**
4. Find your new build (version 2.6.9, build 29)
5. Add **Export Compliance** information (if required)
6. Click **Submit for Review** (for external testing) or add **Internal Testers**

---

## Android Build & Upload to Google Play Console

### Step 1: Prepare Signing Keystore

**If you don't have a keystore yet:**
```bash
keytool -genkey -v \
  -keystore ~/my-release-key.jks \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias my-key-alias
```

**Create/Update `android/key.properties`:**
```bash
cat > android/key.properties << EOF
storePassword=YOUR_KEYSTORE_PASSWORD
keyPassword=YOUR_KEY_PASSWORD
keyAlias=my-key-alias
storeFile=/Users/yourusername/my-release-key.jks
EOF
```

**Ensure `android/app/build.gradle` has signing config:**
```gradle
android {
    ...
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

### Step 2: Update Version in pubspec.yaml

Same as iOS (Step 2 above) - Flutter uses the same version for both platforms.

### Step 3: Clean Previous Build

```bash
flutter clean
flutter pub get
```

### Step 4: Build Android App Bundle (AAB)

**Recommended Format: AAB (App Bundle)**
```bash
flutter build appbundle --release
```

**Output:** `build/app/outputs/bundle/release/app-release.aab`

**Alternative: APK Format (Not recommended for Play Store)**
```bash
flutter build apk --release
```

**Output:** `build/app/outputs/flutter-apk/app-release.apk`

**Expected Output:**
```
✓ Built build/app/outputs/bundle/release/app-release.aab (15.2MB)
```

### Step 5: Upload to Google Play Console

**Method 1: Command Line with Google Play Upload API**

First, set up service account:
1. Go to Google Play Console → Setup → API access
2. Create service account in Google Cloud Console
3. Download JSON key file
4. Grant permissions in Play Console

Install `fastlane`:
```bash
gem install fastlane
```

Create `Fastfile` in `android/fastlane/`:
```ruby
default_platform(:android)

platform :android do
  desc "Upload to Google Play Internal Testing"
  lane :internal do
    upload_to_play_store(
      track: 'internal',
      aab: '../build/app/outputs/bundle/release/app-release.aab',
      json_key: '/path/to/service-account.json',
      skip_upload_screenshots: true,
      skip_upload_images: true
    )
  end
end
```

Upload:
```bash
cd android
fastlane internal
```

**Method 2: Google Play Console Web Interface (Manual)**
1. Go to https://play.google.com/console
2. Select your app
3. Navigate to: **Release** → **Testing** → **Internal testing**
4. Click **Create new release**
5. Upload `build/app/outputs/bundle/release/app-release.aab`
6. Add release notes
7. Click **Review release** → **Start rollout to Internal testing**

**Method 3: Using bundletool (Advanced)**
```bash
# Install bundletool
brew install bundletool

# Upload using Google Play Developer API
# (Requires additional OAuth setup)
```

---

## Troubleshooting

### iOS Issues

**Error: "Invalid Pre-Release Train. The train version 'X.X.X' is closed"**
- **Solution:** Increment version number in `pubspec.yaml`
- Version must be higher than any previous upload

**Error: "CFBundleShortVersionString must contain a higher version"**
- **Solution:** Increment version number in `pubspec.yaml`
- Build number alone is not sufficient

**Error: "Code signing failed"**
- **Solution:** Check Xcode signing settings
- Open `ios/Runner.xcworkspace` in Xcode
- Navigate to: Runner → Signing & Capabilities
- Ensure correct Team and Provisioning Profile selected

**Error: "Failed to load AuthKey file"**
- **Solution:** Verify `.p8` file location
- Must be in: `~/.appstoreconnect/private_keys/`
- Filename must match: `AuthKey_YOUR_KEY_ID.p8`

### Android Issues

**Error: "Keystore not found"**
- **Solution:** Verify `storeFile` path in `android/key.properties`
- Use absolute paths, not relative

**Error: "Failed to read key from store"**
- **Solution:** Check keystore password and key alias
- Verify passwords in `android/key.properties`

**Error: "Version code has already been used"**
- **Solution:** Increment build number in `pubspec.yaml`
- Each upload must have unique version code

**Error: "The package name is already taken"**
- **Solution:** Change `applicationId` in `android/app/build.gradle`
- Format: `com.company.appname`

---

## Quick Reference Commands

### iOS
```bash
# Update version
nano pubspec.yaml

# Clean & build
flutter clean && flutter pub get
flutter build ipa --release

# Upload
xcrun altool --upload-app --type ios \
  --file build/ios/ipa/*.ipa \
  --apiKey YOUR_KEY --apiIssuer YOUR_ISSUER
```

### Android
```bash
# Update version
nano pubspec.yaml

# Clean & build
flutter clean && flutter pub get
flutter build appbundle --release

# Upload (manual)
open https://play.google.com/console
# Or use fastlane: cd android && fastlane internal
```

---

## Version Management Best Practices

1. **Use Semantic Versioning:** `MAJOR.MINOR.PATCH+BUILD`
2. **Always increment build number** for each upload
3. **Increment version number** for user-facing releases
4. **Keep changelog** in version control
5. **Tag releases** in Git: `git tag v2.6.9`
6. **Document breaking changes** in release notes

---

## Automation Tips

### Create Shell Script for iOS Upload

Save as `upload-ios.sh`:
```bash
#!/bin/bash
set -e

# Configuration
API_KEY="YOUR_API_KEY"
ISSUER_ID="YOUR_ISSUER_ID"

# Build
echo "Building iOS release..."
flutter clean
flutter pub get
flutter build ipa --release

# Upload
echo "Uploading to TestFlight..."
xcrun altool --upload-app --type ios \
  --file build/ios/ipa/*.ipa \
  --apiKey "$API_KEY" \
  --apiIssuer "$ISSUER_ID"

echo "✅ Upload complete!"
```

Make executable:
```bash
chmod +x upload-ios.sh
./upload-ios.sh
```

### Create Shell Script for Android Upload

Save as `upload-android.sh`:
```bash
#!/bin/bash
set -e

# Build
echo "Building Android release..."
flutter clean
flutter pub get
flutter build appbundle --release

# Upload (using fastlane)
echo "Uploading to Play Console..."
cd android
fastlane internal

echo "✅ Upload complete!"
```

Make executable:
```bash
chmod +x upload-android.sh
./upload-android.sh
```

---

## Resources

- **Flutter Deployment Docs:** https://docs.flutter.dev/deployment
- **App Store Connect:** https://appstoreconnect.apple.com
- **Google Play Console:** https://play.google.com/console
- **Fastlane Docs:** https://docs.fastlane.tools
- **Apple Developer:** https://developer.apple.com
- **Android Developer:** https://developer.android.com/studio/publish

---

**Last Updated:** October 2025
**Tested with:** Flutter 3.x, Xcode 15+, Android Studio 2024+
