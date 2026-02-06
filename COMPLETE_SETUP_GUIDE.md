# VisualVerses Complete Setup Guide

## Current Status

You've successfully:
1. Created a fully functional React Native 0.83 app with all core features implemented
2. Integrated Bible API for verse retrieval
3. Integrated Replicate API for AI image generation
4. Implemented history and favorites functionality with local storage
5. Created a beautiful Bauhaus-inspired UI design
6. Set up proper navigation between screens

## Your Environment

| Component | Status |
|---|---|
| OS | Fedora 43 (Linux) |
| Node.js | v22.20.0 (meets >=20 requirement) |
| System Java | OpenJDK 25.0.1 (too new -- need JDK 17) |
| JDK 17 | Installed at `/home/claudworkpc/jdk-17.0.18+8` |
| Android SDK | **NOT INSTALLED** -- needs setup |
| ADB | Installed (`/usr/bin/adb`) |
| Connected Device | Yes (`48231FDAQ00414`) -- Android 16 (API 36) |
| node_modules | Installed |

## Next Steps to Run on Android Device

### Option 1: Complete Android Build (Recommended)

#### Step 1: Set Java 17 as active JDK

React Native 0.83 requires JDK 17. You have JDK 17 installed but your system defaults to JDK 25. Set it for your current session (and add to `~/.bashrc` for persistence):

```bash
export JAVA_HOME=/home/claudworkpc/jdk-17.0.18+8
export PATH=$JAVA_HOME/bin:$PATH

# Verify:
java -version  # Should show: openjdk version "17.0.18"
```

To make this permanent, add those two `export` lines to `~/.bashrc`:
```bash
echo 'export JAVA_HOME=/home/claudworkpc/jdk-17.0.18+8' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

#### Step 2: Install Android SDK Command Line Tools

Since Android Studio is not available in Fedora repos, download the SDK command-line tools directly:

```bash
# Create the SDK directory
mkdir -p ~/Android/Sdk/cmdline-tools

# Download the latest command-line tools from Google
# Go to: https://developer.android.com/studio#command-line-tools-only
# Or download directly:
cd /tmp
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip

# Extract and place in the correct directory structure
unzip commandlinetools-linux-*.zip
mv cmdline-tools ~/Android/Sdk/cmdline-tools/latest
```

#### Step 3: Set Android environment variables

```bash
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/emulator
```

Add these to `~/.bashrc` for persistence:
```bash
echo 'export ANDROID_HOME=$HOME/Android/Sdk' >> ~/.bashrc
echo 'export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin' >> ~/.bashrc
echo 'export PATH=$PATH:$ANDROID_HOME/platform-tools' >> ~/.bashrc
echo 'export PATH=$PATH:$ANDROID_HOME/emulator' >> ~/.bashrc
source ~/.bashrc
```

#### Step 4: Install required SDK packages

Use `sdkmanager` to install the platforms and build tools required by React Native 0.83 targeting Android 16 (API 36):

```bash
# Accept licenses first
yes | sdkmanager --licenses

# Install required packages:
# - Android 15 SDK Platform 35 (required by React Native 0.83 for building)
# - Android 16 SDK Platform 36 (your device's API level & compileSdkVersion)
# - Build tools 36.0.0
# - Platform tools (adb, fastboot -- you already have adb system-wide)
sdkmanager \
  "platform-tools" \
  "platforms;android-35" \
  "platforms;android-36" \
  "build-tools;36.0.0" \
  "ndk;27.1.12297006"
```

#### Step 5: Verify local.properties

The file `android/local.properties` should point to your SDK:
```bash
# It should already contain:
# sdk.dir=/home/claudworkpc/Android/Sdk
cat /home/claudworkpc/Documents/projects/ideas/VisualVerses/VisualVerses/android/local.properties
```

If the path is wrong, fix it:
```bash
echo "sdk.dir=/home/claudworkpc/Android/Sdk" > \
  /home/claudworkpc/Documents/projects/ideas/VisualVerses/VisualVerses/android/local.properties
```

#### Step 6: Build and install the app

```bash
cd /home/claudworkpc/Documents/projects/ideas/VisualVerses/VisualVerses

