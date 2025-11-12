# Docker Quick Start - 2 Minutos

> **Guía ultra rápida para comenzar con Docker**

---

## ⚡ Inicio en 3 Pasos

### 1. Verificar Docker

```bash
docker --version
# Docker version 24.x.x
```

### 2. Iniciar Proyecto

**Windows PowerShell:**
```powershell
.\docker\scripts\dev.ps1
```

**Linux/Mac:**
```bash
chmod +x docker/scripts/dev.sh
./docker/scripts/dev.sh
```

**O manualmente:**
```bash
docker-compose --profile dev up -d
```

### 3. Abrir Navegador

```
http://localhost:3000
```

---

## 🎯 Comandos Esenciales

```bash
# Iniciar
docker-compose --profile dev up -d

# Ver logs
docker-compose logs -f frontend-dev

# Detener
docker-compose --profile dev down

# Limpiar todo
docker-compose down -v && docker system prune -a
```

---

## 🔧 Con Makefile (Linux/Mac)

```bash
make dev          # Iniciar desarrollo
make logs         # Ver logs
make stop         # Detener
make clean        # Limpiar todo
make test         # Ejecutar tests
make shell        # Acceder a shell
```

---

## 📦 Instalar Dependencias

```bash
# Método 1: Editar package.json y reconstruir
docker-compose build frontend-dev

# Método 2: Desde Makefile
make install PKG=axios

# Método 3: Directamente en container
docker-compose exec frontend-dev npm install axios
```

---

## 🆘 Problemas Comunes

### Puerto ocupado
```bash
# Cambiar puerto en docker-compose.yml
ports:
  - "3001:3000"
```

### Hot reload no funciona (Windows)
```bash
# Ya está configurado con CHOKIDAR_USEPOLLING=true
# Si no funciona, reiniciar:
docker-compose restart frontend-dev
```

### Cambios en package.json
```bash
# Reconstruir imagen
docker-compose build --no-cache frontend-dev
docker-compose --profile dev up -d
```

---

## 📚 Documentación Completa

Lee [DOCKER.md](./DOCKER.md) para más detalles.

---

## 🌐 URLs

```
Frontend Dev:     http://localhost:3000
Backend Mock:     http://localhost:5000
PostgreSQL:       localhost:5432
Redis:            localhost:6379
```

---

## ✅ Verificación

```bash
# Ver servicios corriendo
docker-compose ps

# Deberías ver:
# bmoi-frontend-dev   Up   0.0.0.0:3000->3000/tcp
```

---

<div align="center">

**¡Listo en 2 minutos! 🚀**

</div>
