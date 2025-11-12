# 🔬 Evaluación Técnica del Stack - bMOI Admin

> **Análisis exhaustivo del stack tecnológico para la interfaz Admin de bMOI**

**Fecha:** 2025-01-05
**Versión:** 1.0.0
**Evaluador:** Claude Code (Sonnet 4.5)

---

## 📊 Resumen Ejecutivo

### Objetivo de la Evaluación

Determinar si el stack tecnológico propuesto en las plantillas actuales es **adecuado** para soportar los requerimientos específicos de bMOI Admin:

- ✅ **KYC**: Revisión de artesanos (documentos, sanciones, revalidaciones)
- ✅ **Moderación**: Briefs, imágenes, portafolios (aprobación/rechazo)
- ✅ **Disputas**: Timeline, evidencias, resolución con notas internas
- ✅ **Configuración Negocio**: Comisiones, fees, cupones, políticas SLA
- ✅ **Panel Ejecutivo**: Dashboards con AOV, embudo, cohortes, fraude
- ✅ **Soporte**: Búsqueda avanzada, acciones seguras
- ✅ **Seguridad**: MFA, RBAC/ABAC, auditoría, IP allowlist, zero-PAN

### Conclusión

| Métrica | Resultado |
|---------|-----------|
| **Stack Base** | ✅ **EXCELENTE** - React 18 + TS + Vite es ideal |
| **Gaps Identificados** | ⚠️ **7 categorías** requieren librerías adicionales |
| **Ajustes Necesarios** | ✅ **MENORES** - Solo agregar dependencias específicas |
| **Recomendación Final** | ✅ **APROBAR con extensiones** |

---

## 🛠️ Stack Actual (Plantillas Base)

### ✅ Core - APROBADO

| Tecnología | Versión | Evaluación | Justificación |
|------------|---------|------------|---------------|
| **React** | 18.3+ | ✅ **EXCELENTE** | Concurrent features ideales para UX fluida en workflows complejos |
| **TypeScript** | 5.6+ (strict) | ✅ **CRÍTICO** | Type-safety esencial para evitar errores en operaciones sensibles (KYC, disputas) |
| **Vite** | 5+ | ✅ **PERFECTO** | HMR instantáneo acelera desarrollo; build optimizado para producción |
| **npm** | Latest | ✅ **ADECUADO** | Lock file + audit integrado; estable para CI/CD |

**Veredicto Core:** ✅ **Mantener sin cambios**

---

### ✅ State Management - APROBADO

| Tecnología | Versión | Evaluación | Justificación |
|------------|---------|------------|---------------|
| **Zustand** | 4+ | ✅ **PERFECTO** | Ligero (1KB), ideal para auth state, UI state, tenant selection |
| **React Query** | 5+ | ✅ **EXCELENTE** | Caching automático, stale-while-revalidate ideal para dashboards en tiempo real |

**Casos de uso confirmados:**
```typescript
// Zustand: Client-side state
- authStore (token, user, selected_tenant_id)
- uiStore (sidebar, theme, notifications)
- permissionsStore (RBAC/ABAC rules)

// React Query: Server-side state
- useKYCPendingQueue() → Auto-refetch cada 30s
- useDashboardMetrics() → Stale 5min, background refetch
- useDisputeTimeline(id) → Real-time updates con refetchInterval
```

**Veredicto State:** ✅ **Mantener sin cambios**

---

### ✅ Forms & Validation - APROBADO

| Tecnología | Versión | Evaluación | Justificación |
|------------|---------|------------|---------------|
| **React Hook Form** | 7+ | ✅ **EXCELENTE** | Uncontrolled forms = mejor performance; critical para forms complejos de KYC/Moderación |
| **Zod** | 3+ | ✅ **CRÍTICO** | Schema validation type-safe; compartible frontend-backend (coherencia) |

**Casos de uso confirmados:**
```typescript
// KYC Rejection Form
const kycRejectionSchema = z.object({
  reason: z.enum(['invalid_doc', 'expired', 'sanctions', 'incomplete']),
  notes: z.string().min(20).max(500),
  requiresRevalidation: z.boolean(),
});

// Business Config Form
const commissionConfigSchema = z.object({
  categoryId: z.string().uuid(),
  percentage: z.number().min(0).max(100),
  flatFee: z.number().min(0),
  applies_from: z.date(),
});
```

