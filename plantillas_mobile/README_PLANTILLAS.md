# 📄 README_PLANTILLAS.md - Mobile App Template Guide

## Overview

This document is the **complete guide** for using the bMOI Mobile App templates. These are **instruction templates** (not working code) designed to guide developers and AI assistants in building the Flutter mobile application from scratch.

**Template Type:** Instruction/Documentation templates
**Purpose:** Guide for creating Flutter 3.19+ mobile app (iOS + Android)
**Architecture:** Clean Architecture (Feature-First)
**State Management:** Riverpod 2.5+

---

## 🎯 What Are These Templates?

These templates provide **comprehensive instructions** for building the bMOI mobile app with:

✅ **Complete documentation** for AI assistants (Claude Code, GitHub Copilot, Cursor)
✅ **Architecture decisions** explained (why Flutter vs React Native, why Riverpod vs BLoC, etc.)
✅ **Naming conventions** enforced (snake_case for files, PascalCase for classes)
✅ **Code examples** for all layers (Data, Domain, Presentation)
✅ **Multi-tenant integration** patterns
✅ **Offline-first** strategies
✅ **Docker configuration** for CI/CD builds
✅ **Deployment guides** for App Store and Google Play

---

## 📁 Template Structure

```
plantillas_mobile/
├── .claude/                        # AI Assistant Context (Claude Code)
│   ├── context.md                  # Complete project context & architecture
│   ├── conventions.md              # Mandatory naming conventions
│   └── session-state.md            # Development state tracking
│
├── .copilot/                       # GitHub Copilot Instructions
│   └── instructions.md             # Copilot-specific guidance
│
├── .cursorrules                    # Cursor AI Rules
│
├── .docs/                          # Comprehensive Documentation
│   ├── README.md                   # Project overview
│   ├── SETUP.md                    # Step-by-step setup guide
│   ├── ARQUITECTURA.md             # Architecture decisions (Flutter vs RN, etc.)
│   ├── DEPLOYMENT.md               # Deployment to App Store & Google Play
│   ├── CODING_STANDARDS.md         # Coding standards with examples
│   └── EJEMPLOS_CODIGO.md          # Complete code examples
│
├── CLAUDE.md                       # Quick reference for Claude Code
├── README.md                       # Standard README
├── README_PLANTILLAS.md            # This file (template guide)
├── pubspec.yaml                    # Flutter dependencies template
├── analysis_options.yaml           # Dart analyzer config
├── .gitignore                      # Git ignore rules
├── .dockerignore                   # Docker ignore rules
├── Dockerfile                      # Production Docker build (CI/CD)
└── docker-compose.yml              # Docker for CI/CD builds
```

---

## 📚 Documentation Files Explained

### `.claude/` Folder - AI Assistant Context

#### `context.md` (20KB)
**Purpose:** Complete project context for Claude Code

**Contains:**
- Project identity and purpose
- Technology stack decisions (Flutter, Riverpod, Go Router)
- Architecture decisions explained (Clean Architecture, feature-first)
- Multi-tenant integration strategy
- Offline-first capabilities design
- Folder structure breakdown
- Development workflow (local + Docker)
- Key features & screens
- Integration with bMOI ecosystem
- Firebase integration
- Performance optimizations
- Security considerations
- Accessibility guidelines
- Internationalization strategy (i18n)
- Development phases breakdown (0-6)
- Next steps from zero to production

**When to read:** First thing when starting any work on this project

#### `conventions.md` (18KB)
**Purpose:** Mandatory naming conventions (CRITICAL for Dart!)

**Contains:**
- File naming conventions (snake_case for all Dart files)
- Code naming conventions (PascalCase, lowerCamelCase)
- Folder structure conventions
- Widget naming conventions
- Riverpod provider naming
- Model & entity naming
- Use case naming conventions
- Repository naming (interface + implementation)
- Extension naming
- Import/export conventions (import ordering)
- Test file naming
- Asset path constants
- Git commit conventions (Conventional Commits)
- Branch naming conventions
- Common pitfalls to avoid

**When to read:** Before writing any code (conventions are DIFFERENT from TypeScript!)

**Key difference:** Dart uses snake_case for files and lowerCamelCase for constants (unlike TypeScript)

#### `session-state.md` (8KB)
**Purpose:** Track development progress

