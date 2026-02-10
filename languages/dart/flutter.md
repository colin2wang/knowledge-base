# Flutter Development Environment

Flutter is Google's UI toolkit for building natively compiled applications for mobile, web, and desktop from a single codebase. Proper environment setup is crucial for efficient Flutter development.

## Environment Configuration

### China Mirror Configuration

For developers in China, configuring domestic mirrors significantly improves package download speeds.

#### Linux/macOS
```bash
# Add to ~/.bashrc or ~/.zshrc
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn

# Reload configuration
source ~/.bashrc
# or
source ~/.zshrc
```

#### Windows
```cmd
# Command Prompt
set PUB_HOSTED_URL=https://pub.flutter-io.cn
set FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn

# PowerShell
$env:PUB_HOSTED_URL="https://pub.flutter-io.cn"
$env:FLUTTER_STORAGE_BASE_URL="https://storage.flutter-io.cn"

# Permanent PowerShell configuration
[Environment]::SetEnvironmentVariable("PUB_HOSTED_URL", "https://pub.flutter-io.cn", "User")
[Environment]::SetEnvironmentVariable("FLUTTER_STORAGE_BASE_URL", "https://storage.flutter-io.cn", "User")
```

#### Verification
```bash
# Check current configuration
flutter config

# List all environment variables
flutter doctor -v
```

## Flutter SDK Management

### Installation

#### Using Flutter CLI
```bash
# Download Flutter SDK
wget https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_3.10.0-stable.tar.xz

# Extract
tar xf flutter_linux_3.10.0-stable.tar.xz

# Add to PATH
export PATH="$PATH:`pwd`/flutter/bin"
```

#### Using Package Managers
```bash
# macOS with Homebrew
brew install flutter

# Windows with Chocolatey
choco install flutter

# Linux with Snap
sudo snap install flutter --classic
```

### Version Management

#### Switching Flutter Channels
```bash
# List available channels
flutter channel

# Switch to stable channel
flutter channel stable

# Switch to beta channel
flutter channel beta

# Upgrade to latest version
flutter upgrade
```

#### Managing Multiple Versions
```bash
# Using FVM (Flutter Version Management)
# Install FVM
dart pub global activate fvm

# Install specific Flutter version
fvm install 3.10.0

# Use specific version in project
fvm use 3.10.0

# List installed versions
fvm list
```

## Development Tools Setup

### IDE Configuration

#### Android Studio
1. Install Flutter and Dart plugins
2. Configure Flutter SDK path
3. Set up Android SDK
4. Configure device emulators

#### Visual Studio Code
```json
// .vscode/settings.json
{
  "dart.flutterSdkPath": "/path/to/flutter",
  "dart.pubAdditionalArgs": [],
  "dart.vmAdditionalArgs": [],
  "dart.debugExternalLibraries": true,
  "dart.debugSdkLibraries": false
}
```

### Device Setup

#### Android Emulator
```bash
# List available AVDs
flutter emulators

# Launch emulator
flutter emulators --launch <emulator_name>

# Create new AVD
avdmanager create avd -n <name> -k "system-images;android-33;google_apis;x86_64"
```

#### iOS Simulator
```bash
# Launch iOS simulator
open -a Simulator

# List connected devices
flutter devices
```

## Project Configuration

### pubspec.yaml Management

#### Dependencies
```yaml
name: my_flutter_app
description: A new Flutter project.
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.0.0 <4.0.0'
  flutter: '>=3.10.0'

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.2
  http: ^0.15.0
  provider: ^6.1.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^2.0.0

flutter:
  uses-material-design: true
  assets:
    - assets/images/
    - assets/icons/
  fonts:
    - family: CustomFont
      fonts:
        - asset: assets/fonts/CustomFont-Regular.ttf
```

#### Environment-Specific Configurations
```yaml
# pubspec.yaml
flutter:
  assets:
    - assets/config/$env$/app_config.json

# Create environment-specific files
assets/config/dev/app_config.json
assets/config/prod/app_config.json
assets/config/staging/app_config.json
```

### Build Configuration

#### Android
```gradle
// android/app/build.gradle
android {
  compileSdkVersion 33
  
  defaultConfig {
    applicationId "com.example.myapp"
    minSdkVersion 21
    targetSdkVersion 33
    versionCode flutterVersionCode.toInteger()
    versionName flutterVersionName
  }
  
  buildTypes {
    release {
      signingConfig signingConfigs.release
    }
  }
}
```

#### iOS
```xml
<!-- ios/Runner.xcodeproj/project.pbxproj -->
/* Set deployment target */
IPHONEOS_DEPLOYMENT_TARGET = 11.0;

/* Enable bitcode for App Store */
ENABLE_BITCODE = YES;
```

## Performance Optimization

### Build Performance

#### Enable Build Caching
```bash
# Enable build cache
flutter config --enable-build-cache

# Clear build cache when needed
flutter clean
```

#### Fast Development Mode
```bash
# Hot reload development
flutter run --hot

# Profile mode for performance testing
flutter run --profile

# Release mode for final builds
flutter run --release
```

### Code Optimization

#### Analysis Options
```yaml
# analysis_options.yaml
analyzer:
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"
  errors:
    unused_import: warning
    unused_local_variable: warning

linter:
  rules:
    - prefer_const_constructors
    - prefer_final_fields
    - prefer_relative_imports
    - unnecessary_new
```