**Veredicto Forms:** ✅ **Mantener sin cambios**

---

### ✅ UI Framework - APROBADO CON AJUSTE MENOR

| Tecnología | Versión | Evaluación | Justificación |
|------------|---------|------------|---------------|
| **Tailwind CSS** | 3.4+ | ✅ **EXCELENTE** | Utility-first ideal para customización; JIT mode = bundle pequeño |
| **Shadcn/UI** | Latest | ✅ **PERFECTO** | Copy-paste approach = full control; Radix UI = accesibilidad WCAG AA |
| **Lucide React** | Latest | ✅ **BUENO** | Icons consistentes, tree-shakeable |

**Ajuste requerido:** Agregar componentes adicionales de Shadcn/UI:

```bash
# Componentes adicionales necesarios para bMOI Admin
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add dropdown-menu
npx shadcn-ui@latest add tabs
npx shadcn-ui@latest add badge
npx shadcn-ui@latest add alert
npx shadcn-ui@latest add toast
npx shadcn-ui@latest add select
npx shadcn-ui@latest add textarea
npx shadcn-ui@latest add switch
npx shadcn-ui@latest add separator
npx shadcn-ui@latest add avatar
npx shadcn-ui@latest add progress
npx shadcn-ui@latest add skeleton
```

**Veredicto UI:** ✅ **Mantener con componentes adicionales**

---

## ⚠️ GAPS Identificados - Requieren Extensión

### 1. 📄 Visualización de Documentos (KYC)

**Requerimiento:**
- Viewer de PDFs (documentos de identidad)
- Lightbox para imágenes (selfies, comprobantes)
- Zoom, rotación, anotaciones
- Preview antes de subir

**Stack actual:** ❌ No incluye

**Solución recomendada:**

| Librería | Propósito | Evaluación | Alternativas |
|----------|-----------|------------|--------------|
| **react-pdf** | Viewer de PDFs | ✅ Recomendado | pdf.js (más bajo nivel) |
| **yet-another-react-lightbox** | Lightbox moderno | ✅ Recomendado | react-image-lightbox (deprecated) |
| **react-image-crop** | Crop/edición básica | ✅ Opcional | - |

**Instalación:**
```bash
npm install react-pdf yet-another-react-lightbox
npm install -D @types/react-pdf
```

**Ejemplo de uso:**
```typescript
// src/components/kyc/DocumentViewer.tsx
import { Document, Page } from 'react-pdf';
import Lightbox from 'yet-another-react-lightbox';

export const DocumentViewer = ({ document }) => {
  if (document.type === 'pdf') {
    return (
      <Document file={document.url}>
        <Page pageNumber={1} />
      </Document>
    );
  }

  return (
    <Lightbox
      open={isOpen}
      close={() => setIsOpen(false)}
      slides={[{ src: document.url }]}
    />
  );
};
```

**Impacto:** 🔴 **CRÍTICO** - Sin esto, KYC no funciona

---

### 2. 📊 Charts y Visualización (Dashboard Ejecutivo)

**Requerimiento:**
- Line/Bar charts (tendencias)
- Funnel chart (embudo conversión)
- Cohort table (análisis cohortes)
- Real-time metrics (counter animado)
- Heatmap (actividad por hora/día)

**Stack actual:** ❌ No incluye

**Solución recomendada:**

| Librería | Propósito | Evaluación | Bundle Size | Alternativas |
|----------|-----------|------------|-------------|--------------|
| **Recharts** | Charts generales | ✅ **Recomendado** | 96KB | Chart.js (no React-first) |
| **@tremor/react** | Dashboard components | ⚠️ Opcional | 120KB | Visx (más bajo nivel) |
| **react-countup** | Animated counters | ✅ Recomendado | 8KB | - |

**Instalación:**
```bash
npm install recharts react-countup
npm install -D @types/recharts
```

**Por qué Recharts:**
- ✅ Composable components (React-first)
- ✅ Responsive by default
- ✅ TypeScript support nativo
- ✅ Customizable con Tailwind
- ✅ Tree-shakeable (solo importas lo que usas)

