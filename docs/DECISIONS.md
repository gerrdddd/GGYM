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

La referencia debe ser única y permitir búsquedas eficientes.

## Motivo

Separar el dominio del sistema de la implementación específica del hardware.

---

# ADR-008 — Historial de ventas

## Decisión

Las ventas no deben eliminarse físicamente para corregir errores.

## Motivo

Las operaciones financieras requieren trazabilidad.

Las correcciones deben utilizar:

- cancelación;
- ajustes;
- motivo;
- auditoría.

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

Evitar mezclar el mecanismo de pago con el registro financiero interno.

---

# ADR-012 — Tickets internos

## Decisión

Los tickets del sistema serán comprobantes internos.

Una `Receipt` pertenece al checkout representado por una `Sale`.

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

- ventas;
- movimientos financieros;
- accesos;
- auditoría;
- historial de membresías;
- devoluciones;
- pagos.

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

---

# ADR-018 — Código de barras opcional en productos

## Decisión

`barcode` será opcional en `Product`.

Si existe, debe ser único.

## Motivo

No todos los productos tienen código de barras.

---

# ADR-019 — Arquitectura de carrito en ventas

## Decisión

El proceso de venta implementará un carrito de compras.

## Motivo

Permite agregar múltiples productos, modificar cantidades, eliminar productos, recalcular subtotales, aplicar descuentos y recalcular totales antes de confirmar.

---

# ADR-020 — Funcionalidad de devoluciones en ventas

## Decisión

El sistema contempla devoluciones mediante `Return` y `ReturnItem`.

La venta original no se elimina ni se modifica físicamente para representar la devolución.

## Motivo

Mantener la trazabilidad histórica.

---

# ADR-021 — Soporte para descuentos en ventas

## Decisión

Las ventas podrán tener:

```text
subtotal
discount
total
```

Las reglas exactas de autorización para descuentos quedan sujetas a la matriz de permisos.

---

# ADR-022 — Historial visual del cliente

## Decisión

El sistema permitirá consultar visualmente información histórica relevante del cliente.

Incluye:

- información personal;
- membresías;
- historial de acceso;
- información de recepción.

---

# ADR-023 — Alertas accionables en Dashboard

## Decisión

El dashboard mostrará alertas sobre:

- stock bajo;
- productos próximos a vencer;
- membresías próximas a vencer;
- equipos en mantenimiento.

Las alertas permitirán navegar a la sección correspondiente.

---

# ADR-024 — Sesión de caja asociada a usuario

## Decisión

Las operaciones de caja estarán asociadas al usuario responsable de la sesión.

Se conservarán:

- usuario que abrió;
- usuario que cerró;
- apertura;
- esperado;
- real;
- diferencia;
- timestamps.

---

# ADR-025 — Métodos de pago iniciales limitados

## Decisión

La primera versión soportará únicamente:

```text
CASH
CARD
TRANSFER
```

No se implementarán pasarelas de pago online en esta fase.

---

# ADR-026 — Eliminación de entidad Inventory independiente

## Decisión

No existirá una entidad `Inventory` independiente.

El stock se gestionará mediante:

```text
ProductBatch.quantity
```

## Motivo

Evitar redundancia y problemas de sincronización.

---

# ADR-027 — Estrategia de lotes FEFO

## Decisión

El sistema utilizará FEFO:

```text
First Expire, First Out
```

Se consumirán primero los lotes no vencidos con fecha de vencimiento más próxima.

---

# ADR-028 — Entidad intermedia SaleItemBatch

## Decisión

Se crea `SaleItemBatch` entre `SaleItem` y `ProductBatch`.

## Motivo

Una línea de venta puede consumir unidades de varios lotes.

```text
SaleItem
   ↓
SaleItemBatch
   ↓
ProductBatch
```

---

# ADR-029 — Modelo de Devoluciones

## Decisión

Se crean:

```text
Return
ReturnItem
```

`ReturnItem` incluirá `batchId`.

## Motivo

Permitir devolver correctamente unidades al lote correspondiente y conservar historial independiente.

---

# ADR-030 — Asociación opcional de CashSession a FinancialTransaction

## Decisión

