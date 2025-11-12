# Mobile App Context - bMOI Project

## Project Identity

**Project Name:** bMOI Mobile App
**Type:** Cross-platform Mobile Application (iOS + Android)
**Repository:** bmoi-mobile-app
**Part of:** bMOI Multi-Tenant Ecosystem
**Status:** Template/Plantilla (Starting from Zero)

## Core Purpose

The bMOI Mobile App is a **cross-platform mobile application** designed to:
- Provide mobile access to the bMOI platform for business users
- Support both iOS and Android with a single codebase
- Integrate with the backend API for multi-tenant data access
- Offer offline-first capabilities for critical features
- Deliver native performance and UX

**This is NOT:**
- A hybrid web app (WebView-based)
- A React Native application
- A native-only iOS or Android app
- A consumer-facing marketplace app

## Technology Stack

### Core Framework: Flutter 3.19+ ✅

**Decision:** Use **Flutter** (not React Native, not Native iOS/Android)

**Why Flutter:**
- ✅ **Single codebase** - iOS + Android with one code
- ✅ **Native performance** - Compiled to native ARM code
- ✅ **Beautiful UI** - Material Design + Cupertino widgets
- ✅ **Hot reload** - Instant feedback during development
- ✅ **Strong typing** - Dart with null safety
- ✅ **Rich ecosystem** - 40,000+ packages on pub.dev
- ✅ **Excellent documentation** - Official docs are comprehensive
- ✅ **Cost-effective** - No need for separate iOS/Android teams

**Alternatives discarded:**
- ❌ React Native: Good but less performant than Flutter
- ❌ Native (Swift/Kotlin): Double development cost
- ❌ Ionic/Capacitor: WebView-based, not truly native

### Language: Dart 3.3+ ✅

**Why Dart:**
- ✅ **Null safety** - Eliminates null pointer errors
- ✅ **Strong typing** - Type-safe at compile time
- ✅ **Async/await** - Clean asynchronous code
- ✅ **Fast compilation** - Quick dev cycles
- ✅ **Modern syntax** - Clean and readable

### State Management: Riverpod 2.5+ ✅

**Decision:** Use **Riverpod** (not Provider, not BLoC, not GetX)

**Why Riverpod:**
- ✅ **Compile-time safety** - No runtime errors
- ✅ **Testable** - Easy to mock and test
- ✅ **No BuildContext required** - Access state anywhere
- ✅ **Auto-disposal** - Automatic cleanup
- ✅ **Code generation** - Less boilerplate with riverpod_generator
- ✅ **DevTools support** - Excellent debugging

**Alternatives discarded:**
- ❌ Provider: Less powerful than Riverpod
- ❌ BLoC: Too much boilerplate
- ❌ GetX: Magic strings, hard to maintain

### Navigation: Go Router 13+ ✅

**Why Go Router:**
- ✅ **Declarative routing** - URL-based navigation
- ✅ **Deep linking** - Native deep link support
- ✅ **Type-safe routes** - No magic strings
- ✅ **Nested navigation** - Complex navigation trees
- ✅ **Redirects & guards** - Authentication flow support

### HTTP Client: Dio 5+ ✅

**Why Dio:**
- ✅ **Interceptors** - Global error handling, auth tokens
- ✅ **Request cancellation** - Cancel in-flight requests
- ✅ **Progress callbacks** - Upload/download progress
- ✅ **Form data** - Easy file uploads
- ✅ **Better error handling** - Than http package

### Local Storage: Hive / Shared Preferences / Flutter Secure Storage ✅

**Strategy:**
- **Hive** - Structured local data (user profiles, cached data)
- **Shared Preferences** - Simple key-value (settings, flags)
- **Flutter Secure Storage** - Sensitive data (tokens, credentials)

### Code Generation: Freezed + JSON Serializable ✅

**Why:**
- ✅ **Immutable models** - Freezed creates immutable data classes
- ✅ **JSON serialization** - Auto-generate fromJson/toJson
- ✅ **Union types** - Sealed classes for state management
- ✅ **Less boilerplate** - Generated code is type-safe

