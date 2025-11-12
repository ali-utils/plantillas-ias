# bMOI Mobile App - Project Overview

## 🎯 Project Purpose

The bMOI Mobile App is a **cross-platform mobile application** (iOS + Android) designed to:

- **Provide mobile access** to the bMOI platform
- **Support offline-first** capabilities for critical features
- **Deliver native performance** on both iOS and Android
- **Integrate seamlessly** with multi-tenant backend
- **Offer excellent UX** with Material Design + Cupertino widgets

## 🏗️ Architecture

### Clean Architecture (Feature-First)

```
Presentation Layer (UI, State)
        ↕️
Domain Layer (Business Logic)
        ↕️
Data Layer (API, Local Storage)
```

**Benefits:**
- ✅ **Testable** - Each layer independent
- ✅ **Maintainable** - Clear separation of concerns
- ✅ **Scalable** - Easy to add features
- ✅ **Team-friendly** - Different teams on different features

### Feature-First Organization

```
lib/features/auth/
├── data/            # API clients, models, repo implementations
├── domain/          # Entities, repo interfaces, use cases
└── presentation/    # Providers, screens, widgets
```

**Why feature-first?**
- All auth code in one place
- Easy to find and modify
- Self-contained modules
- Better team scalability

## 🛠️ Technology Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| **Flutter** | 3.19+ | Cross-platform framework |
| **Dart** | 3.3+ | Programming language |
| **Riverpod** | 2.5+ | State management |
| **Go Router** | 13+ | Navigation |
| **Dio** | 5+ | HTTP client |
| **Freezed** | 2.4+ | Immutable models |
| **Hive** | 2.2+ | Local storage |
| **Flutter Secure Storage** | 9.0+ | Secure storage (JWT) |

### Why Flutter?

✅ **Single codebase** - iOS + Android with one code
✅ **Native performance** - Compiled to native ARM
✅ **Hot reload** - Instant feedback (dev productivity++)
✅ **Beautiful UI** - Material + Cupertino widgets
✅ **Strong typing** - Dart null safety
✅ **Cost-effective** - No separate iOS/Android teams

**Comparison:**

| Framework | Performance | Dev Speed | Ecosystem | Native Feel |
|-----------|-------------|-----------|-----------|-------------|
| **Flutter** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| React Native | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Native (2 codebases) | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

### Why Riverpod?

✅ **Compile-time safety** - Errors caught at compile time
✅ **No BuildContext** - Access state anywhere
✅ **Auto-disposal** - Automatic memory management
✅ **Testable** - Easy to mock providers
✅ **Code generation** - Less boilerplate

## 📦 Project Structure

```
lib/
├── core/                       # Shared functionality
│   ├── network/
│   │   ├── dio_client.dart    # Configured Dio instance
│   │   ├── interceptors/      # Auth, Tenant, Error interceptors
│   │   └── api_client.dart    # Base API client
│   ├── storage/
│   │   ├── secure_storage.dart
│   │   └── local_storage.dart
│   ├── theme/
│   │   ├── app_theme.dart
│   │   └── app_colors.dart
│   ├── constants/
│   │   ├── api_constants.dart
│   │   └── app_constants.dart
│   └── utils/
│       ├── logger.dart
│       └── validators.dart
│
├── features/                   # Feature modules
│   ├── auth/
│   │   ├── data/
│   │   │   ├── models/
│   │   │   │   └── user_model.dart        # JSON model
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository_impl.dart
│   │   │   └── data_sources/
│   │   │       ├── auth_remote_data_source.dart
│   │   │       └── auth_local_data_source.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── user.dart              # Domain entity
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart   # Interface
│   │   │   └── usecases/
│   │   │       ├── login_usecase.dart
│   │   │       └── logout_usecase.dart
│   │   └── presentation/
│   │       ├── providers/
│   │       │   └── auth_provider.dart
│   │       ├── screens/
│   │       │   ├── login_screen.dart
│   │       │   └── register_screen.dart
│   │       └── widgets/
│   │           └── login_form.dart
│   │
│   ├── dashboard/              # Dashboard feature
│   ├── products/               # Products feature
│   ├── orders/                 # Orders feature
│   └── profile/                # Profile feature
│
├── shared/                     # Shared UI components
│   ├── widgets/
│   │   ├── custom_button.dart
│   │   ├── custom_text_field.dart
│   │   └── loading_indicator.dart
│   └── services/
│       └── connectivity_service.dart
│
└── main.dart                   # App entry point
```

## 🎯 Key Features

### 1. Authentication
- Login with email/password
- Biometric authentication (Face ID, Fingerprint)
- JWT token management
- Auto-refresh tokens
- Secure storage

### 2. Dashboard
- Overview metrics
- Recent activity
- Quick actions
- Notifications

### 3. Products Management
- List all products
- View product details
- Add/Edit products
- Image upload
- Barcode scanning (optional)

