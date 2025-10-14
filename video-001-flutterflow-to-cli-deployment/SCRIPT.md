# Video Script: From FlutterFlow to CLI - Deploy Directly from GitHub

## Video Title
"Deploy FlutterFlow Apps to Play Store & TestFlight Automatically with GitHub Actions"

## Video Description
Learn how to migrate your FlutterFlow project to GitHub and set up automated CI/CD deployment to Google Play Internal Testing, TestFlight, and GitHub Releases. No more manual builds - push to GitHub and your app deploys automatically!

**Timestamps:**
0:00 - Introduction
1:15 - What You'll Need (Prerequisites)
3:30 - Part 1: Export FlutterFlow to GitHub
6:45 - Part 2: Android Setup (Keystore & Secrets)
15:20 - Part 3: iOS Setup (Certificates & Secrets)
22:40 - Part 4: Create GitHub Actions Workflows
30:15 - Part 5: First Deployment & Testing
35:00 - Troubleshooting & Best Practices
40:30 - Conclusion & Next Steps

---

## Introduction (0:00 - 1:15)

**[Screen: Intro animation/channel logo]**

Hey everyone, Dimitar here! Today I'm going to show you how to completely automate your FlutterFlow app deployment using GitHub Actions.

**[Screen: Show comparison - Manual vs Automated]**

Imagine this: You make changes in FlutterFlow, push one button, and your app automatically:
- Builds for Android and iOS
- Deploys to Google Play Internal Testing
- Publishes to TestFlight
- Creates GitHub releases with downloadable APKs

No more manual builds, no more command line frustration. Just push and deploy.

**[Screen: Example CheapestAPP project]**

I'll be using a real production app I built called "CheapestAPP" as our reference. This app is live on both app stores and uses this exact deployment process.

Let's get started!

---

## What You'll Need (1:15 - 3:30)

**[Screen: Split screen with checklist]**

Before we start, let's go over what you'll need. Don't worry if you don't have everything yet - I'll show you how to get each piece.

**Accounts:**

**[Highlight each as mentioned]**

1. **FlutterFlow Account** - You probably already have this
2. **GitHub Account** - Free tier works fine
3. **Google Play Console** - $25 one-time fee for Android deployment
4. **Apple Developer Account** - $99/year for iOS deployment

**[Screen: Show folder structure]**

