# QUICKSTART - Backend en 5 Minutos

> **Guía rápida para tener el backend funcionando en 5 minutos**

---

## ⚡ Inicio Rápido

### Prerrequisitos

- ✅ **Docker** (20.10+)
- ✅ **Docker Compose** (2.0+)
- ✅ **Git**

### Paso 1: Clonar el Repositorio (30 segundos)

```bash
git clone https://github.com/your-org/bmoi-backend-api.git
cd bmoi-backend-api
```

### Paso 2: Configurar Variables de Entorno (1 minuto)

```bash
# Copiar archivo de ejemplo
cp .env.example .env

# Editar .env con tus valores
# (Para desarrollo local, los valores por defecto funcionan)
```

**Variables mínimas requeridas:**

```env
# Database
DATABASE_URL="postgresql://postgres:postgres@postgres:5432/bmoi?schema=public"

# JWT (generar claves con: ssh-keygen -t rsa -b 4096)
JWT_SECRET="your-secret-key-change-in-production"
JWT_PRIVATE_KEY="path/to/private.key"
JWT_PUBLIC_KEY="path/to/public.key"

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# MinIO
MINIO_ENDPOINT=minio
MINIO_PORT=9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin

# App
NODE_ENV=development
PORT=5000
```

### Paso 3: Iniciar con Docker (2 minutos)

```bash
# Iniciar todos los servicios
docker compose up -d

# Ver logs
docker compose logs -f backend
```

**Servicios iniciados:**
- ✅ Backend (NestJS) → http://localhost:5000
- ✅ PostgreSQL 15 → localhost:5432
- ✅ Redis 7 → localhost:6379
- ✅ MinIO → http://localhost:9000

### Paso 4: Ejecutar Migraciones (1 minuto)

```bash
# Ejecutar migraciones de Prisma
docker compose exec backend npx prisma migrate dev

# Seed data inicial
docker compose exec backend npm run seed
```

### Paso 5: Verificar que Funciona (30 segundos)

```bash
# Health check
curl http://localhost:5000/health

# Swagger docs
open http://localhost:5000/api/docs
```

---

## ✅ Verificación Rápida

### API está funcionando

```bash
curl http://localhost:5000/health
```

**Respuesta esperada:**
```json
{
  "status": "ok",
  "database": "connected",
  "redis": "connected"
}
```

### Swagger está disponible

Abre en tu navegador: http://localhost:5000/api/docs

### Login de prueba

```bash
curl -X POST http://localhost:5000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@test.com",
    "password": "Admin123!"
  }'
```

**Respuesta esperada:**
```json
{
  "access_token": "eyJhbGc...",
  "refresh_token": "eyJhbGc...",
  "user": {
    "id": "uuid",
    "email": "admin@test.com",
    "name": "Admin User"
  },
  "tenants": [
    {
      "id": "tenant-uuid",
      "name": "Demo Tenant",
      "plan": "plan1"
    }
  ]
}
```

---

## 🚀 Siguiente Paso: Desarrollo

### Ver logs en tiempo real

```bash
docker compose logs -f backend
```

### Ejecutar tests

```bash
# Unit tests
docker compose exec backend npm run test

# E2E tests
docker compose exec backend npm run test:e2e

# Coverage
docker compose exec backend npm run test:cov
```

### Abrir Prisma Studio

```bash
docker compose exec backend npx prisma studio
```

Abre en tu navegador: http://localhost:5555

### Hot-reload está activo

El backend ya está en modo development con hot-reload. Edita cualquier archivo `.ts` y se recargará automáticamente.

---

## 🛠️ Comandos Útiles

### Instalar nueva dependencia

```bash
docker compose exec backend npm install [package]
```

### Crear nueva migración

```bash
docker compose exec backend npx prisma migrate dev --name [nombre]
```

### Generar Prisma Client

```bash
docker compose exec backend npx prisma generate
```

### Crear nuevo módulo

```bash
docker compose exec backend nest g module modules/products
docker compose exec backend nest g controller modules/products
docker compose exec backend nest g service modules/products
```

### Ver base de datos

```bash
# Conectar a PostgreSQL
docker compose exec postgres psql -U postgres -d bmoi

# Listar tablas
\dt

# Ver estructura de tabla
\d users

# Salir
\q
```

---

## 🐛 Troubleshooting

### Puerto 5000 ya en uso

Edita `docker-compose.yml`:

```yaml
services:
  backend:
    ports:
      - "3001:5000"  # Cambiar puerto local
```

### Backend no inicia

```bash
# Ver logs detallados
docker compose logs backend

# Rebuild image
docker compose build backend

# Restart
docker compose restart backend
```

### Database connection error

```bash
# Verificar que PostgreSQL está corriendo
docker compose ps postgres

# Ver logs de PostgreSQL
docker compose logs postgres

# Restart PostgreSQL
docker compose restart postgres
```

### Migrations error

```bash
# Reset database (⚠️ borra todos los datos)
docker compose exec backend npx prisma migrate reset

# Re-run migrations
docker compose exec backend npx prisma migrate dev
```

---

## 📚 Próximos Pasos

1. **Leer documentación completa:**
   - `PROJECT_INSTRUCTIONS.md` - Guía completa
   - `CODING_STANDARDS.md` - Estándares de código
   - `ARQUITECTURA.md` - Arquitectura detallada

2. **Revisar contexto para IAs:**
   - `.claude/context.md` - Contexto completo
   - `.claude/conventions.md` - Convenciones obligatorias

3. **Explorar código base:**
   - `src/modules/auth/` - Autenticación
   - `src/modules/tenants/` - Gestión de tenants
   - `src/modules/users/` - Gestión de usuarios

4. **Empezar a desarrollar:**
   - Lee `START.md` para guía paso a paso
   - Sigue las convenciones en `CODING_STANDARDS.md`

---

## 🎯 Endpoints Principales

| Endpoint | Método | Descripción | Auth |
|----------|--------|-------------|------|
| `/health` | GET | Health check | No |
| `/api/docs` | GET | Swagger UI | No |
| `/auth/login` | POST | Login | No |
| `/auth/refresh` | POST | Refresh token | No |
| `/tenants` | GET | List tenants | Sí |
| `/users` | GET | List users | Sí |
| `/users` | POST | Create user | Sí |

---

## 📞 Ayuda

- **Documentación:** `.docs/` folder
- **Issues:** GitHub Issues
- **Slack:** #bmoi-backend

---

<div align="center">

**¡Backend listo en 5 minutos!**

Ahora puedes empezar a desarrollar

[Documentación](.docs/) • [API Docs](http://localhost:5000/api/docs) • [Arquitectura](ARQUITECTURA.md)

</div>
