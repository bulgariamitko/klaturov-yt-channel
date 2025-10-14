# From FlutterFlow to CLI: Deploy Directly from GitHub

## Video Overview

This tutorial demonstrates how to migrate a FlutterFlow project to direct GitHub deployment using CI/CD pipelines. Learn how to push your FlutterFlow code to GitHub and automatically deploy to Google Play Internal Testing, TestFlight, and GitHub Releases.

## What You'll Learn

- How to export FlutterFlow projects to GitHub
- Setting up GitHub Actions for automated Android & iOS builds
- Configuring signing certificates and credentials
- Deploying to Google Play Internal Testing automatically
- Publishing to TestFlight automatically
- Creating GitHub Releases with downloadable APKs
- Working with LLM CLI tools (Claude Code) for project management

---

## Prerequisites

### Required Accounts

1. **FlutterFlow Account**
   - Active project ready for export
   - Access to Settings > GitHub integration

2. **GitHub Account**
   - Free or paid account
   - Permission to create repositories
   - Access to GitHub Secrets

3. **Google Play Console Account** (for Android)
   - Developer account ($25 one-time fee)
   - App created in console
   - Internal testing track enabled

4. **Apple Developer Account** (for iOS)
   - Active membership ($99/year)
   - App Store Connect access
   - App registered with Bundle ID

### Required Software

- Git (for local repository management)
- Text editor or IDE (VS Code, Android Studio, etc.)
- Terminal/Command line access
- OpenSSL (for certificate generation - comes with Git on Windows)

---

## Step 1: Export FlutterFlow Project to GitHub

### In FlutterFlow

1. Open your FlutterFlow project
2. Navigate to **Settings** > **GitHub**
3. Click **Create GitHub Repository**
   - Choose repository name (e.g., `my-app`)
   - Select **Private**
4. Copy the generated repository URL (e.g., `https://github.com/username/my-app`)
5. Click **Associate** button
6. Authenticate with GitHub when prompted
7. Click **Connect GitHub with FlutterFlow**
8. Click **Push to GitHub** button

Your FlutterFlow code is now on GitHub!

---

## Step 2: Prepare Android Signing Credentials

### Generate Upload Keystore

Run these commands in your terminal:

```bash
# Navigate to a secure location for your keystore
cd ~/Documents/app-keys

# Generate keystore (replace values with your info)
keytool -genkey -v -keystore upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload \
  -storepass YOUR_STORE_PASSWORD \
  -keypass YOUR_KEY_PASSWORD
```

**Important:** Save these values securely:
- Keystore password
- Key password
- Key alias (e.g., "upload")
- Path to `.jks` file

### Convert Keystore to Base64

```bash
base64 -i upload-keystore.jks -o keystore-base64.txt
```

Copy the contents of `keystore-base64.txt` - you'll need this for GitHub Secrets.

### Create Google Play Service Account

1. Go to **Google Play Console** > **Setup** > **API access**
2. Click **Create new service account**
3. Follow link to Google Cloud Console
4. Create service account (e.g., `github-actions`)
5. Download JSON key file
6. Back in Play Console, grant access:
   - **Admin** (View app information and download bulk reports)
   - **Release to testing tracks** (Release apps to testing tracks)

### Convert Service Account JSON to Base64

```bash
base64 -i google-play-service-account.json -o service-account-base64.txt
```

---

## Step 3: Prepare iOS Signing Credentials

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

### Generate iOS Distribution Certificate Key (Optional but Recommended)

This allows GitHub Actions to create certificates automatically:

```bash
# Generate private key
openssl genrsa -out ios_distribution_key.pem 2048

# Convert to base64
base64 -i ios_distribution_key.pem -o ios-key-base64.txt
```

---

## Step 4: Configure GitHub Secrets

### Navigate to Repository Secrets

1. Go to your GitHub repository
2. Click **Settings** > **Secrets and variables** > **Actions**
3. Click **New repository secret**

### Add Android Secrets

| Secret Name | Value | Description |
|------------|-------|-------------|
| `ANDROID_KEYSTORE_BASE64` | Contents of `keystore-base64.txt` | Base64-encoded keystore |
| `ANDROID_KEYSTORE_PASSWORD` | Your keystore password | Password for keystore |
| `ANDROID_KEY_PASSWORD` | Your key password | Password for key alias |
| `ANDROID_KEY_ALIAS` | Your key alias (e.g., "upload") | Alias name in keystore |
| `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` | Contents of `service-account-base64.txt` | Base64-encoded service account JSON |

### Add iOS Secrets

| Secret Name | Value | Description |
|------------|-------|-------------|
| `APP_STORE_CONNECT_ISSUER_ID` | From App Store Connect Keys page | Issuer ID |
| `APP_STORE_CONNECT_KEY_ID` | From your API key | Key ID |
| `APP_STORE_CONNECT_PRIVATE_KEY` | Contents of `.p8` file | Full private key including headers |
| `IOS_DISTRIBUTION_CERTIFICATE_KEY_BASE64` | Contents of `ios-key-base64.txt` | Base64-encoded certificate key (optional) |

