# Sistema de Memoria y Checkpoints - Backend

> **Sistema Anti-Olvido para Asistentes de IA**

---

## 🧠 ¿Por Qué Este Sistema?

Las IAs pueden "olvidar" contexto en conversaciones largas. Este sistema asegura que siempre recuerdes:

1. Qué backend estamos construyendo (NestJS multi-tenant)
2. Convenciones obligatorias (kebab-case, PascalCase, RLS)
3. Dónde quedamos (endpoints, módulos)
4. Qué sigue (próxima fase)

---

## ⏰ Checkpoint Cada 5 Mensajes

### Trigger

Cuando veas este mensaje o hayas respondido 5 veces:

```
🧠 CHECKPOINT DE MEMORIA - BACKEND
```

### Verificación Automática

```markdown
## Checkpoint de Memoria Backend - [FECHA]

### ✅ Información del Proyecto
- [ ] Nombre: bMOI Backend API Multi-Tenant
- [ ] Framework: NestJS 10 + TypeScript 5
- [ ] Database: PostgreSQL 15 con Row-Level Security
- [ ] Cache: Redis 7
- [ ] ORM: Prisma
- [ ] Docker: TODO en containers

### ✅ Convenciones OBLIGATORIAS
- [ ] Archivos: kebab-case (users.controller.ts)
- [ ] Clases: PascalCase (UsersController)
- [ ] Métodos: camelCase (findAll)
- [ ] Constantes: UPPER_SNAKE_CASE (JWT_SECRET)
- [ ] ❌ PROHIBIDO: snake_case en TypeScript
- [ ] ❌ PROHIBIDO: any type
- [ ] ❌ PROHIBIDO: console.log en producción

### ✅ Reglas Multi-Tenant OBLIGATORIAS
- [ ] Siempre usar @UseGuards(JwtAuthGuard, TenantGuard)
- [ ] Siempre @UseInterceptors(TenantInterceptor)
- [ ] Siempre set PostgreSQL session (RLS)
  ```typescript
  await this.prisma.$executeRaw`
    SELECT set_config('app.current_tenant_id', ${tenantId}, true)
  `
  ```
- [ ] Siempre incluir tenant_id en inserts
- [ ] Validar permisos con @RequirePermission()
- [ ] Verificar límites con @CheckPlanLimit()
- [ ] NUNCA queries sin tenant_id

### ✅ Seguridad
- [ ] JWT con RS256 (asymmetric)
- [ ] Refresh tokens (rotación)
- [ ] Passwords con bcrypt (10 rounds)
- [ ] NUNCA retornar passwords
- [ ] Validación con class-validator
- [ ] Rate limiting habilitado
- [ ] CORS configurado correctamente

### ✅ Testing
- [ ] Coverage objetivo: >80%
- [ ] Unit tests para services
- [ ] E2E tests para endpoints críticos
- [ ] Mock de PrismaService en tests

### ✅ Estado Actual (Actualizar)
- Último módulo implementado: ___________
- Endpoints creados hoy: ___________
- Tests escritos: ___________
- Próximo paso: ___________
```

---

## 📝 Checkpoint de Sesión (Fin del Día)

### Al Finalizar

Actualizar `session-state.md` con:

```markdown
## Última Actualización
- Fecha: YYYY-MM-DD HH:mm
- IA: Claude Code
- Fase: Fase X

## ✅ Completado Hoy

### Módulos
- [x] Auth Module
  - [x] JWT Strategy implementado
  - [x] Refresh Token Strategy
  - [x] Login endpoint (POST /auth/login)
  - [x] Logout endpoint (POST /auth/logout)

### Endpoints
- [x] POST /auth/login
- [x] POST /auth/refresh
- [x] POST /auth/logout
- [x] GET /users (con tenant isolation)
- [x] POST /users (con plan limits)

### Guards/Interceptors
- [x] JwtAuthGuard
- [x] TenantGuard
- [x] TenantInterceptor
- [x] AuditInterceptor

### Tests
- [x] auth.controller.spec.ts
- [x] auth.service.spec.ts
- [x] auth.e2e-spec.ts

## 📊 Métricas
- Módulos creados: X
- Endpoints implementados: Y
- Tests escritos: Z
- Coverage: W%

## 🎯 Decisiones Técnicas Tomadas

### 1. PostgreSQL Row-Level Security
**Decisión**: Usar RLS con tenant_id en cada tabla
**Razón**: Aislamiento garantizado a nivel de DB
**Implementación**:
```sql
CREATE POLICY tenant_isolation ON users
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);
```

### 2. JWT con RS256
**Decisión**: Usar algoritmo asimétrico RS256
**Razón**: Mayor seguridad, public key para validación
**Trade-off**: Más lento que HS256, pero más seguro

## ⚠️ Problemas Encontrados

### Problema 1: RLS no se aplicaba
- **Descripción**: Queries retornaban datos de todos los tenants
- **Causa**: No se ejecutaba set_config antes del query
- **Solución**: Crear método helper en PrismaService
```typescript
async setTenantContext(tenantId: string) {
  await this.$executeRaw`
    SELECT set_config('app.current_tenant_id', ${tenantId}, true)
  `
}
```

## 🔜 Próximos Pasos

### Mañana
- [ ] Implementar Users Module completo
- [ ] Agregar pagination a GET /users
- [ ] Implementar filters en GET /users
- [ ] Tests E2E para users endpoints

### Esta Semana
- [ ] Dashboard Module (métricas)
- [ ] Audit Module (logging)
- [ ] Plan Limits enforcement
- [ ] Rate limiting por tenant

## 📝 Notas para la Próxima Sesión

### Recordatorios Importantes
1. **RLS**: Siempre ejecutar set_config antes de queries
2. **Passwords**: Siempre hashear con bcrypt antes de guardar
3. **DTOs**: Todos los DTOs deben tener validación class-validator
4. **Swagger**: Actualizar decorators @Api* para documentación

### Contexto a Recordar
- JWT Secret está en .env (JWT_SECRET)
- Puerto backend: 5000
- PostgreSQL en docker: postgres:5432
- Redis en docker: redis:6379
- Comando para migrations: `npx prisma migrate dev`

### Archivos Críticos a Revisar
- `src/common/guards/tenant.guard.ts` - Validación de tenant
- `src/common/interceptors/tenant.interceptor.ts` - Inyección tenant context
- `prisma/schema.prisma` - Definición de modelos
```

