# Building GreenTriangle with Custom Hermes (TextDecoder Support)

This document explains how to build the GreenTriangle Android app with this custom Hermes that includes TextDecoder support.

## Overview

React Native 0.79 uses Hermes as its JavaScript engine. The standard Hermes build doesn't include `TextDecoder` support. This branch (`rn079-textdecoder`) adds TextDecoder by cherry-picking the relevant commit onto the RN 0.79-compatible Hermes base.

## Branch Information

- **Branch**: `rn079-textdecoder`
- **Base**: Hermes commit `7f9a871ee` (the version used by RN 0.79.x)
- **Cherry-picked**: TextDecoder commit `b450ba9a8`

## Prerequisites

### 1. Java 17

Gradle 7.4 (used by Hermes and RN) doesn't support Java 21+. Install Java 17:

```bash
# macOS with Homebrew
brew install openjdk@17

# Set JAVA_HOME (add to ~/.zshrc or ~/.bashrc)
export JAVA_HOME="/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home"
```

### 2. Android SDK

Install Android SDK with required components:

```bash
# Set ANDROID_HOME
export ANDROID_HOME="$HOME/Library/Android/sdk"

# Install required components
$ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager \
    "platforms;android-35" \
    "build-tools;35.0.0" \
    "ndk;27.1.12297006" \
    "platform-tools" \
    "cmake;3.22.1"
```

### 3. CMake Host Build

Hermes requires a host (macOS) build of `hermesc` compiler before building for Android. Build it once:

```bash
cd ~/rn/hermes
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --target hermesc -j$(sysctl -n hw.ncpu)
```

This creates `build/ImportHermesc.cmake` which is needed by the Android build.

## Building the GT App

### Quick Start

```bash
cd /path/to/agro/app/android
./build-with-custom-hermes.sh
```

### Manual Build

```bash
cd /path/to/agro/app/android
REACT_NATIVE_OVERRIDE_HERMES_DIR=~/rn/hermes ./gradlew assembleDebug
```

The key is the `REACT_NATIVE_OVERRIDE_HERMES_DIR` environment variable - it tells React Native to build Hermes from your custom source instead of downloading the prebuilt version.

## How It Works

React Native's build system checks for `REACT_NATIVE_OVERRIDE_HERMES_DIR`. When set:

1. Instead of using the prebuilt `hermes-android` artifact from Maven
2. RN builds Hermes from source using the specified directory
3. The build uses CMake with Android NDK cross-compilation
4. The resulting `libhermes.so` includes your custom changes

This is the official way to use a custom Hermes - it ensures ABI compatibility with `libhermestooling.so` and other React Native native code.

## Troubleshooting

### "Unsupported class file major version 65"
You're using Java 21+. Switch to Java 17.

### "Hermes host build not found"
Run the CMake host build (see Prerequisites step 3).

### "cannot locate symbol" crashes at runtime
Don't try to build Hermes AAR separately and publish to Maven. Use `REACT_NATIVE_OVERRIDE_HERMES_DIR` instead.

### Build is slow
First build compiles Hermes from source for all architectures (~15-20 min). Subsequent builds are cached.

## Testing TextDecoder

After installing the app, you can verify TextDecoder works:

```javascript
// In React Native
const decoder = new TextDecoder('utf-8');
const bytes = new Uint8Array([72, 101, 108, 108, 111]);
console.log(decoder.decode(bytes)); // "Hello"
```
