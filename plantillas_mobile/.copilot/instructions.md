# GitHub Copilot Instructions - bMOI Mobile App

## Project Context

You are working on the **bMOI Mobile App**, a cross-platform mobile application built with **Flutter 3.19+ and Dart 3.3+**.

**Purpose:** Provide mobile access to the bMOI multi-tenant platform for business users (iOS + Android)
**Architecture:** Clean Architecture (Feature-First)
**State Management:** Riverpod 2.5+

---

## Core Technologies

- **Framework:** Flutter 3.19+
- **Language:** Dart 3.3+ (null safety)
- **State Management:** Riverpod 2.5+
- **Navigation:** Go Router 13+
- **HTTP Client:** Dio 5+
- **Code Generation:** Freezed + JSON Serializable + Riverpod Generator
- **Local Storage:** Hive + Shared Preferences + Flutter Secure Storage

---

## Mandatory Conventions

### File Naming

```dart
✅ CORRECT:
lib/features/auth/presentation/screens/login_screen.dart  // snake_case
lib/features/products/data/models/product_model.dart
lib/core/network/api_client.dart
lib/shared/widgets/custom_button.dart

❌ INCORRECT:
lib/features/auth/LoginScreen.dart           // Don't use PascalCase for files
lib/features/products/product-model.dart     // Don't use kebab-case
lib/core/network/ApiClient.dart              // Files must be snake_case
```

**Important:** Dart uses snake_case for file names, unlike TypeScript!

### Code Naming

```dart
✅ CORRECT:
// Classes: PascalCase
class LoginScreen extends StatelessWidget {}
class ProductModel {}
class AuthRepository {}

// Variables & parameters: lowerCamelCase
final String userName = 'John';
final int productCount = 10;
bool isLoading = false;

// Constants: lowerCamelCase (Dart convention!)
const String apiBaseUrl = 'https://api.bmoi.com';
const int maxRetries = 3;

// Private members: underscore prefix
final String _apiKey;
void _processData() {}

// Enums: PascalCase name, lowerCamelCase values
enum UserRole {
  admin,
  user,
  guest,
}

❌ INCORRECT:
class loginScreen extends StatelessWidget {}  // Should be PascalCase
final String UserName = 'John';                // Should be lowerCamelCase
const String API_BASE_URL = 'https://...';     // Dart uses lowerCamelCase, not UPPER_SNAKE_CASE
enum userRole { Admin, User }                  // Should be PascalCase name, lowerCamelCase values
```

---

## Clean Architecture Structure

```dart
lib/
├── core/                    # Shared across features
│   ├── network/             # HTTP client, interceptors
│   ├── storage/             # Local storage services
│   ├── theme/               # App theme
│   └── constants/           # Global constants
│
├── features/                # Feature-first organization
│   └── auth/                # Example feature
│       ├── data/            # Data layer
│       │   ├── models/      # Data models (JSON)
│       │   ├── repositories/# Repository implementations
│       │   └── data_sources/# API clients, local storage
│       ├── domain/          # Domain layer
│       │   ├── entities/    # Domain entities
│       │   ├── repositories/# Repository interfaces
│       │   └── usecases/    # Business logic
│       └── presentation/    # Presentation layer
│           ├── providers/   # Riverpod providers
│           ├── screens/     # Screens/Pages
│           └── widgets/     # Feature-specific widgets
│
├── shared/                  # Reusable UI components
│   └── widgets/
│
└── main.dart
```

---

## Common Patterns

### 1. Freezed Model (Data Layer)

```dart
// File: lib/features/auth/data/models/user_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user_model.freezed.dart';
part 'user_model.g.dart';

@freezed
class UserModel with _$UserModel {
  const factory UserModel({
    required String id,
    required String email,
    required String name,
    String? avatarUrl,
  }) = _UserModel;

  factory UserModel.fromJson(Map<String, dynamic> json) =>
      _$UserModelFromJson(json);
}
```

### 2. Domain Entity

```dart
// File: lib/features/auth/domain/entities/user.dart
class User {
  final String id;
  final String email;
  final String name;
  final String? avatarUrl;

  const User({
    required this.id,
    required this.email,
    required this.name,
    this.avatarUrl,
  });
}
```

### 3. Repository Interface (Domain)

```dart
// File: lib/features/auth/domain/repositories/auth_repository.dart
import 'package:dartz/dartz.dart';

abstract class AuthRepository {
  Future<Either<Failure, User>> login({
    required String email,
    required String password,
  });

  Future<Either<Failure, void>> logout();
}
```

### 4. Repository Implementation (Data)

```dart
// File: lib/features/auth/data/repositories/auth_repository_impl.dart
class AuthRepositoryImpl implements AuthRepository {
  final AuthRemoteDataSource remoteDataSource;
  final AuthLocalDataSource localDataSource;

  const AuthRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
  });

  @override
  Future<Either<Failure, User>> login({
    required String email,
    required String password,
  }) async {
    try {
      final userModel = await remoteDataSource.login(email, password);
      await localDataSource.saveUser(userModel);
      return Right(userModel.toEntity());
    } on NetworkException catch (e) {
      return Left(NetworkFailure(e.message));
    } catch (e) {
      return Left(UnknownFailure(e.toString()));
    }
  }
}
```

