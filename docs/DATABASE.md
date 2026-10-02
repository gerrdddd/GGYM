# GYM APP — Database Design

## 1. Objetivo

Documentar el modelo conceptual de datos del sistema antes de implementar definitivamente `schema.prisma`.

Este documento representa el diseño funcional y conceptual.

No debe interpretarse automáticamente como una instrucción para crear exactamente las mismas tablas sin revisar relaciones, restricciones, índices y necesidades técnicas.

---

# 2. Tecnología

Base de datos:
Receipt
```text
PostgreSQL
```

ORM:

```text
Prisma
```

---

# 3. Convenciones generales

## IDs

Las entidades principales utilizarán UUID.

## Fechas

Utilizar tipos de fecha/hora apropiados.

## Dinero

Los valores monetarios deben utilizar `Decimal`.

No utilizar `Float` para dinero.

## Estados

Cuando corresponda, utilizar enums.

## Auditoría

Las entidades importantes deben conservar información temporal como:

- `createdAt`
- `updatedAt`

cuando aplique.

## Historial

Las operaciones históricas importantes no deben eliminarse físicamente sin una justificación explícita.

---


## 4. Modelo conceptual

```text
Person
│
├── Member
│   ├── Membership
│   ├── Fingerprint
│   └── AccessLog
│
└── Employee
    └── UserAccount
        ├── UserRole
        │    └── Role
        │         └── RolePermission
        │              └── Permission


MembershipPlan
└── Membership
      └── Sale (operación que la originó, cuando aplique)


Product
├── ProductBatch
├── SaleItem
│    └── SaleItemBatch
└── PurchaseItem


Supplier
└── Purchase
    └── PurchaseItem


Sale
├── SaleMember
├── SaleItem
│    └── SaleItemBatch
├── Payment
├── Receipt
├── Membership (cuando la operación incluye una membresía)
└── FinancialTransaction


Equipment
└── EquipmentMaintenance
       └── FinancialTransaction (cuando genera gasto)


FinancialTransaction
└── CashSession (solo cuando corresponde a efectivo)


AuditLog
```

La entidad `Sale` representa la operación comercial/checkout completa.

Una venta puede ser:

- solamente de productos;
- solamente de una membresía;
- de productos y una membresía al mismo tiempo.

---

## 14. Membership

Representa una membresía concreta perteneciente a un cliente.

Campos conceptuales:

```text
id
memberId
membershipPlanId
saleId?
pricePaid
startDate
endDate
status
createdAt
updatedAt
```

Relaciones:

```text
Member 1 ─── N Membership
MembershipPlan 1 ─── N Membership
Sale 1 ─── 0..N Membership
```

`pricePaid` conserva el importe realmente pagado por esa membresía.

Esto es importante porque:

```text
MembershipPlan.price
```

representa el precio vigente/referencial del plan, mientras que:

```text
Membership.pricePaid
```

representa el precio histórico efectivamente pagado.

`Membership.saleId` permite identificar la operación comercial que originó la membresía.

La relación es nullable para permitir registros históricos o administrativos que no provengan de una venta formal. En el flujo normal de compra/renovación, la membresía deberá quedar vinculada a su `Sale`, mientras que `SaleMember` representa los miembros involucrados en la venta.

---

## 15. Fingerprint

Representa la referencia biométrica asociada a un cliente.

Campos conceptuales:

```text
id
memberId
reference
status
createdAt
updatedAt
```

Reglas:

- `reference` debe ser `UNIQUE`.
- Debe estar indexado para búsquedas rápidas.
- No se debe almacenar la huella cruda ni una imagen de la huella.
- `reference` representa un identificador/referencia compatible con el dispositivo o adaptador biométrico.
- La integración con hardware debe mantenerse desacoplada del dominio.

Relación:

```text
Member 1 ─── N Fingerprint
```

La aplicación utiliza `reference` para localizar rápidamente al cliente correspondiente.

---

## 20. Product

Representa un producto comercializable.

Campos conceptuales:

```text
id
name
barcode?
category
description?
purchasePrice
salePrice
minimumStock
status
createdAt
updatedAt
```

Reglas:

- `barcode` es opcional.
- Cuando existe, debe ser único.
- `purchasePrice` representa el costo de compra actual/de referencia del producto.
- El costo histórico real de cada adquisición se conserva en `ProductBatch.purchasePrice`.
- `minimumStock >= 0`.

Por lo tanto:

```text
Product.purchasePrice
        ↓
costo actual/referencial

ProductBatch.purchasePrice
        ↓
costo histórico real del lote
```

