# Guía: Cómo Solicitar Plantillas Completas para un Nuevo Proyecto

**Propósito:** Este documento explica cómo solicitar a Claude Code la generación completa de plantillas para un nuevo proyecto multi-plataforma (similar a bMOI) en una sola instrucción.

**Fecha:** 2025-01-04
**Versión:** 1.0.0

---

## 📋 Tabla de Contenidos

1. [¿Qué son las Plantillas?](#qué-son-las-plantillas)
2. [Pre-requisitos](#pre-requisitos)
3. [Estructura de las Plantillas](#estructura-de-las-plantillas)
4. [Template del Prompt Completo](#template-del-prompt-completo)
5. [Ejemplo Real de Prompt](#ejemplo-real-de-prompt)
6. [Qué Esperar como Resultado](#qué-esperar-como-resultado)
7. [Checklist de Información Necesaria](#checklist-de-información-necesaria)
8. [FAQ](#faq)

---

## 🎯 ¿Qué son las Plantillas?

Las **plantillas** son conjuntos completos de documentación, configuración y estructura de código que sirven como **punto de partida** para desarrollar cada interfaz de un proyecto multi-plataforma.

### Ejemplo: Proyecto bMOI

```
bo-backend-api/
├── plantillas_backend/           # API NestJS + PostgreSQL
├── plantillas_cpanel-multitenant/ # Panel Web React (Tenants)
├── plantillas_admin/             # Panel Web React (Super Admin)
├── plantillas_landing/           # Landing Page Astro
└── plantillas_mobile/            # App Móvil Flutter
```

Cada plantilla contiene:
- ✅ Documentación para IAs (Claude, Copilot, Cursor)
- ✅ Arquitectura y decisiones técnicas justificadas
- ✅ Estándares de código específicos del stack
- ✅ Configuración Docker completa
- ✅ Guías de setup paso a paso
- ✅ Ejemplos de código
- ✅ Integración con otras interfaces del ecosistema

---

## 📦 Pre-requisitos

### Antes de solicitar las plantillas, debes tener:

#### 1. Diagrama de Arquitectura (OBLIGATORIO)
**Formato:** Imagen (JPG, PNG) o Mermaid/PlantUML

**Debe mostrar:**
- Todas las interfaces del proyecto (frontends, backend, móvil)
- Capa de orquestación (API Gateway, BFF si aplica)
- Servicios de dominio (microservicios si aplica)
- Capa de datos (bases de datos, cache, storage)
- Flujo de comunicación entre componentes

**Ejemplo:** `Arquitectura_MiProyecto.jpg`

#### 2. Documento Técnico Funcional (OBLIGATORIO)
**Formato:** DOCX, PDF, Markdown

**Debe contener:**
- Concepto de negocio
- Casos de uso principales
- Requerimientos funcionales
- Requerimientos no funcionales
- Stack tecnológico (si ya está definido)
- Restricciones y consideraciones

**Ejemplo:** `Documento_Tecnico_Funcional_MiProyecto.docx`

#### 3. Información del Proyecto (RECOMENDADO)
- Nombre del proyecto
- Propósito/objetivo
- Usuarios objetivo
- Características especiales (multi-tenant, tiempo real, etc.)

---

## 🏗️ Estructura de las Plantillas

Cada plantilla generada tendrá esta estructura:

```
plantillas_[nombre]/
│
├── .claude/                         # Configuración Claude Code
│   ├── context.md                   # Contexto completo del proyecto
│   ├── conventions.md               # Convenciones obligatorias
│   ├── memory-checkpoints.md        # Sistema anti-olvido
│   └── session-state.md             # Estado del desarrollo
│
├── .copilot/                        # Configuración GitHub Copilot
│   └── instructions.md              # Instrucciones para Copilot
│
├── .docs/                           # Documentación técnica
│   ├── README.md                    # Overview del proyecto
│   ├── SETUP.md                     # Setup paso a paso desde cero
│   ├── ARQUITECTURA.md              # Decisiones arquitectónicas
│   ├── CODING_STANDARDS.md          # Estándares de código
│   ├── RESPONSIVE_DESIGN.md         # Guía responsive (si es web)
│   ├── EJEMPLOS_CODIGO.md           # Código listo para copiar
│   └── CONFIGURACIONES.md           # Archivos de configuración
│
├── .cursorrules                     # Reglas para Cursor AI
├── .dockerignore                    # Exclusiones Docker
├── .env.example                     # Template variables entorno
├── CLAUDE.md                        # Guía rápida para Claude (raíz)
├── README_PLANTILLAS.md             # Este archivo (instrucciones uso)
├── docker-compose.yml               # Orquestación Docker
├── Dockerfile                       # Build de producción
├── Dockerfile.dev                   # Build de desarrollo
└── nginx.conf                       # Configuración Nginx (si aplica)
```

---

## 📝 Template del Prompt Completo

### COPIAR Y ADAPTAR ESTE PROMPT:

```markdown
# Solicitud: Generar Plantillas Completas para Proyecto [NOMBRE_PROYECTO]

Hola Claude, necesito que me generes **plantillas completas** para un nuevo proyecto multi-plataforma similar a cómo hiciste las plantillas de bMOI (plantillas_backend, plantillas_cpanel, plantillas_landing, plantillas_mobile, plantillas_admin).

## 📋 Información del Proyecto

### Nombre del Proyecto
**[NOMBRE DEL PROYECTO]**

### Propósito/Objetivo
[Descripción breve del proyecto: qué hace, para quién es, problema que resuelve]

### Características Especiales
- [ ] Multi-tenant (aislamiento por organización)
- [ ] Tiempo real (WebSockets, notificaciones)
- [ ] Offline-first (PWA, sincronización)
- [ ] Multi-idioma (i18n)
- [ ] Geolocalización
- [ ] Pagos online
- [ ] [Otra característica importante]

---

## 🏗️ Arquitectura del Proyecto

**IMPORTANTE:** He adjuntado el diagrama de arquitectura en este mensaje:
- Archivo: `[NOMBRE_ARCHIVO_ARQUITECTURA]` (ej: Arquitectura_MiProyecto.jpg)

**Descripción de la arquitectura:**
[Si el diagrama no es claro, describe brevemente:
- Cuántos frontends hay (web admin, web cliente, móvil, etc.)
- Tipo de backend (monolito, microservicios, serverless)
- Bases de datos y tecnologías
- Integraciones externas importantes]

---

## 📄 Documento Técnico Funcional

**IMPORTANTE:** He adjuntado el documento técnico funcional:
- Archivo: `[NOMBRE_ARCHIVO_DOC_TECNICO]` (ej: Documento_Tecnico_MiProyecto.docx)

**Por favor lee este documento para entender:**
- Concepto de negocio
- Casos de uso principales
- Requerimientos funcionales y no funcionales
- Restricciones técnicas

---

## 🛠️ Stack Tecnológico

### Backend
- **Framework:** [NestJS / Express / FastAPI / Django / Spring Boot / etc.]
- **Lenguaje:** [TypeScript / Python / Java / Go / etc.]
- **Base de Datos:** [PostgreSQL / MySQL / MongoDB / etc.]
- **Cache:** [Redis / Memcached / etc.]
- **ORM:** [Prisma / TypeORM / Mongoose / etc.]

### Frontend Web 1: [Nombre Panel/Dashboard]
- **Framework:** [React / Vue / Angular / Svelte / etc.]
- **Lenguaje:** [TypeScript / JavaScript]
- **Build Tool:** [Vite / Next.js / Webpack / etc.]
- **UI Library:** [Ant Design / Material-UI / Tailwind + Shadcn/UI / etc.]
- **State:** [Zustand / Redux / Pinia / etc.]

### Frontend Web 2: [Nombre Panel Admin] (si aplica)
- [Mismo formato que Frontend Web 1]

### Landing Page
- **Framework:** [Astro / Next.js / Gatsby / Hugo / etc.]
- **Enfoque:** [SSG / SSR]

### App Móvil
- **Framework:** [Flutter / React Native / Native (Swift+Kotlin) / etc.]
- **Lenguaje:** [Dart / TypeScript / Swift+Kotlin]
- **State:** [Riverpod / Redux / Provider / etc.]

---

## 📦 Plantillas a Generar

Por favor genera las siguientes plantillas completas:

1. **plantillas_backend** → [Nombre del backend]
2. **plantillas_[nombre_web_1]** → [Descripción]
3. **plantillas_[nombre_web_2]** → [Descripción] (si aplica)
4. **plantillas_landing** → Landing page
5. **plantillas_mobile** → App móvil

---

## ✅ Requerimientos de las Plantillas

### Para CADA plantilla, necesito:

#### 1. Documentación para IAs (.claude/, .copilot/, .cursorrules)
- `context.md` - Contexto completo del proyecto
- `conventions.md` - Convenciones obligatorias
- `session-state.md` - Estado del desarrollo
- `instructions.md` (Copilot)
- `.cursorrules` (Cursor AI)

#### 2. Documentación Técnica (.docs/)
- `README.md` - Overview
- `SETUP.md` - Setup paso a paso desde cero
- `ARQUITECTURA.md` - Decisiones técnicas justificadas
- `CODING_STANDARDS.md` - Estándares de código detallados
- `RESPONSIVE_DESIGN.md` - Guía responsive (solo frontends web)
- `EJEMPLOS_CODIGO.md` - Código listo para copiar
- `CONFIGURACIONES.md` - Archivos de configuración

#### 3. Docker Setup
- `docker-compose.yml` - Con profiles (dev/prod/full)
- `Dockerfile` - Build de producción
- `Dockerfile.dev` - Build de desarrollo
- `.dockerignore` - Optimizado
- `nginx.conf` - Si aplica (frontends web)

#### 4. Variables de Entorno
- `.env.example` - Template completo

#### 5. Guías Raíz
- `CLAUDE.md` - Guía rápida para Claude
- `README_PLANTILLAS.md` - Cómo usar las plantillas

#### 6. Documento de Integración (EN LA RAÍZ DEL PROYECTO)
- `INTEGRATION.md` - Cómo se comunican todas las interfaces entre sí
  - URLs por entorno (dev, staging, prod)
  - Headers obligatorios
  - Flujo de autenticación
  - Endpoints principales
  - Convenciones por lenguaje (justificadas)

#### 7. Análisis y Resumen (EN LA RAÍZ DEL PROYECTO)
- `ANALISIS_PLANTILLAS.md` - Análisis de completitud y coherencia
- `RESUMEN_OPTIMIZACIONES.md` - Resumen ejecutivo

---

## 🎯 Criterios de Calidad

### Cada plantilla debe:
1. ✅ Ser **100% coherente** con las demás plantillas del ecosistema
2. ✅ Documentar **decisiones técnicas justificadas** (por qué se eligió X sobre Y)
3. ✅ Incluir **ejemplos de código completos** listos para copiar
4. ✅ Tener **setup desde cero** paso a paso (10-15 pasos)
5. ✅ Ser **responsive** si es frontend web (mobile-first)
6. ✅ Tener **Docker mandatorio** para desarrollo
7. ✅ Incluir **estándares de código específicos** del stack
8. ✅ Documentar **URLs, headers, login flow** coherentes
9. ✅ Incluir **naming conventions** claras y obligatorias
10. ✅ Tener **sistema de memoria para IAs** (anti-olvido)

---

## 📊 Entregables Esperados

Al finalizar, espero recibir:

### Estructura de Carpetas
```
[proyecto_raiz]/
├── plantillas_backend/           # Plantilla completa
├── plantillas_[frontend_1]/      # Plantilla completa
├── plantillas_[frontend_2]/      # Plantilla completa (si aplica)
├── plantillas_landing/           # Plantilla completa
├── plantillas_mobile/            # Plantilla completa
├── INTEGRATION.md                # Guía de integración global
├── ANALISIS_PLANTILLAS.md        # Análisis de coherencia
└── RESUMEN_OPTIMIZACIONES.md     # Resumen ejecutivo
```

### Métricas de Completitud
- Promedio de completitud de todas las plantillas: >= 95%
- Coherencia entre plantillas: >= 95%
- Documentación total: >= 150KB por plantilla

---

## ⚙️ Proceso Sugerido

1. **Leer arquitectura y doc funcional** para entender el proyecto
2. **Identificar interfaces necesarias** (cuántas plantillas)
3. **Definir stack tecnológico** para cada interfaz (si no está definido)
4. **Generar plantilla_backend** primero (base del ecosistema)
5. **Generar plantillas de frontends** con coherencia al backend
6. **Generar INTEGRATION.md** con URLs, headers, flows
7. **Generar ANALISIS_PLANTILLAS.md** evaluando coherencia
8. **Generar RESUMEN_OPTIMIZACIONES.md** con métricas finales

---

## 🚀 Notas Importantes

1. **Prioriza coherencia sobre perfección individual**
   - Mejor que todas las plantillas sean coherentes entre sí, que una sola sea perfecta

2. **Justifica decisiones técnicas**
   - Por qué elegiste React sobre Vue
   - Por qué NestJS sobre Express
   - Trade-offs aceptados

3. **Responsive es obligatorio para frontends web**
   - Mobile-first approach
   - Breakpoints documentados
   - Ejemplos de layouts responsive

4. **Docker es mandatorio para desarrollo**
   - No opciones de desarrollo sin Docker
   - Profiles para diferentes escenarios

5. **Documenta diferencias de convenciones**
   - TypeScript usa camelCase
   - Dart usa lowerCamelCase para constantes
   - Justifica por qué

---

## 🎯 Objetivo Final

Que al finalizar, tenga un ecosistema completo de plantillas que:
- ✅ Me permita iniciar desarrollo de cada interfaz en **< 30 minutos**
- ✅ Garantice **coherencia tecnológica** entre todas las interfaces
- ✅ Sirva como **documentación profesional** para el equipo y para IAs
- ✅ Incluya **ejemplos de código** probados y listos para usar
- ✅ Tenga **guías de integración** claras entre componentes
- ✅ Sea **mantenible** y **escalable** a largo plazo

---

**¿Puedes generar todas estas plantillas siguiendo estas especificaciones?**

Por favor confirma que entendiste los requerimientos y procede con la generación.
```

---

## 💡 Ejemplo Real de Prompt

### Caso: Proyecto E-Learning Platform

```markdown
# Solicitud: Generar Plantillas Completas para Proyecto EduFlow

Hola Claude, necesito que me generes **plantillas completas** para un nuevo proyecto multi-plataforma similar a cómo hiciste las plantillas de bMOI.

## 📋 Información del Proyecto

### Nombre del Proyecto
**EduFlow** - Plataforma E-Learning Multi-Tenant

### Propósito/Objetivo
Plataforma SaaS que permite a instituciones educativas crear y gestionar cursos online con seguimiento de progreso, evaluaciones y certificaciones. Cada institución tiene su propio panel aislado.

### Características Especiales
- [x] Multi-tenant (aislamiento por institución educativa)
- [x] Tiempo real (clases en vivo, chat)
- [x] Offline-first (app móvil puede descargar cursos)
- [x] Multi-idioma (ES, EN, PT)
- [ ] Geolocalización
- [x] Pagos online (suscripciones institucionales)
- [x] Video streaming (AWS S3 + CloudFront)

---

## 🏗️ Arquitectura del Proyecto

**IMPORTANTE:** He adjuntado el diagrama de arquitectura:
- Archivo: `Arquitectura_EduFlow.jpg`

**Descripción de la arquitectura:**
- Frontend 1: Panel Web para **estudiantes** (ver cursos, tomar clases, evaluaciones)
- Frontend 2: Panel Web para **profesores** (crear cursos, subir contenido, calificar)
- Frontend 3: Panel Web para **administradores de institución** (gestionar usuarios, reportes)
- Backend: API NestJS monolítica con microservicio separado para video streaming
- Bases de datos: PostgreSQL (principal) + Redis (cache) + S3 (videos)
- Integraciones: Stripe (pagos), Twilio (notificaciones)

---

## 📄 Documento Técnico Funcional

**IMPORTANTE:** He adjuntado el documento técnico funcional:
- Archivo: `EduFlow_Documento_Tecnico.docx`

---

## 🛠️ Stack Tecnológico

### Backend
- **Framework:** NestJS 10
- **Lenguaje:** TypeScript 5
- **Base de Datos:** PostgreSQL 15 con Row-Level Security
- **Cache:** Redis 7
- **ORM:** Prisma

### Frontend Web 1: Panel Estudiantes
- **Framework:** React 18
- **Lenguaje:** TypeScript 5
- **Build Tool:** Vite 5
- **UI Library:** Tailwind CSS + Shadcn/UI
- **State:** Zustand + React Query

### Frontend Web 2: Panel Profesores
- **Framework:** React 18
- **Lenguaje:** TypeScript 5
- **Build Tool:** Vite 5
- **UI Library:** Tailwind CSS + Shadcn/UI
- **State:** Zustand + React Query

### Frontend Web 3: Panel Admin Institucional
- **Framework:** React 18
- **Lenguaje:** TypeScript 5
- **Build Tool:** Vite 5
- **UI Library:** Ant Design 5
- **State:** Zustand + React Query

### Landing Page
- **Framework:** Astro 4
- **Enfoque:** SSG (Static Site Generation)

### App Móvil
- **Framework:** Flutter 3.19
- **Lenguaje:** Dart 3.3
- **State:** Riverpod

---

## 📦 Plantillas a Generar

1. **plantillas_backend** → API NestJS + PostgreSQL + Prisma
2. **plantillas_estudiantes** → Panel web para estudiantes
3. **plantillas_profesores** → Panel web para profesores
4. **plantillas_admin_institucion** → Panel web para admin institucional
5. **plantillas_landing** → Landing page estática
6. **plantillas_mobile** → App móvil Flutter

---

## ✅ Requerimientos de las Plantillas

[Copiar la sección completa del template anterior]

---

## 🎯 Criterios de Calidad

[Copiar la sección completa del template anterior]

---

**¿Puedes generar todas estas plantillas siguiendo estas especificaciones?**
```

---

## 🎯 Qué Esperar como Resultado

### Tiempo Estimado
- **Análisis inicial:** 10-15 minutos
- **Generación de plantillas:** 60-90 minutos (dependiendo de complejidad)
- **Revisión y coherencia:** 15-20 minutos
- **Total:** ~2 horas para proyecto completo

### Proceso de Claude

1. **Lee y analiza** arquitectura + doc funcional
2. **Identifica interfaces** y stack tecnológico
3. **Genera plantilla_backend** primero (base)
4. **Genera plantillas frontends** una por una con coherencia
5. **Crea INTEGRATION.md** con URLs, headers, flows
6. **Genera ANALISIS_PLANTILLAS.md** evaluando coherencia
7. **Crea RESUMEN_OPTIMIZACIONES.md** con métricas

### Archivos Generados (Ejemplo para 5 interfaces)

```
Total de archivos: ~50-60 archivos
Total de líneas: ~15,000-20,000 líneas
Documentación: ~800KB-1MB
```

**Por plantilla:**
- `.claude/` (4 archivos)
- `.copilot/` (1 archivo)
- `.docs/` (6-8 archivos)
- Docker (3-4 archivos)
- Configuración (2-3 archivos)
- Guías raíz (2 archivos)

**Global:**
- `INTEGRATION.md`
- `ANALISIS_PLANTILLAS.md`
- `RESUMEN_OPTIMIZACIONES.md`

---

## ✅ Checklist de Información Necesaria

Antes de enviar el prompt, asegúrate de tener:

### Obligatorio
- [ ] Diagrama de arquitectura (imagen clara)
- [ ] Documento técnico funcional (DOCX/PDF/MD)
- [ ] Nombre del proyecto
- [ ] Propósito/objetivo claro
- [ ] Lista de interfaces necesarias

### Recomendado
- [ ] Stack tecnológico definido (o al menos preferencias)
- [ ] Características especiales (multi-tenant, tiempo real, etc.)
- [ ] Restricciones técnicas conocidas
- [ ] Integraciones externas importantes

### Opcional (Claude puede sugerir)
- [ ] Naming conventions específicas
- [ ] Paleta de colores
- [ ] Estructura de carpetas personalizada
- [ ] Decisiones arquitectónicas específicas

---

## ❓ FAQ

### ¿Puedo pedir plantillas en múltiples sesiones?
Sí, pero es menos eficiente. Es mejor generarlas todas de una vez para garantizar coherencia máxima.

### ¿Qué pasa si no tengo el stack tecnológico definido?
Claude puede sugerirte el stack más apropiado basándose en tu arquitectura y requerimientos. Solo menciónalo en el prompt.

### ¿Puedo pedir más de 5 interfaces?
Sí, pero considera que:
- Más de 6 interfaces puede tomar 3+ horas
- La coherencia es más difícil de mantener
- Considera dividir en fases si tienes 8+ interfaces

### ¿Las plantillas incluyen código funcional?
No, las plantillas son **documentación y configuración**. Incluyen:
- ✅ Ejemplos de código completos
- ✅ Configuración Docker funcional
- ✅ Setup paso a paso
- ❌ No incluyen código de features completas

### ¿Qué hago después de recibir las plantillas?
1. Crear directorios de proyectos
2. Copiar plantillas correspondientes
3. Seguir `SETUP.md` de cada plantilla
4. Inicializar proyecto con `npm create vite`, `nest new`, etc.
5. Empezar desarrollo con guías de `EJEMPLOS_CODIGO.md`

### ¿Puedo personalizar las plantillas después?
¡Por supuesto! Las plantillas son punto de partida. Puedes:
- Agregar más documentación
- Cambiar configuraciones
- Adaptar convenciones
- Agregar scripts personalizados

### ¿Las plantillas funcionan con otros IAs?
Sí, las plantillas incluyen configuración para:
- ✅ Claude Code (`.claude/`)
- ✅ GitHub Copilot (`.copilot/`)
- ✅ Cursor AI (`.cursorrules`)

---

## 📚 Recursos Adicionales

### Ejemplos de Plantillas Completas
- Ver `plantillas_backend/` del proyecto bMOI
- Ver `plantillas_cpanel-multitenant/` del proyecto bMOI

### Documentos de Referencia
- `INTEGRATION.md` - Ejemplo de guía de integración
- `ANALISIS_PLANTILLAS_BMOI.md` - Ejemplo de análisis
- `RESUMEN_OPTIMIZACIONES.md` - Ejemplo de resumen

---

## 🎉 Conclusión

Con este template de prompt, puedes solicitar la generación completa de plantillas para cualquier proyecto multi-plataforma en **una sola instrucción**.

**Ventajas:**
- ✅ Ahorra tiempo (2 horas vs múltiples sesiones)
- ✅ Garantiza coherencia máxima
- ✅ Documentación profesional desde día 1
- ✅ Onboarding rápido para equipo y IAs
- ✅ Base sólida para desarrollo

**Recuerda:**
1. Proporciona arquitectura clara
2. Proporciona doc funcional completo
3. Define stack tecnológico (o pide sugerencias)
4. Copia el template del prompt y adapta
5. Espera ~2 horas para generación completa

---

**Última actualización:** 2025-01-04
**Versión:** 1.0.0
**Basado en:** Experiencia con proyecto bMOI (4 plantillas generadas exitosamente)
