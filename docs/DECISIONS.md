# GYM APP — Architecture Decisions

# ADR-001 — Frontend desacoplado del backend

## Decisión

El frontend será desarrollado con Next.js y consumirá una API REST desarrollada con NestJS.

```text
Next.js
   ↓
NestJS
```

## Motivo

Separar responsabilidades y permitir evolución independiente de frontend y backend.

## Consecuencia

El frontend no debe conectarse directamente a PostgreSQL.

---

# ADR-002 — PostgreSQL como base de datos

## Decisión

PostgreSQL será la base de datos principal.

## Motivo

Es la base definida para el proyecto y adecuada para las relaciones y operaciones transaccionales requeridas.

---

# ADR-003 — Prisma como ORM

## Decisión

Prisma será el ORM utilizado por NestJS.

## Motivo

Permite definir el modelo, relaciones, consultas y migraciones de forma integrada.

---

# ADR-004 — Cliente separado de UserAccount

## Decisión

Un cliente del gimnasio no necesita una cuenta administrativa.

## Motivo

El acceso del cliente se realiza principalmente mediante huella y membresía.

Modelo:

```text
Fingerprint
    ↓
Member
    ↓
Membership
```

---

# ADR-005 — Puesto laboral separado de Role

## Decisión

El puesto laboral de un empleado no determina directamente sus permisos.

## Motivo

Permite controlar autorización de manera más flexible.

```text
Employee
   |
   └── jobPosition

UserAccount
   |
   └── Role
          |
          └── Permission
```

---

# ADR-006 — No existe rol CLIENT

## Decisión

No crear un rol administrativo `CLIENT`.

## Motivo

Los clientes no necesitan acceso al panel administrativo.

---

# ADR-007 — Huella como referencia biométrica

## Decisión

La aplicación no tratará la huella como contraseña.

Se utilizará una referencia/identificador compatible con el dispositivo biométrico.

## Motivo

Separar el dominio del sistema de la implementación específica del hardware.

---

# ADR-008 — Historial de ventas

## Decisión

Las ventas no deben eliminarse físicamente para corregir errores.

## Motivo

Las operaciones financieras requieren trazabilidad.

Las correcciones deben utilizar:

- Cancelación.
- Ajustes.
- Motivo.
- Auditoría.

---

# ADR-009 — Movimientos del día

## Decisión

El sistema tendrá una funcionalidad para consultar los movimientos realizados durante el día.

## Motivo

El recepcionista necesita identificar y corregir errores operativos rápidamente.

---

# ADR-010 — Dashboard financiero limitado

## Decisión

El dashboard operativo muestra ingresos del día.

Los reportes financieros más amplios requieren permisos.

## Motivo

Separar información operativa de información financiera sensible.

---

# ADR-011 — Payment separado de FinancialTransaction

## Decisión

El pago y el movimiento financiero serán conceptos separados.

```text
Payment
=
cómo se realizó el pago

FinancialTransaction
=
movimiento financiero registrado
```

## Motivo

Evitar mezclar el mecanismo de pago con la contabilidad interna del sistema.

---

# ADR-012 — Tickets internos

## Decisión

Los tickets del sistema serán comprobantes internos.

## Motivo

Un ticket interno no implica automáticamente CFDI.

La facturación fiscal mexicana será una futura integración si se requiere.

---

# ADR-013 — Dinero como Decimal

## Decisión

Los valores monetarios deben utilizar `Decimal`.

## Motivo

Evitar problemas de precisión asociados a tipos de punto flotante.

---

# ADR-014 — No eliminar historial importante

## Decisión

Las operaciones históricas importantes deben conservarse.

## Aplicación

Incluye, entre otras:

- Ventas.
- Movimientos financieros.
- Accesos.
- Auditoría.
- Historial de membresías.

---

# ADR-015 — Trabajo por tickets

## Decisión

El agente trabajará ticket por ticket.

Flujo:

```text
EPIC
 ↓
USER STORY
 ↓
TASK
 ↓
PLAN
 ↓
IMPLEMENTACIÓN
 ↓
PRUEBAS
 ↓
REVISIÓN
 ↓
DONE
```

## Motivo

Evitar modificaciones fuera de alcance y facilitar trazabilidad.

---

# ADR-016 — Documentación operativa en Markdown

## Decisión

El documento maestro original permanece en la raíz.

La documentación operativa se mantiene dentro de `docs/`.

## Motivo

Permitir que agentes y desarrolladores consulten información específica sin depender de un único documento extenso.

---

# ADR-017 — Revisión del modelo antes de Prisma

## Decisión

No construir inmediatamente `schema.prisma`.

Primero:

```text
Modelo conceptual
 ↓
Modelo relacional
 ↓
Revisión
 ↓
schema.prisma
```

## Motivo

Evitar que decisiones de base de datos prematuras obliguen a rehacer migraciones posteriormente.