Los cálculos históricos de compras, inventario y rentabilidad deben utilizar el costo del lote cuando corresponda.

---

## 21. ProductBatch

No existe una entidad `Inventory` independiente.

El stock se controla directamente mediante:

```text
ProductBatch.quantity
```

Campos conceptuales:

```text
id
productId
batchNumber?
expirationDate?
quantity
purchasePrice
createdAt
updatedAt
```

Reglas:

- `quantity >= 0`.
- `batchNumber` es opcional.
- Cuando exista `batchNumber`, su unicidad debe considerarse junto con `productId`.
- No se requiere que el número de lote sea globalmente único entre todos los productos.
- El costo histórico de adquisición se almacena en `purchasePrice`.
- Los productos vencidos no pueden venderse.
- Para consumir stock se utilizará FEFO:

```text
First Expire, First Out
```

Es decir, entre lotes válidos del mismo producto se consume primero el lote con fecha de expiración más próxima.

La trazabilidad de una venta se realiza mediante:

```text
SaleItem
    ↓
SaleItemBatch
    ↓
ProductBatch
```

Una misma línea de venta puede consumir unidades de más de un lote.

---

## 23. Sale

Representa la operación comercial/checkout completa.

Campos conceptuales:

```text
id
userId
saleDate
subtotal
discount
total
status
createdAt
updatedAt
```

Una `Sale` puede incluir:

```text
0..N SaleMember
0..N SaleItem
0..N Payment
0..1 Receipt
0..N Membership
0..N FinancialTransaction
```

Casos válidos:

### Venta de productos

```text
Sale
 └── SaleItem
```

### Compra de membresía

```text
Sale
 └── Membership
```

### Operación combinada

```text
Sale
 ├── SaleItem
 └── Membership
```

Ejemplo:

```text
Membresía       $500
Proteína        $700
Bebida           $30
--------------------
Total          $1230
```

Esta operación se registra como una sola `Sale`.

La venta utiliza una arquitectura de carrito antes de su confirmación.

Una vez confirmada:

```text
Sale
 ↓
Items
 ↓
Payments
 ↓
Receipt
 ↓
Inventory changes
 ↓
Financial transactions
 ↓
Audit
```

La venta confirmada no debe eliminarse físicamente.

---


## 23.1 SaleMember

Entidad intermedia para relacionar una venta con los miembros involucrados.

Campos conceptuales:

```text
saleId
memberId
```

PK compuesta: `(saleId, memberId)`.

Relaciones:

```text
Sale 1 ─── N SaleMember
Member 1 ─── N SaleMember
```

Esto reemplaza el antiguo `Sale.memberId` y permite que una misma venta incluya membresías para múltiples miembros diferentes.

## 24. SaleItem

Representa un producto dentro de una venta.

Campos:

```text
id
saleId
productId
quantity
unitPrice
subtotal
```

Reglas:

```text
quantity > 0
unitPrice >= 0
subtotal >= 0
```

El `unitPrice` conserva el precio aplicado en el momento de la venta.

No debe recalcularse posteriormente utilizando el precio actual de `Product`.

---

## 24.1 SaleItemBatch

Entidad intermedia para relacionar una línea de venta con los lotes utilizados.

Campos:

```text
id
saleItemId
batchId
quantity
```

Reglas:

```text
quantity > 0
```

La suma de las cantidades de `SaleItemBatch` correspondientes a un `SaleItem` debe coincidir con la cantidad realmente descontada de ese producto.

Esta entidad permite trazabilidad incluso cuando una sola línea consume múltiples lotes.

---

## 24.2 Return

Representa una devolución.

Campos:

```text
id
saleId
userId
reason
totalAmount
status
createdAt
updatedAt
```

Una devolución nunca elimina la venta original.

La devolución conserva:

```text
Sale original
    ↓
Return
```

El sistema debe poder conocer:

- venta original;
- usuario que realizó la devolución;
- fecha;
- motivo;
- importe;
- productos devueltos;
- lotes involucrados.

---

## 24.3 ReturnItem

Representa un producto específico devuelto.

Campos:

```text
id
returnId
saleItemId
productId
batchId
quantity
unitPrice
subtotal
```

Reglas:

```text
quantity > 0
```

`batchId` permite identificar el lote al que deben regresar las unidades cuando corresponda.

La devolución conserva la trazabilidad de inventario y de la operación original.

---

# 25. Payment

Representa un pago realizado dentro de una `Sale`.

Campos:

```text
id
saleId
amount
method
createdAt
updatedAt
```

