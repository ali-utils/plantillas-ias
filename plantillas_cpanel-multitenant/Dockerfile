# ============================================
# Dockerfile - bMOI CPanel Multi-Tenant BO
# Multi-stage build para desarrollo y producción
# ============================================

# ============================================
# STAGE 1: Base
# ============================================
FROM node:20-alpine AS base

# Instalar dependencias del sistema necesarias
RUN apk add --no-cache \
    git \
    python3 \
    make \
    g++

# Configurar directorio de trabajo
WORKDIR /app

# Copiar archivos de dependencias
COPY package.json package-lock.json ./

# ============================================
# STAGE 2: Dependencies
# ============================================
FROM base AS dependencies

# Instalar TODAS las dependencias (incluyendo devDependencies)
RUN npm ci --legacy-peer-deps

# ============================================
# STAGE 3: Development
# ============================================
FROM base AS development

# Variables de entorno para desarrollo
ENV NODE_ENV=development
ENV VITE_PORT=3000
ENV CHOKIDAR_USEPOLLING=true

# Copiar dependencias del stage anterior
COPY --from=dependencies /app/node_modules ./node_modules

# Copiar código fuente
COPY . .

# Exponer puerto de desarrollo
EXPOSE 3000

# Comando para desarrollo con hot reload
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]

# ============================================
# STAGE 4: Builder (Build de producción)
# ============================================
FROM base AS builder

# Variables de entorno para build
ENV NODE_ENV=production

# Copiar dependencias
COPY --from=dependencies /app/node_modules ./node_modules

# Copiar código fuente
COPY . .

# Crear archivo .env vacío si no existe (para build)
RUN touch .env

# Build de producción
RUN npm run build

# ============================================
# STAGE 5: Production
# ============================================
FROM node:20-alpine AS production

# Variables de entorno
ENV NODE_ENV=production

# Instalar serve para servir archivos estáticos
RUN npm install -g serve

WORKDIR /app

# Copiar solo los archivos build
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

# Usuario no-root por seguridad
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Cambiar ownership de archivos
RUN chown -R nodejs:nodejs /app

# Cambiar a usuario no-root
USER nodejs

# Exponer puerto
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000', (r) => {if (r.statusCode !== 200) throw new Error(r.statusCode)})"

# Comando para servir archivos estáticos
CMD ["serve", "-s", "dist", "-l", "3000", "-n"]
