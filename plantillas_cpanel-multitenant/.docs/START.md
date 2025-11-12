# START - Guía de Inicio con IA

> **Pasos para comenzar desarrollo con Claude Code, Copilot o Cursor**
> Versión: 2.0 | Noviembre 2025

---

## 🎯 Objetivo de este Documento

Este documento te guía **paso a paso** para configurar tu entorno de desarrollo y empezar a programar el proyecto **cpanel-multitenant-bo** con la ayuda de asistentes de IA (Claude Code, GitHub Copilot, Cursor, etc.).

---

## 📋 Antes de Empezar

### ✅ Checklist de Prerrequisitos

- [ ] Node.js >= 18.0.0 instalado
- [ ] npm >= 9.0.0 (o pnpm >= 8.0.0)
- [ ] Git configurado
- [ ] Editor: VS Code / Cursor
- [ ] Extensiones:
  - [ ] ESLint
  - [ ] Prettier
  - [ ] GitHub Copilot (opcional)
- [ ] Claude Code CLI instalado (si usas Claude)

---

## 🚀 FASE 1: Instalación del Proyecto

### Paso 1.1: Clonar o Crear Proyecto

```bash
# Opción A: Si el repo existe
git clone https://github.com/your-org/cpanel-multitenant-bo.git
cd cpanel-multitenant-bo

# Opción B: Crear desde cero
mkdir cpanel-multitenant-bo
cd cpanel-multitenant-bo
git init
```

### Paso 1.2: Inicializar Proyecto Vite + React + TypeScript

```bash
# Crear proyecto con Vite
npm create vite@latest . -- --template react-ts

# Instalar dependencias base
npm install
```

### Paso 1.3: Instalar Dependencias del Proyecto

```bash
# UI y Estilos
npm install antd
npm install tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Estado y Data Fetching
npm install zustand
npm install @tanstack/react-query
npm install axios

# Forms y Validación
npm install react-hook-form
npm install zod @hookform/resolvers

# Routing
npm install react-router-dom

# i18n
npm install react-i18next i18next

# DevDependencies
npm install -D @types/node
npm install -D eslint prettier
npm install -D vitest @testing-library/react @testing-library/jest-dom
npm install -D husky lint-staged
```

### Paso 1.4: Configurar Variables de Entorno

```bash
# Copiar template
cp .env.example .env.local
```

Editar `.env.local`:
```env
VITE_API_URL=http://localhost:5000/api/v1
VITE_APP_NAME=bMOI Panel de Control
VITE_TENANT_HEADER=X-Tenant-ID
VITE_DEFAULT_LOCALE=es-ES
```

---

## 📁 FASE 2: Crear Estructura de Carpetas

### Paso 2.1: Estructura Base del Proyecto

```bash
# Crear estructura completa
mkdir -p src/{components/{atoms,molecules,organisms,templates},pages,hooks,context,store,services/api,types,utils,config,locales,styles,router}

# Crear carpetas de testing
mkdir -p tests/{unit,integration,e2e}
mkdir -p tests/__mocks__

# Crear carpetas de scripts
mkdir -p scripts

# Crear docs
mkdir -p docs/{architecture,api,components,guides}
```

### Paso 2.2: Crear Archivos Iniciales

```bash
# Config files (ya los tienes en la raíz)
# Los archivos README.md, PROJECT_INSTRUCTIONS.md, etc. ya existen

# Crear archivos base src
touch src/main.tsx
touch src/App.tsx
touch src/types/index.ts
touch src/config/env.ts
touch src/utils/constants.ts
```

---

## 🤖 FASE 3: Configurar Asistentes de IA

### Para Claude Code

#### Paso 3.1: Crear Carpeta `.claude/`

```bash
mkdir -p .claude/skills
```

#### Paso 3.2: Crear Archivos de Contexto

Ya están creados en el proyecto. Verifica que existan:

```
.claude/
├── context.md                 ← Contexto del proyecto
├── conventions.md             ← Convenciones obligatorias
├── memory-checkpoints.md      ← Sistema anti-olvido
├── session-state.md           ← Estado actual
└── skills/
    ├── multi-tenant-skill.md  ← Patterns multi-tenant
    └── react-conventions.md   ← Patterns React
```

