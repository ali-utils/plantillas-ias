# CLAUDE.md - Mobile App

This file provides guidance to Claude Code (claude.ai/code) when working on the **bMOI Mobile App**.

## Project Overview

**bMOI Mobile App** is a cross-platform mobile application (iOS + Android) built with Flutter that provides mobile access to the bMOI multi-tenant platform.

**Framework:** Flutter 3.19+ + Dart 3.3+
**Architecture:** Clean Architecture (Feature-First)
**State Management:** Riverpod 2.5+

**Current Status:** Template/Plantilla (Starting from Zero - 0% complete)

## 📚 Critical Documentation

Read these files in order when starting:

1. **`.claude/context.md`** - Complete project context (MUST READ FIRST)
2. **`.claude/conventions.md`** - Mandatory naming conventions (snake_case files!)
3. **`.claude/session-state.md`** - Current development status
4. **This file (CLAUDE.md)** - Quick reference
5. **`.docs/SETUP.md`** - Step-by-step setup guide
6. **`.docs/ARQUITECTURA.md`** - Architecture decisions

## 🎯 Key Points

### This Project IS
- ✅ Flutter 3.19+ mobile app (iOS + Android)
- ✅ Clean Architecture (feature-first)
- ✅ Riverpod state management
- ✅ Multi-tenant (X-Tenant-ID header)
- ✅ Offline-first capabilities
- ✅ Null-safe Dart code

### This Project is NOT
- ❌ React Native app
- ❌ Native iOS/Android only
- ❌ Hybrid WebView app
- ❌ Consumer marketplace app

## 🛠️ Tech Stack

- **Framework:** Flutter 3.19+
- **Language:** Dart 3.3+ (null safety)
- **State Management:** Riverpod 2.5+
- **Navigation:** Go Router 13+
- **HTTP:** Dio 5+
- **Code Gen:** Freezed + JSON Serializable + Riverpod Generator
- **Storage:** Hive + Shared Preferences + Flutter Secure Storage

## 📝 Naming Conventions (CRITICAL!)

### Files: snake_case
```dart
✅ login_screen.dart
✅ product_model.dart
✅ auth_repository.dart

❌ LoginScreen.dart
❌ product-model.dart
❌ AuthRepository.dart
```

### Classes: PascalCase
```dart
✅ class LoginScreen {}
✅ class ProductModel {}

❌ class loginScreen {}
❌ class product_model {}
```

### Variables/Functions: lowerCamelCase
```dart
✅ final String userName = 'John';
✅ void fetchProducts() {}

❌ final String UserName = 'John';
❌ void FetchProducts() {}
```

### Constants: lowerCamelCase (Dart convention!)
```dart
✅ const String apiUrl = 'https://api.bmoi.com';

❌ const String API_URL = 'https://api.bmoi.com';  // NOT in Dart!
```

**Important:** Dart uses snake_case for files and lowerCamelCase for constants (unlike other languages!)

## 🏗️ Clean Architecture

```
lib/
├── features/
│   └── auth/
│       ├── data/            # Models, Repo Impl, Data Sources
│       ├── domain/          # Entities, Repo Interface, Use Cases
│       └── presentation/    # Providers, Screens, Widgets
├── core/                    # Network, Storage, Theme
└── shared/                  # Reusable Widgets
```

**Flow:** Presentation → UseCase → Repository → DataSource

## 🎨 Common Patterns

### Freezed Model

```dart
@freezed
class UserModel with _$UserModel {
  const factory UserModel({
    required String id,
    required String email,
  }) = _UserModel;

  factory UserModel.fromJson(Map<String, dynamic> json) =>
      _$UserModelFromJson(json);
}
```

### Repository Interface

```dart
abstract class AuthRepository {
  Future<Either<Failure, User>> login({
    required String email,
    required String password,
  });
}
```

### Riverpod Provider

```dart
@riverpod
class Auth extends _$Auth {
  @override
  AsyncValue<User?> build() => const AsyncValue.data(null);

  Future<void> login(String email, String password) async {
    // Implementation
  }
}
```

### Screen Widget

```dart
class LoginScreen extends ConsumerWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(authProvider);
    return Scaffold(/* ... */);
  }
}
```

## 🚀 Development Commands

```bash
# Check Flutter installation
flutter doctor

# Get dependencies
flutter pub get

# Generate code
dart run build_runner build --delete-conflicting-outputs

# Run app
flutter run

# Run on specific device
flutter run -d <device-id>

# Run tests
flutter test

# Build APK
flutter build apk --release

# Build iOS (macOS only)
flutter build ios --release
```

## 🔗 Multi-Tenant Integration

Auto-inject headers in all API requests:

```dart
class AuthInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) async {
    final token = await secureStorage.read(key: 'jwt_token');
    final tenantId = await secureStorage.read(key: 'tenant_id');

    if (token != null) options.headers['Authorization'] = 'Bearer $token';
    if (tenantId != null) options.headers['X-Tenant-ID'] = tenantId;

    handler.next(options);
  }
}
```