---

## Architecture Decisions

### 1. Clean Architecture (Feature-First) ✅

**Decision:** Use **Clean Architecture** organized by features

**Structure:**
```
lib/
├── core/                    # Core functionality (shared across features)
│   ├── network/             # HTTP client, interceptors
│   ├── storage/             # Local storage services
│   ├── theme/               # App theme, colors, typography
│   ├── constants/           # Global constants
│   └── utils/               # Utility functions
│
├── features/                # Business features (feature-first organization)
│   ├── auth/                # Authentication feature
│   │   ├── data/            # Data layer (repositories, data sources)
│   │   │   ├── models/      # Data models (from JSON)
│   │   │   ├── repositories/# Repository implementations
│   │   │   └── data_sources/# API clients, local storage
│   │   ├── domain/          # Domain layer (entities, use cases)
│   │   │   ├── entities/    # Domain entities
│   │   │   ├── repositories/# Repository interfaces
│   │   │   └── usecases/    # Business logic
│   │   └── presentation/    # Presentation layer (UI, state)
│   │       ├── providers/   # Riverpod providers
│   │       ├── screens/     # Screens/Pages
│   │       └── widgets/     # Feature-specific widgets
│   │
│   ├── dashboard/           # Dashboard feature
│   ├── products/            # Products management
│   ├── orders/              # Orders management
│   └── profile/             # User profile
│
├── shared/                  # Shared UI components & services
│   ├── widgets/             # Reusable widgets
│   ├── services/            # Shared services
│   └── extensions/          # Dart extensions
│
└── main.dart                # Entry point
```

**Why Clean Architecture:**
- ✅ **Separation of concerns** - Clear boundaries between layers
- ✅ **Testable** - Domain logic independent of UI/data
- ✅ **Scalable** - Easy to add new features
- ✅ **Maintainable** - Clear organization

**Why Feature-First:**
- ✅ **Faster navigation** - All files for one feature together
- ✅ **Team scalability** - Different teams work on different features
- ✅ **Easier refactoring** - Self-contained features

### 2. Multi-Tenant Integration ✅

**Decision:** Integrate with backend multi-tenant system

**Flow:**
1. User logs in → Backend returns JWT + tenant info
2. App stores JWT + tenant ID securely
3. All API requests include:
   - `Authorization: Bearer {jwt}`
   - `X-Tenant-ID: {tenantId}`
4. Data filtered by tenant at backend (RLS)

**Local storage:**
```dart
// Secure storage for tokens
await secureStorage.write(key: 'jwt_token', value: token);
await secureStorage.write(key: 'tenant_id', value: tenantId);

// Hive for user profile
box.put('user_profile', userProfile);
```

### 3. Offline-First Capabilities ✅

**Decision:** Critical features work offline

**Strategy:**
- **Read operations:** Cache data locally, serve from cache first
- **Write operations:** Queue mutations, sync when online
- **Conflict resolution:** Last-write-wins or backend timestamp

**Implementation:**
```dart
// Repository pattern with cache-first strategy
Future<List<Product>> getProducts() async {
  try {
    // Try network first
    final products = await api.getProducts();
    await cache.saveProducts(products);
    return products;
  } catch (e) {
    // Fallback to cache
    return cache.getProducts();
  }
}
```

### 4. Platform-Specific Features ✅

**Decision:** Support platform-specific features where appropriate

**iOS-specific:**
- Face ID / Touch ID authentication
- iOS share sheet integration
- Apple Pay (future)

**Android-specific:**
- Fingerprint authentication
- Android share sheet
- Google Pay (future)

**Implementation:**
```dart
import 'dart:io';

if (Platform.isIOS) {
  // iOS-specific code
} else if (Platform.isAndroid) {
  // Android-specific code
}
```

### 5. Theming & Design System ✅

**Decision:** Hybrid Material + Cupertino design

**Strategy:**
- **Android:** Material Design 3 widgets
- **iOS:** Cupertino widgets where appropriate
- **Shared:** Custom theme with brand colors

