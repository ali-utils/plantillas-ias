# Coding Standards - bMOI Mobile App (Flutter + Dart)

**Lenguaje:** Dart 3.3+ | **Framework:** Flutter 3.19+ | **Style Guide:** [Effective Dart](https://dart.dev/guides/language/effective-dart)

---

## 📋 Convenciones de Nombres (OBLIGATORIAS)

### Archivos: snake_case

```dart
// ✅ CORRECTO
login_screen.dart
product_model.dart
auth_repository.dart
string_extensions.dart

// ❌ INCORRECTO
LoginScreen.dart          // PascalCase (NO en Dart)
product-model.dart        // kebab-case (NO en Dart)
AuthRepository.dart       // PascalCase (NO en Dart)
```

### Clases: PascalCase

```dart
// ✅ CORRECTO
class LoginScreen extends StatelessWidget { }
class ProductModel { }
class AuthRepository { }

// ❌ INCORRECTO
class loginScreen { }     // lowerCamelCase
class product_model { }   // snake_case
```

### Variables y Funciones: lowerCamelCase

```dart
// ✅ CORRECTO
String userName = 'John';
int productCount = 10;
void fetchProducts() { }
Future<User> getCurrentUser() async { }

// ❌ INCORRECTO
String UserName = 'John';      // PascalCase
int product_count = 10;        // snake_case
void FetchProducts() { }       // PascalCase
```

### Constantes: lowerCamelCase (Dart convention!)

```dart
// ✅ CORRECTO (Dart usa lowerCamelCase para constantes)
const String apiUrl = 'https://api.bmoi.com';
const int maxRetries = 3;
const Duration timeout = Duration(seconds: 30);

// ❌ INCORRECTO (UPPER_SNAKE_CASE es de otros lenguajes)
const String API_URL = 'https://...';  // NO en Dart
const int MAX_RETRIES = 3;             // NO en Dart
```

### Privados: Prefijo _

```dart
// ✅ CORRECTO
class User {
  final String _password;  // Privado
  String get password => _password;  // Getter público

  void _validateEmail() { }  // Método privado
}
```

---

## 🏗️ Estructura de Archivos

### Imports Order

```dart
// 1. Dart SDK
import 'dart:async';
import 'dart:convert';

// 2. Flutter
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

// 3. Third-party packages
import 'package:dio/dio.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 4. Local imports
import 'package:bmoi_mobile_app/core/network/dio_client.dart';
import 'package:bmoi_mobile_app/features/auth/domain/entities/user.dart';
```

### Organización de Clase

```dart
class ProductScreen extends ConsumerStatefulWidget {
  // 1. Constructor parameters (const si es posible)
  const ProductScreen({
    super.key,
    required this.productId,
  });

  // 2. Properties
  final String productId;

  // 3. createState
  @override
  ConsumerState<ProductScreen> createState() => _ProductScreenState();
}

class _ProductScreenState extends ConsumerState<ProductScreen> {
  // 1. State variables
  bool _isLoading = false;
  String? _error;

  // 2. Lifecycle methods
  @override
  void initState() {
    super.initState();
    _loadProduct();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  // 3. Private methods
  Future<void> _loadProduct() async { }

  void _handleSubmit() { }

  // 4. Build method (always last)
  @override
  Widget build(BuildContext context) {
    return Scaffold( /* ... */ );
  }
}
```

---

## 🎨 Widget Best Practices

### 1. Use Const Constructors

```dart
// ✅ CORRECTO - Mejor performance
const Text('Hello');
const SizedBox(height: 16);
const Icon(Icons.check);

// ⚠️ EVITAR - Reconstruye en cada build
Text('Hello');
SizedBox(height: 16);
```

### 2. Extract Widgets

```dart
// ❌ INCORRECTO - Widget gigante
class ProductScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // 100 líneas de UI aquí...
          Container(
            child: Column(
              children: [
                // Más nested widgets...
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// ✅ CORRECTO - Widgets extraídos
class ProductScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          const _ProductHeader(),
          const _ProductDetails(),
          _ProductActions(),
        ],
      ),
    );
  }
}

class _ProductHeader extends StatelessWidget {
  const _ProductHeader();

  @override
  Widget build(BuildContext context) {
    return Container( /* ... */ );
  }
}
```

### 3. Responsive Sizing

```dart
// ✅ CORRECTO - Responsive
SizedBox(
  width: MediaQuery.of(context).size.width * 0.8,
  height: MediaQuery.of(context).size.height * 0.3,
)

// O mejor aún, usar LayoutBuilder
LayoutBuilder(
  builder: (context, constraints) {
    return Container(
      width: constraints.maxWidth * 0.8,
    );
  },
)
```

---

## 🔄 Riverpod Patterns

### Provider Definition

```dart
// ✅ CORRECTO - Con code generation
@riverpod
class Products extends _$Products {
  @override
  FutureOr<List<Product>> build() async {
    final repository = ref.watch(productsRepositoryProvider);
    return repository.getProducts();
  }

  Future<void> createProduct(Product product) async {
    state = const AsyncValue.loading();
    final repository = ref.read(productsRepositoryProvider);

    final result = await repository.createProduct(product);

    result.fold(
      (failure) => state = AsyncValue.error(failure, StackTrace.current),
      (product) {
        // Update state
        state = AsyncValue.data([...state.value!, product]);
      },
    );
  }
}

// ✅ Uso en UI
class ProductsScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final productsAsync = ref.watch(productsProvider);

    return productsAsync.when(
      data: (products) => ListView.builder( /* ... */ ),
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

---

## 💾 Data Models

### Freezed Models

```dart
// ✅ CORRECTO - Immutable con Freezed
@freezed
class User with _$User {
  const factory User({
    required String id,
    required String email,
    required String name,
    String? avatar,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) =>
      _$UserFromJson(json);
}

// Auto-genera:
// - copyWith
// - ==, hashCode
// - toString
// - fromJson, toJson
```

### Entities vs Models

```dart
// Domain Entity (business logic)
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

// Data Model (from API)
@freezed
class UserModel with _$UserModel {
  const factory UserModel({
    required String id,
    required String email,
    required String name,
    @JsonKey(name: 'avatar_url') String? avatarUrl,
  }) = _UserModel;

  factory UserModel.fromJson(Map<String, dynamic> json) =>
      _$UserModelFromJson(json);

  // Mapper to Entity
  User toEntity() {
    return User(
      id: id,
      email: email,
      name: name,
    );
  }
}
```

---

## 🔐 Error Handling

### Use Either for Results

```dart
// ✅ CORRECTO - Result pattern con dartz
import 'package:dartz/dartz.dart';

Future<Either<Failure, User>> login(String email, String password) async {
  try {
    final response = await dio.post('/auth/login', data: {
      'email': email,
      'password': password,
    });

    final user = UserModel.fromJson(response.data).toEntity();
    return Right(user);
  } on DioException catch (e) {
    return Left(NetworkFailure(e.message ?? 'Network error'));
  } catch (e) {
    return Left(UnknownFailure(e.toString()));
  }
}

// Uso:
final result = await authRepository.login(email, password);

result.fold(
  (failure) {
    // Handle error
    showError(failure.message);
  },
  (user) {
    // Handle success
    navigateToDashboard();
  },
);
```

### Failure Classes

```dart
abstract class Failure {
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

## ✅ Testing

### Widget Test

```dart
void main() {
  group('LoginScreen', () {
    testWidgets('shows error when email is invalid', (tester) async {
      await tester.pumpWidget(
        const ProviderScope(
          child: MaterialApp(
            home: LoginScreen(),
          ),
        ),
      );

      // Find email field
      final emailField = find.byKey(const Key('email_field'));

      // Enter invalid email
      await tester.enterText(emailField, 'invalid-email');

      // Tap submit
      await tester.tap(find.byKey(const Key('submit_button')));
      await tester.pump();

      // Expect error
      expect(find.text('Invalid email'), findsOneWidget);
    });
  });
}
```

### Unit Test

```dart
void main() {
  group('LoginUseCase', () {
    late MockAuthRepository mockRepository;
    late LoginUseCase useCase;

    setUp(() {
      mockRepository = MockAuthRepository();
      useCase = LoginUseCase(mockRepository);
    });

    test('returns User when login succeeds', () async {
      // Arrange
      final user = User(id: '1', email: 'test@test.com', name: 'Test');
      when(() => mockRepository.login('test@test.com', 'password'))
          .thenAnswer((_) async => Right(user));

      // Act
      final result = await useCase('test@test.com', 'password');

      // Assert
      expect(result, Right(user));
      verify(() => mockRepository.login('test@test.com', 'password')).called(1);
    });
  });
}
```

---

## 🚨 Prohibiciones

```dart
// ❌ NUNCA usar dynamic sin razón
dynamic data = await fetchData();  // NO

// ❌ NUNCA ignorar errores
try {
  await riskyOperation();
} catch (e) {
  // Ignorado - MAL
}

// ❌ NUNCA usar print en producción
print('User logged in');  // Usar logger

// ❌ NUNCA hardcodear strings
Text('Login');  // Usar i18n
```

---

**Última actualización:** 2025-01-04