## 🧠 AI Memory Checkpoints

Every ~5 messages, verify you remember:
- [ ] Project: **bMOI Mobile App**
- [ ] Framework: **Flutter 3.19+** (NOT React Native)
- [ ] Language: **Dart 3.3+** (null safety)
- [ ] State: **Riverpod** (NOT Provider, NOT BLoC)
- [ ] Architecture: **Clean Architecture** (feature-first)
- [ ] Files: **snake_case** (e.g., `login_screen.dart`)
- [ ] Classes: **PascalCase** (e.g., `LoginScreen`)
- [ ] Variables: **lowerCamelCase** (e.g., `userName`)
- [ ] Constants: **lowerCamelCase** (e.g., `const apiUrl`, NOT `API_URL`)
- [ ] Private: **_prefix** (e.g., `_privateMethod`)
- [ ] Multi-tenant: **X-Tenant-ID** header auto-injected
- [ ] Offline: **Cache-first** strategy where possible

## 🚨 Common Mistakes to Avoid

### ❌ DON'T

```dart
// ❌ Don't use PascalCase for file names
LoginScreen.dart  // WRONG! Use login_screen.dart

// ❌ Don't use UPPER_SNAKE_CASE for constants (Dart uses lowerCamelCase)
const String API_URL = 'https://...';  // WRONG! Use apiUrl

// ❌ Don't use dynamic types
dynamic data;  // WRONG! Type everything

// ❌ Don't ignore null safety
String? name;
print(name.length);  // WRONG! Could be null

// ❌ Don't skip architecture layers
// Presentation → DataSource directly is WRONG
```

### ✅ DO

```dart
// ✅ Use snake_case for files
login_screen.dart  // CORRECT

// ✅ Use lowerCamelCase for constants
const String apiUrl = 'https://...';  // CORRECT

// ✅ Type everything
List<Product> products = [];  // CORRECT

// ✅ Handle nulls safely
String? name;
print(name?.length ?? 0);  // CORRECT

// ✅ Follow architecture layers
Presentation → UseCase → Repository → DataSource  // CORRECT
```

## 📊 Development Phases

```
Phase 0 (Setup):           ░░░░░░░░░░  0%  - Flutter + Clean Arch + Docker
Phase 1 (Infrastructure):  ░░░░░░░░░░  0%  - Network, Storage, Theme
Phase 2 (Authentication):  ░░░░░░░░░░  0%  - Login, Register, JWT
Phase 3 (Core Features):   ░░░░░░░░░░  0%  - Dashboard, Products, Orders
Phase 4 (Offline):         ░░░░░░░░░░  0%  - Hive, Sync, Cache
Phase 5 (Testing):         ░░░░░░░░░░  0%  - Unit, Widget, Integration
Phase 6 (Deployment):      ░░░░░░░░░░  0%  - Firebase, Store submission
```

## 🔍 Troubleshooting AI Behavior

### If AI forgets conventions:

```
🧠 CONVENTION REMINDER - MOBILE APP:

This project uses STRICTLY:
- Files: snake_case (login_screen.dart, product_model.dart)
- Classes: PascalCase (LoginScreen, ProductModel)
- Variables: lowerCamelCase (userName, fetchProducts)
- Constants: lowerCamelCase (const apiUrl, NOT API_URL)
- Private: _prefix (_privateMethod)

Framework: Flutter 3.19+ (NOT React Native)
State: Riverpod (code generation)
Architecture: Clean Architecture (feature-first)
Multi-tenant: X-Tenant-ID header auto-injected

Please confirm you'll remember this.
```

### If AI uses wrong file naming:

```
⚠️ ERROR: Incorrect file naming

Files MUST be snake_case in Dart:
✅ login_screen.dart
✅ product_model.dart
✅ auth_repository.dart

❌ LoginScreen.dart
❌ product-model.dart
❌ AuthRepository.dart

Please correct the file names above.
```

## 📚 Additional Resources

### Core Documentation
- **Complete context:** `.claude/context.md`
- **Conventions:** `.claude/conventions.md`
- **Current state:** `.claude/session-state.md`

### Guides (`.docs/` folder)
- **Project overview:** `.docs/README.md`
- **Setup guide:** `.docs/SETUP.md`

**Future guides** (to be created):
- `.docs/ARQUITECTURA.md` - Architecture decisions
- `.docs/CODING_STANDARDS.md` - Coding standards
- `.docs/EJEMPLOS_CODIGO.md` - Code examples
- `.docs/DEPLOYMENT.md` - Deployment to stores

## 🔗 Related Projects

- Backend API: `bo-backend-api/CLAUDE.md`
- Admin Panel: `plantillas_admin/.claude/context.md`
- Landing Page: `plantillas_landing/.claude/context.md`

---

<div align="center">

**Cross-Platform Mobile App with Flutter**

iOS + Android | Clean Architecture | Offline-First

</div>