**Theme structure:**
```dart
class AppTheme {
  static final light = ThemeData(
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
    useMaterial3: true,
  );

  static final dark = ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.blue,
      brightness: Brightness.dark,
    ),
    useMaterial3: true,
  );
}
```

### 6. Error Handling & Logging ✅

**Decision:** Centralized error handling with Firebase Crashlytics

**Error types:**
```dart
sealed class Failure {
  const Failure();
}

class NetworkFailure extends Failure {
  final String message;
  const NetworkFailure(this.message);
}

class AuthFailure extends Failure {
  final String message;
  const AuthFailure(this.message);
}

class CacheFailure extends Failure {
  final String message;
  const CacheFailure(this.message);
}
```

**Logging:**
```dart
import 'package:logger/logger.dart';

final logger = Logger();

logger.d('Debug message');
logger.i('Info message');
logger.w('Warning message');
logger.e('Error message');
```

### 7. Testing Strategy ✅

**Three layers of testing:**

1. **Unit Tests** - Business logic (use cases, repositories)
2. **Widget Tests** - UI components
3. **Integration Tests** - Full user flows

**Coverage target:** 80% minimum

**Structure:**
```
test/
├── unit/
│   ├── features/
│   │   └── auth/
│   │       ├── data/
│   │       └── domain/
│   └── core/
├── widget/
│   └── features/
│       └── auth/
│           └── presentation/
└── integration/
    └── auth_flow_test.dart
```

---

## Development Workflow

### Local Development (With Docker - OPTIONAL)

```bash
# Docker is OPTIONAL for Flutter mobile dev
# Primarily used for CI/CD builds

# Development is typically done with Flutter SDK locally
flutter doctor
flutter run
```

### Local Development (Recommended)

```bash
# Check Flutter installation
flutter doctor

# Get dependencies
flutter pub get

# Generate code (Freezed, Riverpod)
dart run build_runner build --delete-conflicting-outputs

# Run on device/emulator
flutter run

# Run on specific device
flutter run -d <device-id>

# Build APK (Android)
flutter build apk --release

# Build IPA (iOS, macOS only)
flutter build ios --release
```

### Docker (CI/CD Only)

```bash
# Build APK in Docker
docker compose run --rm flutter-build flutter build apk --release

# Build App Bundle
docker compose run --rm flutter-build flutter build appbundle --release

# Run tests in Docker
docker compose run --rm flutter-test flutter test
```

---

## Key Features & Screens

### 1. Authentication Flow
- **Login Screen** - Email/password login
- **Register Screen** - Create new account (links to backend)
- **Forgot Password** - Reset password flow
- **Biometric Auth** - Face ID / Fingerprint (optional)

### 2. Dashboard
- **Overview** - Key metrics, recent activity
- **Quick Actions** - Common tasks (create order, add product)
- **Notifications** - In-app notifications

### 3. Products Management
- **Product List** - View all products
- **Product Details** - View/edit product
- **Add Product** - Create new product (with image upload)
- **Barcode Scanner** - Scan product barcode (optional)

### 4. Orders Management
- **Order List** - View all orders
- **Order Details** - View/edit order
- **Create Order** - New order with products
- **Order Status** - Track order progress

### 5. User Profile
- **Profile Screen** - View/edit user profile
- **Settings** - App settings (theme, notifications)
- **Tenant Switcher** - Switch between tenants (if multi-tenant user)
- **Logout** - Sign out

---

## Integration with bMOI Ecosystem

### API Integration

**Base URL:**
```dart
const String baseUrl = 'https://api.bmoi.com';
```

**Authentication:**
```dart
// Login
POST /auth/login
Body: { email, password }
Response: { accessToken, refreshToken, user, tenant }

// Refresh token
POST /auth/refresh
Body: { refreshToken }
Response: { accessToken }
```

**Products:**
```dart
// Get products (auto-filtered by tenant via X-Tenant-ID header)
GET /products
Headers: { Authorization: Bearer {token}, X-Tenant-ID: {tenantId} }
Response: { data: [products] }

// Create product
POST /products
Headers: { Authorization: Bearer {token}, X-Tenant-ID: {tenantId} }
Body: { name, price, description, ... }
Response: { data: product }
```