## Testing and Debugging

### Testing Setup

#### Unit Testing
```dart
// test/widget_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/main.dart';

void main() {
  testWidgets('Counter increments smoke test', (WidgetTester tester) async {
    // Build our app and trigger a frame.
    await tester.pumpWidget(const MyApp());

    // Verify that our counter starts at 0.
    expect(find.text('0'), findsOneWidget);
    expect(find.text('1'), findsNothing);

    // Tap the '+' icon and trigger a frame.
    await tester.tap(find.byIcon(Icons.add));
    await tester.pump();

    // Verify that our counter has incremented.
    expect(find.text('0'), findsNothing);
    expect(find.text('1'), findsOneWidget);
  });
}
```

#### Integration Testing
```dart
// integration_test/app_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('end-to-end test', () {
    testWidgets('tap on the floating action button; verify counter',
        (tester) async {
      app.main();
      await tester.pumpAndSettle();

      // Finds the floating action button to tap on.
      final Finder fab = find.byTooltip('Increment');

      // Emulates a tap on the floating action button.
      await tester.tap(fab);

      // Trigger a frame.
      await tester.pumpAndSettle();

      // Verify the counter increments by 1.
      expect(find.text('1'), findsOneWidget);
    });
  });
}
```

### Debugging Tools

#### DevTools
```bash
# Launch DevTools
flutter pub global activate devtools
flutter pub global run devtools

# Connect to running app
flutter run
# Then open DevTools URL shown in console
```

#### Performance Profiling
```bash
# Run with profiling enabled
flutter run --profile

# Record performance timeline
flutter driver --target=test_driver/app.dart --profile
```

## Deployment Configuration

### Android Signing

#### Generate Keystore
```bash
# Generate keystore
keytool -genkey -v -keystore ~/upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload

# Create key.properties
echo "storePassword=<password>" > android/key.properties
echo "keyPassword=<password>" >> android/key.properties
echo "keyAlias=upload" >> android/key.properties
echo "storeFile=../app/upload-keystore.jks" >> android/key.properties
```

#### Configure Gradle
```gradle
// android/app/build.gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
  signingConfigs {
    release {
      keyAlias keystoreProperties['keyAlias']
      keyPassword keystoreProperties['keyPassword']
      storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
      storePassword keystoreProperties['storePassword']
    }
  }
}
```

### iOS Configuration

#### Update Info.plist
```xml
<!-- ios/Runner/Info.plist -->
<key>CFBundleDisplayName</key>
<string>My App</string>
<key>CFBundleIdentifier</key>
<string>com.example.myapp</string>
<key>CFBundleShortVersionString</key>
<string>$(FLUTTER_BUILD_NAME)</string>
<key>CFBundleVersion</key>
<string>$(FLUTTER_BUILD_NUMBER)</string>
```

## Best Practices

### Project Structure
```
lib/
├── main.dart
├── app/
│   ├── app.dart
│   └── routes/
├── core/
│   ├── constants/
│   ├── utils/
│   └── theme/
├── features/
│   ├── authentication/
│   ├── home/
│   └── profile/
└── shared/
    ├── widgets/
    ├── services/
    └── models/
```

### Environment Variables
```dart
// lib/core/constants/env_config.dart
class EnvConfig {
  static const String apiUrl = String.fromEnvironment(
    'API_URL',
    defaultValue: 'https://api.example.com',
  );
  
  static const bool isDebug = bool.fromEnvironment(
    'DEBUG',
    defaultValue: false,
  );
}

// Usage in build
flutter build apk --dart-define=API_URL=https://staging-api.example.com --dart-define=DEBUG=true
```

### CI/CD Integration
```yaml
# .github/workflows/flutter.yml
name: Flutter CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: subosito/flutter-action@v2
      with:
        flutter-version: '3.10.0'
    - run: flutter pub get
    - run: flutter analyze
    - run: flutter test
    - run: flutter build apk --release
```

## Troubleshooting

### Common Issues

#### Dependency Resolution Errors
```bash
# Clear pub cache
flutter pub cache repair

# Downgrade conflicting packages
flutter pub downgrade

# Override dependency versions
flutter pub upgrade --major-versions
```

#### Build Failures
```bash
# Clean build artifacts
flutter clean

# Regenerate platform-specific files
flutter create .

# Update CocoaPods (iOS)
pod install --repo-update
```

#### Performance Issues
```bash
# Enable verbose logging
flutter run -v

# Check for memory leaks
flutter run --profile

# Analyze widget rebuilds
flutter run --debug --observatory-port=8888
```

## Resources

### Official Documentation
- [Flutter Documentation](https://docs.flutter.dev/)
- [Flutter API Reference](https://api.flutter.dev/)
- [Flutter Samples](https://flutter.github.io/samples/)
- [Flutter Codelabs](https://codelabs.developers.google.com/codelabs/flutter/)

### Community Resources
- [Flutter Community](https://flutter.dev/community)
- [Awesome Flutter](https://github.com/Solido/awesome-flutter)
- [Flutter Weekly](https://flutterweekly.net/)
- [Reddit r/FlutterDev](https://www.reddit.com/r/FlutterDev/)