### 4. Orders Management
- List all orders
- View order details
- Create orders
- Update order status
- Order history

### 5. User Profile
- View/edit profile
- Settings
- Tenant switcher (multi-tenant users)
- Theme switcher
- Logout

## 🔄 Multi-Tenant Integration

### Auto-Header Injection

```dart
// All API requests automatically include:
Authorization: Bearer {jwt}
X-Tenant-ID: {tenantId}

// Dio interceptor handles this automatically
class AuthInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) async {
    final token = await secureStorage.read(key: 'jwt_token');
    final tenantId = await secureStorage.read(key: 'tenant_id');

    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }

    if (tenantId != null) {
      options.headers['X-Tenant-ID'] = tenantId;
    }

    handler.next(options);
  }
}
```

### Offline-First Architecture

```
User Action → Check Network
               ↓
        ┌──────┴──────┐
        ↓             ↓
    Online        Offline
        ↓             ↓
   API Call      Cache Read
        ↓             ↓
    Save Cache    Queue Write
        ↓             ↓
    Show Data     Show Data
                      ↓
              Sync when online
```

## 🚀 Performance Targets

### App Performance

| Metric | Target | Importance |
|--------|--------|------------|
| **Cold start** | < 2s | Critical |
| **Hot start** | < 1s | Critical |
| **Frame rate** | 60 FPS | Critical |
| **Memory** | < 200MB | Important |
| **APK size** | < 50MB | Important |

### Best Practices

- ✅ Use `const` widgets where possible
- ✅ Implement lazy loading for lists
- ✅ Optimize images (compress, cache)
- ✅ Use `ListView.builder` for large lists
- ✅ Avoid unnecessary rebuilds
- ✅ Profile with DevTools

## 🔐 Security

### Secure Storage

```dart
// Store sensitive data
await secureStorage.write(key: 'jwt_token', value: token);
await secureStorage.write(key: 'tenant_id', value: tenantId);

// Read sensitive data
final token = await secureStorage.read(key: 'jwt_token');

// Delete on logout
await secureStorage.delete(key: 'jwt_token');
await secureStorage.deleteAll();
```

### Certificate Pinning (Optional)

```dart
// For production apps
class CertificatePinning {
  static Future<Dio> createDio() async {
    final dio = Dio();
    (dio.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate =
      (client) {
        client.badCertificateCallback =
          (cert, host, port) => validateCertificate(cert, host);
        return client;
      };
    return dio;
  }
}
```

### Code Obfuscation

```bash
# Build with obfuscation (production)
flutter build apk --obfuscate --split-debug-info=build/debug-info
flutter build ios --obfuscate --split-debug-info=build/debug-info
```

## 📱 Platform-Specific Features

### iOS
- ✅ Face ID / Touch ID
- ✅ iOS share sheet
- ✅ Apple Pay (future)
- ✅ Cupertino widgets where appropriate

### Android
- ✅ Fingerprint authentication
- ✅ Android share sheet
- ✅ Google Pay (future)
- ✅ Material Design 3

## 🌍 Internationalization (i18n) - Future

**Initial:** Spanish only
**Future:** English, Portuguese

```dart
MaterialApp(
  localizationsDelegates: [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
  ],
  supportedLocales: [
    Locale('es', ''),  // Spanish
    Locale('en', ''),  // English
  ],
);
```

## 🧪 Testing Strategy

```
Unit Tests (80%)       → Business logic, use cases
        ↓
Widget Tests (15%)     → UI components
        ↓
Integration Tests (5%) → Full user flows
```

**Coverage target:** 80% minimum

## 🚢 Deployment

### Android (Google Play)

```bash
# Build App Bundle
flutter build appbundle --release

# Output: build/app/outputs/bundle/release/app-release.aab
```

### iOS (App Store)

```bash
# Build IPA (macOS only)
flutter build ios --release

# Open Xcode for archive
open ios/Runner.xcworkspace
```

## 📊 Monitoring

### Firebase Services

- **Analytics** - User behavior tracking
- **Crashlytics** - Crash reporting
- **Cloud Messaging** - Push notifications
- **Remote Config** - Feature flags (optional)

### Key Metrics to Track

- Daily active users (DAU)
- Session duration
- Crash-free rate (> 99%)
- API response times
- Network errors

## 📚 Related Documentation

- **[SETUP.md](SETUP.md)** - Step-by-step setup guide
- **[ARQUITECTURA.md](ARQUITECTURA.md)** - Architecture decisions
- **[DEPLOYMENT.md](DEPLOYMENT.md)** - Deployment guides
- **[CODING_STANDARDS.md](CODING_STANDARDS.md)** - Coding standards
- **[EJEMPLOS_CODIGO.md](EJEMPLOS_CODIGO.md)** - Code examples

---

**Version:** 1.0.0
**Last Updated:** 2025-01-04

<div align="center">

**Cross-Platform Mobile App with Flutter**

iOS + Android | Clean Architecture | Offline-First

</div>