### Dio Interceptor for Auto-Headers

```dart
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

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      // Token expired, try refresh
      await refreshToken();
      handler.resolve(await retry(err.requestOptions));
    } else {
      handler.next(err);
    }
  }
}
```

---

## Environment Variables

```dart
// .env
API_BASE_URL=https://api.bmoi.com
APP_NAME=bMOI
VERSION=1.0.0

// Load with flutter_dotenv
import 'package:flutter_dotenv/flutter_dotenv.dart';

await dotenv.load();
final apiUrl = dotenv.env['API_BASE_URL'];
```

---

## Firebase Integration

### Firebase Services Used

1. **Firebase Analytics** - User behavior tracking
2. **Firebase Crashlytics** - Crash reporting
3. **Firebase Cloud Messaging (FCM)** - Push notifications
4. **Firebase Remote Config** - Feature flags (optional)

### Setup

```dart
// main.dart
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(MyApp());
}
```

---

## Performance Optimizations

### Image Optimization

```dart
// Use cached_network_image for network images
CachedNetworkImage(
  imageUrl: product.imageUrl,
  placeholder: (context, url) => Shimmer(...),
  errorWidget: (context, url, error) => Icon(Icons.error),
);

// Optimize image file size before upload
final compressedImage = await FlutterImageCompress.compressWithFile(
  file.path,
  quality: 85,
);
```

### List Performance

```dart
// Use ListView.builder for long lists
ListView.builder(
  itemCount: products.length,
  itemBuilder: (context, index) {
    return ProductCard(product: products[index]);
  },
);

// Use const widgets where possible
const Text('Static text');
```

### State Management

```dart
// Use select() to rebuild only when specific data changes
final userName = ref.watch(userProvider.select((user) => user.name));
```

---

## Security Considerations

### Secure Storage

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

final secureStorage = FlutterSecureStorage();

// Store sensitive data
await secureStorage.write(key: 'jwt_token', value: token);

// Read sensitive data
final token = await secureStorage.read(key: 'jwt_token');

// Delete sensitive data
await secureStorage.delete(key: 'jwt_token');
```

### Certificate Pinning (Optional)

```dart
// For production apps, pin SSL certificates
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

---

## Accessibility (a11y)

### Semantic Labels

```dart
// Add semantic labels for screen readers
Semantics(
  label: 'Product name',
  child: Text(product.name),
);

// Button with semantic label
ElevatedButton(
  onPressed: () {},
  child: Text('Add to Cart'),
  // Semantics handled automatically by Material widgets
);
```

### Text Scaling

```dart
// Support text scaling
Text(
  'Product name',
  style: Theme.of(context).textTheme.bodyLarge,
  // Automatically respects user's text size settings
);
```

### Color Contrast

- Ensure text has sufficient contrast (WCAG 2.1 AA: 4.5:1)
- Test with color blindness simulators

---

## Internationalization (i18n) - Optional Phase 2

**Initial:** Spanish only
**Future:** English, Portuguese

**Implementation:**
```dart
// Using flutter_localizations
import 'package:flutter_localizations/flutter_localizations.dart';

MaterialApp(
  localizationsDelegates: [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  supportedLocales: [
    Locale('es', ''), // Spanish
    Locale('en', ''), // English
  ],
);
```

---

## AI Assistant Guidelines

### When Working on This Project

**ALWAYS remember:**
- This is a **Flutter mobile app** (NOT React Native, NOT native iOS/Android)
- Framework is **Flutter 3.19+** with **Dart 3.3+**
- State management is **Riverpod** (NOT Provider, NOT BLoC)
- Architecture is **Clean Architecture** (feature-first)
- API integration is **multi-tenant** (X-Tenant-ID header)
- **Offline-first** where possible
- **Performance is critical** - Optimize lists, images, state
- **Type safety** - Use Freezed, JSON Serializable, no dynamic types