`method` únicamente puede ser:

```text
CASH
CARD
TRANSFER
```

La relación es:

```text
Sale 1 ─── N Payment
```

`Payment.saleId` es obligatorio.

No existe una relación alternativa:

```text
Payment → Membership
```

La membresía, si forma parte del checkout, pertenece a la misma `Sale`.

Esto permite pagos divididos.

Ejemplo:

```text
Sale.total = $1230

Payment #1
CASH
$500

Payment #2
CARD
$730
```

Regla:

```text
SUM(Payment.amount) = Sale.total
```

al momento de confirmar la operación.

Un checkout puede utilizar uno o varios métodos de pago.

Importante:

```text
Payment != FinancialTransaction
```

`Payment` representa cómo se pagó.

`FinancialTransaction` representa el movimiento financiero generado dentro del sistema.

---

# 26. Receipt

Representa un comprobante interno.

Campos conceptuales:

```text
id
saleId
receiptNumber
issuedAt
total
status
```

Relación:

```text
Sale 1 ─── 1 Receipt
```

El `Receipt` pertenece a la `Sale`, no directamente a una membresía.

Esto permite que una sola operación genere un único comprobante aunque incluya:

```text
Productos
+
Membresía
+
Uno o varios pagos
```

El ticket interno no implica automáticamente CFDI fiscal.

La facturación fiscal mexicana sería una futura integración independiente.

---

# 27. FinancialTransaction

Representa un movimiento financiero.

Campos conceptuales:

```text
id
type
category
amount
description?
paymentMethod?
userId
saleId?
purchaseId?
paymentId?
equipmentMaintenanceId?
cashSessionId?
createdAt
updatedAt
```

Una `FinancialTransaction` puede estar asociada a:

- una venta;
- una compra;

- un pago (`paymentId` con `UNIQUE`);
- un gasto de mantenimiento (`equipmentMaintenanceId`);
- otra operación financiera permitida.

En el caso de una `Sale`, cada `Payment` confirmado genera su correspondiente movimiento financiero.

Ejemplo:

```text
Sale = $1230

Payment CASH = $500
        ↓
FinancialTransaction INCOME $500
        ↓
CashSession

Payment CARD = $730
        ↓
FinancialTransaction INCOME $730
        ↓
sin CashSession
```

No debe generarse adicionalmente una tercera transacción de `$1230` que duplique el ingreso.

`cashSessionId` solo se utiliza cuando el movimiento corresponde a efectivo administrado por una sesión de caja.

---

# 28. CashSession

Representa una sesión de caja.

Campos:

```text
id
openedBy
closedBy?
openedAt
closedAt?
openingAmount
expectedAmount?
actualAmount?
difference?
status
```

La sesión está asociada al usuario que administra la caja. Pueden existir múltiples sesiones abiertas simultáneamente.

Solamente los movimientos que afectan físicamente el efectivo deben impactar la sesión de caja.

Por lo tanto:

```text
CASH
    ↓
CashSession
```

Mientras:

```text
CARD
TRANSFER
```

no incrementan el efectivo físico de la caja.

El permiso:

```text
MANAGE_CASH
```

controla las operaciones sensibles relacionadas con caja.

---

# 31. AuditLog

Representa acciones importantes realizadas dentro del sistema.

Campos:

```text
id
actorType
userId?
action
entity
entityId
description
createdAt
```

Debe permitir conocer:

```text
Quién
Qué hizo
Sobre qué entidad
Cuándo
Por qué
```

cuando corresponda.

Reglas:
- `actorType = USER` → `userId` requerido
- `actorType = SYSTEM` → `userId` NULL

Acciones importantes incluyen, entre otras:

```text
SALE_CREATED
SALE_CANCELLED
SALE_CORRECTED
PAYMENT_CREATED
RETURN_CREATED
STOCK_ADJUSTED
CASH_SESSION_OPENED
CASH_SESSION_CLOSED
FINANCIAL_TRANSACTION_CREATED
FINANCIAL_TRANSACTION_MODIFIED
```

El catálogo definitivo de acciones se implementará mediante enums o una estrategia equivalente durante el diseño relacional.

---

# 32. Relaciones principales