**Ejemplo de uso:**
```typescript
// src/components/dashboard/MetricsChart.tsx
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip } from 'recharts';

export const AOVChart = ({ data }) => (
  <LineChart width={600} height={300} data={data}>
    <XAxis dataKey="date" />
    <YAxis />
    <CartesianGrid strokeDasharray="3 3" />
    <Tooltip />
    <Line type="monotone" dataKey="aov" stroke="#8884d8" />
  </LineChart>
);
```

**Impacto:** 🔴 **CRÍTICO** - Dashboard ejecutivo depende de esto

---

### 3. 🔐 Seguridad Avanzada (MFA, RBAC/ABAC)

**Requerimiento:**
- MFA/2FA con TOTP
- QR code generation
- Session management avanzado
- Permission checks granulares

**Stack actual:** ⚠️ Parcial (solo Zustand + JWT básico)

**Solución recomendada:**

| Librería | Propósito | Evaluación | Alternativas |
|----------|-----------|------------|--------------|
| **otpauth** | TOTP generation/validation | ✅ Recomendado | speakeasy (Node-only) |
| **qrcode.react** | QR code component | ✅ Recomendado | qrcode (más bajo nivel) |
| **@casl/ability** | RBAC/ABAC rules engine | ✅ **Crítico** | Custom (reinventar rueda) |

**Instalación:**
```bash
npm install otpauth qrcode.react @casl/ability @casl/react
```

**Por qué @casl/ability:**
- ✅ Tipo-seguro con TypeScript
- ✅ Soporta RBAC y ABAC
- ✅ Define reglas en backend, evalúa en frontend
- ✅ React hooks: `useAbility()`, `Can` component

**Ejemplo de uso:**
```typescript
// src/lib/permissions/ability.ts
import { defineAbility } from '@casl/ability';

export const defineUserAbility = (user) => {
  return defineAbility((can, cannot) => {
    if (user.role === 'superadmin') {
      can('manage', 'all');
    }

    if (user.role === 'kyc.reviewer') {
      can('read', 'KYC');
      can('approve', 'KYC');
      can('reject', 'KYC');
      cannot('delete', 'KYC');
    }

    if (user.role === 'ops.viewer') {
      can('read', 'all');
      cannot('create', 'all');
      cannot('update', 'all');
    }
  });
};

// Uso en componentes
import { Can } from '@casl/react';
import { useAbility } from '@/hooks/useAbility';

export const KYCActions = () => {
  const ability = useAbility();

  return (
    <>
      <Can I="approve" a="KYC" ability={ability}>
        <Button onClick={handleApprove}>Approve</Button>
      </Can>
      <Can I="reject" a="KYC" ability={ability}>
        <Button onClick={handleReject}>Reject</Button>
      </Can>
    </>
  );
};
```

**Impacto:** 🔴 **CRÍTICO** - Seguridad depende de permisos granulares

---

### 4. 📁 Manejo de Archivos (S3, Upload, Preview)

**Requerimiento:**
- Upload a S3 (evidencias, docs KYC)
- Progress bar
- Multiple files
- Image optimization (resize antes de subir)
- Presigned URLs

**Stack actual:** ⚠️ Axios (solo HTTP, no upload avanzado)

**Solución recomendada:**

| Librería | Propósito | Evaluación | Alternativas |
|----------|-----------|------------|--------------|
| **react-dropzone** | Drag & drop upload | ✅ Recomendado | react-upload (menos popular) |
| **browser-image-compression** | Client-side resize | ✅ Recomendado | jimp (Node-only) |
| **AWS SDK v3** | S3 direct upload | ✅ Recomendado | Presigned URLs (más seguro) |

**Instalación:**
```bash
npm install react-dropzone browser-image-compression
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

**Arquitectura recomendada:**
```
Frontend → Request Presigned URL → Backend → Generate URL → S3
Frontend → Upload to S3 with Presigned URL → S3
Frontend → Notify Backend (file uploaded) → Backend → Save metadata
```

**Por qué Presigned URLs (no direct S3):**
- ✅ Backend controla permisos (quién puede subir qué)
- ✅ No exponer AWS credentials en frontend
- ✅ Backend valida tipo de archivo, tamaño, tenant_id
- ✅ Temporal (expira en 15min)

**Ejemplo de uso:**
```typescript
// src/components/upload/FileUploader.tsx
import { useDropzone } from 'react-dropzone';
import imageCompression from 'browser-image-compression';