### 5. Use Case

```dart
// File: lib/features/auth/domain/usecases/login_usecase.dart
class LoginUseCase {
  final AuthRepository repository;

  const LoginUseCase(this.repository);

  Future<Either<Failure, User>> call({
    required String email,
    required String password,
  }) async {
    return await repository.login(email: email, password: password);
  }
}
```

### 6. Riverpod Provider (Generated)

```dart
// File: lib/features/auth/presentation/providers/auth_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'auth_provider.g.dart';

@riverpod
class Auth extends _$Auth {
  @override
  AsyncValue<User?> build() {
    return const AsyncValue.data(null);
  }

  Future<void> login(String email, String password) async {
    state = const AsyncValue.loading();
    final result = await ref.read(loginUseCaseProvider).call(
      email: email,
      password: password,
    );

    state = result.fold(
      (failure) => AsyncValue.error(failure, StackTrace.current),
      (user) => AsyncValue.data(user),
    );
  }

  Future<void> logout() async {
    await ref.read(logoutUseCaseProvider).call();
    state = const AsyncValue.data(null);
  }
}
```

### 7. Screen/Widget

```dart
// File: lib/features/auth/presentation/screens/login_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class LoginScreen extends ConsumerWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(authProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: authState.when(
          data: (user) {
            if (user != null) {
              return const Center(child: Text('Logged in'));
            }
            return LoginForm();
          },
          loading: () => const Center(child: CircularProgressIndicator()),
          error: (error, stack) => Center(
            child: Text('Error: $error'),
          ),
        ),
      ),
    );
  }
}
```

---

## Dio Setup with Interceptors

```dart
// File: lib/core/network/dio_client.dart
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
      ErrorInterceptor(),
      LogInterceptor(requestBody: true, responseBody: true),
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

## Error Handling with Either

```dart
// File: lib/core/error/failures.dart
sealed class Failure {
  final String message;
  const Failure(this.message);
}

class NetworkFailure extends Failure {
  const NetworkFailure(super.message);
}

class CacheFailure extends Failure {
  const CacheFailure(super.message);
}

class AuthFailure extends Failure {
  const AuthFailure(super.message);
}
```

---

## Common Mistakes to Avoid

### ❌ DON'T

```dart
// ❌ Don't use PascalCase for file names
LoginScreen.dart  // WRONG! Use login_screen.dart

// ❌ Don't use UPPER_SNAKE_CASE for constants (Dart uses lowerCamelCase)
const String API_URL = 'https://...';  // WRONG! Use apiUrl

// ❌ Don't use dynamic types
dynamic data;  // WRONG! Always type variables

// ❌ Don't ignore null safety
String? name;
print(name.length);  // WRONG! Could be null

// ❌ Don't skip layers in Clean Architecture
// Presentation → UseCase → Repository → DataSource (follow the flow)

// ❌ Don't use BuildContext across async gaps
void fetchData(BuildContext context) async {
  await Future.delayed(Duration(seconds: 1));
  Navigator.push(context, ...);  // WRONG! Context might be invalid
}
```

### ✅ DO

```dart
// ✅ Use snake_case for file names
login_screen.dart  // CORRECT

// ✅ Use lowerCamelCase for constants
const String apiUrl = 'https://...';  // CORRECT

// ✅ Always type variables
User? user;
List<Product> products = [];

// ✅ Handle null safety properly
String? name;
print(name?.length ?? 0);  // CORRECT

// ✅ Follow Clean Architecture layers
// Presentation → UseCase → Repository → DataSource

// ✅ Check context validity
void fetchData(BuildContext context) async {
  await Future.delayed(Duration(seconds: 1));
  if (context.mounted) {  // CORRECT
    Navigator.push(context, ...);
  }
}
```

---

## Quick Reference

### Import Order

```dart
// 1. Dart imports
import 'dart:async';
import 'dart:io';

// 2. Flutter imports
import 'package:flutter/material.dart';

// 3. Package imports
import 'package:riverpod/riverpod.dart';
import 'package:dio/dio.dart';

// 4. Project imports
import 'package:bmoi_mobile_app/core/network/api_client.dart';
import 'package:bmoi_mobile_app/features/auth/domain/entities/user.dart';
```

### Common Commands

```bash
# Get dependencies
flutter pub get

# Generate code (Freezed, Riverpod)
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

---

## Remember

1. **Files:** snake_case (e.g., `login_screen.dart`)
2. **Classes:** PascalCase (e.g., `LoginScreen`)
3. **Variables:** lowerCamelCase (e.g., `userName`)
4. **Constants:** lowerCamelCase (e.g., `const apiUrl`)
5. **Private:** Underscore prefix (e.g., `_privateMethod`)
6. **Architecture:** Clean Architecture (feature-first)
7. **State:** Riverpod (use code generation)
8. **Types:** Always type everything (no dynamic)
9. **Null safety:** Use Dart null safety features
10. **Multi-tenant:** Auto-inject X-Tenant-ID header

---

**For detailed documentation, refer to:**
- `.claude/context.md` - Complete project context
- `.claude/conventions.md` - Detailed naming conventions
- `.docs/CODING_STANDARDS.md` - Coding standards with examples