#### Paso 3.3: Iniciar Claude Code

```bash
# Dentro de la carpeta del proyecto
claude code

# En tu primera interacción con Claude:
# "Lee todos los archivos en .claude/ y PROJECT_INSTRUCTIONS.md
#  para entender el contexto del proyecto"
```

### Para GitHub Copilot

#### Paso 3.1: Crear `.copilot/`

```bash
mkdir -p .copilot/templates
```

Archivo `.copilot/instructions.md` (ya existe en el proyecto).

#### Paso 3.2: Configurar en VS Code

1. Instalar extensión GitHub Copilot
2. Login con tu cuenta GitHub
3. Copilot leerá automáticamente `.copilot/instructions.md`

### Para Cursor

#### Paso 3.1: Crear `.cursorrules`

```bash
touch .cursorrules
```

Contenido `.cursorrules`:
```
# Proyecto: bMOI CPanel Multi-Tenant BO
# Stack: React 18 + TypeScript 5 + Vite 5

## Convenciones Obligatorias
- Variables: camelCase
- Componentes: PascalCase
- Constantes: UPPER_SNAKE_CASE
- NO usar snake_case NUNCA
- NO usar `any` type

## Multi-Tenant
- Siempre verificar currentTenant
- Query keys deben incluir tenant.id
- Validar permisos con hasPermission()

Leer: PROJECT_INSTRUCTIONS.md para detalles completos
```

---

## 📝 FASE 4: Primera Sesión de Programación

### Paso 4.1: Prompt Inicial para la IA

```markdown
Hola, soy un desarrollador trabajando en el proyecto bMOI CPanel Multi-Tenant BO.

Por favor:
1. Lee COMPLETO el archivo PROJECT_INSTRUCTIONS.md
2. Lee COMPLETO el archivo CODING_STANDARDS.md
3. Revisa el archivo .claude/context.md (o .copilot/instructions.md)

Cuando termines de leer, confirma que entendiste:
- El nombre del proyecto
- La paleta de colores
- Las convenciones de nomenclatura (camelCase, PascalCase, etc.)
- Las reglas multi-tenant
- El stack tecnológico

Luego estaré listo para comenzar a programar.
```

### Paso 4.2: Verificar que la IA Entendió

Pregunta a la IA:

```markdown
¿Cuáles son las convenciones de nomenclatura obligatorias en este proyecto?
¿Qué debo verificar siempre antes de hacer un query en React Query?
¿Cuál es la paleta de colores primaria?
```

Respuestas esperadas:
- Variables: `camelCase`, Componentes: `PascalCase`, Constantes: `UPPER_SNAKE_CASE`
- Verificar `currentTenant` e incluir `tenant.id` en query keys
- Primary: `#2563EB`, Secondary: `#7C3AED`

### Paso 4.3: Primer Task - Crear Configuración Base

```markdown
Vamos a crear la configuración inicial del proyecto.

Crea los siguientes archivos siguiendo EXACTAMENTE las convenciones:

1. src/config/env.ts - Variables de entorno tipadas
2. src/types/common.types.ts - Tipos comunes (IUser, ITenant, etc.)
3. src/services/api/client.ts - Cliente API con interceptores multi-tenant
4. src/context/TenantContext.tsx - Context para tenant actual

Asegúrate de:
- Usar TypeScript estricto (NO any)
- Seguir nomenclatura camelCase para variables
- Incluir JSDoc en funciones
- Verificar tenant en todos los API calls
```

---

## 🧠 FASE 5: Sistema Anti-Olvido

### Cada 5 Mensajes con la IA

```markdown
🧠 Checkpoint de Memoria:

Actualiza tu memoria sobre:
- ¿Qué estamos construyendo? (bMOI Multi-Tenant Panel)
- ¿Qué convenciones OBLIGATORIAS estamos usando? (camelCase, PascalCase, etc.)
- ¿Última feature completada?
- ¿Próximo paso?

Confirma que recuerdas las reglas multi-tenant.
```

### Al Final de Cada Sesión