`FinancialTransaction` tendrá una relación opcional hacia `CashSession`.

```text
cashSessionId?
```

## Motivo

No todas las transacciones afectan el efectivo físico.

---

# ADR-031 — Permiso específico MANAGE_CASH

## Decisión

Se crea:

```text
MANAGE_CASH
```

`CREATE_SALE` no otorga automáticamente permiso para administrar caja.

---

# ADR-032 — Sale como checkout comercial único

## Decisión

`Sale` representa el checkout comercial completo.

Una `Sale` puede contener:

- productos;
- una membresía adquirida o renovada;
- uno o varios pagos;
- un recibo.

Ejemplo:

```text
Sale
 ├── Membership
 ├── SaleItem
 ├── SaleItem
 ├── Payment
 ├── Payment
 └── Receipt
```

## Motivo

Una operación real puede incluir la renovación de una membresía y la compra de productos al mismo tiempo.

Separarlas artificialmente generaría dos operaciones comerciales cuando el cliente realizó un solo checkout.

---

# ADR-033 — Payment pertenece a Sale

## Decisión

`Payment` pertenecerá obligatoriamente a `Sale`.

No se utilizará una relación alternativa:

```text
Payment → Sale XOR Membership
```

La membresía se relacionará con el checkout que la originó.

## Motivo

El pago corresponde al checkout completo y no exclusivamente a uno de sus conceptos.

---

# ADR-034 — Soporte para pagos divididos

## Decisión

Una `Sale` puede tener múltiples `Payment`.

Ejemplo:

```text
Sale total = $1,300

Payment
$500 CASH

Payment
$800 CARD
```

La suma de los pagos debe coincidir con el total de la venta.

## Motivo

Permitir pagos combinados sin duplicar el ingreso financiero.

---

# ADR-035 — FinancialTransaction por Payment

## Decisión

Los movimientos financieros derivados de una venta deben corresponder a los pagos registrados.

Ejemplo:

```text
Sale = $1,300

Payment CASH = $500
        ↓
FinancialTransaction INCOME = $500
        ↓
CashSession

Payment CARD = $800
        ↓
FinancialTransaction INCOME = $800
```

No se debe registrar adicionalmente un ingreso independiente de $1,300 por la misma operación si los dos movimientos anteriores ya representan el ingreso completo.

## Motivo

Mantener trazabilidad entre:

```text
Payment
   ↓
FinancialTransaction
```

y evitar duplicación financiera.

---

# ADR-036 — Receipt pertenece a Sale

## Decisión

Una `Receipt` pertenece al checkout representado por `Sale`.

No se utilizará una relación:

```text
Receipt → Sale XOR Membership
```

## Motivo

Una misma operación puede contener membresía y productos.

Un único ticket puede representar todo el checkout.

---

# ADR-037 — Membership conserva el precio pagado

## Decisión

`Membership` conservará el valor históricamente pagado mediante:

```text
pricePaid
```

También podrá relacionarse con la `Sale` que originó la membresía.

## Motivo

El precio del plan puede cambiar posteriormente. El historial no debe depender del precio actual del `MembershipPlan`.

---

# ADR-038 — Product.purchasePrice y ProductBatch.purchasePrice

## Decisión

Se mantienen ambos valores.

`Product.purchasePrice`:

```text
precio actual / de referencia
```

`ProductBatch.purchasePrice`:

```text
costo histórico real de adquisición del lote
```

## Motivo

Permitir conservar correctamente el costo histórico cuando cambien los precios de compra.

---

# ADR-039 — BatchNumber opcional

## Decisión

`ProductBatch.batchNumber` será opcional.

Cuando exista, su unicidad se considerará dentro del producto:

```text
(productId, batchNumber)
```

## Motivo

Los proveedores pueden utilizar números de lote que se repitan entre productos diferentes.

---

# ADR-040 — Ajustes de inventario

## Decisión

El sistema permitirá ajustes de stock para situaciones que no sean ventas o compras.

Ejemplos:

- producto roto;
- producto vencido;
- producto perdido;
- producto inutilizable;
- corrección de conteo.

La acción de auditoría será:

```text
STOCK_ADJUSTED
```

## Motivo

