# Setup Guide - bMOI Mobile App

Complete setup guide from zero to running app (iOS + Android).

## Prerequisites

### Required Software

| Tool | Minimum Version | Purpose |
|------|----------------|---------|
| **Flutter SDK** | 3.19+ | Framework |
| **Dart** | 3.3+ | Language (comes with Flutter) |
| **Android Studio** | Latest | Android development |
| **Xcode** | 15+ (macOS only) | iOS development |
| **VS Code** | Latest | Code editor (recommended) |
| **Git** | 2.x | Version control |

### Verify Installation

```bash
# Check Flutter installation
flutter doctor -v

# Should show ✓ for:
# ✓ Flutter (3.19.0 or higher)
# ✓ Android toolchain
# ✓ Xcode (macOS only)
# ✓ VS Code or Android Studio
```

---

## Step 1: Install Flutter SDK

### macOS

```bash
# Download Flutter SDK
cd ~/development
git clone https://github.com/flutter/flutter.git -b stable

# Add to PATH
export PATH="$PATH:`pwd`/flutter/bin"

# Add to ~/.zshrc or ~/.bash_profile
echo 'export PATH="$PATH:$HOME/development/flutter/bin"' >> ~/.zshrc

# Verify
flutter doctor
```

### Windows

1. Download Flutter SDK from https://docs.flutter.dev/get-started/install/windows
2. Extract to `C:\src\flutter`
3. Add to PATH: `C:\src\flutter\bin`
4. Verify: `flutter doctor`

### Linux

```bash
# Download
cd ~/development
git clone https://github.com/flutter/flutter.git -b stable

# Add to PATH
export PATH="$PATH:$HOME/development/flutter/bin"
echo 'export PATH="$PATH:$HOME/development/flutter/bin"' >> ~/.bashrc

# Verify
flutter doctor
```

---

## Step 2: Setup Android Development

### Install Android Studio

1. Download from https://developer.android.com/studio
2. Run installer
3. Open Android Studio
4. Go to: Settings → Plugins → Install "Flutter" and "Dart" plugins

### Configure Android SDK

```bash
# Accept licenses
flutter doctor --android-licenses

# Should show:
# ✓ Android toolchain - develop for Android devices
```

### Create Android Emulator

```bash
# List available devices
flutter emulators

# Create emulator
flutter emulators --create

# Or use Android Studio:
# Tools → Device Manager → Create Device
```

---

## Step 3: Setup iOS Development (macOS only)

### Install Xcode

1. Install from App Store (15GB+, takes time)
2. Open Xcode
3. Agree to license: `sudo xcodebuild -license`

### Install CocoaPods

```bash
# Install CocoaPods (dependency manager for iOS)
sudo gem install cocoapods

# Verify
pod --version
```

### Setup iOS Simulator

```bash
# Open simulator
open -a Simulator

# Or from Xcode:
# Xcode → Open Developer Tool → Simulator
```

---

## Step 4: Create Flutter Project

```bash
# Create project
flutter create bmoi_mobile_app --org com.bmoi

# Navigate to project
cd bmoi_mobile_app

# Test run
flutter run
# (Select device when prompted)
```

---

## Step 5: Add Dependencies

### Update `pubspec.yaml`

```yaml
name: bmoi_mobile_app
description: bMOI Mobile App
version: 1.0.0+1

environment:
  sdk: '>=3.3.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter

  # State Management
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0

  # Routing
  go_router: ^13.0.0

  # HTTP Client
  dio: ^5.4.0

  # Code Generation & Immutability
  freezed_annotation: ^2.4.1
  json_annotation: ^4.8.1

  # Storage
  shared_preferences: ^2.2.2
  flutter_secure_storage: ^9.0.0
  hive: ^2.2.3
  hive_flutter: ^1.1.0

  # Utils
  intl: ^0.19.0
  logger: ^2.0.2
  dartz: ^0.10.1

  # Firebase
  firebase_core: ^2.24.2
  firebase_analytics: ^10.8.0
  firebase_crashlytics: ^3.4.9

dev_dependencies:
  flutter_test:
    sdk: flutter

  # Code Generation
  build_runner: ^2.4.8
  riverpod_generator: ^2.3.0
  freezed: ^2.4.6
  json_serializable: ^6.7.1
  hive_generator: ^2.0.1

  # Linting
  flutter_lints: ^3.0.1
  very_good_analysis: ^5.1.0

flutter:
  uses-material-design: true
```

### Install Dependencies

```bash
flutter pub get
```

---

## Step 6: Configure Project Structure

```bash
# Create folders
mkdir -p lib/{core,features,shared}
mkdir -p lib/core/{network,storage,theme,constants,utils}
mkdir -p lib/features/{auth,dashboard,products,orders,profile}
mkdir -p lib/shared/{widgets,services}

# Create data/domain/presentation for auth
mkdir -p lib/features/auth/{data,domain,presentation}
mkdir -p lib/features/auth/data/{models,repositories,data_sources}
mkdir -p lib/features/auth/domain/{entities,repositories,usecases}
mkdir -p lib/features/auth/presentation/{providers,screens,widgets}
```

