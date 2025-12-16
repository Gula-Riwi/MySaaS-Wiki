# Setup Guide - MeetLines Mobile (Android)

## Prerequisites

Install the following:

| Requirement | Version |
|-------------|---------|
| Android Studio | Flamingo+ |
| Android SDK | 33+ |
| Gradle | 8.0+ |
| JDK | 17+ |
| Git | 2.0+ |

## Install Android Studio

1. Download from [developer.android.com](https://developer.android.com/studio)
2. Run installer
3. Complete initial setup wizard
4. Install Android SDK components:
   - SDK Platform 33+
   - Build Tools 34+
   - Android Emulator
   - Intel x86 Emulator Accelerator

## Clone Repository

```bash
git clone https://github.com/Gula-Riwi/MeetLines-Mobile.git
cd MeetLines-Mobile

# Verify Android project structure
ls -la
```

## Set Up Environment

### Create `local.properties`

```properties
sdk.dir=/path/to/android/sdk
ndk.dir=/path/to/android/ndk
```

### Create `app/src/main/AndroidManifest.xml` permissions

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_CALENDAR" />
```

## Install Dependencies

```bash
# Build project (downloads Gradle and dependencies)
./gradlew build

# Or on Windows
gradlew.bat build
```

## Run Development

### Connect Android Device

```bash
# Check devices
adb devices

# Enable USB debugging on device
Settings > Developer Options > USB Debugging
```

### Run on Emulator

1. Open Android Studio
2. Tools → Device Manager
3. Create Virtual Device (API 33+)
4. Start emulator

### Build and Run App

```bash
# Install and run on connected device/emulator
./gradlew installDebug

# Or from Android Studio: Run → Run 'app'
```

## Development

### Edit Code

1. Open in Android Studio
2. Navigate to `app/src/main/java/.../`
3. Edit `.kt` files
4. Run on device with hot reload

### Key Directories

```
app/
├── src/main/java/com/Gula/MeetLines/
│   ├── ui/screens/         ← Edit screens here
│   ├── viewmodel/          ← Edit ViewModels here
│   ├── data/repository/    ← Edit repositories here
│   └── model/              ← Edit data models here
└── src/main/res/
    ├── drawable/           ← Add images/icons
    ├── values/strings.xml  ← Add strings
    └── values/colors.xml   ← Add colors
```

## Build for Production

```bash
# Create signed APK
./gradlew bundleRelease

# Output: app/build/outputs/bundle/release/

# To install APK manually
./gradlew assembleRelease

# Output: app/build/outputs/apk/release/app-release.apk
```

## Debugging

### View Logs

```bash
# Real-time logs from device
adb logcat -v brief

# Filter by app
adb logcat *:S com.Gula.MeetLines:V

# Save logs to file
adb logcat > logcat.txt
```

### Debugger

1. Set breakpoint (click on line number)
2. Run → Debug 'app'
3. Logcat shows breakpoint hits
4. Use Variables panel to inspect

### Android Profiler

1. Run → Edit Configurations
2. Enable profiling
3. Tools → Profiler

## Common Issues

### "Cannot find Android SDK"

```bash
# Set in Android Studio
File → Project Structure → SDK Location
# Or set ANDROID_HOME environment variable
```

### "Gradle sync failed"

```bash
# Clean and rebuild
./gradlew clean
./gradlew build
```

### "Emulator won't start"

```bash
# Kill existing emulator
pkill -f emulator

# Restart from Device Manager
```

### "App crashes on startup"

Check logcat for error messages:
```bash
adb logcat | grep "CRASH\|ERROR"
```

## Testing

```bash
# Run unit tests
./gradlew test

# Run instrumented tests (on device)
./gradlew connectedAndroidTest
```

## Performance

### Profiling

- Android Profiler (Memory, CPU, Network)
- Layout Inspector
- Battery Historian

### Optimization

- Use ViewHolders in RecyclerView
- Lazy load images
- Minimize database queries
- Use Coroutines (not threads)

## Deployment to Play Store

1. Create Google Play Console account
2. Build signed APK/AAB
3. Upload to Play Store
4. Set store listing
5. Submit for review

## Resources

- [Android Jetpack Compose](https://developer.android.com/jetpack/compose)
- [Android Development](https://developer.android.com)
- [Gradle Documentation](https://gradle.org/documentation/)
- [Kotlin Documentation](https://kotlinlang.org/docs/)

## Next Steps

1. ✅ Setup complete
2. 🚀 Run: `./gradlew installDebug`
3. 📖 Read technical architecture
4. 💻 Edit screens in `ui/screens/`
5. 🔧 Build and test features

Welcome to MeetLines Mobile!