**Contains:**
- Current project status
- Phase progress (0-6)
- Recent changes log
- Active development session info
- Technical decisions made
- Problems encountered & solutions
- Files created/modified
- Next steps

**When to update:** At the end of each development session

### `.copilot/` Folder - GitHub Copilot Instructions

#### `instructions.md` (10KB)
**Purpose:** Guide GitHub Copilot when writing code

**Contains:**
- Project context summary
- Core technologies
- Mandatory conventions (snake_case files, lowerCamelCase constants)
- Clean Architecture structure
- Common code patterns (Freezed models, repositories, use cases, providers, screens)
- Dio setup with interceptors
- Error handling with Either
- Import ordering
- Common mistakes to avoid
- Quick reference

**When used:** Automatically by GitHub Copilot when writing code

### `.cursorrules` File - Cursor AI Rules

**Purpose:** Configure Cursor AI behavior for this project

**Contains:**
- Project identity
- Core principles (Clean Architecture, type safety, offline-first)
- Mandatory file naming (snake_case!)
- Code standards (all layers)
- Dio setup with multi-tenant support
- Error handling patterns
- Import ordering
- Common mistakes to avoid
- Testing standards
- Before committing checklist

**When used:** Automatically by Cursor AI when working in this project

---

## 🚀 How to Use These Templates

### For Developers (Manual Setup)

1. **Read the documentation** in this order:
   - `README.md` - Get overview
   - `.docs/SETUP.md` - Follow step-by-step setup
   - `.docs/ARQUITECTURA.md` - Understand decisions
   - `.docs/CODING_STANDARDS.md` - Learn conventions

2. **Install Flutter SDK:**
   ```bash
   # Follow official guide
   # https://docs.flutter.dev/get-started/install

   flutter doctor -v
   ```

3. **Create new Flutter project:**
   ```bash
   flutter create bmoi_mobile_app
   cd bmoi_mobile_app
   ```

4. **Add dependencies to `pubspec.yaml`:**
   - flutter_riverpod
   - riverpod_annotation
   - go_router
   - dio
   - freezed
   - json_serializable
   - etc.

5. **Setup folder structure** following Clean Architecture (see `.claude/context.md`)

6. **Follow the examples** in `.docs/EJEMPLOS_CODIGO.md`

7. **Reference conventions** in `.claude/conventions.md` while coding

### For AI Assistants (Claude Code, Copilot, Cursor)

1. **Claude Code:**
   - Reads `.claude/context.md` first
   - Follows `.claude/conventions.md` for naming
   - Updates `.claude/session-state.md` after each session
   - Uses `CLAUDE.md` for quick reference

2. **GitHub Copilot:**
   - Automatically reads `.copilot/instructions.md`
   - Suggests code following the patterns defined
   - Respects conventions (snake_case files, lowerCamelCase constants)

3. **Cursor AI:**
   - Automatically applies `.cursorrules`
   - Enforces naming conventions
   - Suggests code following project standards

### For Teams

1. **Onboarding new developers:**
   - Share `README.md` for overview
   - Have them read `.docs/SETUP.md` step-by-step
   - Review `.docs/ARQUITECTURA.md` together
   - Pair program using `.docs/EJEMPLOS_CODIGO.md`

2. **Code reviews:**
   - Check against `.claude/conventions.md`
   - Verify Clean Architecture layers respected
   - Test on both iOS and Android
   - Run `flutter analyze` and `flutter test`

3. **AI-assisted development:**
   - Ensure AI reads context files before starting
   - Update `.claude/session-state.md` regularly
   - Use AI memory checkpoints to prevent context loss

---

## 🎯 Key Decisions Explained

### Why Flutter?

**Decision:** Use Flutter 3.19+ (not React Native, not Native iOS/Android)

**Reasons:**
- ✅ **Single codebase** - iOS + Android with one code
- ✅ **Native performance** - Compiled to native ARM code (not WebView)
- ✅ **Beautiful UI** - Material Design + Cupertino widgets
- ✅ **Hot reload** - Instant feedback (dev productivity++)
- ✅ **Strong typing** - Dart with null safety
- ✅ **Cost-effective** - No need for separate iOS/Android teams

**Trade-offs:**
- ❌ Larger app size than native (but acceptable with modern storage)
- ❌ Smaller ecosystem than React Native (but growing rapidly)

