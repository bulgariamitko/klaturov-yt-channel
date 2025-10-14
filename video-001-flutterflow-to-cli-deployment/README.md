# FlutterFlow to GitHub CI/CD Deployment

Technical guide for setting up automated deployment from FlutterFlow projects to Google Play Internal Testing, TestFlight, and GitHub Releases using GitHub Actions.

---

## Prerequisites

### Required Accounts

- **FlutterFlow Account** with active project
- **GitHub Account** with repository access
- **Google Play Console Account** ($25 one-time fee)
- **Apple Developer Account** ($99/year)

### Required Software

- Git
- Terminal/Command line access
- OpenSSL (included with Git on Windows)

---

## Step 1: Export FlutterFlow Project to GitHub

1. Open FlutterFlow project
2. Navigate to **Settings** > **GitHub**
3. Click **Create GitHub Repository**
4. Choose repository name and visibility (Private recommended)
5. Copy generated repository URL
6. Click **Associate** button
7. Authenticate with GitHub
8. Click **Connect GitHub with FlutterFlow**
9. Click **Push to GitHub** button

---

## Step 2: Regenerate App Icons

After exporting from FlutterFlow, regenerate app icons for proper deployment:

1. Clone the repository locally:
   ```bash
   git clone https://github.com/username/your-repo.git
   cd your-repo
   ```

2. Install dependencies:
   ```bash
   flutter pub get
   ```

3. Regenerate app icons:
   ```bash
   dart run flutter_launcher_icons
   ```

4. Commit and push the regenerated icons:
   ```bash
   git add android/app/src/main/res/ ios/Runner/Assets.xcassets/
   git commit -m "Regenerate app icons"
   git push origin main
   ```

**Note:** FlutterFlow generates icons during their build process, but when deploying from GitHub directly, icons need to be regenerated locally using `flutter_launcher_icons`.

---

## Step 3: Prepare Android Signing Credentials

### Download Keystore from FlutterFlow

1. In FlutterFlow, navigate to **Settings** > **Mobile Deployment** > **Android**
2. Click the orange **Key** button to download your `.jks` keystore file
3. Save the file securely (e.g., `~/Documents/app-keys/upload-keystore.jks`)
4. Note the keystore details from FlutterFlow:
   - Keystore password
   - Key password
   - Key alias

**Alternative:** Generate your own keystore using `keytool`:
```bash
cd ~/Documents/app-keys
keytool -genkey -v -keystore upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload \
  -storepass YOUR_STORE_PASSWORD \
  -keypass YOUR_KEY_PASSWORD
```

### Convert Keystore to Base64

```bash
base64 -i upload-keystore.jks -o keystore-base64.txt
```

### Create Google Play Service Account

1. Go to **Google Play Console** > **Setup** > **API access**
2. Click **Create new service account**
3. Follow link to Google Cloud Console
4. Create service account (e.g., `github-actions`)
5. Download JSON key file
6. Back in Play Console, grant permissions:
   - **Admin** (View app information and download bulk reports)
   - **Release to testing tracks** (Release apps to testing tracks)

### Convert Service Account JSON to Base64

```bash
base64 -i google-play-service-account.json -o service-account-base64.txt
```

---

## Step 4: Prepare iOS Signing Credentials

### Get App Store Connect API Keys

1. Go to **App Store Connect** > **Users and Access** > **Keys**
2. Click **+** to generate new key
3. Set name (e.g., "GitHub Actions")
4. Set access: **App Manager** or **Developer**
5. Download `.p8` key file
6. Note these values:
   - **Issuer ID** (top of Keys page)
   - **Key ID** (in the keys list)
   - **Private Key** (contents of `.p8` file)

### Generate iOS Distribution Certificate Key

```bash
openssl genrsa -out ios_distribution_key.pem 2048
base64 -i ios_distribution_key.pem -o ios-key-base64.txt
```

---

## Step 5: Configure GitHub Secrets

1. Go to GitHub repository
2. Click **Settings** > **Secrets and variables** > **Actions**
3. Click **New repository secret**

### Android Secrets

| Secret Name | Value | Description |
|------------|-------|-------------|
| `ANDROID_KEYSTORE_BASE64` | Contents of `keystore-base64.txt` | Base64-encoded keystore |
| `ANDROID_KEYSTORE_PASSWORD` | Your keystore password | Password for keystore |
| `ANDROID_KEY_PASSWORD` | Your key password | Password for key alias |
| `ANDROID_KEY_ALIAS` | Your key alias (e.g., "upload") | Alias name in keystore |
| `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` | Contents of `service-account-base64.txt` | Base64-encoded service account JSON |

### iOS Secrets

| Secret Name | Value | Description |
|------------|-------|-------------|
| `APP_STORE_CONNECT_ISSUER_ID` | From App Store Connect Keys page | Issuer ID |
| `APP_STORE_CONNECT_KEY_ID` | From your API key | Key ID |
| `APP_STORE_CONNECT_PRIVATE_KEY` | Contents of `.p8` file | Full private key including headers |
| `IOS_DISTRIBUTION_CERTIFICATE_KEY_BASE64` | Contents of `ios-key-base64.txt` | Base64-encoded certificate key |

