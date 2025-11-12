# Naming Conventions - bMOI Mobile App

## Overview

This document defines **mandatory naming conventions** for the bMOI Mobile App project. These conventions ensure consistency, readability, and maintainability across the Flutter/Dart codebase.

**Framework:** Flutter 3.19+ + Dart 3.3+
**Architecture:** Clean Architecture (Feature-First)
**State Management:** Riverpod 2.5+

---

## File Naming Conventions

### Dart Files

Use **snake_case** for ALL Dart files:

```
✅ CORRECT:
lib/features/auth/presentation/screens/login_screen.dart
lib/features/products/data/models/product_model.dart
lib/core/network/api_client.dart
lib/shared/widgets/custom_button.dart
lib/features/dashboard/presentation/providers/dashboard_provider.dart

❌ INCORRECT:
lib/features/auth/LoginScreen.dart           // Don't use PascalCase for files
lib/features/products/product-model.dart     // Don't use kebab-case
lib/core/network/ApiClient.dart              // Don't use PascalCase
lib/shared/widgets/CustomButton.dart         // Don't use PascalCase
```

**Important:** Dart convention is snake_case for files, unlike TypeScript/JavaScript!

### Asset Files

Use **kebab-case** for asset files:

```
✅ CORRECT:
assets/images/logo.png
assets/images/product-placeholder.png
assets/icons/home-icon.svg
assets/fonts/inter-regular.ttf

❌ INCORRECT:
assets/images/Logo.png
assets/images/product_placeholder.png    // Use kebab-case, not snake_case
assets/icons/HomeIcon.svg
```

---

## Code Naming Conventions

### Classes

Use **PascalCase** for all classes:

```dart
✅ CORRECT:
class LoginScreen extends StatelessWidget {}
class ProductModel {}
class AuthRepository {}
class GetProductsUseCase {}
class DashboardProvider extends StateNotifier {}

❌ INCORRECT:
class loginScreen extends StatelessWidget {}  // Should be PascalCase
class productModel {}                         // Should be PascalCase
class authRepository {}                       // Should be PascalCase
class get_products_use_case {}                // Should be PascalCase, not snake_case
```

### Variables & Parameters

Use **lowerCamelCase** for variables and parameters:

```dart
✅ CORRECT:
final String userName = 'John';
final int productCount = 10;
final List<Product> productList = [];
String apiBaseUrl = 'https://api.bmoi.com';
bool isLoading = false;

void fetchProducts({required String tenantId}) {}

❌ INCORRECT:
final String UserName = 'John';           // Don't use PascalCase
final int product_count = 10;             // Don't use snake_case
final List<Product> ProductList = [];     // Don't use PascalCase
String APIBaseURL = 'https://...';        // Too many capitals
```

### Constants

Use **lowerCamelCase** for constants (Dart convention):

```dart
✅ CORRECT:
const String apiBaseUrl = 'https://api.bmoi.com';
const int maxRetries = 3;
const double padding16 = 16.0;
const Duration timeoutDuration = Duration(seconds: 30);

// For compile-time constants in classes
class ApiConstants {
  static const String baseUrl = 'https://api.bmoi.com';
  static const String productsEndpoint = '/products';
}

❌ INCORRECT:
const String API_BASE_URL = 'https://...';  // Don't use UPPER_SNAKE_CASE in Dart
const int MAX_RETRIES = 3;                  // Don't use UPPER_SNAKE_CASE
const double PADDING_16 = 16.0;             // Don't use UPPER_SNAKE_CASE
```

**Note:** Unlike other languages, Dart uses lowerCamelCase for constants, not UPPER_SNAKE_CASE!

### Functions & Methods

Use **lowerCamelCase** for all functions and methods:

```dart
✅ CORRECT:
void fetchProducts() {}
Future<List<Product>> getProducts() async {}
String formatDate(DateTime date) {}
bool isValidEmail(String email) {}

// Extension methods
extension StringExtension on String {
  String capitalize() {}
  bool get isValidEmail {}
}

❌ INCORRECT:
void FetchProducts() {}              // Don't use PascalCase
Future<List<Product>> GetProducts() async {}  // Don't use PascalCase
String format_date(DateTime date) {} // Don't use snake_case
```

### Private Members

Use **underscore prefix** for private members:

```dart
✅ CORRECT:
class ProductService {
  final String _apiKey;  // Private field
  String _cachedData = '';  // Private field

  void _processData() {}  // Private method
  int _calculateTotal() {} // Private method
}

// Private top-level functions/variables
final _logger = Logger();
void _helperFunction() {}

❌ INCORRECT:
class ProductService {
  final String apiKey;  // Not marked as private
  String cachedData = '';  // Not marked as private
}
```

