# 🔐 Seguridad y Buenas Prácticas - bMOI Admin

> **Guía completa de seguridad avanzada, RBAC/ABAC, MFA y buenas prácticas de programación para bMOI Admin**

**Prioridad:** 🔴 P0 - CRÍTICO
**Audiencia:** Todos los desarrolladores del equipo frontend bMOI Admin

---

## 📋 Índice

1. [Principios de Seguridad](#principios-de-seguridad)
2. [MFA/2FA Implementation](#mfa2fa-implementation)
3. [RBAC/ABAC - Permisos Granulares](#rbacabac---permisos-granulares)
4. [Auditoría Completa](#auditoría-completa)
5. [Session Management](#session-management)
6. [Zero-PAN y PII Protection](#zero-pan-y-pii-protection)
7. [IP Allowlist](#ip-allowlist)
8. [Buenas Prácticas de Código](#buenas-prácticas-de-código)
9. [Restricciones y Convenciones](#restricciones-y-convenciones)
10. [Testing de Seguridad](#testing-de-seguridad)

---

## 🛡️ Principios de Seguridad

### Mínimo Privilegio (Principle of Least Privilege)

```typescript
// ❌ MAL - Dar permisos amplios por defecto
if (user.role === 'staff') {
  can('manage', 'all'); // Demasiado permisivo
}

// ✅ BIEN - Solo los permisos necesarios
if (user.role === 'kyc.reviewer') {
  can('read', 'KYC');
  can('approve', 'KYC');
  can('reject', 'KYC');
  cannot('delete', 'KYC');
  cannot('export', 'KYC'); // Explícitamente prohibido
}
```

### Segregación de Funciones (SoD)

**Regla:** Un mismo usuario NO puede aprobar Y ejecutar una acción crítica.

```typescript
// Ejemplo: Reembolsos de disputas
// El que aprueba el reembolso NO puede ejecutarlo

// Usuario A (risk.analyst) → Aprueba reembolso
can('approve_refund', 'Dispute');

// Usuario B (finance.admin) → Ejecuta reembolso
can('execute_refund', 'Dispute');

// Nadie tiene ambos permisos
```

### Defense in Depth (Capas de Seguridad)

```
Frontend → Backend → DB
   ↓          ↓        ↓
Permisos   Permisos   RLS
CASL       Guards    Policies
```

**Nunca confiar solo en frontend:**
```typescript
// ❌ MAL - Solo ocultar botón
<Can I="delete" a="User">
  <Button onClick={handleDelete}>Delete</Button>
</Can>

// ✅ BIEN - Ocultar Y validar en backend
<Can I="delete" a="User">
  <Button onClick={handleDelete}>Delete</Button> {/* Frontend */}
</Can>
// Backend SIEMPRE valida permisos también
```

---

## 🔑 MFA/2FA Implementation

### Flujo Completo

```
1. User logs in with email/password
2. Backend validates credentials
3. If MFA enabled → Return { requiresMFA: true, challenge: session_id }
4. Frontend shows TOTP input
5. User enters 6-digit code
6. Backend validates TOTP
7. Return access_token + refresh_token
```

### Setup MFA (Primera vez)

```typescript
// src/pages/settings/MFASetupPage.tsx
import QRCode from 'qrcode.react';
import { TOTP } from 'otpauth';

export const MFASetupPage = () => {
  const { data: user } = useAuthUser();
  const setupMutation = useSetupMFA();
  const [secret, setSecret] = useState<string>('');
  const [qrCodeUrl, setQRCodeUrl] = useState<string>('');
  const [backupCodes, setBackupCodes] = useState<string[]>([]);

  const handleGenerateSecret = async () => {
    // 1. Generate secret
    const totp = new TOTP({
      issuer: 'bMOI Admin',
      label: user.email,
      algorithm: 'SHA1',
      digits: 6,
      period: 30,
    });

    const generatedSecret = totp.secret.base32;
    setSecret(generatedSecret);

    // 2. Generate QR Code URL
    const otpauthUrl = totp.toString();
    setQRCodeUrl(otpauthUrl);

    // 3. Generate backup codes (backend)
    const response = await setupMutation.mutateAsync({ secret: generatedSecret });
    setBackupCodes(response.backupCodes);
  };

  const handleVerifyAndEnable = async (code: string) => {
    // Verify TOTP code before enabling
    await setupMutation.mutateAsync({
      secret,
      verificationCode: code,
    });

    toast.success('MFA habilitada correctamente');
  };

  return (
    <Card>
      <CardHeader>
        <CardTitle>Configurar Autenticación de Dos Factores (2FA)</CardTitle>
      </CardHeader>
      <CardContent className="space-y-6">
        {/* Step 1: Generate Secret */}
        {!secret && (
          <Button onClick={handleGenerateSecret}>Generar Código QR</Button>
        )}

        {/* Step 2: Scan QR */}
        {secret && (
          <>
            <div className="flex flex-col items-center space-y-4">
              <p className="text-sm text-muted-foreground">
                Escanea este código QR con tu app de autenticación (Google Authenticator, Authy, etc.)
              </p>
              <QRCode value={qrCodeUrl} size={200} />

              <div className="text-center">
                <p className="text-xs text-muted-foreground">O ingresa manualmente:</p>
                <code className="text-sm bg-muted px-2 py-1 rounded">{secret}</code>
              </div>
            </div>

            {/* Step 3: Verify Code */}
            <div>
              <Label>Código de Verificación</Label>
              <Input
                placeholder="123456"
                maxLength={6}
                onChange={(e) => {
                  if (e.target.value.length === 6) {
                    handleVerifyAndEnable(e.target.value);
                  }
                }}
              />
            </div>

            {/* Step 4: Backup Codes */}
            {backupCodes.length > 0 && (
              <Alert>
                <AlertCircle className="h-4 w-4" />
                <AlertTitle>Códigos de Recuperación</AlertTitle>
                <AlertDescription>
                  Guarda estos códigos en un lugar seguro. Los necesitarás si pierdes acceso a tu app de autenticación.
                  <div className="grid grid-cols-2 gap-2 mt-4">
                    {backupCodes.map((code, i) => (
                      <code key={i} className="text-sm bg-muted px-2 py-1 rounded">
                        {code}
                      </code>
                    ))}
                  </div>
                </AlertDescription>
              </Alert>
            )}
          </>
        )}
      </CardContent>
    </Card>
  );
};
```

### MFA Login Flow

```typescript
// src/pages/auth/MFAVerificationPage.tsx
export const MFAVerificationPage = () => {
  const navigate = useNavigate();
  const { sessionId } = useLocation().state; // From login page
  const verifyMutation = useVerifyMFA();

  const form = useForm({
    resolver: zodResolver(
      z.object({
        code: z.string().length(6, 'Código debe tener 6 dígitos'),
      })
    ),
  });

  const handleVerify = form.handleSubmit(async (data) => {
    try {
      const response = await verifyMutation.mutateAsync({
        sessionId,
        code: data.code,
      });

      // Store tokens
      localStorage.setItem('access_token', response.accessToken);
      localStorage.setItem('refresh_token', response.refreshToken);

      navigate('/dashboard');
    } catch (error) {
      toast.error('Código inválido');
    }
  });

  return (
    <Card>
      <CardHeader>
        <CardTitle>Verificación en Dos Pasos</CardTitle>
        <CardDescription>Ingresa el código de tu app de autenticación</CardDescription>
      </CardHeader>
      <CardContent>
        <form onSubmit={handleVerify} className="space-y-4">
          <Input
            {...form.register('code')}
            placeholder="123456"
            maxLength={6}
            autoFocus
            className="text-center text-2xl tracking-widest"
          />
          <Button type="submit" className="w-full">
            Verificar
          </Button>
        </form>
      </CardContent>
    </Card>
  );
};
```

### MFA Backend Service

```typescript
// src/services/mfa.service.ts
import { TOTP } from 'otpauth';

export const mfaService = {
  async setupMFA(secret: string, verificationCode: string): Promise<{ backupCodes: string[] }> {
    const response = await apiClient.post('/auth/mfa/setup', {
      secret,
      verificationCode,
    });
    return response.data;
  },

  async verifyMFA(sessionId: string, code: string): Promise<{ accessToken: string; refreshToken: string }> {
    const response = await apiClient.post('/auth/mfa/verify', {
      sessionId,
      code,
    });
    return response.data;
  },

  async disableMFA(password: string): Promise<void> {
    await apiClient.post('/auth/mfa/disable', { password });
  },

  // Validate locally before sending to backend (UX optimization)
  validateTOTP(secret: string, token: string): boolean {
    const totp = new TOTP({ secret: TOTP.Secret.fromBase32(secret) });
    return totp.validate({ token, window: 1 }) !== null;
  },
};
```

---

## 🎭 RBAC/ABAC - Permisos Granulares

### Roles Definidos para bMOI Admin

| Rol | Descripción | Permisos |
|-----|-------------|----------|
| **superadmin** | Acceso total | `manage all` |
| **ops.viewer** | Operaciones (solo lectura) | `read all`, `cannot create/update/delete` |
| **kyc.reviewer** | Revisor KYC | `read KYC`, `approve KYC`, `reject KYC` |
| **kyc.senior** | KYC Senior + forzar revalidación | Todo lo anterior + `force_revalidation KYC` |
| **risk.analyst** | Analista de Riesgo | `manage KYC`, `read Disputes`, `approve_refund Disputes` |
| **moderation.reviewer** | Moderador de Contenido | `read Moderation`, `approve Content`, `reject Content` |
| **finance.admin** | Administrador Finanzas | `read Orders`, `execute_refund Disputes`, `export Financial` |
| **compliance.officer** | Oficial de Cumplimiento | `read all`, `configure Policies`, `export AuditLogs` |
| **support.agent** | Agente de Soporte | `read Users`, `read Orders`, `update Users (limited)` |

### CASL Ability Definition

```typescript
// src/lib/permissions/ability.ts
import { AbilityBuilder, createMongoAbility } from '@casl/ability';

export type Actions = 'manage' | 'create' | 'read' | 'update' | 'delete' | 'approve' | 'reject' | 'export' | 'execute_refund' | 'force_revalidation';
export type Subjects = 'all' | 'KYC' | 'Moderation' | 'Dispute' | 'User' | 'Order' | 'AuditLog' | 'Policy';

export type AppAbility = MongoAbility<[Actions, Subjects]>;

export const defineAbilityFor = (user: { role: string; permissions?: string[] }) => {
  const { can, cannot, build } = new AbilityBuilder<AppAbility>(createMongoAbility);

  // Superadmin
  if (user.role === 'superadmin') {
    can('manage', 'all');
    return build();
  }

  // Ops Viewer
  if (user.role === 'ops.viewer') {
    can('read', 'all');
    cannot('create', 'all');
    cannot('update', 'all');
    cannot('delete', 'all');
    cannot('approve', 'all');
    cannot('reject', 'all');
  }

  // KYC Reviewer
  if (user.role === 'kyc.reviewer') {
    can('read', 'KYC');
    can('approve', 'KYC');
    can('reject', 'KYC');
    cannot('delete', 'KYC');
    cannot('force_revalidation', 'KYC');
  }

  // KYC Senior
  if (user.role === 'kyc.senior') {
    can('read', 'KYC');
    can('approve', 'KYC');
    can('reject', 'KYC');
    can('force_revalidation', 'KYC');
    cannot('delete', 'KYC');
  }

  // Risk Analyst
  if (user.role === 'risk.analyst') {
    can('manage', 'KYC');
    can('read', 'Dispute');
    can('approve_refund', 'Dispute'); // Solo aprueba, no ejecuta
    cannot('execute_refund', 'Dispute'); // SoD
  }

  // Moderation Reviewer
  if (user.role === 'moderation.reviewer') {
    can('read', 'Moderation');
    can('approve', 'Moderation');
    can('reject', 'Moderation');
    cannot('delete', 'Moderation');
  }

  // Finance Admin
  if (user.role === 'finance.admin') {
    can('read', 'Order');
    can('read', 'Dispute');
    can('execute_refund', 'Dispute'); // Solo ejecuta, no aprueba (SoD)
    can('export', 'Order');
    cannot('approve_refund', 'Dispute'); // SoD
  }

  // Compliance Officer
  if (user.role === 'compliance.officer') {
    can('read', 'all');
    can('export', 'AuditLog');
    can('update', 'Policy');
    cannot('create', 'User');
    cannot('delete', 'User');
  }

  // Support Agent
  if (user.role === 'support.agent') {
    can('read', 'User');
    can('read', 'Order');
    can('update', 'User', ['email', 'phone']); // Solo campos específicos
    cannot('update', 'User', ['role', 'permissions']); // Campos críticos prohibidos
  }

  return build();
};
```

### Uso en Componentes

```typescript
// src/components/kyc/KYCActions.tsx
import { Can } from '@casl/react';
import { useAbility } from '@/hooks/useAbility';

export const KYCActions = ({ application }: { application: KYCApplication }) => {
  const ability = useAbility();

  return (
    <div className="flex gap-2">
      <Can I="approve" a="KYC" ability={ability}>
        <Button onClick={handleApprove}>Aprobar</Button>
      </Can>

      <Can I="reject" a="KYC" ability={ability}>
        <Button variant="destructive" onClick={handleReject}>
          Rechazar
        </Button>
      </Can>

      <Can I="force_revalidation" a="KYC" ability={ability}>
        <Button variant="outline" onClick={handleForceRevalidation}>
          Forzar Revalidación
        </Button>
      </Can>

      {/* Este botón solo lo ven superadmins */}
      <Can I="delete" a="KYC" ability={ability}>
        <Button variant="destructive" onClick={handleDelete}>
          Eliminar Permanentemente
        </Button>
      </Can>
    </div>
  );
};
```

### Protección de Rutas

```typescript
// src/router/protected-route.tsx
import { Navigate } from 'react-router-dom';
import { useAbility } from '@/hooks/useAbility';

interface ProtectedRouteProps {
  children: React.ReactNode;
  action: Actions;
  subject: Subjects;
  redirectTo?: string;
}

export const ProtectedRoute = ({
  children,
  action,
  subject,
  redirectTo = '/unauthorized',
}: ProtectedRouteProps) => {
  const ability = useAbility();

  if (!ability.can(action, subject)) {
    return <Navigate to={redirectTo} replace />;
  }

  return <>{children}</>;
};

// Uso en router
{
  path: 'kyc',
  element: (
    <ProtectedRoute action="read" subject="KYC">
      <KYCPage />
    </ProtectedRoute>
  ),
}
```

---

## 📝 Auditoría Completa

### Qué se Audita

```typescript
// src/types/audit.types.ts
export interface AuditLog {
  id: string;
  userId: string;
  userEmail: string;
  userRole: string;
  action: string; // e.g., 'kyc.approved', 'user.updated', 'refund.executed'
  resource: string; // e.g., 'kyc_application', 'user', 'dispute'
  resourceId: string;
  metadata: Record<string, any>; // Detalles específicos de la acción
  ipAddress: string;
  userAgent: string;
  timestamp: Date;
  tenantId?: string; // Si aplica
}

// Ejemplos de acciones auditadas:
'auth.login'
'auth.logout'
'auth.mfa_enabled'
'kyc.approved'
'kyc.rejected'
'moderation.content_approved'
'dispute.refund_approved'
'dispute.refund_executed'
'user.created'
'user.role_changed'
'config.policy_updated'
'export.audit_logs'
```

### Interceptor de Auditoría

```typescript
// src/services/api-client.ts
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
});

// Request interceptor - Agregar headers
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('access_token');
  const tenantId = localStorage.getItem('selected_tenant_id');

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  if (tenantId) {
    config.headers['X-Tenant-ID'] = tenantId;
  }

  // Agregar user agent y timestamp (para auditoría)
  config.headers['X-User-Agent'] = navigator.userAgent;
  config.headers['X-Request-Timestamp'] = new Date().toISOString();

  return config;
});

// Response interceptor - Log errores (opcional)
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    // Log failed requests para debugging
    console.error('[API Error]', {
      url: error.config?.url,
      method: error.config?.method,
      status: error.response?.status,
      message: error.response?.data?.message,
    });

    // Si 401, redirigir a login
    if (error.response?.status === 401) {
      localStorage.removeItem('access_token');
      localStorage.removeItem('refresh_token');
      window.location.href = '/login';
    }

    return Promise.reject(error);
  }
);
```

### Página de Auditoría

```typescript
// src/pages/audit/AuditLogsPage.tsx
export const AuditLogsPage = () => {
  const [filters, setFilters] = useQueryParams({
    userId: StringParam,
    action: StringParam,
    resource: StringParam,
    dateFrom: DateParam,
    dateTo: DateParam,
    page: NumberParam,
  });

  const { data, isLoading } = useAuditLogs(filters);

  return (
    <div className="space-y-4">
      <Card>
        <CardHeader>
          <CardTitle>Audit Logs</CardTitle>
          <CardDescription>
            Historial completo de acciones realizadas por usuarios del sistema
          </CardDescription>
        </CardHeader>
        <CardContent>
          <AuditLogsFilters filters={filters} onChange={setFilters} />
          <AuditLogsTable logs={data?.logs} isLoading={isLoading} />
          <Pagination
            currentPage={filters.page || 1}
            totalPages={data?.totalPages}
            onPageChange={(page) => setFilters({ page })}
          />
        </CardContent>
      </Card>
    </div>
  );
};
```

---

## ⏱️ Session Management

### Configuración

```typescript
// .env
VITE_SESSION_TIMEOUT=30 # minutos
VITE_MAX_IDLE_TIME=15 # minutos sin actividad
VITE_MAX_LOGIN_ATTEMPTS=5
VITE_LOCKOUT_DURATION=15 # minutos
```

### Implementación

```typescript
// src/hooks/useSessionTimeout.ts
import { useEffect, useRef } from 'react';
import { useAuthStore } from '@/stores/auth.store';

const SESSION_TIMEOUT = Number(import.meta.env.VITE_SESSION_TIMEOUT) * 60 * 1000; // ms
const IDLE_TIMEOUT = Number(import.meta.env.VITE_MAX_IDLE_TIME) * 60 * 1000; // ms

export const useSessionTimeout = () => {
  const logout = useAuthStore((state) => state.logout);
  const sessionTimerRef = useRef<NodeJS.Timeout>();
  const idleTimerRef = useRef<NodeJS.Timeout>();

  const resetIdleTimer = () => {
    if (idleTimerRef.current) clearTimeout(idleTimerRef.current);

    idleTimerRef.current = setTimeout(() => {
      toast.warning('Sesión cerrada por inactividad');
      logout();
    }, IDLE_TIMEOUT);
  };

  useEffect(() => {
    // Session timeout (tiempo máximo de sesión)
    sessionTimerRef.current = setTimeout(() => {
      toast.error('Sesión expirada. Por favor inicia sesión nuevamente.');
      logout();
    }, SESSION_TIMEOUT);

    // Idle timeout (inactividad del usuario)
    const events = ['mousedown', 'keydown', 'scroll', 'touchstart'];
    events.forEach((event) => {
      document.addEventListener(event, resetIdleTimer);
    });

    resetIdleTimer();

    return () => {
      if (sessionTimerRef.current) clearTimeout(sessionTimerRef.current);
      if (idleTimerRef.current) clearTimeout(idleTimerRef.current);
      events.forEach((event) => {
        document.removeEventListener(event, resetIdleTimer);
      });
    };
  }, []);
};

// Uso en App.tsx
export const App = () => {
  const isAuthenticated = useAuthStore((state) => state.isAuthenticated);

  useSessionTimeout(); // Auto-logout por timeout o inactividad

  return <RouterProvider router={router} />;
};
```

---

## 🔒 Zero-PAN y PII Protection

### Regla: NUNCA mostrar datos completos de pago

```typescript
// ❌ MAL - Mostrar número de tarjeta completo
<p>Tarjeta: 4532 1234 5678 9010</p>

// ✅ BIEN - Masked
<p>Tarjeta: •••• •••• •••• 9010</p>

// Utility
export const maskCardNumber = (cardNumber: string): string => {
  const last4 = cardNumber.slice(-4);
  return `•••• •••• •••• ${last4}`;
};
```

### PII (Personally Identifiable Information)

```typescript
// src/lib/pii-protection.ts

// Nunca loggear PII en consola o Sentry
export const sanitizeLogs = (data: any): any => {
  const piiFields = ['email', 'phone', 'address', 'ssn', 'cardNumber', 'cvv'];

  if (typeof data !== 'object') return data;

  const sanitized = { ...data };

  for (const key of Object.keys(sanitized)) {
    if (piiFields.includes(key)) {
      sanitized[key] = '[REDACTED]';
    } else if (typeof sanitized[key] === 'object') {
      sanitized[key] = sanitizeLogs(sanitized[key]);
    }
  }

  return sanitized;
};

// Uso en error logging
try {
  await createUser(userData);
} catch (error) {
  console.error('Error creating user:', sanitizeLogs(error));
  Sentry.captureException(error, {
    contexts: {
      user: sanitizeLogs(userData),
    },
  });
}
```

---

## 🌐 IP Allowlist

### Frontend: Warning

```typescript
// src/hooks/useIPCheck.ts
export const useIPCheck = () => {
  const { data: ipInfo, isLoading } = useQuery({
    queryKey: ['ip-check'],
    queryFn: async () => {
      const response = await apiClient.get('/auth/ip-check');
      return response.data;
    },
    retry: false,
  });

  useEffect(() => {
    if (!isLoading && ipInfo?.allowed === false) {
      toast.error(
        'Tu dirección IP no está en la lista permitida. Contacta al administrador.'
      );
      // Opcionalmente, redirigir o bloquear acceso
    }
  }, [ipInfo, isLoading]);
};

// Uso en App.tsx (solo para staff interno)
export const App = () => {
  useIPCheck(); // Valida IP al cargar la app

  return <RouterProvider router={router} />;
};
```

### Backend: Implementación

```typescript
// Backend middleware (NestJS example)
@Injectable()
export class IPAllowlistGuard implements CanActivate {
  private readonly allowedIPs = process.env.ALLOWED_IPS.split(',');

  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const clientIP = request.ip || request.headers['x-forwarded-for'];

    const isAllowed = this.allowedIPs.includes(clientIP);

    if (!isAllowed) {
      this.logger.warn(`Unauthorized IP attempt: ${clientIP}`);
      throw new ForbiddenException('IP not allowed');
    }

    return true;
  }
}
```

---

## 📐 Buenas Prácticas de Código

### 1. TypeScript Strict - SIEMPRE

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "strict": true, // ✅ OBLIGATORIO
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

### 2. Nunca usar `any`

```typescript
// ❌ MAL
const handleSubmit = (data: any) => {
  console.log(data.email); // No type-safety
};

// ✅ BIEN
interface FormData {
  email: string;
  password: string;
}

const handleSubmit = (data: FormData) => {
  console.log(data.email); // Type-safe
};
```

### 3. Validación con Zod - SIEMPRE

```typescript
// ❌ MAL - Sin validación
const formData = form.getValues();
await apiClient.post('/users', formData); // Puede enviar data inválida

// ✅ BIEN - Validación con Zod
const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  role: z.enum(['admin', 'user']),
});

const formData = createUserSchema.parse(form.getValues());
await apiClient.post('/users', formData); // Data garantizada válida
```

### 4. Error Handling Exhaustivo

```typescript
// ❌ MAL - No manejar errores
const { data } = await apiClient.get('/users');

// ✅ BIEN - Manejar todos los casos
try {
  const { data } = await apiClient.get('/users');
  return data;
} catch (error) {
  if (axios.isAxiosError(error)) {
    if (error.response?.status === 404) {
      toast.error('Usuario no encontrado');
    } else if (error.response?.status === 403) {
      toast.error('No tienes permisos para ver esto');
    } else {
      toast.error('Error al obtener usuarios');
    }
  } else {
    Sentry.captureException(error);
    toast.error('Error inesperado');
  }
}
```

### 5. React Query - Configuración Correcta

```typescript
// src/lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 min
      cacheTime: 10 * 60 * 1000, // 10 min
      retry: 1, // Solo 1 retry
      refetchOnWindowFocus: false, // No refetch al cambiar de tab
      refetchOnReconnect: true, // Refetch al reconectar
      // ✅ IMPORTANTE: Error handling global
      onError: (error) => {
        console.error('[React Query Error]', sanitizeLogs(error));
      },
    },
    mutations: {
      retry: 0, // No retry en mutations
      onError: (error) => {
        console.error('[React Query Mutation Error]', sanitizeLogs(error));
        toast.error('Error al realizar la operación');
      },
    },
  },
});
```

---

## 🚫 Restricciones y Convenciones

### ❌ PROHIBIDO

1. **`console.log` en producción**
   ```typescript
   // ❌ MAL
   console.log('User data:', user);

   // ✅ BIEN - Usar logger condicional
   if (import.meta.env.DEV) {
     console.log('User data:', sanitizeLogs(user));
   }
   ```

2. **`any` type**
   ```typescript
   // ❌ PROHIBIDO
   const data: any = response.data;

   // ✅ BIEN
   interface User {
     id: string;
     email: string;
   }
   const data: User = response.data;
   ```

3. **Inline functions en JSX**
   ```typescript
   // ❌ MAL - Re-crea función en cada render
   <Button onClick={() => handleClick(id)}>Click</Button>

   // ✅ BIEN - useCallback
   const handleButtonClick = useCallback(() => handleClick(id), [id]);
   <Button onClick={handleButtonClick}>Click</Button>
   ```

4. **Mutación directa de estado**
   ```typescript
   // ❌ MAL
   user.email = 'new@email.com';
   setUser(user);

   // ✅ BIEN - Inmutabilidad
   setUser({ ...user, email: 'new@email.com' });
   ```

5. **Hardcoded secrets o URLs**
   ```typescript
   // ❌ PROHIBIDO
   const API_KEY = 'sk_live_1234567890';

   // ✅ BIEN
   const API_KEY = import.meta.env.VITE_API_KEY;
   ```

---

## ✅ Testing de Seguridad

### Test - RBAC Permissions

```typescript
// src/lib/permissions/ability.test.ts
import { describe, it, expect } from 'vitest';
import { defineAbilityFor } from './ability';

describe('RBAC Permissions', () => {
  it('kyc.reviewer should only approve/reject, not delete', () => {
    const ability = defineAbilityFor({ role: 'kyc.reviewer' });

    expect(ability.can('approve', 'KYC')).toBe(true);
    expect(ability.can('reject', 'KYC')).toBe(true);
    expect(ability.can('delete', 'KYC')).toBe(false);
    expect(ability.can('force_revalidation', 'KYC')).toBe(false);
  });

  it('finance.admin should execute refunds but not approve them (SoD)', () => {
    const ability = defineAbilityFor({ role: 'finance.admin' });

    expect(ability.can('execute_refund', 'Dispute')).toBe(true);
    expect(ability.can('approve_refund', 'Dispute')).toBe(false); // SoD
  });

  it('ops.viewer should only read, never modify', () => {
    const ability = defineAbilityFor({ role: 'ops.viewer' });

    expect(ability.can('read', 'all')).toBe(true);
    expect(ability.can('create', 'User')).toBe(false);
    expect(ability.can('update', 'User')).toBe(false);
    expect(ability.can('delete', 'User')).toBe(false);
  });
});
```

---

## 📚 Recursos Adicionales

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CASL Documentation](https://casl.js.org/)
- [TOTP/HOTP RFC 6238](https://datatracker.ietf.org/doc/html/rfc6238)
- [TypeScript Strict Mode](https://www.typescriptlang.org/tsconfig#strict)

---

**Elaborado por:** Claude Code (Sonnet 4.5)
**Fecha:** 2025-01-05
**Revisión:** Pendiente