You'll also need:
- A text editor (I'll use Claude Code, but VS Code works great too)
- Basic terminal/command line knowledge
- About 45 minutes to set everything up

**[Screen: Security note]**

Important: We'll be dealing with signing certificates and API keys. These are sensitive - never share them or commit them to public repositories. GitHub Secrets keeps everything safe.

Alright, let's dive into the setup!

---

## Part 1: Export FlutterFlow to GitHub (3:30 - 6:45)

**[Screen: FlutterFlow dashboard]**

First, we need to get our FlutterFlow project into GitHub. This is super straightforward.

**[Screen: Navigate to Settings]**

In your FlutterFlow project:
1. Click **Settings** in the left sidebar
2. Click **GitHub** tab

**[Screen: Show GitHub integration page]**

Here's where the magic happens. FlutterFlow has built-in GitHub integration.

**Step 1: Create GitHub Repository**

**[Click "Create GitHub Repository" button]**

Click this button. FlutterFlow will open a dialog where you can:
- Choose your repository name (I'll use "cheapest-app")
- Select Public or Private (I recommend Private for production apps)

**[Create repository]**

Click create, and FlutterFlow generates a new GitHub repo for you.

**[Screen: Show the generated GitHub URL]**

Copy this URL - we'll need it in a moment.

**Step 2: Associate Repository**

**[Click "Associate" button]**

This links your FlutterFlow project to the GitHub repo.

**[Screen: GitHub OAuth dialog]**

Authorize FlutterFlow to access your GitHub account.

**Step 3: Connect GitHub**

**[Click "Connect GitHub with FlutterFlow" button]**

This sets up the connection between the two platforms.

**Step 4: Push to GitHub**

**[Click "Push to GitHub" button]**

And finally, push your code! This might take a minute depending on project size.

**[Screen: Show GitHub repository]**

Perfect! If we go to GitHub, we can see our Flutter code is now in the repository. All the source code, assets, everything.

**[Point to key files]**

Here's our:
- `lib/` folder with all the Dart code
- `pubspec.yaml` with dependencies
- `android/` and `ios/` folders with native configuration

Now the real fun begins - let's set up automated deployment!

---

## Part 2: Android Setup (6:45 - 15:20)

**[Screen: Terminal/Command line]**

Okay, Android deployment requires a few pieces:
1. An upload keystore (for signing your app)
2. Google Play service account (for API access)
3. GitHub Secrets (to store everything securely)

Let's create each one.

### Creating Upload Keystore (7:00 - 9:30)

**[Screen: Terminal showing command]**

First, we need to generate a keystore. This is like a digital signature that proves you built the app.

Open your terminal and navigate to a secure location:

```bash
cd ~/Documents/app-keys
```

**[Type command]**

Now run this command:

```bash
keytool -genkey -v -keystore upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload
```

**[Screen: Interactive prompts]**

It will ask you several questions:
- Keystore password (choose a strong password)
- Key password (can be the same as keystore password)
- Your name, organization, location, etc.

**[Show the generated file]**

Great! Now we have `upload-keystore.jks` file.

**IMPORTANT:** Keep this file safe and never share it. If you lose it, you can't update your app on Google Play!

**[Screen: Show note/warning]**

I recommend backing this up to a secure location like a password manager or encrypted cloud storage.

### Convert Keystore to Base64 (9:30 - 10:30)

**[Screen: Terminal]**

GitHub Secrets can't store binary files directly, so we need to convert our keystore to Base64.

```bash
base64 -i upload-keystore.jks -o keystore-base64.txt
```

**[Show the output file]**

This creates a text file with the encoded keystore. We'll use this in GitHub Secrets.

### Create Google Play Service Account (10:30 - 13:00)

**[Screen: Google Play Console]**

Next, we need a service account so GitHub Actions can upload to Google Play.

Go to **Google Play Console** → **Setup** → **API access**

**[Click "Create new service account"]**

This will take you to Google Cloud Console.

**[Screen: Google Cloud Console]**

Click **"+ CREATE SERVICE ACCOUNT"**

**[Fill in details]**

Give it a name like "github-actions" and description "GitHub Actions deployment"

**[Click Create and Continue]**

Skip the permissions section - we'll set those in Play Console.

**[Click Done]**

Now click on the service account you just created.

**[Navigate to Keys tab]**

Go to the **Keys** tab → **Add Key** → **Create new key**

**[Select JSON]**

Choose JSON format and click Create.

**[Show downloaded file]**

This downloads a JSON file. Keep this safe!

**[Screen: Back to Play Console]**

Back in Google Play Console, you should now see your service account listed.

**[Click on it and grant access]**

Click "Grant Access" and give these permissions:
- **Admin (View app information and download bulk reports)**
- **Release to testing tracks (Release apps to testing tracks)**

**[Save changes]**

### Convert Service Account to Base64 (13:00 - 13:45)

**[Screen: Terminal]**

Same process - convert to Base64:

```bash
base64 -i google-play-service-account.json -o service-account-base64.txt
```

**[Show output]**

Perfect! Now we have both credentials ready for GitHub.

### Add GitHub Secrets for Android (13:45 - 15:20)

**[Screen: GitHub repository]**

Go to your repository → **Settings** → **Secrets and variables** → **Actions**

**[Click "New repository secret"]**

We need to add 5 secrets. Let me show you each one:

**1. ANDROID_KEYSTORE_BASE64**
**[Paste from keystore-base64.txt]**
This is the entire contents of the `keystore-base64.txt` file.

**2. ANDROID_KEYSTORE_PASSWORD**
**[Type password]**
The password you used when creating the keystore.

**3. ANDROID_KEY_PASSWORD**
**[Type password]**
The key password (might be same as keystore password).

**4. ANDROID_KEY_ALIAS**
**[Type "upload"]**
The alias you used - probably "upload" if you followed my command.

**5. GOOGLE_PLAY_SERVICE_ACCOUNT_JSON**
**[Paste from service-account-base64.txt]**
The entire contents of `service-account-base64.txt`.

**[Screen: Show all 5 secrets added]**

Excellent! Android secrets are done. Now let's tackle iOS.

---

## Part 3: iOS Setup (15:20 - 22:40)

**[Screen: App Store Connect]**

iOS deployment uses App Store Connect API keys instead of certificates. This is actually simpler than the old certificate-based approach.

### Get App Store Connect API Keys (15:45 - 18:00)

**[Screen: App Store Connect]**

Go to **App Store Connect** → **Users and Access** → **Keys** tab

**[Click the + button]**

Click the **+** button to generate a new key.

**[Fill in details]**

- Name: "GitHub Actions"
- Access: Choose **Developer** or **App Manager**

**[Show important info]**

IMPORTANT: You can only download this key ONCE. If you lose it, you'll need to generate a new one.

**[Click Generate]**

**[Show three pieces of information]**

After generating, note these three values:

1. **Issuer ID** - At the top of the Keys page (starts with a UUID)
2. **Key ID** - Listed with your key (10 characters, like "353ZAQ9SV4")
3. **Private Key** - The `.p8` file you download

**[Download the .p8 file]**

Download and keep this file safe.

### Generate iOS Distribution Certificate Key (18:00 - 19:30)

**[Screen: Terminal]**

This optional step lets GitHub Actions automatically create and manage certificates for you. Highly recommended!

```bash
openssl genrsa -out ios_distribution_key.pem 2048
```

**[Show the generated file]**

Now convert to Base64:

```bash
base64 -i ios_distribution_key.pem -o ios-key-base64.txt
```

**[Show output]**

Perfect! This will let Codemagic CLI tools create certificates on the fly.

### Add GitHub Secrets for iOS (19:30 - 22:40)

**[Screen: GitHub Secrets page]**

Back to GitHub repository → **Settings** → **Secrets and variables** → **Actions**

**[Click "New repository secret"]**

We need 4 secrets for iOS:

**1. APP_STORE_CONNECT_ISSUER_ID**
**[Paste Issuer ID]**
The UUID from the top of the Keys page.

**2. APP_STORE_CONNECT_KEY_ID**
**[Paste Key ID]**
The 10-character key ID (e.g., "353ZAQ9SV4").

**3. APP_STORE_CONNECT_PRIVATE_KEY**
**[Open .p8 file in text editor]**
This one's important - copy the ENTIRE contents of the `.p8` file, including the BEGIN and END lines:

```
-----BEGIN PRIVATE KEY-----
MIGTAgEAMBMG...
-----END PRIVATE KEY-----
```

**[Paste entire content]**

**4. IOS_DISTRIBUTION_CERTIFICATE_KEY_BASE64**
**[Paste from ios-key-base64.txt]**
The contents of `ios-key-base64.txt`.

**[Screen: Show all secrets]**

Perfect! All iOS secrets are configured.

**[Screen: Overview of all secrets]**

Let's take a quick look at what we've set up:
- 5 Android secrets for keystore and Google Play access
- 4 iOS secrets for App Store Connect and certificates

Now we can create the GitHub Actions workflows that use these secrets!

---

## Part 4: Create GitHub Actions Workflows (22:40 - 30:15)

**[Screen: Repository file structure]**

GitHub Actions use YAML files in a special directory. Let's create the structure:

```
.github/
  workflows/
    android-internal-testing.yaml
    ios-testflight.yaml
```

**[Screen: Create folder]**

I'll create this using Claude Code, but you can use any text editor or the GitHub web interface.

### Create Android Workflow (23:15 - 26:30)

**[Screen: Creating android-internal-testing.yaml]**

Create `.github/workflows/android-internal-testing.yaml`

**[Show the workflow file - scroll through key sections]**

Let me explain the key parts of this workflow:

**[Highlight trigger section]**
```yaml
on:
  push:
    branches:
      - main
```
This runs every time you push to the main branch.

**[Highlight Java setup]**
```yaml
- name: Set up Java
  uses: actions/setup-java@v4
  with:
    distribution: 'zulu'
    java-version: '17'
```
Sets up Java 17 (required for recent Flutter versions).

**[Highlight Flutter setup]**
```yaml
- name: Set up Flutter
  uses: subosito/flutter-action@v2
  with:
    channel: 'stable'
```
Installs Flutter stable channel.

**[Highlight secrets usage]**
```yaml
- name: Create key.properties file
  run: |
    echo "storePassword=${{ secrets.ANDROID_KEYSTORE_PASSWORD }}" > android/key.properties
```
See how we're using the secrets we created? This creates the signing configuration file.

**[Highlight build number fetch]**
```yaml
- name: Get latest build number from Google Play
  run: |
    LATEST_BUILD=$(google-play get-latest-build-number \
      --package-name YOUR_PACKAGE_NAME \
      ...
```

**IMPORTANT:** You need to replace `YOUR_PACKAGE_NAME` with your actual package name!

**[Show where to find package name]**

Your package name is in `android/app/build.gradle`:
```gradle
applicationId "app.Cheapest"  // This is your package name
```

**[Highlight build steps]**
```yaml
- name: Build Android App Bundle
  run: flutter build appbundle --release --build-number=...

- name: Build Android APK
  run: flutter build apk --release --build-number=...
```
Builds both App Bundle (for Play Store) and APK (for direct distribution).

**[Highlight publish step]**
```yaml
- name: Publish to Google Play Internal Testing
  run: |
    google-play bundles publish \
      --bundle build/app/outputs/bundle/release/app-release.aab \
      --track internal
```
Automatically uploads to Internal Testing!

**[Highlight GitHub Release]**
```yaml
- name: Create GitHub Release
  uses: softprops/action-gh-release@v1
  with:
    tag_name: v${{ steps.build_number.outputs.new_build_number }}
    files: |
      build/app/outputs/flutter-apk/app-release.apk
```
Creates a GitHub Release with the APK attached for direct download!

**[Save file]**

### Create iOS Workflow (26:30 - 29:30)

**[Screen: Creating ios-testflight.yaml]**

Create `.github/workflows/ios-testflight.yaml`

**[Show the workflow file - scroll through]**

iOS workflow is similar but with some key differences:

**[Highlight macOS runner]**
```yaml
runs-on: macos-latest
```
iOS builds require macOS (GitHub provides free macOS runners).

**[Highlight keychain setup]**
```yaml
- name: Set up keychain
  run: keychain initialize
```
Creates a temporary keychain for signing.

**[Highlight fetch signing files]**
```yaml
- name: Fetch signing files
  run: |
    app-store-connect fetch-signing-files YOUR_BUNDLE_ID \
      --issuer-id ${{ secrets.APP_STORE_CONNECT_ISSUER_ID }} \
      --certificate-key @file:keys/ios_distribution_key.pem \
      --type IOS_APP_STORE \
      --create
```

**IMPORTANT:** Replace `YOUR_BUNDLE_ID` with your actual bundle identifier!

**[Show where to find bundle ID]**

Your bundle ID is in `ios/Runner.xcodeproj/project.pbxproj` or just look in Xcode. It's something like `com.example.myapp`.

**[Highlight build number]**
```yaml
- name: Increment build number
  run: |
    cd ios
    agvtool new-version -all $(($(date +%s)))
```
Uses timestamp for unique build numbers (simple and effective).

**[Highlight TestFlight publish]**
```yaml
- name: Publish to TestFlight
  run: |
    app-store-connect publish \
      --path build/ios/ipa/*.ipa
```
Uploads to TestFlight automatically!

**[Save file]**

Perfect! Both workflows are created.

### Configure Android Signing (29:30 - 30:15)

**[Screen: android/app/build.gradle]**

One more thing - we need to update Android's build.gradle to use our keystore.

**[Navigate to android/app/build.gradle]**

Add this signing configuration inside the `android` block:

```gradle
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
    }
}
```

**[Save file]**

This tells Gradle to use our keystore for release builds.

Great! Everything is configured. Let's deploy!

---

## Part 5: First Deployment & Testing (30:15 - 35:00)

**[Screen: Terminal/Git]**

Time for the moment of truth! Let's commit and push our changes.

### Commit Workflow Files (30:30 - 31:30)

**[Type commands]**

```bash
# Add the workflow files
git add .github/

# Also add the modified build.gradle if you changed it
git add android/app/build.gradle

# Commit
git commit -m "Add CI/CD workflows for Android and iOS deployment"

# Push to main
git push origin main
```

**[Screen: Pushing animation]**

Pushing to GitHub...

### Monitor GitHub Actions (31:30 - 33:30)

**[Screen: GitHub repository]**

Now go to your repository and click the **Actions** tab.

**[Show Actions dashboard]**

Look! Two workflows are now running:
- Android Internal Testing Deployment
- iOS TestFlight Deployment

**[Click on Android workflow]**

Let's watch the Android build first.

**[Show workflow steps expanding]**

You can see each step:
- ✅ Checkout repository
- ✅ Set up Java
- ✅ Set up Flutter
- ✅ Install dependencies
- ⏳ Create key.properties file (running)

**[Fast forward or show key steps]**

This takes about 10-15 minutes for the first build. The key steps to watch:
- Build Android App Bundle (takes ~5 minutes)
- Publish to Google Play Internal Testing
- Create GitHub Release

**[Show when complete - all green checkmarks]**

Success! Let's see the results.

### Check Google Play Console (33:30 - 34:15)

**[Screen: Google Play Console]**

Go to **Google Play Console** → Your App → **Internal Testing**

**[Show the new build]**

There it is! Our build was automatically uploaded. You can see:
- Build number (automatically incremented)
- Upload date
- Status: Available for testing

**[Click "Manage testers"]**

Add tester email addresses here, and they'll get an email with installation instructions.

### Check GitHub Releases (34:15 - 34:45)

**[Screen: GitHub repository]**

Go to your repository → **Releases**

**[Show the new release]**

Perfect! There's our release with:
- Version tag (v131 in my case)
- Release notes (auto-generated from commit message)
- **APK download** - Users can download and sideload this

**[Highlight download link]**

Anyone with the link can download this APK and install it directly on Android (with "Unknown Sources" enabled).

### Check TestFlight (34:45 - 35:00)

**[Screen: App Store Connect]**

Go to **App Store Connect** → Your App → **TestFlight**

**[Show processing status]**

iOS builds take a bit longer to process (5-15 minutes after upload). You'll see:
- Status: "Processing" initially
- Then "Ready to Submit" or "Testing" once processed

**[Show tester management]**

Add testers here, and they'll get TestFlight invitations via email.

**[Celebration moment]**

And that's it! Your app is now automatically deploying to both app stores with every push to GitHub!

---

## Troubleshooting & Best Practices (35:00 - 40:30)

**[Screen: Common issues list]**

Let me quickly cover some common issues you might encounter and best practices.

### Common Android Issues (35:15 - 37:00)

**[Screen: Error log example]**

**Issue 1: "Keystore file not found"**

**[Show fix]**
- Double-check your `ANDROID_KEYSTORE_BASE64` secret
- Make sure you didn't include any extra spaces
- Re-encode the keystore if needed

**Issue 2: "Invalid keystore password"**

**[Show fix]**
- Verify the password matches what you used during creation
- Check for typos in the GitHub Secret

**Issue 3: "Package name mismatch"**

**[Show fix]**
- Update the workflow file with your correct package name
- Check `android/app/build.gradle` for the `applicationId`

**Issue 4: "Build number already exists"**

**[Show fix]**
- The workflow auto-increments from Google Play
- If you uploaded manually, just push again - it'll skip to next number

### Common iOS Issues (37:00 - 38:30)

**[Screen: iOS error examples]**

**Issue 1: "Failed to fetch signing files"**

**[Show fix]**
- Verify all three App Store Connect credentials are correct
- Ensure your bundle ID exists in App Store Connect
- Check that your API key has proper permissions (Developer or App Manager)

**Issue 2: "No provisioning profile found"**

**[Show fix]**
- Remove the `--certificate-key` parameter to let Codemagic auto-create
- Manually create provisioning profile in Apple Developer portal
- Ensure you have Admin access to the Apple Developer account

**Issue 3: "Build already uploaded"**

**[Show fix]**
- iOS uses timestamps for build numbers, so conflicts are rare
- If it happens, wait a second and push again

### Security Best Practices (38:30 - 39:30)

**[Screen: Security checklist]**

**Never commit these files:**
- `.jks` or `.keystore` files
- `key.properties`
- `.p8` or `.pem` certificate files
- Service account JSON files

**[Show .gitignore]**

Make sure your `.gitignore` includes:
```
android/key.properties
android/keys/
*.jks
*.keystore
ios/keys/
*.p8
*.pem
**/google-play-service-account.json
```

**[Screen: Backup reminder]**

**Backup your credentials!**
- Store keystore in password manager (like 1Password)
- Save service account JSON in secure cloud storage
- Keep API keys in encrypted notes

**[Show rotation reminder]**

**Rotate credentials periodically:**
- Service account keys every 6-12 months
- App Store Connect keys yearly
- Update GitHub Secrets when rotating

### Development Best Practices (39:30 - 40:30)

**[Screen: Branch strategy diagram]**

**Use branch protection:**
- Set up branch protection rules on `main`
- Require PR reviews before merging
- Run workflows on feature branches for testing

**[Screen: Version management]**

**Version management:**
- Update `version` in `pubspec.yaml` for releases
- Follow semantic versioning (1.2.3)
- Let CI/CD handle build number incrementing

**[Screen: Testing workflow]**

**Test workflow changes:**
- Create a test branch first
- Push workflow changes to test branch
- Verify builds succeed before merging to main

**[Screen: Monitoring]**

**Monitor your builds:**
- Set up email notifications for failed builds
- Check logs regularly for warnings
- Test on real devices from TestFlight/Internal Testing

---

## Conclusion & Next Steps (40:30 - End)

**[Screen: Summary slide]**

Alright, let's recap what we've accomplished today:

✅ Exported FlutterFlow project to GitHub
✅ Set up Android signing and Google Play deployment
✅ Configured iOS certificates and TestFlight publishing
✅ Created GitHub Actions workflows for automated CI/CD
✅ Deployed our first build to both app stores
✅ Set up GitHub Releases for direct APK distribution

**[Screen: Show the complete workflow diagram]**

Now every time you push to GitHub, your app automatically:
1. Builds for Android and iOS
2. Increments build numbers
3. Signs with production certificates
4. Uploads to Google Play Internal Testing
5. Publishes to TestFlight
6. Creates GitHub Releases with APKs

**[Screen: Time savings comparison]**

This saves you hours of manual work on every release!

### Next Steps (41:30 - 42:30)

**[Screen: Next steps checklist]**

Here's what you can do next:

1. **Add more testers** - Invite friends/colleagues to test
2. **Set up additional tracks** - Closed Beta, Open Beta
3. **Configure release notes automation** - Auto-generate changelogs
4. **Add Slack/Discord notifications** - Get alerted on deployment
5. **Set up staging vs production** - Use different branches

**[Screen: LLM CLI tools mention]**

And here's where it gets even better - you can use LLM CLI tools like Claude Code or Cursor to help manage all of this! They can:
- Write workflow configurations for you
- Debug deployment issues
- Suggest optimizations
- Help with version management

I'll be covering LLM CLI tools more in future videos, so subscribe to stay updated!

### Resources (42:30 - 43:00)

**[Screen: Resources list]**

All the code and detailed instructions are available:
- GitHub repo with workflow templates: [Link in description]
- Full written guide: Check the README in the video folder
- CheapestAPP source (my example): [Link]

**[Screen: Call to action]**

If you found this helpful:
- Give it a thumbs up
- Subscribe for more FlutterFlow and CLI tool content
- Drop questions in the comments - I read and respond to all of them!

**[Screen: What's coming next preview]**

Coming soon:
- Deep dive into using Claude Code for Flutter development
- Advanced CI/CD: Multiple environments & feature flags
- Cost optimization for app store deployments

**[Screen: Outro]**

Thanks for watching! Until next time, keep building and automating!

**[End screen with subscribe button and video suggestions]**

---

## B-Roll Suggestions

Throughout the video, overlay these visuals during explanations:
- GitHub Actions running (progress bars, green checkmarks)
- Google Play Console screenshots
- App Store Connect TestFlight page
- Terminal commands executing
- File structure diagrams
- Flowcharts of the deployment process
- Before/after comparisons (manual vs automated)

## Graphics/Animations Needed

- Workflow diagram (FlutterFlow → GitHub → App Stores)
- Timeline showing deployment progress
- Security best practices infographic
- Troubleshooting decision tree
- Architecture diagram

## Notes for Editing

- Speed up long build processes (show key steps only)
- Use picture-in-picture for terminal commands
- Highlight cursor for important clicks
- Add timestamps in description
- Include error examples with solutions overlaid
- Use consistent color scheme (GitHub black, Flutter blue, Android green, iOS gray)
