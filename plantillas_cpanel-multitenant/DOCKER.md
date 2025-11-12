# DOCKER - Guía Completa

> **Configuración Docker para bMOI CPanel Multi-Tenant BO**
> Versión: 2.0 | Noviembre 2025

---

## 📋 Tabla de Contenidos

1. [¿Por Qué Docker?](#-por-qué-docker)
2. [Prerrequisitos](#-prerrequisitos)
3. [Instalación Rápida](#-instalación-rápida)
4. [Modo Desarrollo](#-modo-desarrollo)
5. [Modo Producción](#-modo-producción)
6. [Comandos Útiles](#-comandos-útiles)
7. [Troubleshooting](#-troubleshooting)

---

## 🎯 ¿Por Qué Docker?

### Problemas que Resuelve

```
❌ Sin Docker:
- "En mi máquina funciona"
- Versiones diferentes de Node entre desarrolladores
- Dependencias del sistema incompatibles
- Configuración manual compleja
- Problemas con Windows vs Mac vs Linux

✅ Con Docker:
- Entorno idéntico para todos
- Mismo Node 20, mismas dependencias
- Setup en 2 comandos
- Funciona en cualquier OS
- Aislamiento completo
```

### Beneficios para el Equipo

1. **Consistencia**: Todos usan exactamente las mismas versiones
2. **Onboarding Rápido**: Nuevo dev → 5 minutos para comenzar
3. **Aislamiento**: No contamina tu sistema con dependencias
4. **CI/CD Ready**: Mismo container en dev, staging y prod
5. **Fácil Limpieza**: `docker-compose down` y todo limpio

---

## 🔧 Prerrequisitos

### 1. Instalar Docker Desktop

#### Windows

```powershell
# Descargar Docker Desktop desde:
# https://www.docker.com/products/docker-desktop/

# O con winget:
winget install Docker.DockerDesktop

# Verificar instalación:
docker --version
docker-compose --version
```

#### Mac

```bash
# Descargar desde:
# https://www.docker.com/products/docker-desktop/

# O con Homebrew:
brew install --cask docker

# Verificar:
docker --version
docker-compose --version
```

#### Linux (Ubuntu/Debian)

```bash
# Instalar Docker Engine
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Agregar usuario al grupo docker
sudo usermod -aG docker $USER

# Instalar Docker Compose
sudo apt-get install docker-compose-plugin

# Verificar:
docker --version
docker compose version
```

### 2. Configurar Docker Desktop

```yaml
Configuración Recomendada:
  RAM: 4GB mínimo (8GB recomendado)
  CPUs: 2 mínimo (4 recomendado)
  Disk: 20GB mínimo

Settings → Resources:
  - Memory: 4GB
  - CPUs: 2
  - Swap: 1GB
  - Disk image size: 64GB
```

### 3. Verificar Instalación

```bash
# Verificar que Docker está corriendo
docker info

# Debería mostrar:
# Server Version: 24.x.x
# Operating System: ...
# Total Memory: ...
```

---

## ⚡ Instalación Rápida

### Paso 1: Clonar Proyecto

```bash
git clone https://github.com/your-org/cpanel-multitenant-bo.git
cd cpanel-multitenant-bo
```

### Paso 2: Configurar Variables de Entorno

```bash
# Copiar template
cp .env.example .env.local

# Editar .env.local con tus valores
# (Opcional en desarrollo, usa valores por defecto)
```

### Paso 3: Iniciar con Docker

#### Opción A: Windows PowerShell

```powershell
# Ejecutar script de desarrollo
.\docker\scripts\dev.ps1
```

#### Opción B: Linux/Mac

```bash
# Dar permisos de ejecución
chmod +x docker/scripts/dev.sh

# Ejecutar script
./docker/scripts/dev.sh
```

#### Opción C: Manual

```bash
# Construir y levantar servicios de desarrollo
docker-compose --profile dev up -d

# Ver logs
docker-compose logs -f frontend-dev
```

### Paso 4: Abrir en el Navegador

```
🌐 Frontend: http://localhost:3000
🔧 API Mock: http://localhost:5000
```

---

## 🛠️ Modo Desarrollo

### Características del Modo Dev

```yaml
Hot Reload: ✅ Cambios en código se reflejan automáticamente
Source Maps: ✅ Debugging fácil
Volumes: ✅ Código sincronizado entre host y container
Dev Tools: ✅ React DevTools funcionan
Performance: ⚡ Optimizado para desarrollo
```

### Iniciar Desarrollo

```bash
# Opción 1: Con script (Recomendado)
# Windows:
.\docker\scripts\dev.ps1

# Linux/Mac:
./docker/scripts/dev.sh

# Opción 2: Manual
docker-compose --profile dev up -d

# Ver logs en tiempo real
docker-compose logs -f frontend-dev
```

### Estructura de Servicios en Dev

```
┌─────────────────────────────────────┐
│  Frontend (React + Vite)            │
│  Puerto: 3000                       │
│  Hot Reload: ✅                     │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│  Backend Mock (Mockoon)             │
│  Puerto: 5000                       │
│  API REST Mock                      │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│  PostgreSQL (Opcional)              │
│  Puerto: 5432                       │
│  Base de datos de desarrollo       │
└─────────────────────────────────────┘
           ↓
┌─────────────────────────────────────┐
│  Redis (Opcional)                   │
│  Puerto: 6379                       │
│  Cache                              │
└─────────────────────────────────────┘
```

### Trabajar con Código

```bash
# El código en src/ está montado como volumen
# Edita normalmente en tu IDE y los cambios se reflejan automáticamente

# Estructura montada:
./src           → /app/src          (Hot reload)
./public        → /app/public       (Hot reload)
./index.html    → /app/index.html   (Hot reload)
./vite.config.ts → /app/vite.config.ts

# NO montado (usa del container):
./node_modules  → Usa las del container
```

### Instalar Nuevas Dependencias

```bash
# Método 1: Reconstruir imagen (Recomendado)
# Editar package.json manualmente
docker-compose build frontend-dev
docker-compose --profile dev up -d

# Método 2: Ejecutar npm dentro del container
docker-compose exec frontend-dev npm install nombre-paquete
```

### Ver Logs

```bash
# Logs de todos los servicios
docker-compose logs -f

# Logs solo del frontend
docker-compose logs -f frontend-dev

# Últimas 100 líneas
docker-compose logs --tail=100 frontend-dev

# Logs de un servicio específico
docker-compose logs -f postgres
```

### Detener Servicios

```bash
# Detener todos los servicios (mantiene datos)
docker-compose --profile dev down

# Detener y eliminar volúmenes (datos de DB)
docker-compose --profile dev down -v

# Detener solo frontend
docker-compose stop frontend-dev
```

---

## 🚀 Modo Producción

### Construir para Producción

```bash
# Build de imagen de producción
docker-compose build frontend-prod

# Iniciar servicios de producción
docker-compose --profile prod up -d

# Con Nginx reverse proxy
docker-compose --profile prod up -d frontend-prod nginx
```

### Características del Modo Prod

```yaml
Optimización: ✅ Build minificado
Multi-stage: ✅ Imagen ligera (~50MB)
Security: ✅ Usuario no-root
Health Check: ✅ Verificación automática
Restart: ✅ Reinicio automático
Nginx: ✅ Reverse proxy con SSL
```

### Diferencias Dev vs Prod

| Característica | Desarrollo | Producción |
|----------------|------------|------------|
| **Imagen** | node:20-alpine | node:20-alpine (multi-stage) |
| **Tamaño** | ~1GB | ~50MB |
| **Hot Reload** | ✅ Sí | ❌ No |
| **Source Maps** | ✅ Sí | ❌ No |
| **Minificación** | ❌ No | ✅ Sí |
| **Servidor** | Vite Dev | serve estático |
| **Optimización** | ❌ No | ✅ Sí |
| **Volúmenes** | Código montado | Solo dist/ |

### Deploy a Producción

```bash
# 1. Construir imagen
docker-compose build frontend-prod

# 2. Etiquetar para registry
docker tag bmoi-frontend-prod:latest registry.example.com/bmoi-frontend:v1.0.0

# 3. Subir a registry
docker push registry.example.com/bmoi-frontend:v1.0.0

# 4. En servidor de producción
docker pull registry.example.com/bmoi-frontend:v1.0.0
docker-compose --profile prod up -d
```

---

## 🎨 Comandos Útiles

### Gestión de Containers

```bash
# Ver containers corriendo
docker-compose ps

# Ver todos los containers (incluso detenidos)
docker ps -a

# Reiniciar un servicio
docker-compose restart frontend-dev

# Ejecutar comando en container
docker-compose exec frontend-dev sh

# Ejemplo: Acceder a shell del container
docker-compose exec frontend-dev sh
# Dentro del container:
$ ls -la
$ npm run test
$ exit
```

### Limpieza

```bash
# Detener todos los servicios
docker-compose --profile dev down

# Detener y eliminar volúmenes
docker-compose --profile dev down -v

# Limpiar todo (imágenes, containers, cache)
docker system prune -a --volumes

# Limpiar solo imágenes no usadas
docker image prune -a

# Eliminar volúmenes no usados
docker volume prune
```

### Debugging

```bash
# Ver logs con timestamps
docker-compose logs -f --timestamps frontend-dev

# Inspeccionar container
docker inspect bmoi-frontend-dev

# Ver uso de recursos
docker stats

# Ver procesos dentro del container
docker-compose top frontend-dev

# Ejecutar comando específico
docker-compose exec frontend-dev npm run lint
```

### Gestión de Volúmenes

```bash
# Listar volúmenes
docker volume ls

# Inspeccionar volumen
docker volume inspect bmoi-postgres-data

# Backup de volumen de base de datos
docker run --rm \
  -v bmoi-postgres-data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres-backup.tar.gz -C /data .

# Restaurar desde backup
docker run --rm \
  -v bmoi-postgres-data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/postgres-backup.tar.gz -C /data
```

### Variables de Entorno

```bash
# Ver variables de entorno de un container
docker-compose exec frontend-dev env

# Ejecutar con variables de entorno custom
docker-compose run -e VITE_API_URL=http://custom-api frontend-dev npm run build
```

---

## 🔍 Troubleshooting

### Problema: Puerto 3000 ocupado

```bash
# Ver qué proceso usa el puerto
# Windows:
netstat -ano | findstr :3000

# Linux/Mac:
lsof -i :3000

# Solución 1: Cambiar puerto en docker-compose.yml
ports:
  - "3001:3000"  # Usar 3001 en host

# Solución 2: Matar proceso
# Windows:
taskkill /PID <PID> /F

# Linux/Mac:
kill -9 <PID>
```

### Problema: Hot Reload no funciona

```bash
# En Windows, asegurar que CHOKIDAR_USEPOLLING esté habilitado
# En docker-compose.yml:
environment:
  - CHOKIDAR_USEPOLLING=true

# Reiniciar container
docker-compose restart frontend-dev
```

### Problema: Cambios en package.json no se reflejan

```bash
# Reconstruir imagen
docker-compose build --no-cache frontend-dev
docker-compose --profile dev up -d
```

### Problema: Error "no space left on device"

```bash
# Limpiar imágenes y cache
docker system prune -a --volumes

# Aumentar espacio en Docker Desktop
# Settings → Resources → Disk image size → 64GB
```

### Problema: Container crashea al iniciar

```bash
# Ver logs detallados
docker-compose logs frontend-dev

# Ver últimos logs
docker logs bmoi-frontend-dev --tail 50

# Ejecutar container en modo interactivo
docker-compose run --rm frontend-dev sh
```

### Problema: Permisos en Linux

```bash
# Agregar usuario al grupo docker
sudo usermod -aG docker $USER

# Logout y login de nuevo

# Verificar
docker run hello-world
```

### Problema: Dependencias no se instalan

```bash
# Limpiar cache de npm en container
docker-compose exec frontend-dev rm -rf node_modules package-lock.json
docker-compose exec frontend-dev npm cache clean --force

# Reconstruir sin cache
docker-compose build --no-cache frontend-dev
```

---

## 📊 Monitoreo y Performance

### Ver Uso de Recursos

```bash
# Recursos en tiempo real
docker stats

# Ejemplo de salida:
# CONTAINER          CPU %     MEM USAGE / LIMIT     MEM %
# bmoi-frontend-dev  1.5%      256MiB / 4GiB        6.25%
# bmoi-postgres      0.5%      64MiB / 4GiB         1.56%
```

### Health Checks

```bash
# Ver health status
docker-compose ps

# Inspeccionar health check
docker inspect --format='{{json .State.Health}}' bmoi-frontend-prod
```

---

## 🔐 Seguridad

### Buenas Prácticas

```dockerfile
# ✅ Usar usuario no-root en producción
USER nodejs

# ✅ Multi-stage builds (imagen ligera)
FROM node:20-alpine AS builder
FROM node:20-alpine AS production

# ✅ Health checks
HEALTHCHECK --interval=30s CMD node -e "..."

# ✅ .dockerignore completo
node_modules/
.env
```

### Escaneo de Vulnerabilidades

```bash
# Escanear imagen (Docker Desktop incluye Snyk)
docker scan bmoi-frontend-prod

# O con trivy
docker run aquasec/trivy image bmoi-frontend-prod
```

---

## 📚 Archivos de Configuración

### Estructura de Archivos Docker

```
cpanel-multitenant-bo/
├── Dockerfile                    # Multi-stage build
├── docker-compose.yml            # Orquestación de servicios
├── .dockerignore                 # Archivos a excluir
│
├── docker/
│   ├── scripts/
│   │   ├── dev.sh               # Script Linux/Mac
│   │   └── dev.ps1              # Script Windows
│   │
│   ├── nginx/
│   │   ├── nginx.conf           # Config Nginx
│   │   └── ssl/                 # Certificados SSL
│   │
│   └── postgres/
│       └── init.sql             # Script inicialización DB
│
└── .env.example                  # Variables de entorno template
```

---

## 🎯 Resumen de Comandos Rápidos

```bash
# === DESARROLLO ===
# Iniciar todo
docker-compose --profile dev up -d

# Ver logs
docker-compose logs -f frontend-dev

# Detener
docker-compose --profile dev down

# === PRODUCCIÓN ===
# Build
docker-compose build frontend-prod

# Iniciar
docker-compose --profile prod up -d

# === UTILIDADES ===
# Limpiar todo
docker system prune -a --volumes

# Reconstruir sin cache
docker-compose build --no-cache

# Ejecutar comando en container
docker-compose exec frontend-dev npm run test
```

---

## 🤝 Para el Equipo

### Onboarding Nuevo Desarrollador

```markdown
1. Instalar Docker Desktop
2. Clonar repositorio
3. Ejecutar: `docker-compose --profile dev up -d`
4. Abrir: http://localhost:3000
5. ¡Listo para programar!

Tiempo total: ~5 minutos
```

### Sincronización de Versiones

```yaml
Garantizado por Docker:
  - Node: 20.x.x (Alpine)
  - npm: 10.x.x
  - Todas las dependencias: package-lock.json

Sin Docker:
  - Dev 1: Node 18
  - Dev 2: Node 20
  - CI/CD: Node 19
  ❌ "En mi máquina funciona"
```

---

<div align="center">

**Docker asegura que todos trabajen con el mismo entorno**

Última actualización: Noviembre 2025 | Versión 2.0

</div>