### Why Riverpod?

**Decision:** Use Riverpod 2.5+ (not Provider, not BLoC, not GetX)

**Reasons:**
- ✅ **Compile-time safety** - No runtime errors
- ✅ **Testable** - Easy to mock providers
- ✅ **No BuildContext required** - Access state anywhere
- ✅ **Auto-disposal** - Automatic cleanup
- ✅ **Code generation** - Less boilerplate with riverpod_generator

**Trade-offs:**
- ❌ Learning curve for beginners (but worth it for safety)

### Why Clean Architecture?

**Decision:** Clean Architecture with Feature-First organization

**Reasons:**
- ✅ **Separation of concerns** - Clear layer boundaries
- ✅ **Testable** - Domain logic independent of UI/data
- ✅ **Scalable** - Easy to add new features
- ✅ **Maintainable** - Clear organization

**Layers:**
```
Data Layer (models, repositories impl, data sources)
  ↕️
Domain Layer (entities, repositories interface, use cases)
  ↕️
Presentation Layer (providers, screens, widgets)
```

### Why Feature-First?

**Decision:** Organize by feature, not by layer

**Reasons:**
- ✅ **Faster navigation** - All feature files together
- ✅ **Team scalability** - Different teams on different features
- ✅ **Easier refactoring** - Self-contained features

**Structure:**
```
lib/features/auth/      # All auth-related code here
lib/features/products/  # All product-related code here
lib/features/orders/    # All order-related code here
```

### Why Dart Naming Conventions?

**CRITICAL:** Dart naming is DIFFERENT from TypeScript/JavaScript!

| Type | Dart | TypeScript | Example (Dart) |
|------|------|------------|----------------|
| Files | snake_case | kebab-case | login_screen.dart |
| Classes | PascalCase | PascalCase | LoginScreen |
| Variables | lowerCamelCase | camelCase | userName |
| Constants | lowerCamelCase | UPPER_SNAKE_CASE | const apiUrl |
| Private | _prefix | #prefix (TS) | _privateMethod |

**Why follow Dart conventions?**
- ✅ Consistent with official Dart/Flutter code
- ✅ Dart analyzer enforces these conventions
- ✅ Better integration with pub.dev packages

---

## 🧠 AI Assistant Usage Guidelines

### Best Practices

1. **Always read context first**
   ```
   Before starting any task, read:
   1. .claude/context.md
   2. .claude/conventions.md (CRITICAL for naming!)
   3. .claude/session-state.md
   ```

2. **Follow Dart naming conventions STRICTLY**
   - Files: snake_case (e.g., `login_screen.dart`)
   - Classes: PascalCase (e.g., `LoginScreen`)
   - Variables: lowerCamelCase (e.g., `userName`)
   - Constants: lowerCamelCase (e.g., `const apiUrl`, NOT `API_URL`)

3. **Update session state**
   - Update `.claude/session-state.md` after each session
   - Log decisions made, problems solved, next steps

4. **Use memory checkpoints**
   - Every ~5 messages, verify you remember key facts
   - Project name, framework, conventions, etc.

### AI Memory Checkpoints

Every 5 messages, AI should verify:
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

---

## 📊 Development Phases

### Phase 0: Setup (Days 1-2)
- Install Flutter SDK 3.19+
- Setup Android Studio / VS Code
- Configure iOS (Xcode, CocoaPods)
- Initialize Flutter project
- Add dependencies (Riverpod, Dio, Go Router, Freezed)
- Setup folder structure (Clean Architecture)
- Configure Docker (optional, for CI/CD)

### Phase 1: Core Infrastructure (Days 3-5)
- Setup Dio with interceptors (auth, tenant, error)
- Implement secure storage (Flutter Secure Storage)
- Create app theme (Material + Cupertino)
- Setup Go Router navigation
- Implement error handling (Either/Result)
- Setup Riverpod providers structure
- Configure Firebase (Analytics, Crashlytics, FCM)

### Phase 2: Authentication (Days 6-8)
- Create auth models (User, AuthResponse)
- Implement auth repository (domain + data)
- Create login screen
- Create register screen
- Implement JWT storage (secure)
- Add auto-logout on token expiry
- Add biometric authentication (Face ID, Fingerprint)

