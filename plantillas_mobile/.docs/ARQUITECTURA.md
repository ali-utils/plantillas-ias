# Arquitectura - bMOI Mobile App

**Framework:** Flutter 3.19+ | **Lenguaje:** Dart 3.3+ | **Arquitectura:** Clean Architecture

---

## 📋 Tabla de Contenidos

- [Decisiones Arquitectónicas](#-decisiones-arquitectónicas)
- [Clean Architecture](#-clean-architecture)
- [Feature-First Organization](#-feature-first-organization)
- [State Management](#-state-management-riverpod)
- [Offline-First Strategy](#-offline-first-strategy)
- [Multi-Tenant Integration](#-multi-tenant-integration)
- [Navigation](#-navigation-go-router)
- [Data Flow](#-data-flow)

---

## 🎯 Decisiones Arquitectónicas

### 1. Framework: Flutter 3.19+ ✅

**Decisión:** Usar **Flutter** sobre React Native o Desarrollo Nativo

**Por qué Flutter:**
```
✅ Single codebase → iOS + Android con un solo código
✅ Native performance → Compilado a ARM nativo
✅ Hot reload → Feedback instantáneo durante desarrollo
✅ Rich widgets → Material Design + Cupertino out-of-the-box
✅ Strong typing → Null safety de Dart
✅ Cost-effective → No necesita equipos separados iOS/Android
✅ Better DX → Menos configuración que React Native
```

**Comparación:**

| Criterio | Flutter | React Native | Native (2 códigos) |
|----------|---------|--------------|-------------------|
| **Performance** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Development Speed** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| **Native Feel** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Ecosystem** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Learning Curve** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **Cost** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |

**Resultado:** Flutter es el mejor balance entre performance, velocidad de desarrollo y costo.

---

## 🏛️ Clean Architecture

### Estructura en 3 Capas

```
┌─────────────────────────────────────────────────┐
│         PRESENTATION LAYER (UI)                  │
│  • Screens (Widgets)                            │
│  • State Management (Riverpod Providers)        │
│  • ViewModels / Controllers                     │
└──────────────┬──────────────────────────────────┘
               │ depends on
               ▼
┌─────────────────────────────────────────────────┐
│         DOMAIN LAYER (Business Logic)            │
│  • Entities (Business Models)                   │
│  • Use Cases (Business Rules)                   │
│  • Repository Interfaces (Contracts)            │
└──────────────┬──────────────────────────────────┘
               │ depends on
               ▼
┌─────────────────────────────────────────────────┐
│         DATA LAYER (External)                    │
│  • Models (from JSON)                           │
│  • Repository Implementations                   │
│  • Data Sources (API, Local Storage)            │
└─────────────────────────────────────────────────┘
```

### Beneficios

- ✅ **Testable** - Cada capa independiente
- ✅ **Maintainable** - Separación clara de responsabilidades
- ✅ **Scalable** - Fácil agregar nuevas features
- ✅ **Team-friendly** - Equipos diferentes pueden trabajar en features diferentes

---

## 📁 Feature-First Organization

```
lib/
├── core/                          # Shared functionality
│   ├── network/
│   │   ├── dio_client.dart        # Configured Dio
│   │   └── interceptors/
│   │       ├── auth_interceptor.dart
│   │       └── tenant_interceptor.dart
│   ├── storage/
│   │   ├── secure_storage_service.dart
│   │   └── local_storage_service.dart
│   ├── theme/
│   │   ├── app_theme.dart
│   │   └── app_colors.dart
│   └── utils/
│       ├── logger.dart
│       └── validators.dart
│
├── features/                      # Business features
│   ├── auth/
│   │   ├── data/
│   │   │   ├── models/
│   │   │   │   ├── user_model.dart              # JSON → Dart
│   │   │   │   └── user_model.g.dart            # Generated
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository_impl.dart    # Implementation
│   │   │   └── data_sources/
│   │   │       ├── auth_remote_data_source.dart  # API
│   │   │       └── auth_local_data_source.dart   # Hive
│   │   │
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── user.dart                    # Business model
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart         # Interface
│   │   │   └── usecases/
│   │   │       ├── login_usecase.dart
│   │   │       └── logout_usecase.dart
│   │   │
│   │   └── presentation/
│   │       ├── providers/
│   │       │   └── auth_provider.dart           # Riverpod
│   │       ├── screens/
│   │       │   ├── login_screen.dart
│   │       │   └── register_screen.dart
│   │       └── widgets/
│   │           └── login_form.dart
│   │
│   ├── dashboard/
│   ├── products/
│   └── orders/
│
├── shared/                        # Shared UI components
│   ├── widgets/
│   │   ├── custom_button.dart
│   │   └── loading_indicator.dart
│   └── extensions/
│       └── string_extensions.dart
│
└── main.dart
```

### ¿Por qué Feature-First?

```
✅ Colocación → Todo el código de "auth" en un solo lugar
✅ Navegación → Fácil encontrar archivos relacionados
✅ Escalabilidad → Agregar features no afecta otras
✅ Team collaboration → Equipos trabajan en features separadas
```

---

## 🎨 State Management: Riverpod

### Decisión: Riverpod 2.5+ ✅

**Por qué Riverpod:**
```
✅ Compile-time safety → Errores detectados en compilación
✅ No BuildContext → Acceso al estado desde cualquier lugar
✅ Auto-disposal → Limpieza automática de memoria
✅ Testable → Fácil hacer mocking de providers
✅ Code generation → Menos boilerplate
✅ DevTools → Excelente debugging
```

**Alternativas descartadas:**
```
❌ Provider → Menos potente que Riverpod
❌ BLoC → Demasiado boilerplate
❌ GetX → Magic strings, difícil mantener
❌ setState → No escalable
```

### Patrón de Providers

```dart
// 1. Provider for Repository
@riverpod
AuthRepository authRepository(AuthRepositoryRef ref) {
  final dio = ref.watch(dioClientProvider);
  final secureStorage = ref.watch(secureStorageProvider);

  return AuthRepositoryImpl(
    remoteDataSource: AuthRemoteDataSource(dio),
    localDataSource: AuthLocalDataSource(secureStorage),
  );
}

// 2. Provider for UseCase
@riverpod
LoginUseCase loginUseCase(LoginUseCaseRef ref) {
  final repository = ref.watch(authRepositoryProvider);
  return LoginUseCase(repository);
}

// 3. State Provider (UI Layer)
@riverpod
class Auth extends _$Auth {
  @override
  AsyncValue<User?> build() {
    // Initialize state
    return const AsyncValue.data(null);
  }

  Future<void> login(String email, String password) async {
    state = const AsyncValue.loading();

    final useCase = ref.read(loginUseCaseProvider);
    final result = await useCase(email, password);

    result.fold(
      (failure) => state = AsyncValue.error(failure, StackTrace.current),
      (user) => state = AsyncValue.data(user),
    );
  }
}
```

---

## 💾 Offline-First Strategy

### Arquitectura de Caché

```
User Action
     │
     ▼
Check Network Connection
     │
     ├──── Online ────────────────┐
     │                            │
     │  1. Fetch from API         │
     │  2. Update Local Cache     │
     │  3. Return Data            │
     │                            │
     └──── Offline ───────────────┤
                                  │
        1. Read from Local Cache  │
        2. Return Cached Data     │
        3. Queue Write Operations │
                                  │
                                  ▼
                        Display Data to User
                                  │
                         (When online again)
                                  │
                        Sync Queued Operations
```

### Implementación

```dart
// Repository with Cache-First Strategy
class ProductsRepositoryImpl implements ProductsRepository {
  final ProductsRemoteDataSource remoteDataSource;
  final ProductsLocalDataSource localDataSource;
  final ConnectivityService connectivity;

  @override
  Future<Either<Failure, List<Product>>> getProducts() async {
    try {
      // Check connectivity
      final isOnline = await connectivity.isConnected;

      if (isOnline) {
        // ONLINE: Fetch from API
        final products = await remoteDataSource.getProducts();

        // Save to cache
        await localDataSource.cacheProducts(products);

        return Right(products);
      } else {
        // OFFLINE: Read from cache
        final cachedProducts = await localDataSource.getCachedProducts();

        if (cachedProducts.isEmpty) {
          return Left(CacheFailure('No cached data available'));
        }

        return Right(cachedProducts);
      }
    } catch (e) {
      // Error: Try cache as fallback
      final cachedProducts = await localDataSource.getCachedProducts();
      return Right(cachedProducts);
    }
  }

  @override
  Future<Either<Failure, Product>> createProduct(Product product) async {
    final isOnline = await connectivity.isConnected;

    if (isOnline) {
      // ONLINE: Send immediately
      try {
        final created = await remoteDataSource.createProduct(product);
        await localDataSource.cacheProduct(created);
        return Right(created);
      } catch (e) {
        return Left(NetworkFailure(e.toString()));
      }
    } else {
      // OFFLINE: Queue for sync
      await localDataSource.queueCreate(product);
      return Right(product);  // Optimistic response
    }
  }
}
```

### Sincronización

```dart
// Background sync when connection restored
class SyncService {
  Future<void> syncQueuedOperations() async {
    final queuedCreates = await localDataSource.getQueuedCreates();

    for (final product in queuedCreates) {
      try {
        await remoteDataSource.createProduct(product);
        await localDataSource.removeFromQueue(product.id);
      } catch (e) {
        // Handle sync errors
        logger.e('Sync failed for product ${product.id}');
      }
    }
  }
}
```

---

## 🏢 Multi-Tenant Integration

### Headers Automáticos

```dart
// dio_client.dart
class DioClient {
  late final Dio dio;
  final FlutterSecureStorage secureStorage;

  DioClient({required this.secureStorage}) {
    dio = Dio(BaseOptions(
      baseUrl: 'https://api.bmoi.com/api/v1',
      connectTimeout: const Duration(seconds: 30),
      receiveTimeout: const Duration(seconds: 30),
    ));

    // Add interceptors
    dio.interceptors.addAll([
      AuthInterceptor(secureStorage),     // JWT
      TenantInterceptor(secureStorage),   // X-Tenant-ID
      LogInterceptor(                     // Logging
        requestBody: true,
        responseBody: true,
      ),
    ]);
  }
}

// Auth Interceptor
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

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      // Token expired → Try refresh
      final refreshed = await _refreshToken();

      if (refreshed) {
        // Retry request with new token
        final retry = await dio.fetch(err.requestOptions);
        return handler.resolve(retry);
      }
    }

    handler.next(err);
  }
}

// Tenant Interceptor
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

## 🧭 Navigation: Go Router

### Decisión: Go Router 13+ ✅

**Por qué Go Router:**
```
✅ Declarative routing → URL-based navigation
✅ Deep linking → Native deep link support
✅ Type-safe → No magic strings
✅ Nested navigation → Complex navigation trees
✅ Redirects & guards → Authentication flows
```

### Configuración

```dart
// router.dart
final router = GoRouter(
  initialLocation: '/splash',
  redirect: (context, state) {
    // Redirect logic (auth guard)
    final isAuthenticated = ref.read(authProvider).value != null;
    final isAuthRoute = state.location.startsWith('/auth');

    if (!isAuthenticated && !isAuthRoute) {
      return '/auth/login';
    }

    if (isAuthenticated && isAuthRoute) {
      return '/dashboard';
    }

    return null;  // No redirect
  },
  routes: [
    GoRoute(
      path: '/splash',
      builder: (context, state) => const SplashScreen(),
    ),
    GoRoute(
      path: '/auth/login',
      builder: (context, state) => const LoginScreen(),
    ),
    GoRoute(
      path: '/dashboard',
      builder: (context, state) => const DashboardScreen(),
      routes: [
        GoRoute(
          path: 'products',
          builder: (context, state) => const ProductsScreen(),
          routes: [
            GoRoute(
              path: ':id',
              builder: (context, state) {
                final id = state.pathParameters['id']!;
                return ProductDetailScreen(productId: id);
              },
            ),
          ],
        ),
      ],
    ),
  ],
);
```

---

## 🔄 Data Flow

### Read Flow (Query)

```
UI (Screen)
    │
    │ ref.watch(productsProvider)
    │
    ▼
Riverpod Provider
    │
    │ await useCase(tenantId)
    │
    ▼
Use Case
    │
    │ await repository.getProducts()
    │
    ▼
Repository
    │
    │ Check network
    │
    ├─ Online ──▶ Remote Data Source ──▶ API (Dio)
    │                                         │
    │                                         ▼
    │                                    Cache locally
    │
    └─ Offline ─▶ Local Data Source ──▶ Hive
                                             │
                                             ▼
                                        Return cached data
```

### Write Flow (Mutation)

```
UI (Button Press)
    │
    │ ref.read(productsProvider).createProduct(...)
    │
    ▼
Riverpod Provider
    │
    │ state = loading
    │ await useCase(product)
    │
    ▼
Use Case
    │
    │ Validate business rules
    │ await repository.create(product)
    │
    ▼
Repository
    │
    │ Check network
    │
    ├─ Online ──▶ Remote Data Source ──▶ POST /api/products
    │                                         │
    │                                         ▼
    │                                    Update cache
    │
    └─ Offline ─▶ Queue operation ──▶ Sync later
                                          │
                                          ▼
                                    Return optimistic response
```

---

## 🎯 Performance Optimizations

### 1. Lazy Loading

```dart
// Use provider with autoDispose
@riverpod
Future<List<Product>> products(ProductsRef ref) async {
  // Auto-disposed when not used
  final repository = ref.watch(productsRepositoryProvider);
  return repository.getProducts();
}
```

### 2. Pagination

```dart
// Infinite scroll
class ProductsListScreen extends ConsumerStatefulWidget {
  @override
  _ProductsListScreenState createState() => _ProductsListScreenState();
}

class _ProductsListScreenState extends ConsumerState<ProductsListScreen> {
  final _scrollController = ScrollController();
  int _page = 1;

  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
  }

  void _onScroll() {
    if (_scrollController.position.pixels ==
        _scrollController.position.maxScrollExtent) {
      // Load more
      ref.read(productsProvider).loadMore(_page++);
    }
  }
}
```

### 3. Image Caching

```dart
// Use cached_network_image
CachedNetworkImage(
  imageUrl: product.imageUrl,
  placeholder: (context, url) => Shimmer(...),
  errorWidget: (context, url, error) => Icon(Icons.error),
  memCacheWidth: 800,  // Resize for performance
);
```

---

## 📚 Recursos

- [Flutter Architecture Samples](https://github.com/brianegan/flutter_architecture_samples)
- [Riverpod Documentation](https://riverpod.dev/)
- [Clean Architecture (Uncle Bob)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

---

**Última actualización:** 2025-01-04
**Versión:** 1.0.0
