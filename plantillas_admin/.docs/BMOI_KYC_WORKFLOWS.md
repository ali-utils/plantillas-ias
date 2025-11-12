# 🔍 KYC Workflows - bMOI Admin

> **Sistema completo de revisión y validación KYC para artesanos**

**Módulo:** KYC (Know Your Customer)
**Usuarios:** kyc.reviewer, risk.analyst, compliance.officer, superadmin
**Prioridad:** 🔴 P0 - CRÍTICO

---

## 📋 Índice

1. [Overview](#overview)
2. [Estados del KYC](#estados-del-kyc)
3. [Flujo Completo](#flujo-completo)
4. [Arquitectura Frontend](#arquitectura-frontend)
5. [Componentes UI](#componentes-ui)
6. [Servicios y API](#servicios-y-api)
7. [Validaciones y Reglas](#validaciones-y-reglas)
8. [Integraciones](#integraciones)
9. [Seguridad y Auditoría](#seguridad-y-auditoría)
10. [Testing](#testing)

---

## 🎯 Overview

### Propósito

Validar la identidad y legitimidad de artesanos antes de permitirles operar en la plataforma bMOI. El proceso incluye:

- ✅ Validación de documentos de identidad
- ✅ Verificación contra listas de sanciones
- ✅ Comprobación de datos biométricos (selfie)
- ✅ Validación de información fiscal
- ✅ Revalidaciones periódicas

### Roles y Permisos

| Rol | Permisos |
|-----|----------|
| **kyc.reviewer** | Ver cola, aprobar/rechazar KYC, agregar notas |
| **risk.analyst** | Todo lo anterior + ver historial completo, forzar revalidación |
| **compliance.officer** | Todo lo anterior + configurar reglas, ver reportes |
| **superadmin** | Acceso total |

---

## 🔄 Estados del KYC

### Diagrama de Estados

```
                    ┌──────────────┐
                    │   PENDING    │ ← Artesano completa formulario
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │ IN_REVIEW    │ ← Reviewer toma el caso
                    └──────┬───────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼────┐    ┌───────▼───────┐   ┌────▼────┐
    │APPROVED │    │   REJECTED    │   │PENDING  │
    │         │    │               │   │MORE_INFO│
    └────┬────┘    └───────┬───────┘   └────┬────┘
         │                 │                 │
         │                 │                 └──────┐
         │                 │                        │
    ┌────▼────┐    ┌───────▼───────┐        ┌──────▼──────┐
    │ ACTIVE  │    │   BLOCKED     │        │Info provided│
    │(30 days)│    │(permanent ban)│        │  by artisan │
    └────┬────┘    └───────────────┘        └──────┬──────┘
         │                                          │
         │◄─────────────────────────────────────────┘
         │
    ┌────▼────────┐
    │REVALIDATION │ ← Cada 365 días o trigger manual
    │   REQUIRED  │
    └─────────────┘
```

### Definición de Estados (TypeScript)

```typescript
// src/types/kyc.types.ts
export enum KYCStatus {
  PENDING = 'pending',
  IN_REVIEW = 'in_review',
  PENDING_MORE_INFO = 'pending_more_info',
  APPROVED = 'approved',
  REJECTED = 'rejected',
  ACTIVE = 'active',
  REVALIDATION_REQUIRED = 'revalidation_required',
  BLOCKED = 'blocked',
}

export interface KYCApplication {
  id: string;
  artisanId: string;
  status: KYCStatus;
  documents: KYCDocument[];
  personalInfo: PersonalInfo;
  businessInfo: BusinessInfo;
  sanctionsCheckResult: SanctionsCheckResult;
  reviewHistory: KYCReviewAction[];
  createdAt: Date;
  updatedAt: Date;
  reviewedAt?: Date;
  reviewedBy?: string;
  expiresAt?: Date; // Para revalidaciones
}

export interface KYCDocument {
  id: string;
  type: DocumentType;
  url: string;
  uploadedAt: Date;
  status: 'pending' | 'verified' | 'rejected';
  rejectionReason?: string;
}

export enum DocumentType {
  NATIONAL_ID_FRONT = 'national_id_front',
  NATIONAL_ID_BACK = 'national_id_back',
  SELFIE = 'selfie',
  PROOF_OF_ADDRESS = 'proof_of_address',
  TAX_ID = 'tax_id',
  BUSINESS_REGISTRATION = 'business_registration',
}

export interface SanctionsCheckResult {
  checked: boolean;
  checkedAt?: Date;
  provider: 'ComplyAdvantage' | 'WorldCheck' | 'Internal';
  result: 'clear' | 'match' | 'potential_match' | 'error';
  matches?: SanctionsMatch[];
  riskScore: number; // 0-100
}

export interface KYCReviewAction {
  id: string;
  action: 'approved' | 'rejected' | 'requested_info' | 'revalidation' | 'note_added';
  performedBy: string;
  performedByRole: string;
  timestamp: Date;
  reason?: string;
  notes?: string;
  ipAddress: string;
}
```

---

## 🔁 Flujo Completo

### 1. Artesano Completa KYC

```typescript
// Frontend artesano (no admin)
POST /api/v1/kyc/applications
{
  personalInfo: { ... },
  documents: [ ... ],
  businessInfo: { ... }
}

→ Status: PENDING
```

### 2. Aparece en Cola de Revisión (Admin)

```typescript
// src/pages/kyc/KYCQueuePage.tsx
export const KYCQueuePage = () => {
  const { data: queue, isLoading } = useKYCQueue({
    status: ['pending', 'in_review'],
    sortBy: 'createdAt',
    order: 'asc',
  });

  if (isLoading) return <Skeleton />;

  return (
    <div className="space-y-4">
      <KYCQueueHeader />
      <KYCFilters />
      <KYCQueueTable applications={queue} />
    </div>
  );
};
```

### 3. Reviewer Toma el Caso

```typescript
// src/services/kyc.service.ts
export const kycService = {
  async claimApplication(applicationId: string): Promise<KYCApplication> {
    const response = await apiClient.post(`/kyc/applications/${applicationId}/claim`);
    return response.data;
  },
};

// Hook
export const useClaimKYC = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: kycService.claimApplication,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['kyc', 'queue'] });
      toast.success('Application claimed successfully');
    },
  });
};
```

### 4. Revisión de Documentos

```typescript
// src/pages/kyc/KYCReviewPage.tsx
import { DocumentViewer } from '@/components/kyc/DocumentViewer';
import { SanctionsCheckPanel } from '@/components/kyc/SanctionsCheckPanel';
import { PersonalInfoPanel } from '@/components/kyc/PersonalInfoPanel';
import { ReviewActions } from '@/components/kyc/ReviewActions';

export const KYCReviewPage = () => {
  const { id } = useParams();
  const { data: application, isLoading } = useKYCApplication(id);
  const approveMutation = useApproveKYC();
  const rejectMutation = useRejectKYC();

  if (isLoading) return <Skeleton />;

  return (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
      {/* Left: Documents */}
      <div className="lg:col-span-2 space-y-6">
        <DocumentViewer documents={application.documents} />
      </div>

      {/* Right: Info & Actions */}
      <div className="space-y-6">
        <PersonalInfoPanel info={application.personalInfo} />
        <SanctionsCheckPanel result={application.sanctionsCheckResult} />
        <ReviewActions
          applicationId={application.id}
          onApprove={(notes) => approveMutation.mutate({ id: application.id, notes })}
          onReject={(reason, notes) =>
            rejectMutation.mutate({ id: application.id, reason, notes })
          }
        />
      </div>
    </div>
  );
};
```

---

## 🎨 Componentes UI

### DocumentViewer

```typescript
// src/components/kyc/DocumentViewer.tsx
import { Document, Page, pdfjs } from 'react-pdf';
import Lightbox from 'yet-another-react-lightbox';
import { Download, ZoomIn, RotateCw } from 'lucide-react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';

// Configure worker
pdfjs.GlobalWorkerOptions.workerSrc = `//unpkg.com/pdfjs-dist@${pdfjs.version}/build/pdf.worker.min.js`;

interface DocumentViewerProps {
  documents: KYCDocument[];
}

export const DocumentViewer = ({ documents }: DocumentViewerProps) => {
  const [selectedDoc, setSelectedDoc] = useState<KYCDocument | null>(null);
  const [rotation, setRotation] = useState(0);

  const getDocumentLabel = (type: DocumentType): string => {
    const labels: Record<DocumentType, string> = {
      [DocumentType.NATIONAL_ID_FRONT]: 'ID Nacional (Frente)',
      [DocumentType.NATIONAL_ID_BACK]: 'ID Nacional (Reverso)',
      [DocumentType.SELFIE]: 'Selfie',
      [DocumentType.PROOF_OF_ADDRESS]: 'Comprobante de Domicilio',
      [DocumentType.TAX_ID]: 'RFC / Tax ID',
      [DocumentType.BUSINESS_REGISTRATION]: 'Registro de Negocio',
    };
    return labels[type];
  };

  const handleDownload = async (doc: KYCDocument) => {
    const response = await fetch(doc.url);
    const blob = await response.blob();
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `${doc.type}_${doc.id}.${doc.url.split('.').pop()}`;
    a.click();
  };

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
      {documents.map((doc) => {
        const isPDF = doc.url.endsWith('.pdf');
        const isImage = /\.(jpg|jpeg|png|webp)$/i.test(doc.url);

        return (
          <Card key={doc.id}>
            <CardHeader className="flex flex-row items-center justify-between">
              <CardTitle className="text-sm">{getDocumentLabel(doc.type)}</CardTitle>
              <Badge
                variant={
                  doc.status === 'verified'
                    ? 'success'
                    : doc.status === 'rejected'
                    ? 'destructive'
                    : 'secondary'
                }
              >
                {doc.status}
              </Badge>
            </CardHeader>
            <CardContent className="space-y-2">
              {/* Preview */}
              <div
                className="relative aspect-video bg-muted rounded-md overflow-hidden cursor-pointer group"
                onClick={() => setSelectedDoc(doc)}
              >
                {isPDF && (
                  <Document file={doc.url}>
                    <Page pageNumber={1} width={300} />
                  </Document>
                )}
                {isImage && (
                  <img
                    src={doc.url}
                    alt={doc.type}
                    className="w-full h-full object-cover"
                    style={{ transform: `rotate(${rotation}deg)` }}
                  />
                )}
                <div className="absolute inset-0 bg-black/50 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center">
                  <ZoomIn className="text-white" size={32} />
                </div>
              </div>

              {/* Actions */}
              <div className="flex gap-2">
                <Button size="sm" variant="outline" onClick={() => handleDownload(doc)}>
                  <Download size={16} className="mr-1" />
                  Descargar
                </Button>
                {isImage && (
                  <Button
                    size="sm"
                    variant="outline"
                    onClick={() => setRotation((r) => r + 90)}
                  >
                    <RotateCw size={16} />
                  </Button>
                )}
              </div>

              {doc.rejectionReason && (
                <p className="text-sm text-destructive mt-2">
                  Razón: {doc.rejectionReason}
                </p>
              )}
            </CardContent>
          </Card>
        );
      })}

      {/* Lightbox for fullscreen view */}
      {selectedDoc && (
        <Lightbox
          open={!!selectedDoc}
          close={() => setSelectedDoc(null)}
          slides={[{ src: selectedDoc.url }]}
        />
      )}
    </div>
  );
};
```

### ReviewActions

```typescript
// src/components/kyc/ReviewActions.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import * as z from 'zod';
import { Button } from '@/components/ui/button';
import { Textarea } from '@/components/ui/textarea';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { CheckCircle, XCircle, AlertCircle } from 'lucide-react';
import { Can } from '@casl/react';
import { useAbility } from '@/hooks/useAbility';

const approvalSchema = z.object({
  notes: z.string().min(10, 'Mínimo 10 caracteres').max(500),
});

const rejectionSchema = z.object({
  reason: z.enum([
    'invalid_document',
    'expired_document',
    'sanctions_match',
    'incomplete_info',
    'fake_document',
    'underage',
    'duplicate_account',
    'other',
  ]),
  notes: z.string().min(20, 'Debe explicar detalladamente la razón del rechazo').max(500),
  requiresRevalidation: z.boolean(),
});

interface ReviewActionsProps {
  applicationId: string;
  onApprove: (notes: string) => void;
  onReject: (reason: string, notes: string, requiresRevalidation: boolean) => void;
}

export const ReviewActions = ({ applicationId, onApprove, onReject }: ReviewActionsProps) => {
  const ability = useAbility();
  const [action, setAction] = useState<'approve' | 'reject' | null>(null);

  const approveForm = useForm({
    resolver: zodResolver(approvalSchema),
  });

  const rejectForm = useForm({
    resolver: zodResolver(rejectionSchema),
    defaultValues: {
      requiresRevalidation: false,
    },
  });

  const handleApprove = approveForm.handleSubmit((data) => {
    onApprove(data.notes);
  });

  const handleReject = rejectForm.handleSubmit((data) => {
    onReject(data.reason, data.notes, data.requiresRevalidation);
  });

  return (
    <Card>
      <CardHeader>
        <CardTitle>Acciones de Revisión</CardTitle>
      </CardHeader>
      <CardContent className="space-y-4">
        {/* Action Selection */}
        {!action && (
          <div className="flex flex-col gap-2">
            <Can I="approve" a="KYC" ability={ability}>
              <Button
                variant="default"
                className="w-full"
                onClick={() => setAction('approve')}
              >
                <CheckCircle className="mr-2" size={16} />
                Aprobar KYC
              </Button>
            </Can>

            <Can I="reject" a="KYC" ability={ability}>
              <Button
                variant="destructive"
                className="w-full"
                onClick={() => setAction('reject')}
              >
                <XCircle className="mr-2" size={16} />
                Rechazar KYC
              </Button>
            </Can>
          </div>
        )}

        {/* Approval Form */}
        {action === 'approve' && (
          <form onSubmit={handleApprove} className="space-y-4">
            <div>
              <label className="text-sm font-medium">Notas de Aprobación</label>
              <Textarea
                {...approveForm.register('notes')}
                placeholder="Describe cualquier observación relevante..."
                rows={4}
              />
              {approveForm.formState.errors.notes && (
                <p className="text-sm text-destructive mt-1">
                  {approveForm.formState.errors.notes.message}
                </p>
              )}
            </div>

            <div className="flex gap-2">
              <Button type="submit" className="flex-1">
                Confirmar Aprobación
              </Button>
              <Button type="button" variant="outline" onClick={() => setAction(null)}>
                Cancelar
              </Button>
            </div>
          </form>
        )}

        {/* Rejection Form */}
        {action === 'reject' && (
          <form onSubmit={handleReject} className="space-y-4">
            <div>
              <label className="text-sm font-medium">Razón del Rechazo</label>
              <Select {...rejectForm.register('reason')}>
                <SelectTrigger>
                  <SelectValue placeholder="Selecciona una razón" />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="invalid_document">Documento inválido</SelectItem>
                  <SelectItem value="expired_document">Documento expirado</SelectItem>
                  <SelectItem value="sanctions_match">Coincidencia con sanciones</SelectItem>
                  <SelectItem value="incomplete_info">Información incompleta</SelectItem>
                  <SelectItem value="fake_document">Documento falso</SelectItem>
                  <SelectItem value="underage">Menor de edad</SelectItem>
                  <SelectItem value="duplicate_account">Cuenta duplicada</SelectItem>
                  <SelectItem value="other">Otro</SelectItem>
                </SelectContent>
              </Select>
            </div>

            <div>
              <label className="text-sm font-medium">Notas Detalladas</label>
              <Textarea
                {...rejectForm.register('notes')}
                placeholder="Explica detalladamente la razón del rechazo..."
                rows={5}
              />
              {rejectForm.formState.errors.notes && (
                <p className="text-sm text-destructive mt-1">
                  {rejectForm.formState.errors.notes.message}
                </p>
              )}
            </div>

            <div className="flex items-center gap-2">
              <input
                type="checkbox"
                id="revalidation"
                {...rejectForm.register('requiresRevalidation')}
              />
              <label htmlFor="revalidation" className="text-sm">
                Permitir revalidación (artesano puede volver a enviar documentos)
              </label>
            </div>

            <div className="flex gap-2">
              <Button type="submit" variant="destructive" className="flex-1">
                Confirmar Rechazo
              </Button>
              <Button type="button" variant="outline" onClick={() => setAction(null)}>
                Cancelar
              </Button>
            </div>
          </form>
        )}
      </CardContent>
    </Card>
  );
};
```

---

## 🔌 Servicios y API

### KYC Service

```typescript
// src/services/kyc.service.ts
import { apiClient } from './api-client';
import type { KYCApplication, KYCStatus } from '@/types/kyc.types';

interface KYCQueueParams {
  status?: KYCStatus[];
  sortBy?: 'createdAt' | 'updatedAt' | 'riskScore';
  order?: 'asc' | 'desc';
  page?: number;
  limit?: number;
}

interface ApproveKYCDto {
  id: string;
  notes: string;
}

interface RejectKYCDto {
  id: string;
  reason: string;
  notes: string;
  requiresRevalidation: boolean;
}

export const kycService = {
  async getQueue(params: KYCQueueParams): Promise<{ data: KYCApplication[]; total: number }> {
    const response = await apiClient.get('/kyc/queue', { params });
    return response.data;
  },

  async getApplication(id: string): Promise<KYCApplication> {
    const response = await apiClient.get(`/kyc/applications/${id}`);
    return response.data;
  },

  async claimApplication(id: string): Promise<KYCApplication> {
    const response = await apiClient.post(`/kyc/applications/${id}/claim`);
    return response.data;
  },

  async approve(dto: ApproveKYCDto): Promise<KYCApplication> {
    const response = await apiClient.post(`/kyc/applications/${dto.id}/approve`, {
      notes: dto.notes,
    });
    return response.data;
  },

  async reject(dto: RejectKYCDto): Promise<KYCApplication> {
    const response = await apiClient.post(`/kyc/applications/${dto.id}/reject`, {
      reason: dto.reason,
      notes: dto.notes,
      requiresRevalidation: dto.requiresRevalidation,
    });
    return response.data;
  },

  async triggerSanctionsCheck(id: string): Promise<SanctionsCheckResult> {
    const response = await apiClient.post(`/kyc/applications/${id}/sanctions-check`);
    return response.data;
  },

  async forceRevalidation(id: string, reason: string): Promise<void> {
    await apiClient.post(`/kyc/applications/${id}/force-revalidation`, { reason });
  },
};
```

### React Query Hooks

```typescript
// src/hooks/queries/use-kyc-queue.ts
import { useQuery } from '@tanstack/react-query';
import { kycService } from '@/services/kyc.service';

export const useKYCQueue = (params: KYCQueueParams) => {
  return useQuery({
    queryKey: ['kyc', 'queue', params],
    queryFn: () => kycService.getQueue(params),
    staleTime: 30 * 1000, // 30 seconds
    refetchInterval: 60 * 1000, // Auto-refetch every minute
  });
};

export const useKYCApplication = (id: string) => {
  return useQuery({
    queryKey: ['kyc', 'application', id],
    queryFn: () => kycService.getApplication(id),
    enabled: !!id,
  });
};

export const useApproveKYC = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: kycService.approve,
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: ['kyc'] });
      toast.success(`KYC aprobado para ${data.personalInfo.fullName}`);
    },
    onError: (error) => {
      toast.error(`Error al aprobar: ${error.message}`);
    },
  });
};

export const useRejectKYC = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: kycService.reject,
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: ['kyc'] });
      toast.success(`KYC rechazado para ${data.personalInfo.fullName}`);
    },
    onError: (error) => {
      toast.error(`Error al rechazar: ${error.message}`);
    },
  });
};
```

---

## 🔐 Seguridad y Auditoría

### Auditoría Automática

```typescript
// Cada acción de KYC se audita automáticamente
POST /api/v1/audit-logs
{
  userId: reviewer.id,
  action: 'kyc.approved',
  resource: 'kyc_application',
  resourceId: application.id,
  metadata: {
    artisanId: application.artisanId,
    previousStatus: 'in_review',
    newStatus: 'approved',
    notes: 'Documentos válidos...',
  },
  ipAddress: req.ip,
  userAgent: req.headers['user-agent'],
  timestamp: new Date(),
}
```

### Permisos CASL

```typescript
// src/lib/permissions/ability.ts
import { defineAbility } from '@casl/ability';