```markdown
📝 Fin de Sesión:

Actualiza el archivo .claude/session-state.md con:
- Fecha y hora
- Componentes creados hoy
- Decisiones técnicas tomadas
- Próximos pasos
- Problemas pendientes

Genera un resumen de lo que hicimos hoy.
```

---

## 🎨 FASE 6: Desarrollo Modular por Fases

### Fase 1: Autenticación y Layout (Semana 1-2)

```markdown
Vamos a implementar la Fase 1: Autenticación Multi-Tenant

Componentes a crear:
1. LoginPage con formulario (react-hook-form + zod)
2. TenantSelector (dropdown de tenants)
3. MainLayout (header + sidebar + content)
4. useAuth hook
5. useTenant hook
6. API service para auth

Orden de desarrollo:
1. Primero: Tipos y interfaces
2. Segundo: Services (API)
3. Tercero: Hooks
4. Cuarto: Componentes UI
5. Quinto: Tests

Comenzamos?
```

### Fase 2: Dashboard Multi-Tenant (Semana 3-4)

```markdown
Implementar Dashboard con métricas por tenant:

1. DashboardPage
2. MetricCard componente
3. TenantMetricsWidget
4. useDashboardData hook
5. Dashboard service

Recuerda:
- Query keys con tenant.id
- Validar permisos
- Verificar límites del plan
```

### Fase 3: Gestión de Usuarios (Semana 5-6)

```markdown
CRUD de usuarios multi-tenant:

1. UsersPage
2. UsersList componente
3. UserCreateModal
4. UserEditModal
5. useUsersData hook
6. Users service

Validaciones:
- Límites por plan (free: 10 users, plan1: 50, etc.)
- Permisos por rol
```

### Fase 4: Productos y Pedidos (Semana 7-8)

```markdown
Catálogo de productos y pedidos por tenant:

1. ProductsPage
2. OrdersPage
3. ProductsList
4. OrdersList
5. useProducts hook
6. useOrders hook

Con filtros y paginación.
```

---

## 🔄 FASE 7: Flujo de Trabajo Diario

### Inicio del Día

```markdown
Buenos días! Vamos a continuar con el proyecto bMOI Multi-Tenant.

Por favor:
1. Lee .claude/session-state.md para saber dónde quedamos
2. Revisa PROJECT_INSTRUCTIONS.md si necesitas recordar convenciones
3. Confirma que recuerdas:
   - Nomenclatura: camelCase, PascalCase
   - Multi-tenant: verificar currentTenant
   - NO usar any type

¿Listo? ¿Qué vamos a desarrollar hoy?
```

### Durante el Desarrollo

```markdown
✅ Antes de crear un componente:
- Verificar que el nombre sigue PascalCase
- Incluir types en archivo separado
- Agregar JSDoc
- Verificar tenant si usa API

✅ Antes de crear un hook:
- Nombre: use + camelCase
- Incluir tipos de retorno
- Validar tenant si hace queries

✅ Antes de commit:
npm run validate
```

### Fin del Día

```markdown
📊 Resumen del día:

Genera un resumen con:
- ✅ Componentes creados
- ✅ Features completadas
- ⚠️ Problemas encontrados
- 📝 Decisiones técnicas tomadas
- 🔜 Próximos pasos

Actualiza .claude/session-state.md
```

---

## 🛠️ FASE 8: Comandos Útiles

### Generadores Automáticos

```bash
# Generar componente
npm run generate:component UserCard molecule

# Generar página
npm run generate:page Users

# Generar hook
npm run generate:hook useUsers

# Generar service
npm run generate:service users
```

### Validación

```bash
# Validación completa
npm run validate

# Por separado
npm run lint
npm run type-check
npm run test
npm run format
```

### IA Helpers

```bash
# Checkpoint de memoria (cada 5 mensajes)
npm run ai:checkpoint

# Recordar contexto del proyecto
npm run ai:remind

# Validar que el código sigue convenciones
npm run ai:validate
```

---

## 📚 FASE 9: Documentos de Referencia

### Consultar Durante Desarrollo