**Important:** When adding the private key, copy the **entire contents** of the `.p8` file, including:
```
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----
```

---

## Step 5: Create GitHub Actions Workflows

### Create Workflow Directory

In your repository root, create:
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
      contents: write  # Required for creating releases

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

**Important:** Replace `YOUR_PACKAGE_NAME` with your app's package name (e.g., `com.example.myapp`).

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

**Important:** Replace `YOUR_BUNDLE_ID` with your app's bundle identifier (e.g., `com.example.myapp`).

---

## Step 6: Configure Android Build Settings

### Update `android/app/build.gradle`

Add signing configuration:

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

Ensure these files are NOT committed:

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

## Step 7: Deploy Your First Build

### Push to GitHub

```bash
# Add workflow files
git add .github/workflows/

# Commit changes
git commit -m "Add CI/CD workflows for Android and iOS"

# Push to main branch
git push origin main
```

### Monitor Build Progress

1. Go to your GitHub repository
2. Click **Actions** tab
3. Watch the running workflows
4. Check logs if any errors occur

### Expected Results

After successful deployment:

1. **Google Play Console** > **Internal Testing** - New build available
2. **App Store Connect** > **TestFlight** - New build processing
3. **GitHub Releases** - New release with downloadable APK

---

## Distribution

### Android

**Option 1: Google Play Internal Testing**
- Invite testers via email in Google Play Console
- Testers receive email with Play Store link

**Option 2: Direct APK Download**
- Go to your GitHub repo → **Releases**
- Share the latest release URL
- Users download APK and install (requires "Unknown Sources" enabled)

### iOS

**TestFlight**
- Invite testers via email in App Store Connect
- Testers install TestFlight app
- Builds appear automatically after processing (5-15 minutes)

---

## Troubleshooting

### Android Build Fails

**"Keystore file not found"**
- Check that `ANDROID_KEYSTORE_BASE64` secret is correctly set
- Verify base64 encoding is correct

**"Invalid keystore password"**
- Double-check `ANDROID_KEYSTORE_PASSWORD` matches keystore creation

**"Package name mismatch"**
- Update workflow file with correct package name
- Check `android/app/build.gradle` → `applicationId`

### iOS Build Fails

**"Failed to fetch signing files"**
- Verify App Store Connect API credentials are correct
- Ensure bundle ID exists in App Store Connect
- Check that API key has proper permissions

**"No provisioning profile found"**
- Remove `--certificate-key` parameter to let Codemagic create certificates
- Ensure you have Admin access in Apple Developer account

**"Build number conflict"**
- iOS workflow uses timestamp for build numbers
- Should not conflict, but can manually increment in `ios/Runner.xcodeproj/project.pbxproj`

### Google Play Upload Fails

**"Service account has insufficient permissions"**
- In Play Console, verify service account has:
  - **Release to testing tracks** permission
  - **App access** for your specific app

**"Build number already exists"**
- Workflow auto-increments from latest build
- If manual upload was done, workflow will skip to next number

---

## Best Practices

### Security

- Never commit sensitive files (`.jks`, `.p8`, service account JSON)
- Use GitHub Secrets for all credentials
- Rotate API keys periodically
- Limit service account permissions to minimum required

### Version Management

- Update `version` in `pubspec.yaml` for each release
- Let CI/CD auto-increment build numbers
- Use semantic versioning (e.g., `1.2.3`)

### Testing

- Test workflow changes on a separate branch first
- Use `continue-on-error: true` for deployment steps during initial setup
- Monitor build logs carefully for first few deployments

### FlutterFlow Integration

- When making changes in FlutterFlow, push to GitHub
- CI/CD will automatically build and deploy
- Keep custom code in designated areas to avoid overwriting

---

## Resources

- [Flutter Deployment Guide](https://docs.flutter.dev/deployment)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Codemagic CLI Tools](https://github.com/codemagic-ci-cd/cli-tools)
- [Google Play Console](https://play.google.com/console)
- [App Store Connect](https://appstoreconnect.apple.com)

---

## Next Steps

After successful deployment:

1. Add more testers to Internal Testing/TestFlight
2. Collect feedback and iterate
3. Configure additional testing tracks (Closed Beta, Open Beta)
4. Prepare for production release
5. Set up monitoring and analytics
6. Explore using LLM CLI tools (Claude Code, Cursor) for development

---

## Questions?

If you encounter issues:
1. Check GitHub Actions logs for detailed error messages
2. Review the troubleshooting section
3. Verify all secrets are correctly configured
4. Comment on the YouTube video for help!