**Note:** For `APP_STORE_CONNECT_PRIVATE_KEY`, include the entire `.p8` file content:
```
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----
```

---

## Step 6: Create GitHub Actions Workflows

### Create Workflow Directory

```
.github/
  workflows/
    android-internal-testing.yaml
    ios-testflight.yaml
```

### Android Workflow (`android-internal-testing.yaml`)

```yaml
name: Android Internal Testing Deployment

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy-android:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: 'stable'

      - name: Install dependencies
        run: flutter pub get

      - name: Install Codemagic CLI tools
        run: pip install codemagic-cli-tools

      - name: Create key.properties file
        run: |
          echo "storePassword=${{ secrets.ANDROID_KEYSTORE_PASSWORD }}" > android/key.properties
          echo "keyPassword=${{ secrets.ANDROID_KEY_PASSWORD }}" >> android/key.properties
          echo "keyAlias=${{ secrets.ANDROID_KEY_ALIAS }}" >> android/key.properties
          echo "storeFile=../keys/upload-keystore.jks" >> android/key.properties

      - name: Decode keystore
        run: |
          mkdir -p android/keys
          echo "${{ secrets.ANDROID_KEYSTORE_BASE64 }}" | base64 --decode > android/keys/upload-keystore.jks

      - name: Decode Google Play service account
        run: |
          mkdir -p android/keys
          echo "${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT_JSON }}" | base64 --decode > android/keys/google-play-service-account.json

      - name: Get latest build number from Google Play
        id: build_number
        run: |
          LATEST_BUILD=$(google-play get-latest-build-number \
            --package-name YOUR_PACKAGE_NAME \
            --credentials @file:android/keys/google-play-service-account.json \
            --tracks internal)
          NEW_BUILD=$((LATEST_BUILD + 1))
          echo "new_build_number=$NEW_BUILD" >> $GITHUB_OUTPUT
          echo "New build number will be: $NEW_BUILD"

      - name: Build Android App Bundle
        run: |
          flutter build appbundle \
            --release \
            --build-number=${{ steps.build_number.outputs.new_build_number }}

      - name: Build Android APK
        run: |
          flutter build apk \
            --release \
            --build-number=${{ steps.build_number.outputs.new_build_number }}

      - name: Publish to Google Play Internal Testing
        continue-on-error: true
        id: google_play_upload
        run: |
          google-play bundles publish \
            --bundle build/app/outputs/bundle/release/app-release.aab \
            --track internal \
            --credentials @file:android/keys/google-play-service-account.json

      - name: Get version info
        id: version
        run: |
          VERSION_NAME=$(grep 'version:' pubspec.yaml | sed 's/version: //' | cut -d'+' -f1)
          echo "version_name=$VERSION_NAME" >> $GITHUB_OUTPUT

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          tag_name: v${{ steps.build_number.outputs.new_build_number }}
          name: v${{ steps.version.outputs.version_name }} (Build ${{ steps.build_number.outputs.new_build_number }})
          body: |
            ## Android Release
            **Version:** ${{ steps.version.outputs.version_name }}
            **Build Number:** ${{ steps.build_number.outputs.new_build_number }}

            ### Downloads
            📱 **APK:** Download `app-release.apk` below and install on Android

            ### Installation
            1. Download the APK file
            2. Enable "Install from Unknown Sources" on your Android device
            3. Open the APK file to install

            ### Changes
            ${{ github.event.head_commit.message }}

            ---
            ${{ steps.google_play_upload.outcome == 'success' && '✅ Deployed to Google Play Internal Testing' || '⚠️ Google Play upload skipped (manual deployment required)' }}
          files: |
            build/app/outputs/flutter-apk/app-release.apk
          draft: false
          prerelease: false
```

**Replace `YOUR_PACKAGE_NAME`** with your app's package name from `android/app/build.gradle` (e.g., `com.example.myapp`).

### iOS Workflow (`ios-testflight.yaml`)