### Enums

Use **PascalCase** for enum names and **lowerCamelCase** for values:

```dart
✅ CORRECT:
enum UserRole {
  admin,
  user,
  guest,
}

enum OrderStatus {
  pending,
  confirmed,
  shipped,
  delivered,
}

// Usage
final role = UserRole.admin;
if (status == OrderStatus.pending) {}

❌ INCORRECT:
enum userRole {  // Should be PascalCase
  Admin,         // Should be lowerCamelCase
  User,
  Guest,
}

enum ORDER_STATUS {  // Should be PascalCase
  PENDING,           // Should be lowerCamelCase
  CONFIRMED,
}
```

### Type Definitions (typedef)

Use **PascalCase** for typedef names:

```dart
✅ CORRECT:
typedef UserId = String;
typedef ProductCallback = void Function(Product product);
typedef JsonMap = Map<String, dynamic>;

❌ INCORRECT:
typedef userId = String;  // Should be PascalCase
typedef product_callback = void Function(Product);  // Should be PascalCase
```

---

## Folder Structure Conventions

### Feature Structure

```
✅ CORRECT:
lib/
├── core/                    # Lowercase folders
│   ├── network/
│   ├── storage/
│   └── theme/
├── features/
│   ├── auth/                # Feature name in lowercase
│   │   ├── data/
│   │   │   ├── models/
│   │   │   │   └── user_model.dart         # snake_case
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository_impl.dart
│   │   │   └── data_sources/
│   │   │       └── auth_remote_data_source.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── user.dart
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart
│   │   │   └── usecases/
│   │   │       └── login_usecase.dart
│   │   └── presentation/
│   │       ├── providers/
│   │       │   └── auth_provider.dart
│   │       ├── screens/
│   │       │   └── login_screen.dart
│   │       └── widgets/
│   │           └── login_form.dart
│   └── products/
├── shared/
│   └── widgets/
│       └── custom_button.dart
└── main.dart

❌ INCORRECT:
lib/
├── Core/                    # Don't capitalize folders
├── Features/
│   └── Auth/
│       ├── Data/
│       │   └── Models/
│       │       └── UserModel.dart  # File should be snake_case
│       └── Presentation/
│           └── Screens/
│               └── LoginScreen.dart  # File should be snake_case
```

---

## Widget Naming

### StatelessWidget / StatefulWidget

Use **PascalCase** with descriptive suffix:

```dart
✅ CORRECT:
class LoginScreen extends StatelessWidget {}
class ProductCard extends StatelessWidget {}
class CustomButton extends StatelessWidget {}
class LoadingIndicator extends StatelessWidget {}
class DashboardPage extends StatefulWidget {}

❌ INCORRECT:
class login_screen extends StatelessWidget {}  // Should be PascalCase
class productcard extends StatelessWidget {}   // Should be PascalCase
class Button extends StatelessWidget {}        // Too generic
```

### Widget File Names

```
✅ CORRECT:
login_screen.dart          # Contains LoginScreen widget
product_card.dart          # Contains ProductCard widget
custom_button.dart         # Contains CustomButton widget

❌ INCORRECT:
LoginScreen.dart           # File should be snake_case
product-card.dart          # Use snake_case, not kebab-case
button.dart                # Too generic
```

---

## Riverpod Provider Naming

### Provider Variables

Use **lowerCamelCase** with descriptive suffix:

```dart
✅ CORRECT:
final authProvider = StateNotifierProvider<AuthNotifier, AuthState>(...);
final productsProvider = FutureProvider<List<Product>>(...);
final userProfileProvider = StateProvider<User?>(...);
final cartItemsProvider = StateNotifierProvider<CartNotifier, List<CartItem>>(...);

// Generated providers (with riverpod_generator)
@riverpod
Future<List<Product>> products(ProductsRef ref) async {}

@riverpod
class Auth extends _$Auth {}

❌ INCORRECT:
final AuthProvider = StateNotifierProvider...  // Should be lowerCamelCase
final Products = FutureProvider...            // Should be lowerCamelCase
final auth_provider = StateNotifierProvider... // Don't use snake_case
```

---

## Model & Entity Naming

### Data Models (JSON serialization)

