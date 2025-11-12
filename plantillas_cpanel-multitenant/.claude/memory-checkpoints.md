# Sistema de Memoria y Checkpoints

> **Sistema Anti-Olvido para Asistentes de IA**

---

## 🧠 ¿Por Qué Este Sistema?

Las IAs pueden "olvidar" contexto en conversaciones largas. Este sistema asegura que siempre recuerdes:

1. Qué estamos construyendo
2. Qué convenciones seguimos
3. Dónde quedamos
4. Qué sigue

---

## ⏰ Checkpoint Cada 5 Mensajes

### Trigger

Cuando veas este mensaje o hayas respondido 5 veces:

```
🧠 CHECKPOINT DE MEMORIA
```

### Verificación Automática

```markdown
## Checkpoint de Memoria - [FECHA]

### ✅ Información del Proyecto
- [ ] Nombre: bMOI CPanel Multi-Tenant BO
- [ ] Logo: bMOI
- [ ] Stack: React 18 + TypeScript 5 + Vite 5
- [ ] Arquitectura: Multi-Tenant con aislamiento de datos

### ✅ Convenciones OBLIGATORIAS
- [ ] Variables: camelCase
- [ ] Componentes: PascalCase
- [ ] Constantes: UPPER_SNAKE_CASE
- [ ] Interfaces: I + PascalCase
- [ ] Hooks: use + camelCase
- [ ] ❌ PROHIBIDO: snake_case en JS/TS
- [ ] ❌ PROHIBIDO: any type

### ✅ Reglas Multi-Tenant
- [ ] Siempre verificar `currentTenant`
- [ ] Query keys incluyen `tenant.id`
- [ ] Validar permisos con `hasPermission()`
- [ ] Verificar límites con `isWithinLimits()`
- [ ] Headers: `X-Tenant-ID` automático

### ✅ Paleta de Colores
- [ ] Primary: #2563EB (Blue-600)
- [ ] Secondary: #7C3AED (Violet-600)
- [ ] Success: #059669
- [ ] Warning: #D97706
- [ ] Danger: #DC2626

### ✅ Estado Actual (Actualizar)
- Última feature: ___________
- Componentes creados hoy: ___________
- Próximo paso: ___________
```

---

## 📝 Checkpoint de Sesión (Fin del Día)

### Al Finalizar

Actualizar `session-state.md` con:

```markdown
## Última Actualización
- Fecha: YYYY-MM-DD HH:mm
- IA: Claude Code
- Fase: Fase X

## ✅ Completado Hoy
- [x] Feature X implementada
- [x] Componente Y creado
- [x] Tests de Z escritos

## 📊 Métricas
- Componentes creados: X
- Tests escritos: Y
- Cobertura: Z%

## 🎯 Decisiones Tomadas
1. Decisión técnica importante
   - Razón: ...
   - Alternativas: ...

## ⚠️ Problemas Encontrados
- Problema X: Solución Y

## 🔜 Próximos Pasos
- [ ] Tarea pendiente 1
- [ ] Tarea pendiente 2

## 📝 Notas para Mañana
- Contexto importante
- Recordatorios
```

---

## 🔄 Recordatorios Automáticos

### Frases Trigger para Revisar Contexto

Si el usuario dice:

| Frase | Acción |
|-------|--------|
| "Recuérdame..." | Leer `context.md` completo |
| "¿Cuáles son las convenciones?" | Mostrar `conventions.md` |
| "¿Cómo funciona multi-tenant?" | Explicar flujo de tenant |
| "Empecemos de nuevo" | Leer `START.md` y `PROJECT_INSTRUCTIONS.md` |
| "Estado del proyecto" | Mostrar `session-state.md` |

---

## ✅ Checklist Antes de Responder

Antes de **cada respuesta**, verifica:

- [ ] ¿Estoy usando las convenciones correctas? (camelCase, PascalCase)
- [ ] ¿He verificado el tenant context si es código con API?
- [ ] ¿Los tipos están correctamente definidos? (NO any)
- [ ] ¿He incluido validaciones multi-tenant?
- [ ] ¿El código seguiría pasando `npm run validate`?

---

## 📊 Historial de Checkpoints

### [2025-11-03 15:00] - Setup Inicial
- ✅ Proyecto configurado
- ✅ Documentación creada
- ✅ Estructura de carpetas definida
- 🔜 Próximo: Implementar autenticación multi-tenant

### [FECHA PRÓXIMO CHECKPOINT]
- Última feature: ___________
- Estado: ___________
- Próximo: ___________

---

## 🆘 Recuperación de Contexto Perdido

### Si perdiste el contexto

1. **Lee inmediatamente:**
   - `PROJECT_INSTRUCTIONS.md`
   - `.claude/context.md`
   - `.claude/conventions.md`
   - `.claude/session-state.md`

2. **Confirma que recuerdas:**
   - Nombre del proyecto: **bMOI CPanel Multi-Tenant BO**
   - Convenciones: **camelCase, PascalCase, NO snake_case**
   - Multi-tenant: **Verificar currentTenant siempre**
   - Paleta: **Primary #2563EB, Secondary #7C3AED**

3. **Pregunta al usuario:**
   ```
   He revisado el contexto del proyecto. Confirmo:
   - Proyecto: bMOI Multi-Tenant BO
   - Convenciones: camelCase, PascalCase, UPPER_SNAKE_CASE
   - Multi-tenant: Validación de tenant obligatoria
   - NO snake_case, NO any type

   ¿En qué estábamos trabajando?
   ```

---

## 🎯 Métricas de Memoria

### Efectividad del Sistema

| Métrica | Objetivo | Actual |
|---------|----------|--------|
| Recordar proyecto | 100% | -% |
| Seguir convenciones | 100% | -% |
| Validar tenant | 100% | -% |
| NO usar snake_case | 100% | -% |
| NO usar any | 95% | -% |

---

<div align="center">

**Este sistema asegura coherencia entre sesiones**

Usa checkpoints regularmente

</div>