```yaml
name: iOS TestFlight Deployment

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy-ios:
    runs-on: macos-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: 'stable'

      - name: Install dependencies
        run: flutter pub get

      - name: Install Codemagic CLI tools
        run: pip install codemagic-cli-tools

      - name: Set up keychain
        run: |
          keychain initialize

      - name: Decode iOS certificate key
        run: |
          mkdir -p keys
          echo "${{ secrets.IOS_DISTRIBUTION_CERTIFICATE_KEY_BASE64 }}" | base64 --decode > keys/ios_distribution_key.pem
          chmod 600 keys/ios_distribution_key.pem

      - name: Fetch signing files
        run: |
          app-store-connect fetch-signing-files YOUR_BUNDLE_ID \
            --issuer-id ${{ secrets.APP_STORE_CONNECT_ISSUER_ID }} \
            --key-id ${{ secrets.APP_STORE_CONNECT_KEY_ID }} \
            --private-key '${{ secrets.APP_STORE_CONNECT_PRIVATE_KEY }}' \
            --certificate-key @file:keys/ios_distribution_key.pem \
            --type IOS_APP_STORE \
            --create

      - name: Use signing files
        run: |
          keychain add-certificates
          xcode-project use-profiles

      - name: Increment build number
        run: |
          cd ios
          agvtool new-version -all $(($(date +%s)))
          cd ..

      - name: Build iOS app
        run: |
          flutter build ipa \
            --release \
            --export-options-plist=$HOME/export_options.plist

      - name: Publish to TestFlight
        run: |
          app-store-connect publish \
            --path build/ios/ipa/*.ipa \
            --issuer-id ${{ secrets.APP_STORE_CONNECT_ISSUER_ID }} \
            --key-id ${{ secrets.APP_STORE_CONNECT_KEY_ID }} \
            --private-key '${{ secrets.APP_STORE_CONNECT_PRIVATE_KEY }}'

      - name: Clean up keychain
        if: always()
        run: |
          keychain delete
```

**Replace `YOUR_BUNDLE_ID`** with your app's bundle identifier (e.g., `com.example.myapp`).

---

## Step 7: Configure Android Build Settings

### Update `android/app/build.gradle`

Add signing configuration inside the `android` block:

```gradle
android {
    // ... existing config

    signingConfigs {
        release {
            def keystorePropertiesFile = rootProject.file("key.properties")
            def keystoreProperties = new Properties()
            if (keystorePropertiesFile.exists()) {
                keystoreProperties.load(new FileInputStream(keystorePropertiesFile))

                keyAlias keystoreProperties['keyAlias']
                keyPassword keystoreProperties['keyPassword']
                storeFile file(keystoreProperties['storeFile'])
                storePassword keystoreProperties['storePassword']
            }
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            // ... other release config
        }
    }
}
```

### Update `.gitignore`

Add these entries:

```gitignore
# Android signing
android/key.properties
android/keys/
*.jks
*.keystore

# iOS signing
ios/keys/
*.p8
*.pem
*.p12
*.mobileprovision

# Sensitive files
**/google-play-service-account.json
```

---

## Step 8: Deploy

### Push to GitHub

```bash
git add .github/workflows/ android/app/build.gradle .gitignore
git commit -m "Add CI/CD workflows for Android and iOS"
git push origin main
```

### Monitor Build Progress

1. Go to GitHub repository
2. Click **Actions** tab
3. Watch running workflows
4. Check logs for errors

### Verify Deployment

After successful build:

- **Google Play Console** > **Internal Testing** - New build available
- **App Store Connect** > **TestFlight** - New build processing (5-15 minutes)
- **GitHub Releases** - New release with downloadable APK

---

## Distribution

### Android

**Google Play Internal Testing:**
- Invite testers via email in Google Play Console
- Testers receive Play Store link

**Direct APK Download:**
- Navigate to GitHub repository > **Releases**
- Share release URL
- Users download APK (requires "Unknown Sources" enabled)

### iOS

**TestFlight:**
- Invite testers via email in App Store Connect
- Testers install TestFlight app
- Builds appear after processing

---

## Troubleshooting

### Android

**"Keystore file not found"**
- Verify `ANDROID_KEYSTORE_BASE64` secret is set correctly
- Check base64 encoding has no extra spaces

**"Invalid keystore password"**
- Verify `ANDROID_KEYSTORE_PASSWORD` matches keystore creation password

**"Package name mismatch"**
- Update `YOUR_PACKAGE_NAME` in workflow file
- Check `android/app/build.gradle` → `applicationId`

**"Build number already exists"**
- Workflow auto-increments from latest Google Play build
- If manual upload occurred, workflow skips to next number

### iOS

**"Failed to fetch signing files"**
- Verify all App Store Connect credentials are correct
- Ensure bundle ID exists in App Store Connect
- Check API key has proper permissions

**"No provisioning profile found"**
- Remove `--certificate-key` parameter to let Codemagic auto-create
- Ensure Admin access in Apple Developer account

**"Build number conflict"**
- Workflow uses timestamp for build numbers
- Manually increment in `ios/Runner.xcodeproj/project.pbxproj` if needed

### Google Play Upload

**"Service account has insufficient permissions"**
- Verify service account has "Release to testing tracks" permission
- Verify service account has access to specific app

---

## Security Notes

- Never commit `.jks`, `.p8`, or service account JSON files
- Store credentials in GitHub Secrets only
- Backup keystore and certificates securely
- Rotate API keys periodically
- Limit service account permissions to minimum required

---

## Version Management

- Update `version` in `pubspec.yaml` for each release
- CI/CD auto-increments build numbers
- Use semantic versioning (e.g., `1.2.3`)