export const EvidenceUploader = ({ disputeId, onUploadComplete }) => {
  const { getRootProps, getInputProps } = useDropzone({
    accept: { 'image/*': ['.png', '.jpg', '.jpeg', '.webp'] },
    maxSize: 10 * 1024 * 1024, // 10MB
    onDrop: async (files) => {
      for (const file of files) {
        // 1. Compress image
        const compressed = await imageCompression(file, {
          maxSizeMB: 1,
          maxWidthOrHeight: 1920,
        });

        // 2. Request presigned URL
        const { url, key } = await evidenceService.getPresignedUrl({
          disputeId,
          filename: file.name,
        });

        // 3. Upload to S3
        await axios.put(url, compressed, {
          headers: { 'Content-Type': compressed.type },
        });

        // 4. Notify backend
        await evidenceService.notifyUploaded({ disputeId, key });
      }

      onUploadComplete();
    },
  });

  return (
    <div {...getRootProps()} className="border-2 border-dashed p-8">
      <input {...getInputProps()} />
      <p>Drag & drop files here, or click to select</p>
    </div>
  );
};
```

**Impacto:** 🔴 **CRÍTICO** - KYC y Disputas dependen de upload de evidencias

---

### 5. 🔔 Notificaciones en Tiempo Real

**Requerimiento:**
- Alertas de fraude en tiempo real
- Nuevas disputas creadas
- KYC pendientes de revisión
- Notificaciones push

**Stack actual:** ❌ No incluye

**Solución recomendada:**

| Tecnología | Propósito | Evaluación | Alternativas |
|------------|-----------|------------|--------------|
| **EventSource (SSE)** | Server-Sent Events | ✅ **Recomendado** | WebSockets (overkill) |
| **WebSockets** | Bidireccional (si necesario) | ⚠️ Solo si SSE no basta | Polling (ineficiente) |
| **React Query refetchInterval** | Polling inteligente | ✅ Backup | - |

**Por qué SSE > WebSockets para este caso:**
- ✅ Unidireccional (servidor → cliente) es suficiente
- ✅ Auto-reconnect nativo
- ✅ HTTP/2 (usa misma conexión que API)
- ✅ Más simple de implementar
- ✅ Firewall-friendly

**Instalación:**
```bash
# No requiere npm install (nativo en browser)
# Backend debe implementar endpoint SSE
```

**Ejemplo de uso:**
```typescript
// src/hooks/useRealTimeNotifications.ts
import { useEffect, useState } from 'react';

export const useRealTimeNotifications = () => {
  const [notifications, setNotifications] = useState([]);

  useEffect(() => {
    const token = localStorage.getItem('access_token');
    const eventSource = new EventSource(
      `${API_BASE_URL}/notifications/stream?token=${token}`
    );

    eventSource.addEventListener('fraud_alert', (event) => {
      const data = JSON.parse(event.data);
      setNotifications((prev) => [...prev, data]);
      toast.error(`Fraud Alert: ${data.message}`);
    });

    eventSource.addEventListener('dispute_created', (event) => {
      const data = JSON.parse(event.data);
      toast.info(`New Dispute: #${data.disputeId}`);
    });

    return () => eventSource.close();
  }, []);

  return { notifications };
};
```

**Impacto:** 🟡 **IMPORTANTE** - UX mejorada, pero no bloqueante (puede usar polling)

---

### 6. 📤 Exportación de Datos (Excel, CSV, PDF)

**Requerimiento:**
- Export audit logs a CSV
- Export dashboard metrics a Excel
- Generate PDF reports

**Stack actual:** ❌ No incluye

**Solución recomendada:**

| Librería | Propósito | Evaluación | Bundle Size | Alternativas |
|----------|-----------|------------|-------------|--------------|
| **xlsx** | Excel generation | ✅ Recomendado | 200KB | SheetJS (paid) |
| **papaparse** | CSV parsing/export | ✅ Ligero | 45KB | - |
| **jsPDF + autoTable** | PDF generation | ⚠️ Opcional | 150KB | Backend generation (mejor) |

**Instalación:**
```bash
npm install xlsx papaparse
npm install -D @types/papaparse
```

**Recomendación:** Excel/CSV en frontend, PDF en backend

**Por qué:**
- ✅ Frontend: Excel/CSV = rápido, no sobrecarga servidor
- ✅ Backend: PDF = mejor formatting, server-side rendering

**Ejemplo de uso:**
```typescript
// src/lib/export/excel-export.ts
import * as XLSX from 'xlsx';

