# Checklist de Calidad - bMOI Backend API

> **Lista de verificación para mantener estándares de calidad**
> **Usar antes de cada commit, PR y release**

---

## 📋 Índice

1. [Pre-Commit Checklist](#pre-commit-checklist)
2. [Pull Request Checklist](#pull-request-checklist)
3. [Pre-Release Checklist](#pre-release-checklist)
4. [Code Review Checklist](#code-review-checklist)
5. [Security Checklist](#security-checklist)
6. [Performance Checklist](#performance-checklist)

---

## ✅ Pre-Commit Checklist

### Antes de cada `git commit`:

#### Código
- [ ] No hay `console.log()` (usar Logger en su lugar)
- [ ] No hay `any` types
- [ ] No hay `@ts-ignore` o `@ts-expect-error` sin comentario justificando
- [ ] No hay código comentado (eliminar o documentar por qué está)
- [ ] No hay TODOs sin issue asociado
- [ ] Variables y funciones tienen nombres descriptivos
- [ ] Funciones <50 líneas (idealmente <30)
- [ ] Clases <300 líneas

#### Nomenclatura
- [ ] Archivos en `kebab-case` (users.controller.ts)
- [ ] Clases en `PascalCase` (UsersController)
- [ ] Métodos/variables en `camelCase` (findAll)
- [ ] Constantes en `UPPER_SNAKE_CASE` (JWT_SECRET)
- [ ] **CERO** `snake_case` en TypeScript

#### Multi-Tenant
- [ ] Todos los controllers tienen `@UseGuards(JwtAuthGuard, TenantGuard)`
- [ ] Todos los controllers tienen `@UseInterceptors(TenantInterceptor)`
- [ ] Todos los servicios ejecutan `set_config` antes de queries
- [ ] Todos los inserts incluyen `tenant_id` explícitamente
- [ ] No hay queries sin tenant context

#### Seguridad
- [ ] Passwords hasheados con bcrypt
- [ ] Passwords **NUNCA** en responses
- [ ] DTOs validados con class-validator
- [ ] No hay secrets hardcodeados
- [ ] SQL parametrizado (no concatenación)

#### Testing
- [ ] Tests unitarios escritos para servicios nuevos
- [ ] Tests E2E escritos para endpoints nuevos
- [ ] Tests pasan localmente (`npm run test`)
- [ ] Coverage >80% en nuevos archivos

#### Linting
- [ ] `npm run lint` pasa sin errores
- [ ] `npm run format` ejecutado
- [ ] No hay warnings de ESLint ignorados

#### Documentación
- [ ] Métodos públicos tienen JSDoc
- [ ] Swagger decorators en controllers (`@ApiOperation`, `@ApiResponse`)
- [ ] README actualizado si es necesario
- [ ] CHANGELOG actualizado si es release

---

## 📝 Pull Request Checklist

### Antes de crear un PR:

#### General
- [ ] Branch actualizado con `main` o `develop`
- [ ] Título del PR es descriptivo (ej: `feat: add user export endpoint`)
- [ ] Descripción explica QUÉ cambia y POR QUÉ
- [ ] PR vinculado a issue correspondiente
- [ ] Screenshots/videos si hay cambios en UI (N/A backend)

#### Testing
- [ ] Todos los tests pasan en CI
- [ ] Coverage no disminuyó
- [ ] Tests E2E pasan
- [ ] Tests de integración pasan

#### Código
- [ ] Sin conflictos de merge
- [ ] Sin cambios de formato masivos (usar commits separados)
- [ ] Sin cambios no relacionados
- [ ] Archivos de configuración no commitados (`.env`, `.DS_Store`)

#### Documentación
- [ ] README actualizado si es necesario
- [ ] API docs actualizadas (Swagger)
- [ ] Migration script incluido si hay cambios de DB
- [ ] Seed script actualizado si es necesario

#### Seguridad
- [ ] No se exponen secrets en código
- [ ] No se exponen datos sensibles en logs
- [ ] Validación de inputs en todos los endpoints
- [ ] Aislamiento multi-tenant verificado

#### Performance
- [ ] No hay N+1 queries
- [ ] Queries optimizadas con indexes
- [ ] Cache implementado donde es apropiado
- [ ] No hay memory leaks evidentes

---

## 🚀 Pre-Release Checklist

### Antes de cada release a producción:

#### Testing
- [ ] Todos los tests unitarios pasan
- [ ] Todos los tests E2E pasan
- [ ] Tests de carga ejecutados (si aplica)
- [ ] Tests de seguridad ejecutados (OWASP ZAP, etc.)
- [ ] Smoke tests en staging

#### Database
- [ ] Migrations testeadas en staging
- [ ] Backup de producción realizado
- [ ] Rollback script preparado
- [ ] RLS policies verificadas
- [ ] Indexes creados/actualizados

#### Seguridad
- [ ] Secrets rotados (si es necesario)
- [ ] Dependencies actualizadas (`npm audit`)
- [ ] OWASP Top 10 verificado
- [ ] Vulnerabilidades conocidas parcheadas
- [ ] JWT keys válidas

#### Configuración
- [ ] Variables de entorno configuradas en producción
- [ ] Feature flags configurados
- [ ] Rate limits configurados
- [ ] CORS configurado correctamente
- [ ] Logging levels correctos (INFO en prod, DEBUG en dev)

#### Documentación
- [ ] CHANGELOG actualizado
- [ ] Version bump (`package.json`)
- [ ] API docs publicadas
- [ ] Migration guide (si breaking changes)
- [ ] Deployment docs actualizadas

#### Infraestructura
- [ ] Health checks funcionando
- [ ] Monitoring configurado (Prometheus/Grafana)
- [ ] Alerts configurados
- [ ] Log aggregation funcionando
- [ ] Backups automáticos configurados

#### Deployment
- [ ] Deployment script testeado
- [ ] Rollback plan documentado
- [ ] On-call schedule definido
- [ ] Stakeholders notificados
- [ ] Maintenance window comunicado (si aplica)

---

## 👀 Code Review Checklist

### Al revisar PRs de otros:

#### Arquitectura
- [ ] Código sigue arquitectura de capas
- [ ] No hay lógica de negocio en controllers
- [ ] No hay acceso directo a DB en controllers
- [ ] Repository pattern usado correctamente
- [ ] Dependency injection usado correctamente

#### Código
- [ ] Código legible y mantenible
- [ ] No hay duplicación (DRY)
- [ ] Funciones tienen responsabilidad única (SRP)
- [ ] Manejo de errores apropiado
- [ ] No hay side effects inesperados

#### Multi-Tenant
- [ ] Guards aplicados en todos los endpoints
- [ ] Tenant context establecido en servicios
- [ ] RLS verificado
- [ ] No hay queries cross-tenant
- [ ] Aislamiento garantizado

#### Testing
- [ ] Tests cubren casos edge
- [ ] Tests son legibles
- [ ] Mocks usados apropiadamente
- [ ] No hay tests flaky
- [ ] Coverage adecuado

#### Performance
- [ ] No hay queries N+1
- [ ] Queries optimizadas
- [ ] Indexes apropiados
- [ ] Cache usado donde corresponde
- [ ] No hay blocking operations innecesarias

#### Seguridad
- [ ] Input validation completa
- [ ] Output sanitization
- [ ] No hay SQL injection risks
- [ ] No hay XSS risks
- [ ] Passwords nunca en logs

---

## 🔒 Security Checklist

### Verificar periódicamente:

#### Authentication & Authorization
- [ ] JWT validation funciona correctamente
- [ ] Refresh token rotation implementado
- [ ] Passwords hasheados con bcrypt (10+ rounds)
- [ ] Rate limiting en endpoints de auth
- [ ] Account lockout después de X intentos fallidos

#### Input Validation
- [ ] Todos los DTOs validados con class-validator
- [ ] Validación de tipos
- [ ] Validación de rangos
- [ ] Validación de formatos (email, URL, etc.)
- [ ] Sanitización de HTML (si aplica)

#### Data Protection
- [ ] Passwords nunca en responses
- [ ] Tokens nunca en logs
- [ ] PII encriptada en DB (si es requerido)
- [ ] HTTPS enforced en producción
- [ ] Sensitive headers no expuestos

#### Database Security
- [ ] RLS policies activas
- [ ] SQL parametrizado (no concatenación)
- [ ] Least privilege en DB users
- [ ] Connection pooling configurado
- [ ] Backups encriptados

#### Dependencies
- [ ] `npm audit` sin vulnerabilidades críticas/altas
- [ ] Dependencies actualizadas regularmente
- [ ] No hay dependencies deprecated
- [ ] Lockfile commiteado (`package-lock.json`)

#### Infrastructure
- [ ] Secrets en variables de entorno (no en código)
- [ ] Secrets Manager en producción (Vault, AWS Secrets)
- [ ] Firewall configurado
- [ ] WAF configurado (si aplica)
- [ ] DDoS protection activa

---

## ⚡ Performance Checklist

### Optimizaciones a verificar:

#### Database
- [ ] Indexes en columnas frecuentemente filtradas
- [ ] Indexes compuestos para queries multi-columna
- [ ] `EXPLAIN ANALYZE` ejecutado en queries complejas
- [ ] No hay full table scans en queries frecuentes
- [ ] Connection pool size apropiado

#### Caching
- [ ] Cache implementado en queries pesadas
- [ ] TTL apropiado configurado
- [ ] Cache invalidation implementado
- [ ] Cache hit rate monitoreado
- [ ] No hay cache stampede

#### API
- [ ] Pagination implementada en list endpoints
- [ ] Limit máximo enforceado (ej: max 100 items)
- [ ] Response size monitoreado
- [ ] Compression habilitado (gzip)
- [ ] ETags implementados (si aplica)

#### Code
- [ ] No hay N+1 queries
- [ ] Lazy loading usado apropiadamente
- [ ] Async/await usado correctamente
- [ ] No hay blocking operations
- [ ] Memory leaks verificados

#### Monitoring
- [ ] Response times <500ms para p95
- [ ] Error rate <1%
- [ ] CPU usage <70%
- [ ] Memory usage <80%
- [ ] Database connections <80% pool

---

## 📊 Métricas de Calidad

### Targets mínimos:

| Métrica | Target | Crítico |
|---------|--------|---------|
| Test Coverage | >80% | >95% para auth/tenant modules |
| Response Time (p95) | <500ms | <1s |
| Error Rate | <1% | <0.1% en prod |
| Uptime | >99.9% | >99.99% en prod |
| Security Vulnerabilities | 0 críticas | 0 altas en prod |
| Technical Debt | <10% | <20% |
| Code Duplication | <5% | <10% |
| Cyclomatic Complexity | <10 per función | <20 |

---

## 🔄 Checklist Automático

### Configurar en CI/CD:

```yaml
# .github/workflows/quality.yml
name: Quality Checks

on: [push, pull_request]

jobs:
  lint:
    - npm run lint
    - npm run format:check

  test:
    - npm run test
    - npm run test:e2e
    - npm run test:cov

  security:
    - npm audit
    - snyk test (opcional)

  build:
    - npm run build
```

---

## 📝 Templates

### Commit Message Template

```
<type>(<scope>): <subject>

<body>

<footer>

🤖 Generated with Claude Code
Co-Authored-By: Claude <noreply@anthropic.com>
```

**Types**: feat, fix, docs, style, refactor, test, chore

### PR Template

```markdown
## Description
[Describe what changes and why]

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Checklist
- [ ] Tests pass
- [ ] Lint passes
- [ ] Documentation updated
- [ ] No breaking changes (or documented)

## Related Issues
Closes #[issue number]
```

---

<div align="center">

**Calidad es responsabilidad de todos**

Usa este checklist en cada fase del desarrollo

</div>
