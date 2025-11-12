# QUICKSTART - bMOI CPanel Multi-Tenant BO

> **Guía de inicio rápido en 5 minutos**

---

## ⚡ Instalación Express

### 1. Prerrequisitos

```bash
# Verificar versiones
node --version   # >= 18.0.0
npm --version    # >= 9.0.0
```

### 2. Clonar e Instalar

```bash
# Clonar repositorio
git clone https://github.com/your-org/cpanel-multitenant-bo.git
cd cpanel-multitenant-bo

# Instalar dependencias
npm install

# Copiar variables de entorno
cp .env.example .env.local
```

### 3. Configurar `.env.local`

```env
VITE_API_URL=http://localhost:5000/api/v1
VITE_APP_NAME=bMOI Panel
VITE_TENANT_HEADER=X-Tenant-ID
```

### 4. Iniciar Desarrollo

```bash
npm run dev
```

Abrir: http://localhost:3000

---

## 🎯 Primeros Pasos

### Crear Primer Componente

```bash
npm run generate:component UserCard molecule
```

Esto crea:
```
src/components/molecules/UserCard/
├── UserCard.tsx
├── UserCard.types.ts
├── UserCard.module.css
├── UserCard.test.tsx
└── index.ts
```

### Ejecutar Tests

```bash
npm run test
npm run test:coverage
```

### Validar Código

```bash
npm run validate
```

Esto ejecuta:
- ✓ ESLint
- ✓ Prettier
- ✓ TypeScript type-check
- ✓ Tests

---

## 📚 Documentos Importantes

Antes de programar, lee:

1. **[START.md](./START.md)** - Desarrollo con IA
2. **[PROJECT_INSTRUCTIONS.md](./PROJECT_INSTRUCTIONS.md)** - Guía completa
3. **[CODING_STANDARDS.md](./CODING_STANDARDS.md)** - Estándares

---

## 🚀 Comandos Útiles

```bash
# Desarrollo
npm run dev              # Servidor desarrollo
npm run build            # Build producción
npm run preview          # Preview build

# Testing
npm run test             # Tests
npm run test:watch       # Tests en watch mode
npm run test:coverage    # Coverage report

# Calidad
npm run lint             # ESLint
npm run lint:fix         # Fix automático
npm run format           # Prettier
npm run type-check       # TypeScript

# Generadores
npm run generate:component [nombre] [tipo]
npm run generate:page [nombre]
npm run generate:hook [nombre]

# IA Helpers
npm run ai:checkpoint    # Guardar estado
npm run ai:remind        # Recordar contexto
```

---

## 🎨 Estructura Básica

```
src/
├── components/      # Atomic Design
│   ├── atoms/
│   ├── molecules/
│   ├── organisms/
│   └── templates/
├── pages/          # Rutas
├── hooks/          # Custom hooks
├── store/          # Zustand stores
├── services/       # API services
└── types/          # TypeScript types
```

---

## ✅ Checklist Primera Sesión

- [ ] Proyecto instalado y corriendo
- [ ] `.env.local` configurado
- [ ] Tests pasan (`npm run test`)
- [ ] Lint sin errores (`npm run lint`)
- [ ] Leído START.md
- [ ] Leído PROJECT_INSTRUCTIONS.md
- [ ] Primer componente generado
- [ ] Git configurado

---

## 🆘 Problemas Comunes

### Puerto 3000 ocupado

```bash
# Cambiar puerto en package.json
"dev": "vite --port 3001"
```

### Errores de TypeScript

```bash
npm run type-check
```

### Tests fallan

```bash
npm run test -- --watch
```

---

<div align="center">

**¡Listo! Ahora lee [START.md](./START.md) para comenzar a programar con IA**

</div>