export const exportToExcel = (data: any[], filename: string) => {
  const worksheet = XLSX.utils.json_to_sheet(data);
  const workbook = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(workbook, worksheet, 'Data');
  XLSX.writeFile(workbook, `${filename}.xlsx`);
};

// Uso
const handleExportAuditLogs = () => {
  const data = auditLogs.map(log => ({
    Date: log.createdAt,
    User: log.user.email,
    Action: log.action,
    Resource: log.resource,
    IP: log.ipAddress,
  }));

  exportToExcel(data, `audit_logs_${new Date().toISOString()}`);
};
```

**Impacto:** 🟡 **IMPORTANTE** - Nice to have, no bloqueante

---

### 7. 🔍 Búsqueda Avanzada y Filtros

**Requerimiento:**
- Búsqueda full-text en usuarios/pedidos
- Filtros complejos (date range, status, amount range)
- URL-synced filters (shareable URLs)

**Stack actual:** ⚠️ React Query (solo fetch básico)

**Solución recomendada:**

| Librería | Propósito | Evaluación | Alternativas |
|----------|-----------|------------|--------------|
| **use-query-params** | Sync URL ↔ State | ✅ Recomendado | Custom hooks (complejo) |
| **date-fns** | Date parsing/formatting | ✅ Recomendado | dayjs (similar) |
| **React Query filters** | Cache invalidation | ✅ Incluido | - |

**Instalación:**
```bash
npm install use-query-params serialize-query-params date-fns
```

**Ejemplo de uso:**
```typescript
// src/pages/users/UsersPage.tsx
import { useQueryParams, StringParam, NumberParam } from 'use-query-params';

export const UsersPage = () => {
  const [query, setQuery] = useQueryParams({
    search: StringParam,
    status: StringParam,
    page: NumberParam,
    limit: NumberParam,
  });

  const { data, isLoading } = useUsers(query);

  // URL: /users?search=john&status=active&page=2&limit=20
  // Shareable, bookmarkable, back button works
};
```

**Impacto:** 🟡 **IMPORTANTE** - UX mejorada, no bloqueante

---

## 🐳 Docker - Ajustes Necesarios

### Stack Actual (docker-compose.yml)

```yaml
services:
  frontend:  ✅ OK
  backend:   ✅ OK (profile: full)
  postgres:  ✅ OK (profile: full)
  redis:     ✅ OK (profile: full)
```

### Servicios Adicionales Recomendados

**Para desarrollo completo de bMOI Admin:**

```yaml
# Agregar a docker-compose.yml

  # ============================================
  # S3 Compatible Storage (MinIO)
  # ============================================
  minio:
    container_name: bmoi-minio
    image: minio/minio:latest
    ports:
      - "9000:9000"  # API
      - "9001:9001"  # Console
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    command: server /data --console-address ":9001"
    volumes:
      - minio-data:/data
    networks:
      - bmoi-network
    profiles:
      - full

  # ============================================
  # Event Streaming (NATS for SSE/real-time)
  # ============================================
  nats:
    container_name: bmoi-nats
    image: nats:alpine
    ports:
      - "4222:4222"  # Client
      - "8222:8222"  # Monitoring
    networks:
      - bmoi-network
    profiles:
      - full

  # ============================================
  # Search Engine (Meilisearch for full-text)
  # ============================================
  meilisearch:
    container_name: bmoi-meilisearch
    image: getmeili/meilisearch:latest
    ports:
      - "7700:7700"
    environment:
      MEILI_MASTER_KEY: masterKey_dev_only
    volumes:
      - meilisearch-data:/meili_data
    networks:
      - bmoi-network
    profiles:
      - full

volumes:
  minio-data:
  meilisearch-data:
```

**Justificación:**
- **MinIO**: S3-compatible storage para evidencias/docs KYC (local dev)
- **NATS**: Event streaming para notificaciones en tiempo real
- **Meilisearch**: Full-text search (alternativa ligera a Elasticsearch)

**Impacto:** 🟡 **Opcional** - Solo para desarrollo local completo

---

## 📦 Resumen de Dependencias a Agregar

### Críticas (P0 - Sin esto bMOI Admin no funciona)

```bash
# Document viewing (KYC)
npm install react-pdf yet-another-react-lightbox

# Charts (Dashboard Ejecutivo)
npm install recharts react-countup

# Security (RBAC/ABAC)
npm install otpauth qrcode.react @casl/ability @casl/react

# File upload (Evidencias)
npm install react-dropzone browser-image-compression
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

### Importantes (P1 - Mejoran UX significativamente)

```bash
# Data export
npm install xlsx papaparse

# Advanced filters
npm install use-query-params serialize-query-params date-fns

# Notifications UI
npm install sonner  # Toast notifications (si no está)
```

### Opcionales (P2 - Nice to have)

```bash
# PDF generation (mejor en backend)
npm install jspdf jspdf-autotable

# Advanced data viz
npm install @tremor/react
```

---

## 📋 Checklist de Implementación

### Phase 1: Stack Extensions (1-2 días)

- [ ] Instalar dependencias críticas
- [ ] Configurar react-pdf (worker setup)
- [ ] Configurar CASL ability
- [ ] Setup react-dropzone + S3 presigned URLs
- [ ] Integrar Recharts en dashboard

### Phase 2: Infrastructure (1 día)

- [ ] Agregar MinIO a docker-compose.yml
- [ ] Agregar NATS a docker-compose.yml
- [ ] Actualizar .env.example con nuevas vars
- [ ] Configurar S3 buckets en MinIO

### Phase 3: Security (2 días)

- [ ] Implementar MFA/2FA flow
- [ ] Definir RBAC/ABAC rules
- [ ] Implementar permission checks en rutas
- [ ] Agregar auditoría en interceptors

### Phase 4: Features (5-7 días)

- [ ] Implementar KYC workflows
- [ ] Implementar sistema de moderación
- [ ] Implementar resolución de disputas
- [ ] Implementar dashboard ejecutivo
- [ ] Implementar búsqueda avanzada

---

## 🎯 Recomendación Final

### ✅ APROBAR Stack con Extensiones

**Stack base (React 18 + TS + Vite + Zustand + React Query + Shadcn/UI) es EXCELENTE.**

**Requiere 7 extensiones específicas para bMOI Admin:**

| # | Extensión | Prioridad | Estimación | Impacto |
|---|-----------|-----------|------------|---------|
| 1 | Document Viewing | 🔴 P0 | 4h | KYC bloqueado sin esto |
| 2 | Charts (Recharts) | 🔴 P0 | 6h | Dashboard bloqueado sin esto |
| 3 | RBAC/ABAC (CASL) | 🔴 P0 | 8h | Seguridad crítica |
| 4 | File Upload (S3) | 🔴 P0 | 6h | Evidencias bloqueadas |
| 5 | Real-time (SSE) | 🟡 P1 | 4h | UX mejorada |
| 6 | Data Export | 🟡 P1 | 3h | Nice to have |
| 7 | Advanced Search | 🟡 P1 | 5h | UX mejorada |

**Total estimado:** ~36 horas (4-5 días) de setup adicional

**ROI:** 🚀 **ALTO** - Evita reinventar la rueda, usa librerías battle-tested

---

## 📚 Próximos Pasos

1. ✅ **Aprobar este documento**
2. 📝 **Crear los 7 documentos específicos de bMOI** (BMOI_KYC_WORKFLOWS.md, etc.)
3. 🔧 **Actualizar .env.example** con nuevas variables
4. 🐳 **Actualizar docker-compose.yml** con servicios opcionales
5. 📖 **Actualizar ARQUITECTURA.md** con stack extendido
6. 💻 **Crear ejemplos de código** para workflows críticos

---

**Elaborado por:** Claude Code (Sonnet 4.5)
**Revisión:** Pendiente
**Aprobación:** Pendiente