```text
Person
 ├── Member
 │    ├── Membership
 │    │    └── Sale (saleId?)
 │    ├── Fingerprint
 │    └── AccessLog
 │
 └── Employee
      └── UserAccount
           ├── UserRole
           │    └── Role
           │         └── RolePermission
           │              └── Permission


MembershipPlan
 └── Membership


Product
 ├── ProductBatch
 ├── SaleItem
 │    └── SaleItemBatch
 └── PurchaseItem


Supplier
 └── Purchase
      └── PurchaseItem


Sale
 ├── SaleItem
 │    └── SaleItemBatch
 ├── Payment
 ├── Receipt
 ├── Membership
 └── FinancialTransaction


FinancialTransaction
 └── CashSession?


Equipment
 └── EquipmentMaintenance


Return
 └── ReturnItem


AuditLog
```

---

# 33. Integridad y restricciones

Las relaciones históricas importantes deben protegerse contra eliminación accidental.

Regla general:

```text
Historial importante
        ↓
RESTRICT / NO ACTION
```

Las relaciones intermedias puras pueden utilizar `CASCADE` cuando sea seguro hacerlo.

Las relaciones opcionales pueden utilizar `SET NULL` cuando conservar el registro histórico tenga sentido.

No deben utilizarse cascadas destructivas sobre:

- ventas;
- pagos;
- movimientos financieros;
- membresías históricas;
- accesos;
- auditoría;
- devoluciones.

---

# 34. Cantidades

Las cantidades deben respetar reglas lógicas.

Stock:

```text
ProductBatch.quantity >= 0
Product.minimumStock >= 0
```

Líneas de operación:

```text
SaleItem.quantity > 0
SaleItemBatch.quantity > 0
PurchaseItem.quantity > 0
ReturnItem.quantity > 0
```

No se establecerán límites artificiales absurdamente pequeños.

Los límites físicos razonables se validarán posteriormente a nivel de DTO, dominio y base de datos cuando corresponda.

---

# 35. Índices

Se deben utilizar índices para claves foráneas y búsquedas frecuentes.

Como mínimo se deberán evaluar índices para:

```text
Fingerprint.reference
Product.barcode
ProductBatch.productId
ProductBatch.expirationDate

Sale.saleDate
Sale.userId

SaleMember.saleId
SaleMember.memberId

Payment.saleId

FinancialTransaction.createdAt
FinancialTransaction.cashSessionId
FinancialTransaction.saleId

AccessLog.memberId
AccessLog.createdAt

AuditLog.entityId
AuditLog.createdAt
```

También deben evaluarse índices compuestos cuando el patrón real de consulta lo justifique.

No se deben agregar índices arbitrariamente.

Si posteriormente una consulta compleja lo justifica, podrán utilizarse:

- índices adicionales;
- vistas;
- funciones;
- consultas especializadas.

La necesidad deberá demostrarse por el comportamiento real del sistema.

---


---

# 37. Entidades faltantes documentadas conceptualmente

- **Person:** Entidad base (`id`).
- **Member:** Cliente (`id`, `personId` UNIQUE).
- **Employee:** Empleado (`id`, `personId` UNIQUE).
- **UserAccount/Role/Permission:** Sistema de autorización (`Employee 1 ─── 0..1 UserAccount`).
- **AccessLog:** Registro de entrada (`memberId`, `accessType`, `granted`).
- **Equipment:** Unidad física (`id`, sin `quantity`, `serialNumber?` UNIQUE).
- **EquipmentMaintenance:** Mantenimiento (`equipmentId`, `performedByEmployeeId?`, `providerName?`).

---

# 38. Claves Foráneas Compuestas

Para garantizar la coherencia de datos, las relaciones que involucran lotes deben utilizar FK compuestas:

```text
(productId, batchId)
        ↓
ProductBatch(productId, id)
```

Aplicable a `PurchaseItem`, `SaleItemBatch` y `ReturnItem`.

---

# 39. Reglas de Lógica de Negocio (NestJS)

No todo se resuelve mediante restricciones de base de datos. NestJS debe validar mediante transacciones:
- Una venta tiene al menos un componente comercial.
- Suma de `Payments` = `Sale.total`.
- Suma de `SaleItemBatch` no supera `SaleItem.quantity`.
- FEFO y control estricto de stock sin vender negativos.
- No devolver más unidades de las permitidas.
- Empleado interno vs proveedor externo en `EquipmentMaintenance`.
- Correcciones financieras mediante movimientos compensatorios.

# 36. Regla para schema.prisma

No crear el schema definitivo copiando automáticamente este documento.

Proceso:

```text
Modelo conceptual
      ↓
Revisión
      ↓
Modelo relacional
      ↓
Revisión
      ↓
schema.prisma
      ↓
prisma validate
      ↓
migration
      ↓
pruebas
```

Este documento define la intención funcional y conceptual.

El diseño relacional definitivo todavía debe revisarse antes de crear la primera migración.