```dart
✅ CORRECT:
// File: user_model.dart
@freezed
class UserModel with _$UserModel {
  const factory UserModel({
    required String id,
    required String email,
    required String name,
  }) = _UserModel;

  factory UserModel.fromJson(Map<String, dynamic> json) =>
      _$UserModelFromJson(json);
}

❌ INCORRECT:
// File: UserModel.dart  // File should be snake_case
class userModel with _$userModel {}  // Class should be PascalCase
```

### Domain Entities

```dart
✅ CORRECT:
// File: user.dart
class User {
  final String id;
  final String email;
  final String name;

  const User({
    required this.id,
    required this.email,
    required this.name,
  });
}

❌ INCORRECT:
// File: User.dart  // File should be snake_case
class user {}  // Class should be PascalCase
```

---

## Use Case Naming

Use **PascalCase** ending with `UseCase`:

```dart
✅ CORRECT:
// File: login_usecase.dart
class LoginUseCase {
  Future<Either<Failure, User>> call({
    required String email,
    required String password,
  }) async {}
}

// File: get_products_usecase.dart
class GetProductsUseCase {
  Future<Either<Failure, List<Product>>> call() async {}
}

❌ INCORRECT:
// File: LoginUseCase.dart  // File should be snake_case
class loginUseCase {}  // Class should be PascalCase
class Login {}  // Should have UseCase suffix
class get_products {}  // Should be PascalCase
```

---

## Repository Naming

### Repository Interface (Domain)

```dart
✅ CORRECT:
// File: auth_repository.dart
abstract class AuthRepository {
  Future<Either<Failure, User>> login({
    required String email,
    required String password,
  });

  Future<Either<Failure, void>> logout();
}

❌ INCORRECT:
// File: AuthRepository.dart  // File should be snake_case
abstract class authRepository {}  // Class should be PascalCase
abstract class Auth_Repository {}  // Don't use snake_case in class names
```

### Repository Implementation (Data)

```dart
✅ CORRECT:
// File: auth_repository_impl.dart
class AuthRepositoryImpl implements AuthRepository {
  @override
  Future<Either<Failure, User>> login({
    required String email,
    required String password,
  }) async {}
}

❌ INCORRECT:
// File: AuthRepositoryImpl.dart  // File should be snake_case
class authRepositoryImpl implements AuthRepository {}  // Class should be PascalCase
```

---

## Extension Naming

Use **PascalCase** with descriptive name + `Extension`:

```dart
✅ CORRECT:
extension StringExtension on String {
  String capitalize() {
    return '${this[0].toUpperCase()}${substring(1)}';
  }

  bool get isValidEmail {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(this);
  }
}

extension DateTimeExtension on DateTime {
  String toFormattedString() {
    return '$day/$month/$year';
  }
}

❌ INCORRECT:
extension stringExtension on String {}  // Should be PascalCase
extension String_Extension on String {}  // Don't use snake_case
extension StringExt on String {}        // Use full "Extension" suffix
```

---

## Import/Export Conventions

### Import Ordering

```dart
✅ CORRECT:
// 1. Dart imports
import 'dart:async';
import 'dart:io';

// 2. Flutter imports
import 'package:flutter/material.dart';
import 'package:flutter/cupertino.dart';

// 3. Package imports (external)
import 'package:riverpod/riverpod.dart';
import 'package:dio/dio.dart';

// 4. Project imports (internal)
import 'package:bmoi_mobile_app/core/network/api_client.dart';
import 'package:bmoi_mobile_app/features/auth/domain/entities/user.dart';

❌ INCORRECT:
// Mixed order, hard to read
import 'package:bmoi_mobile_app/core/network/api_client.dart';
import 'dart:async';
import 'package:dio/dio.dart';
import 'package:flutter/material.dart';
```

### Relative vs Absolute Imports

```dart
✅ CORRECT:
// Use absolute imports (package imports)
import 'package:bmoi_mobile_app/features/auth/domain/entities/user.dart';
import 'package:bmoi_mobile_app/core/network/api_client.dart';

❌ AVOID:
// Avoid relative imports across features
import '../../../core/network/api_client.dart';
import '../../domain/entities/user.dart';

✅ ACCEPTABLE:
// Relative imports within same feature are OK
import '../entities/user.dart';  // Within same feature
import 'models/user_model.dart';  // Within same folder
```

---

## Test File Naming