---

## Step 7: Configure Dio HTTP Client

### Create `lib/core/network/dio_client.dart`

```dart
import 'package:dio/dio.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class DioClient {
  late final Dio dio;
  final FlutterSecureStorage secureStorage;

  DioClient({required this.secureStorage}) {
    dio = Dio(BaseOptions(
      baseUrl: 'https://api.bmoi.com',
      connectTimeout: const Duration(seconds: 30),
      receiveTimeout: const Duration(seconds: 30),
    ));

    dio.interceptors.addAll([
      AuthInterceptor(secureStorage),
      TenantInterceptor(secureStorage),
      LogInterceptor(
        requestBody: true,
        responseBody: true,
      ),
    ]);
  }
}

class AuthInterceptor extends Interceptor {
  final FlutterSecureStorage secureStorage;

  AuthInterceptor(this.secureStorage);

  @override
  void onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    final token = await secureStorage.read(key: 'jwt_token');
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  }
}

class TenantInterceptor extends Interceptor {
  final FlutterSecureStorage secureStorage;

  TenantInterceptor(this.secureStorage);

  @override
  void onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    final tenantId = await secureStorage.read(key: 'tenant_id');
    if (tenantId != null) {
      options.headers['X-Tenant-ID'] = tenantId;
    }
    handler.next(options);
  }
}
```

---

## Step 8: Configure Theme

### Create `lib/core/theme/app_theme.dart`

```dart
import 'package:flutter/material.dart';

class AppTheme {
  static final light = ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.blue,
      brightness: Brightness.light,
    ),
    useMaterial3: true,
    appBarTheme: const AppBarTheme(
      centerTitle: true,
      elevation: 0,
    ),
  );

  static final dark = ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.blue,
      brightness: Brightness.dark,
    ),
    useMaterial3: true,
    appBarTheme: const AppBarTheme(
      centerTitle: true,
      elevation: 0,
    ),
  );
}
```

---

## Step 9: Update Main Entry Point

### Update `lib/main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'core/theme/app_theme.dart';

void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'bMOI',
      theme: AppTheme.light,
      darkTheme: AppTheme.dark,
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('bMOI Mobile'),
      ),
      body: const Center(
        child: Text('Welcome to bMOI!'),
      ),
    );
  }
}
```

---

## Step 10: Run the App

### Android

```bash
# List devices
flutter devices

# Run on Android emulator
flutter run

# Or specify device
flutter run -d <device-id>
```

### iOS (macOS only)

```bash
# Open iOS simulator first
open -a Simulator

# Run app
flutter run -d iPhone

# Or from VS Code: F5
```

---

## Step 11: Code Generation Setup

```bash
# Generate code (Freezed, Riverpod, JSON Serializable)
dart run build_runner build --delete-conflicting-outputs

# Watch mode (auto-generate on save)
dart run build_runner watch --delete-conflicting-outputs
```

---

## Step 12: Configure Firebase (Optional)

### Install FlutterFire CLI

```bash
dart pub global activate flutterfire_cli
```

### Configure Firebase

```bash
# Login to Firebase
firebase login

# Initialize Firebase in project
flutterfire configure
# Select your Firebase project
# Select platforms (iOS, Android)
```

This creates `firebase_options.dart` automatically.

### Update `lib/main.dart`

```dart
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const ProviderScope(child: MyApp()));
}
```

---

## Troubleshooting

### Issue: CocoaPods not found (iOS)

```bash
sudo gem install cocoapods
pod setup
```

### Issue: Android licenses not accepted

```bash
flutter doctor --android-licenses
# Accept all licenses
```

### Issue: Build fails on iOS

```bash
cd ios
pod deintegrate
pod install
cd ..
flutter clean
flutter pub get
flutter run
```

### Issue: Hot reload not working

```bash
flutter clean
flutter pub get
flutter run
```

---

## Useful Commands

```bash
# Development
flutter run                    # Run app
flutter run -d <device-id>     # Run on specific device
flutter run --release          # Run in release mode

# Build
flutter build apk              # Build APK (Android)
flutter build appbundle        # Build App Bundle (Google Play)
flutter build ios              # Build iOS (macOS only)

# Testing
flutter test                   # Run all tests
flutter test --coverage        # Generate coverage

# Code quality
flutter analyze                # Static analysis
dart format .                  # Format code

# Utilities
flutter devices                # List connected devices
flutter emulators              # List available emulators
flutter clean                  # Clean build files
flutter doctor                 # Check installation
```

---

## Next Steps

After completing setup:

1. ✅ **Read [ARQUITECTURA.md](ARQUITECTURA.md)** - Understand architecture
2. ✅ **Read [CODING_STANDARDS.md](CODING_STANDARDS.md)** - Learn conventions
3. ✅ **Read [EJEMPLOS_CODIGO.md](EJEMPLOS_CODIGO.md)** - See code examples
4. ✅ **Start building features** - Begin with auth module
5. ✅ **Setup CI/CD** - Configure GitHub Actions (optional)

---

**Setup complete! 🚀**

You now have a fully configured Flutter project ready for development!