### Phase 3: Core Features (Days 9-15)
- Dashboard screen (overview, metrics)
- Products list screen
- Product details screen
- Add/Edit product screen (with image upload)
- Orders list screen
- Order details screen
- Create order screen
- User profile screen
- Settings screen

### Phase 4: Offline Support (Days 16-18)
- Setup Hive for local storage
- Implement cache-first repository strategy
- Add sync queue for offline writes
- Handle network connectivity changes
- Show offline indicator in UI
- Implement conflict resolution

### Phase 5: Polish & Testing (Days 19-21)
- Unit tests (repositories, use cases, providers)
- Widget tests (screens, widgets)
- Integration tests (full flows)
- Performance optimization (list rendering, images)
- Accessibility audit (screen readers, contrast)
- Fix bugs found in testing

### Phase 6: Deployment (Days 22-25)
- Configure Firebase completely
- Setup Android signing (keystore)
- Setup iOS signing (certificates, provisioning profiles)
- Build APK/AAB for Google Play
- Build IPA for App Store
- Create app store listings (screenshots, descriptions)
- Submit to Google Play Store
- Submit to Apple App Store

---

## 🔗 Integration with bMOI Ecosystem

The mobile app integrates with the bMOI ecosystem:

```
Mobile App → Backend API (JWT + X-Tenant-ID) → PostgreSQL (RLS)
```

**Authentication Flow:**
1. User logs in via mobile app
2. Backend returns JWT + tenant info
3. App stores JWT + tenant ID (Flutter Secure Storage)
4. All API requests auto-inject: `Authorization: Bearer {jwt}` + `X-Tenant-ID: {tenantId}`
5. Backend filters data by tenant (RLS)

**API Endpoints Used:**
- `POST /auth/login` - Login
- `POST /auth/refresh` - Refresh token
- `GET /products` - Get products (auto-filtered by tenant)
- `POST /products` - Create product
- `GET /orders` - Get orders
- etc.

**Dio Interceptor (Auto-Inject Headers):**
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
}
```

---

## 🎨 Code Examples Quick Reference

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

### Use Case

```dart
class LoginUseCase {
  final AuthRepository repository;

  Future<Either<Failure, User>> call({
    required String email,
    required String password,
  }) {
    return repository.login(email: email, password: password);
  }
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

---

## 📞 Support & Resources

### Documentation
- **Flutter Docs:** https://docs.flutter.dev/
- **Dart Docs:** https://dart.dev/guides
- **Riverpod Docs:** https://riverpod.dev/
- **Go Router:** https://pub.dev/packages/go_router

### Related Projects
- **Backend API:** `bo-backend-api/CLAUDE.md`
- **Admin Panel:** `plantillas_admin/.claude/context.md`
- **Landing Page:** `plantillas_landing/.claude/context.md`

### Getting Help
If you encounter issues:
1. Check `.docs/` folder for detailed guides
2. Review `.claude/context.md` for architecture decisions
3. Consult `.docs/EJEMPLOS_CODIGO.md` for code examples
4. Check `.claude/session-state.md` for recent changes

---

## ✅ Template Checklist

Before starting development, ensure:
- [ ] Read `.claude/context.md` completely
- [ ] Understood `.claude/conventions.md` (CRITICAL for Dart!)
- [ ] Reviewed `.docs/ARQUITECTURA.md`
- [ ] Checked `.docs/SETUP.md` for setup steps
- [ ] Configured your AI assistant (Claude/Copilot/Cursor)

During development:
- [ ] Follow Dart naming conventions (snake_case files, lowerCamelCase constants)
- [ ] Respect Clean Architecture layers (Data → Domain → Presentation)
- [ ] Type everything (no dynamic types)
- [ ] Handle null safety properly
- [ ] Test on both iOS and Android
- [ ] Run `flutter analyze` (no warnings)
- [ ] Run `flutter test` (all tests pass)

After development:
- [ ] Update `.claude/session-state.md`
- [ ] Document technical decisions
- [ ] Run build and check for errors
- [ ] Test on real devices (iOS + Android)
- [ ] Verify offline functionality

---

**Template Version:** 1.0.0
**Last Updated:** 2025-01-04
**Status:** Initial Creation

<div align="center">

**Complete instruction templates for building the bMOI Mobile App with Flutter**

</div>