Permitir corregir existencias reales sin convertir artificialmente el ajuste en una venta o compra.

---

# ADR-041 — Auditoría de operaciones sensibles

## Decisión

Las operaciones sensibles deberán conservar:

```text
userId
action
entity
entityId
description
createdAt
```

Se contemplan acciones como:

```text
STOCK_ADJUSTED
CREATE_SALE
CANCEL_SALE
CORRECT_SALE
CREATE_RETURN
CREATE_PAYMENT
OPEN_CASH_SESSION
CLOSE_CASH_SESSION
```

## Motivo

Mantener trazabilidad de quién realizó cada modificación importante.

---

# ADR-042 — SaleMember y múltiples miembros por venta

## Decisión

La relación entre `Sale` y `Member` se implementa mediante una tabla intermedia `SaleMember`.

Se elimina `Sale.memberId` del modelo.

```text
SaleMember
──────────────
saleId
memberId
```

PK compuesta: `(saleId, memberId)`.

Relaciones:

```text
Sale 1 ─── N SaleMember
Member 1 ─── N SaleMember
```

Una venta puede involucrar múltiples miembros.

Ejemplo:

```text
Sale #100
 ├── SaleMember → Juan
 ├── SaleMember → Ana
 ├── SaleItem → Agua
 ├── Membership → Juan
 └── Membership → Ana
```

## Motivo

El campo `Sale.memberId` solo permitía asociar una venta con un único miembro. En la operación real, una venta puede incluir membresías de diferentes miembros pagadas en una sola transacción.

`SaleMember` permite representar esta relación N:M correctamente.

---

# ADR-043 — Sale puede generar múltiples Memberships

## Decisión

Una `Sale` puede generar N membresías.

```text
Sale 1 ─── N Membership
```

`Membership.saleId` es nullable para permitir membresías históricas o administrativas que no provengan de una venta.

Cuando una `Membership` forma parte de una venta, se relaciona con `Sale` mediante `saleId`, mientras que `SaleMember` representa los miembros involucrados en la venta.

## Motivo

Permitir que una sola operación comercial genere membresías para múltiples miembros y mantener la trazabilidad de cada membresía con su venta de origen.

---

# ADR-044 — Payment 1:0..1 FinancialTransaction

## Decisión

La relación entre `Payment` y `FinancialTransaction` es 1:0..1.

Se implementa mediante:

```text
FinancialTransaction.paymentId?
```

con restricción `UNIQUE`.

Un `Payment` puede generar como máximo un `FinancialTransaction`.

Ejemplo:

```text
Payment $600 CASH
        ↓
FinancialTransaction $600
        ↓
CashSession

Payment $400 CARD
        ↓
FinancialTransaction $400
        (sin CashSession)
```

## Motivo

Establecer una trazabilidad directa y unívoca entre el pago y su movimiento financiero, sin utilizar `membershipId` en `FinancialTransaction`.

---

# ADR-045 — FinancialTransaction.amount siempre positivo

## Decisión

```text
FinancialTransaction.amount > 0
```

El importe se almacena siempre como valor positivo.

No se representan salidas mediante importes negativos.

El campo `type` determina el significado financiero del movimiento:

```text
INCOME
EXPENSE
REFUND
ADJUSTMENT
```

Ejemplo:

```text
INCOME amount = 1000   → ingreso de $1000
REFUND amount = 200    → reembolso de $200
EXPENSE amount = 1500  → gasto de $1500
```

## Motivo

Evitar ambigüedad entre el signo del importe y el tipo de operación. El tipo describe el significado; el importe describe la magnitud.

---

# ADR-046 — FinancialTransaction.equipmentMaintenanceId

## Decisión

`FinancialTransaction` incluye un campo opcional:

```text
equipmentMaintenanceId?
```

Permite vincular un movimiento financiero de tipo `EXPENSE` con un registro de `EquipmentMaintenance`.

Ejemplo:

```text
EquipmentMaintenance (caminadora, costo $1500)
        ↓
FinancialTransaction
type = EXPENSE
amount = 1500
equipmentMaintenanceId = ...
```

## Motivo

Proporcionar trazabilidad directa entre un gasto de mantenimiento y su movimiento financiero correspondiente.