---

## 🔄 Recordatorios Automáticos

### Frases Trigger para Revisar Contexto

Si el usuario dice:

| Frase | Acción |
|-------|--------|
| "Recuérdame..." | Leer `context.md` completo |
| "¿Cuáles son las convenciones?" | Mostrar `conventions.md` |
| "¿Cómo funciona multi-tenant?" | Explicar RLS + interceptor |
| "Empecemos de nuevo" | Leer `PROJECT_INSTRUCTIONS.md` |
| "Estado del backend" | Mostrar `session-state.md` |

---

## ✅ Checklist Antes de Responder

Antes de **cada respuesta**, verifica:

- [ ] ¿Estoy usando kebab-case para archivos?
- [ ] ¿Clases en PascalCase?
- [ ] ¿Métodos en camelCase?
- [ ] ¿He agregado @UseGuards(TenantGuard)?
- [ ] ¿He agregado @UseInterceptors(TenantInterceptor)?
- [ ] ¿El service ejecuta set_config para RLS?
- [ ] ¿Los DTOs tienen validación class-validator?
- [ ] ¿NO estoy retornando passwords?
- [ ] ¿Los tests mockean PrismaService correctamente?

---

## 📊 Historial de Checkpoints

### [2025-11-03 20:00] - Setup Inicial Backend
- ✅ Proyecto NestJS creado
- ✅ Docker Compose configurado (backend + postgres + redis)
- ✅ Prisma configurado
- ✅ Estructura de carpetas modular
- 🔜 Próximo: Auth Module

### [FECHA PRÓXIMO CHECKPOINT]
- Último módulo: ___________
- Endpoints: ___________
- Tests: ___________
- Próximo: ___________

---

## 🆘 Recuperación de Contexto Perdido

### Si perdiste el contexto

1. **Lee inmediatamente:**
   - `PROJECT_INSTRUCTIONS.md`
   - `.claude/context.md`
   - `.claude/conventions.md`
   - `.claude/session-state.md`

2. **Confirma que recuerdas:**
   - Framework: **NestJS 10**
   - Database: **PostgreSQL 15 con RLS**
   - Multi-tenant: **Header X-Tenant-ID + RLS**
   - Convenciones: **kebab-case (archivos), PascalCase (clases), camelCase (métodos)**
   - NO usar: **snake_case, any type, console.log**

3. **Pregunta al usuario:**
   ```
   He revisado el contexto del backend. Confirmo:
   - Framework: NestJS 10 + TypeScript 5
   - Database: PostgreSQL 15 con Row-Level Security
   - Multi-tenant: X-Tenant-ID + RLS automático
   - Convenciones: kebab-case, PascalCase, camelCase
   - NO snake_case, NO any type

   ¿En qué módulo/endpoint estábamos trabajando?
   ```

---

## 🎯 Métricas de Memoria

### Efectividad del Sistema

| Métrica | Objetivo | Actual |
|---------|----------|--------|
| Recordar framework | 100% | -% |
| Seguir convenciones | 100% | -% |
| Validar tenant (RLS) | 100% | -% |
| NO usar snake_case | 100% | -% |
| NO usar any | 95% | -% |
| Tests coverage | 80% | -% |

---

<div align="center">

**Este sistema asegura coherencia entre sesiones**

Usa checkpoints regularmente

Backend Multi-Tenant Profesional

</div>