```
✅ CORRECT:
test/
├── unit/
│   ├── features/
│   │   └── auth/
│   │       ├── data/
│   │       │   └── repositories/
│   │       │       └── auth_repository_impl_test.dart
│   │       └── domain/
│   │           └── usecases/
│   │               └── login_usecase_test.dart
│   └── core/
│       └── network/
│           └── api_client_test.dart
├── widget/
│   └── features/
│       └── auth/
│           └── presentation/
│               └── screens/
│                   └── login_screen_test.dart
└── integration/
    └── auth_flow_test.dart

❌ INCORRECT:
test/
├── AuthRepositoryImplTest.dart  // Should be snake_case + _test suffix
├── LoginUseCaseTest.dart        // Should be snake_case + _test suffix
└── login-screen-test.dart       // Use snake_case, not kebab-case
```

---

## Asset Path Constants

```dart
✅ CORRECT:
class AppAssets {
  // Images
  static const String logo = 'assets/images/logo.png';
  static const String productPlaceholder = 'assets/images/product-placeholder.png';

  // Icons
  static const String homeIcon = 'assets/icons/home-icon.svg';
  static const String profileIcon = 'assets/icons/profile-icon.svg';
}

// Usage
Image.asset(AppAssets.logo);

❌ INCORRECT:
class AppAssets {
  static const String LOGO = 'assets/images/logo.png';  // Don't use UPPER_SNAKE_CASE
  static const String product_placeholder = '...';      // Use lowerCamelCase
}
```

---

## Git Commit Conventions

Use **Conventional Commits** format:

```bash
✅ CORRECT:
feat: add login screen
fix: correct product image loading
docs: update README with setup instructions
style: format code with dartfmt
refactor: simplify auth repository
perf: optimize product list rendering
test: add unit tests for login use case
chore: update dependencies

❌ INCORRECT:
Added login screen              # Missing type prefix
Fixed bug                       # Not descriptive
Updated files                   # Too vague
FEAT: Add login screen          # Don't capitalize type
```

---

## Branch Naming

Use **kebab-case** with type prefix:

```bash
✅ CORRECT:
feature/add-product-screen
feature/implement-offline-sync
fix/login-validation-error
fix/product-image-upload
refactor/auth-repository
docs/update-architecture

❌ INCORRECT:
feature_add_product_screen      # Use kebab-case, not snake_case
AddProductScreen                # Don't use PascalCase
add-product-screen              # Missing type prefix
```

---

## Common Pitfalls to Avoid

### 1. Don't Mix Conventions

```dart
❌ AVOID:
const String API_URL = 'https://...';  // Dart uses lowerCamelCase, not UPPER_SNAKE_CASE
final String UserName = 'John';        // Use lowerCamelCase for variables
class product_model {}                 // Use PascalCase for classes
void Fetch_Products() {}               // Use lowerCamelCase for functions

✅ CORRECT:
const String apiUrl = 'https://...';   // lowerCamelCase
final String userName = 'John';        // lowerCamelCase
class ProductModel {}                  // PascalCase
void fetchProducts() {}                // lowerCamelCase
```

### 2. File Names Must Be snake_case

```
❌ AVOID:
lib/features/auth/LoginScreen.dart     # Files should be snake_case
lib/features/products/ProductCard.dart
lib/core/network/ApiClient.dart

✅ CORRECT:
lib/features/auth/login_screen.dart    # snake_case for files
lib/features/products/product_card.dart
lib/core/network/api_client.dart
```

### 3. Don't Use Abbreviations Unless Standard

```dart
❌ AVOID:
final usrNm = 'John';  // Unclear abbreviation
final prdCnt = 10;     // Unclear abbreviation
final tmpData = [];    // What is tmp?

✅ CORRECT:
final userName = 'John';
final productCount = 10;
final temporaryData = [];

✅ ACCEPTABLE (standard abbreviations):
final apiUrl = 'https://...';  // API is standard
final httpClient = Dio();      // HTTP is standard
final userId = '123';          // ID is standard
```

---

## AI Assistant Reminders

When working with this codebase, ALWAYS:

1. **Files:** snake_case (e.g., `login_screen.dart`, `product_model.dart`)
2. **Classes:** PascalCase (e.g., `LoginScreen`, `ProductModel`)
3. **Variables/Functions:** lowerCamelCase (e.g., `userName`, `fetchProducts()`)
4. **Constants:** lowerCamelCase (e.g., `const apiUrl`, NOT `API_URL`)
5. **Private members:** Prefix with underscore (e.g., `_privateMethod`)
6. **Enums:** PascalCase name, lowerCamelCase values (e.g., `enum UserRole { admin, user }`)
7. **Imports:** Order by Dart → Flutter → Packages → Project
8. **Test files:** snake_case with `_test.dart` suffix

If you're unsure, check this file or ask before implementing.

---

**Last Updated:** 2025-01-04
**Convention Version:** 1.0.0