export const defineKYCAbility = (user) => {
  return defineAbility((can, cannot) => {
    if (user.role === 'kyc.reviewer') {
      can('read', 'KYC');
      can('claim', 'KYC');
      can('approve', 'KYC');
      can('reject', 'KYC');
      can('add_note', 'KYC');
      cannot('delete', 'KYC');
      cannot('force_revalidation', 'KYC');
    }

    if (user.role === 'risk.analyst') {
      can('manage', 'KYC'); // Todo lo anterior +
      can('force_revalidation', 'KYC');
      can('view_history', 'KYC');
    }

    if (user.role === 'superadmin') {
      can('manage', 'all');
    }
  });
};
```

---

## ✅ Testing

### Unit Test - DocumentViewer

```typescript
// src/components/kyc/DocumentViewer.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { DocumentViewer } from './DocumentViewer';
import type { KYCDocument } from '@/types/kyc.types';

describe('DocumentViewer', () => {
  const mockDocuments: KYCDocument[] = [
    {
      id: '1',
      type: DocumentType.NATIONAL_ID_FRONT,
      url: 'https://example.com/doc1.jpg',
      uploadedAt: new Date(),
      status: 'pending',
    },
  ];

  it('should render documents', () => {
    render(<DocumentViewer documents={mockDocuments} />);
    expect(screen.getByText('ID Nacional (Frente)')).toBeInTheDocument();
  });

  it('should handle download', async () => {
    global.fetch = vi.fn(() =>
      Promise.resolve({
        blob: () => Promise.resolve(new Blob()),
      })
    ) as any;

    render(<DocumentViewer documents={mockDocuments} />);
    const downloadButton = screen.getByText('Descargar');
    fireEvent.click(downloadButton);

    expect(global.fetch).toHaveBeenCalledWith(mockDocuments[0].url);
  });
});
```

---

## 📚 Recursos Adicionales

- [Estándares de KYC](https://www.fatf-gafi.org/)
- [ComplyAdvantage API](https://docs.complyadvantage.com/)
- [react-pdf Docs](https://github.com/wojtekmaj/react-pdf)
- [CASL Docs](https://casl.js.org/v6/en/)

---

**Elaborado por:** Claude Code (Sonnet 4.5)
**Fecha:** 2025-01-05
**Revisión:** Pendiente