# Make sure Java 17 and Android SDK are in PATH (from steps 1 & 3)
export JAVA_HOME=/home/claudworkpc/jdk-17.0.18+8
export PATH=$JAVA_HOME/bin:$PATH
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Run the install script
./install-on-device.sh
```

Or manually:
```bash
cd /home/claudworkpc/Documents/projects/ideas/VisualVerses/VisualVerses

# Bundle the JS
npx react-native bundle \
  --platform android \
  --dev false \
  --entry-file index.js \
  --bundle-output android/app/src/main/assets/index.android.bundle \
  --assets-dest android/app/src/main/res/

# Build the APK
cd android
./gradlew assembleDebug

# Install on device
adb install app/build/outputs/apk/debug/app-debug.apk

# Launch the app
adb shell am start -n com.visualverses/.MainActivity
```

### Option 2: Use Expo (Easier, good for development)

Since Expo 54 is already in your dependencies, you can use Expo for quick iteration:

1. **Install Expo Go on your Android device**:
   - Download from Google Play Store

2. **Start the development server**:
   ```bash
   cd /home/claudworkpc/Documents/projects/ideas/VisualVerses/VisualVerses
   npx expo start
   ```

3. **Scan the QR code with Expo Go** on your Android device

Note: Expo Go may have limitations with some native modules. For full functionality, use the native build (Option 1).

## Build Configuration

### Android target versions (in `android/build.gradle`)

| Setting | Value | Meaning |
|---|---|---|
| `compileSdkVersion` | 36 | Android 16 (Baklava) |
| `targetSdkVersion` | 36 | Android 16 (Baklava) |
| `minSdkVersion` | 24 | Android 7.0 (Nougat) minimum |
| `buildToolsVersion` | 36.0.0 | Latest build tools |
| `ndkVersion` | 27.1.12297006 | Native Development Kit |
| `kotlinVersion` | 2.1.20 | Kotlin compiler |

### Gradle

- Gradle wrapper: **8.13** (in `gradle-wrapper.properties`)
- Android Gradle Plugin: managed by React Native Gradle Plugin

## Environment Variables

For full functionality, set these environment variables:
```bash
export REPLICATE_API_KEY=your_replicate_api_key_here
```

You can get a Replicate API key from: https://replicate.com/

## Troubleshooting

### Java Issues
If you get errors about unsupported class file version or Java compatibility:
```bash
# Ensure JDK 17 is active (NOT the system JDK 25)
export JAVA_HOME=/home/claudworkpc/jdk-17.0.18+8
export PATH=$JAVA_HOME/bin:$PATH
java -version  # Must show Java 17, NOT 25
```

React Native 0.83 recommends JDK 17. Higher versions (like 25) will cause build failures.

### Gradle Issues
If Gradle fails, try cleaning:
```bash
cd android
./gradlew clean
./gradlew assembleDebug
```

If you get "SDK location not found", verify `android/local.properties` has the correct `sdk.dir` path.

### ADB Issues
Your device is already connected (`48231FDAQ00414`). If it stops being recognized:
1. Ensure USB Debugging is enabled (Settings > Developer Options > USB Debugging)
2. Try different USB cables
3. Try different USB ports
4. Run `adb kill-server && adb start-server`
5. Check `adb devices` -- device should show as "device" not "unauthorized"

### Missing SDK Platform
If you get errors about missing SDK platform:
```bash
sdkmanager "platforms;android-36" "platforms;android-35"
```

## App Features

The app is fully functional with:
- Bible verse lookup by book, chapter, and verse
- AI-powered image generation from verse text (via Replicate/Stable Diffusion)
- History tracking with timestamp and favorites
- Local storage persistence with AsyncStorage
- Beautiful Bauhaus-inspired UI design
- Tab navigation (Home, History, Settings)

## Quick Reference

```bash
# Full environment setup in one block:
export JAVA_HOME=/home/claudworkpc/jdk-17.0.18+8
export PATH=$JAVA_HOME/bin:$PATH
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Then build and install:
cd /home/claudworkpc/Documents/projects/ideas/VisualVerses/VisualVerses
./install-on-device.sh
```