---

# ADR-047 — Devoluciones y trazabilidad financiera

## Decisión

Las devoluciones generan movimientos financieros de tipo `REFUND` sin modificar los `FinancialTransaction` originales.

En caso de pagos divididos, la distribución del reembolso entre métodos de pago se determina en lógica de negocio.

Cada parte del reembolso conserva trazabilidad propia. No se duplican importes.

No se crea una entidad adicional de refund. Si en el futuro se necesitara, se documentará como decisión futura.

La cadena de trazabilidad debe preservar:

```text
Sale
 ↓
Payments originales
 ↓
Return
 ↓
movimiento(s) financieros de devolución
```

## Motivo

Mantener integridad financiera, evitar duplicación de importes y conservar el historial completo de la operación original y su devolución.

---

# ADR-048 — CashSession permite múltiples sesiones abiertas

## Decisión

Pueden existir múltiples `CashSession` con status `OPEN` simultáneamente.

No se impone una restricción global de una sola sesión abierta.

```text
Usuario A → CashSession OPEN
Usuario B → CashSession OPEN
```

## Motivo

Permitir que varios usuarios operen sesiones de caja independientes de forma simultánea.

---

# ADR-049 — Equipment como unidad física individual

## Decisión

Cada registro `Equipment` representa una unidad física individual.

No existe un campo `quantity`.

```text
Equipment #1
Caminadora
Serial: ABC123

Equipment #2
Caminadora
Serial: ABC124
```

`serialNumber` es opcional pero `UNIQUE` cuando existe.

## Motivo

Permitir rastrear individualmente el estado, historial de mantenimiento y ubicación de cada unidad de equipo.

---

# ADR-050 — EquipmentMaintenance con empleado o proveedor externo

## Decisión

Un mantenimiento puede ser realizado por:

### Empleado interno

```text
performedByEmployeeId?
```

### Proveedor externo

```text
providerName?
providerPhone?
providerEmail?
```

La regla conceptual es:

```text
empleado interno
        O
proveedor externo
```

No ambos simultáneamente. El contacto externo es opcional.

## Motivo

Soportar ambos escenarios de mantenimiento y mantener trazabilidad del responsable.

---

# ADR-051 — AuditLog con actorType USER/SYSTEM

## Decisión

`AuditLog` utiliza:

```text
actorType: USER | SYSTEM
userId?
```

Reglas:

```text
actorType = USER  → userId requerido
actorType = SYSTEM → userId NULL
```

No se crea un `UserAccount` falso llamado `SYSTEM`.

## Motivo

Diferenciar operaciones realizadas por usuarios de operaciones automáticas del sistema sin contaminar la tabla de usuarios.

---

# ADR-052 — FK compuestas para coherencia Product/ProductBatch

## Decisión

Las entidades que referencian tanto un `Product` como un `ProductBatch` deben garantizar que el lote pertenezca al producto indicado.

Estrategia:

```text
(productId, batchId)
        ↓
ProductBatch(productId, id)
```

mediante FK compuesta cuando Prisma/PostgreSQL lo permitan.

Aplica a:

- `PurchaseItem`
- `SaleItemBatch`
- `ReturnItem`

Además de la restricción de BD, NestJS debe validar la coherencia dentro de las operaciones.

## Motivo

Evitar que un lote de un producto sea referenciado desde una operación de un producto diferente.

---

# ADR-053 — Restricciones de integridad y protección del historial

## Decisión

Las operaciones históricas importantes no se eliminan físicamente.

Entidades protegidas:

```text
Sale
Payment
FinancialTransaction
Membership
Purchase
Return
AccessLog
AuditLog
```

Se prefieren estados:

```text
ACTIVE
INACTIVE
CANCELLED
CORRECTED
```

Las relaciones críticas utilizan `RESTRICT` / `NO ACTION`.

`CASCADE` se utiliza únicamente para relaciones técnicas/intermedias que no destruyan historial.

Las correcciones financieras se realizan mediante movimientos compensatorios, nunca mediante eliminación o modificación destructiva del movimiento original.

## Motivo

Garantizar trazabilidad, integridad financiera e integridad del historial operativo del gimnasio.