| Duda | Documento |
|------|-----------|
| ¿Cómo nombrar variables? | [CODING_STANDARDS.md](./CODING_STANDARDS.md) |
| ¿Cómo funciona multi-tenant? | [ARQUITECTURA.md](./ARQUITECTURA.md) |
| ¿Qué paleta de colores usar? | [PROJECT_INSTRUCTIONS.md](./PROJECT_INSTRUCTIONS.md) |
| ¿Cómo crear componentes? | [CODING_STANDARDS.md](./CODING_STANDARDS.md) |
| ¿Instalación rápida? | [QUICKSTART.md](./QUICKSTART.md) |

---

## ✅ CHECKLIST FINAL ANTES DE PROGRAMAR

- [ ] Proyecto instalado (`npm install`)
- [ ] `.env.local` configurado
- [ ] Tests pasan (`npm run test`)
- [ ] Lint sin errores (`npm run lint`)
- [ ] IA configurada (Claude/Copilot/Cursor)
- [ ] IA leyó `PROJECT_INSTRUCTIONS.md`
- [ ] IA leyó `CODING_STANDARDS.md`
- [ ] IA leyó `.claude/context.md` o `.copilot/instructions.md`
- [ ] IA confirmó que entendió las convenciones
- [ ] Git inicializado
- [ ] Estructura de carpetas creada

---

## 🎯 PRIMER PROMPT COMPLETO

Copia este prompt para tu primera sesión:

```markdown
Hola! Soy desarrollador del proyecto bMOI CPanel Multi-Tenant BO.

PASO 1: Lee estos documentos COMPLETOS (son críticos):
- PROJECT_INSTRUCTIONS.md
- CODING_STANDARDS.md
- ARQUITECTURA.md
- .claude/context.md (o .copilot/instructions.md)

PASO 2: Cuando termines de leer, confirma que entendiste:
1. Nombre del proyecto y logo
2. Stack tecnológico (React + TypeScript + Vite)
3. Paleta de colores (Primary #2563EB, Secondary #7C3AED)
4. Convenciones OBLIGATORIAS:
   - Variables: camelCase
   - Componentes: PascalCase
   - Constantes: UPPER_SNAKE_CASE
   - NO snake_case NUNCA
   - NO any type
5. Reglas Multi-Tenant:
   - Verificar currentTenant siempre
   - Query keys incluyen tenant.id
   - Validar permisos con hasPermission()

PASO 3: Cuando confirmes que entendiste todo lo anterior,
comenzaremos con la Fase 1: Autenticación y Layout.

¿Listo para empezar?
```

---

## 🆘 Troubleshooting

### "La IA no recuerda las convenciones"

```markdown
🧠 Recordatorio de Convenciones:

Este proyecto usa ESTRICTAMENTE:
- camelCase: variables y funciones
- PascalCase: componentes React
- UPPER_SNAKE_CASE: constantes
- NO snake_case NUNCA en JS/TS
- NO any type

Por favor confirma que lo recordarás.
```

### "La IA crea código sin tenant context"

```markdown
⚠️ ERROR: Falta validación de tenant

TODO código que use API debe:
1. const { currentTenant } = useTenant()
2. if (!currentTenant) return null
3. queryKey: ['resource', currentTenant.id]

Corrige el código anterior.
```

### "La IA usa snake_case"

```markdown
❌ STOP: Detecté snake_case

Este proyecto PROHÍBE snake_case en JavaScript/TypeScript.

Ejemplos CORRECTOS:
- userName (camelCase)
- UserProfile (PascalCase)
- API_BASE_URL (UPPER_SNAKE_CASE)

Por favor corrige y NO vuelvas a usar snake_case.
```

---

## 🎉 ¡Listo para Comenzar!

Ahora estás preparado para desarrollar con la ayuda de IA.

**Recuerda**:
1. Las IAs son tus asistentes, pero TÚ decides
2. Siempre verifica el código generado
3. Ejecuta `npm run validate` antes de commits
4. Usa checkpoints de memoria cada 5 mensajes
5. Actualiza `.claude/session-state.md` al final del día

---

<div align="center">

**¡Feliz Coding con IA! 🚀**

¿Dudas? Consulta [PROJECT_INSTRUCTIONS.md](./PROJECT_INSTRUCTIONS.md)

</div>