**NEVER:**
- Use dynamic types (always type everything)
- Ignore null safety (use Dart null safety features)
- Create god classes (follow Clean Architecture)
- Hardcode strings (use constants or i18n)
- Forget error handling (use Either or Result types)
- Skip tests (aim for 80% coverage)

### Preferred AI Workflow

1. **Start task** → Read `.claude/context.md` (this file)
2. **Check conventions** → Read `.claude/conventions.md`
3. **Understand current state** → Read `.claude/session-state.md`
4. **Implement** → Follow coding standards in `.docs/CODING_STANDARDS.md`
5. **Test** → Check examples in `.docs/EJEMPLOS_CODIGO.md`
6. **Update state** → Update `.claude/session-state.md` before ending

---

## Next Steps (From Zero to Production)

### Phase 0: Setup (Days 1-2)
- [ ] Install Flutter SDK
- [ ] Setup Android Studio / VS Code
- [ ] Configure iOS (Xcode, CocoaPods)
- [ ] Initialize Flutter project
- [ ] Add dependencies (Riverpod, Dio, Go Router, Freezed)
- [ ] Setup folder structure (Clean Architecture)
- [ ] Configure Docker (optional, for CI/CD)

### Phase 1: Core Infrastructure (Days 3-5)
- [ ] Setup Dio with interceptors
- [ ] Implement secure storage (Flutter Secure Storage)
- [ ] Create theme (Material + Cupertino)
- [ ] Setup Go Router navigation
- [ ] Implement error handling (Either/Result)
- [ ] Setup Riverpod providers structure

### Phase 2: Authentication (Days 6-8)
- [ ] Create auth models (User, AuthResponse)
- [ ] Implement auth repository
- [ ] Create login screen
- [ ] Create register screen
- [ ] Implement JWT storage
- [ ] Add auto-logout on token expiry
- [ ] Add biometric authentication (optional)

### Phase 3: Core Features (Days 9-15)
- [ ] Dashboard screen
- [ ] Products list screen
- [ ] Product details screen
- [ ] Add/Edit product screen
- [ ] Orders list screen
- [ ] Order details screen
- [ ] User profile screen

### Phase 4: Offline Support (Days 16-18)
- [ ] Setup Hive for local storage
- [ ] Implement cache-first strategy
- [ ] Add sync queue for offline writes
- [ ] Handle network connectivity changes
- [ ] Show offline indicator

### Phase 5: Polish & Testing (Days 19-21)
- [ ] Unit tests (repositories, use cases)
- [ ] Widget tests (screens, widgets)
- [ ] Integration tests (full flows)
- [ ] Performance optimization
- [ ] Accessibility audit
- [ ] Fix bugs

### Phase 6: Deployment (Days 22-25)
- [ ] Setup Firebase (Analytics, Crashlytics, FCM)
- [ ] Configure Android signing (keystore)
- [ ] Configure iOS signing (certificates, provisioning profiles)
- [ ] Build APK/AAB for Google Play
- [ ] Build IPA for App Store
- [ ] Submit to stores (Google Play, App Store)

---

## Additional Resources

### Documentation Files (`.docs/` folder)

The following documentation files provide comprehensive guides for this project:

- **`.docs/README.md`** - Project overview with architecture, tech stack, features, and multi-tenant integration
- **`.docs/SETUP.md`** - Complete step-by-step setup guide (Flutter SDK, Android/iOS setup, dependencies, configuration)

**Future documentation** (to be created as needed):
- `.docs/ARQUITECTURA.md` - Detailed architecture decisions
- `.docs/CODING_STANDARDS.md` - Flutter/Dart coding standards
- `.docs/EJEMPLOS_CODIGO.md` - Code examples and patterns
- `.docs/DEPLOYMENT.md` - Deployment guides (Google Play, App Store)

### Related Projects Documentation

- **Backend API:** See `bo-backend-api/CLAUDE.md` for API endpoints
- **Admin Panel:** See `plantillas_admin/.claude/context.md` for web app
- **Landing Page:** See `plantillas_landing/.claude/context.md` for marketing site

---

**Last Updated:** 2025-01-04
**Template Version:** 1.0.0
**Status:** Initial Creation
