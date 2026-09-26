# GYM APP — Database Design

## 1. Objetivo

Documentar el modelo conceptual de datos del sistema antes de implementar definitivamente `schema.prisma`.

Este documento representa el diseño funcional y conceptual.

No debe interpretarse automáticamente como una instrucción para crear exactamente las mismas tablas sin revisar relaciones, restricciones, índices y necesidades técnicas.

---

# 2. Tecnología

Base de datos:

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

# 4. Modelo conceptual

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
        ├── Role
        └── Permission


MembershipPlan
└── Membership


Product
├── ProductBatch
├── SaleItem
└── PurchaseItem


Supplier
└── Purchase
    └── PurchaseItem


Sale
├── SaleItem
├── Payment
├── Receipt
└── FinancialTransaction


Equipment
└── EquipmentMaintenance


FinancialTransaction
└── AuditLog
```

---

# 5. Person

Representa a una persona físicamente.

Campos conceptuales:

```text
id
firstName
lastName
phone
email
photoUrl
createdAt
updatedAt
```

Una persona puede estar relacionada con:

```text
Person
├── Member
└── Employee
```

No debe asumirse que una persona solo puede tener uno de estos conceptos.

---

# 6. Member

Representa al cliente del gimnasio.

Campos:

```text
id
personId
status
createdAt
updatedAt
```

Relación:

```text
Person 1 ─── 1 Member
```

Un Member puede tener:

```text
Member
├── Membership
├── Fingerprint
└── AccessLog
```

---

# 7. Employee

Representa a una persona que trabaja en el gimnasio.

Campos:

```text
id
personId
jobPosition
hireDate
status
observations
createdAt
updatedAt
```

Importante:

```text
jobPosition != Role
```

El puesto laboral no determina automáticamente los permisos del sistema.

---

# 8. UserAccount

Representa la cuenta utilizada para acceder al panel administrativo.

Campos:

```text
id
employeeId
email
passwordHash
status
lastLoginAt
createdAt
updatedAt
```

Relación:

```text
Employee 1 ─── 0..1 UserAccount
```

No todos los empleados necesitan una cuenta.

---

# 9. Role

Representa un conjunto de permisos.

Campos:

```text
id
name
description
createdAt
updatedAt
```

Roles iniciales:

```text
ADMIN
RECEPTIONIST
COACH
CLEANING
MAINTENANCE
```

No se debe crear un rol `CLIENT` únicamente para controlar el acceso mediante huella.

---

# 10. Permission

Representa una acción concreta que un usuario puede realizar.

Campos:

```text
id
name
description
createdAt
updatedAt
```

Permisos iniciales/documentados:

```text
VIEW_CLIENTS
MANAGE_CLIENTS
VIEW_MEMBERSHIPS
MANAGE_MEMBERSHIPS
CREATE_SALE
VIEW_TODAY_MOVEMENTS
CANCEL_SALE
VIEW_PRODUCTS
MANAGE_INVENTORY
VIEW_FINANCIAL_REPORTS
MANAGE_FINANCES
VIEW_AUDIT_LOG
```

La relación conceptual es:

```text
Role
  |
  └── Permission
```

La relación entre roles y permisos es muchos a muchos.

---

# 11. UserRole

Entidad intermedia conceptual:

```text
userId
roleId
```

Relación:

```text
UserAccount N ─── N Role
```

---

# 12. RolePermission

Entidad intermedia:

```text
roleId
permissionId
```

Relación:

```text
Role N ─── N Permission
```

---

# 13. MembershipPlan

Define los planes que ofrece el gimnasio.

Campos:

```text
id
name
description
price
durationDays
status
createdAt
updatedAt
```

Ejemplo:

```text
Plan mensual
$500
30 días
```

---

# 14. Membership

Representa una membresía concreta perteneciente a un cliente.

Campos:

```text
id
memberId
membershipPlanId
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
```

El historial de membresías debe conservarse.

Ejemplo:

```text
Juan
├── Enero — Mensual
├── Febrero — Mensual
└── Marzo — Trimestral
```

No se debe sobrescribir el historial anterior.

---

# 15. Fingerprint

Representa una referencia biométrica asociada a un Member.

Conceptualmente:

```text
id
memberId
reference
status
createdAt
updatedAt
```

La aplicación no debe tratar la huella como una contraseña.

Debe utilizarse una referencia/identificador compatible con el dispositivo biométrico.

La implementación física del dispositivo debe mantenerse desacoplada del dominio siempre que sea posible.

---

# 16. Device

Representa un dispositivo utilizado por el sistema.

Conceptualmente:

```text
id
name
type
location
status
createdAt
updatedAt
```

Puede relacionarse con:

```text
AccessLog
```

---

# 17. AccessLog

Representa un intento o resultado de acceso.

Conceptualmente:

```text
id
memberId?
deviceId
result
reason?
createdAt
```

Debe permitir conservar información como:

- Cliente.
- Fecha.
- Hora.
- Resultado.
- Dispositivo.
- Motivo cuando corresponda.

---

# 18. Product

Representa un producto vendido por el gimnasio.

Campos conceptuales:

```text
id
name
category
description
purchasePrice
salePrice
minimumStock
status
createdAt
updatedAt
```

Categorías iniciales:

```text
PROTEIN
CREATINE
PRE_WORKOUT
DRINK
CLOTHING
ACCESSORY
OTHER
```

---

# 19. ProductBatch

Permite manejar lotes y vencimientos.

Conceptualmente:

```text
id
productId
batchNumber
expirationDate
quantity
purchasePrice
createdAt
updatedAt
```

Esto permite controlar:

- Existencias.
- Lotes.
- Fechas de vencimiento.
- Productos próximos a vencer.
- Productos vencidos.

---

# 20. Supplier

Representa a un proveedor.

Campos:

```text
id
name
phone
email
address
status
createdAt
updatedAt
```

---

# 21. Purchase

Representa una compra realizada a un proveedor.

Conceptualmente:

```text
id
supplierId
userId
purchaseDate
total
invoiceReference
status
createdAt
updatedAt
```

Una compra puede generar:

```text
Purchase
   |
   ├── Inventory entry
   ├── Financial expense
   └── Receipt/comprobante
```

---

# 22. PurchaseItem

Representa los productos incluidos en una compra.

Campos conceptuales:

```text
id
purchaseId
productId
batchId
quantity
unitPrice
subtotal
```

---

# 23. Sale

Representa una venta.

Campos conceptuales:

```text
id
userId
memberId?
saleDate
subtotal
discount
total
status
createdAt
updatedAt
```

Una venta puede incluir varios productos.

---

# 24. SaleItem

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

Relación:

```text
Sale 1 ─── N SaleItem
```

---

# 25. Payment

Representa cómo se realizó un pago.

Conceptualmente:

```text
id
saleId?
membershipId?
amount
method
createdAt
updatedAt
```

Importante:

```text
Payment != FinancialTransaction
```

Payment representa el pago.

FinancialTransaction representa el movimiento financiero generado dentro del sistema.

---

# 26. Receipt

Representa un comprobante interno.

Conceptualmente:

```text
id
saleId?
membershipId?
receiptNumber
issuedAt
total
status
```

El ticket interno no implica automáticamente CFDI fiscal.

La facturación fiscal mexicana sería una futura integración/módulo independiente.

---

# 27. FinancialTransaction

Representa un movimiento financiero.

Conceptualmente:

```text
id
type
category
amount
description
paymentMethod
userId
saleId?
purchaseId?
membershipId?
createdAt
updatedAt
```

Ejemplos:

```text
SALE
MEMBERSHIP
PURCHASE
EXPENSE
OTHER
```

Las categorías y enums definitivos deberán revisarse antes de implementarse.

---

# 28. CashSession

Representa una sesión de caja.

Conceptualmente:

```text
id
openedBy
closedBy
openedAt
closedAt
openingAmount
expectedAmount
actualAmount
difference
status
```

Permite controlar:

- Apertura.
- Operaciones.
- Efectivo esperado.
- Efectivo real.
- Diferencia.
- Corte.

---

# 29. Equipment

Representa equipamiento del gimnasio.

Campos conceptuales:

```text
id
name
category
description
quantity
brand
model
serialNumber
location
status
acquisitionDate
createdAt
updatedAt
```

Categorías:

```text
Máquinas
Mancuernas
Discos
Barras
Cables/Poleas
Bancos
Accesorios
Otros
```

---

# 30. EquipmentMaintenance

Representa mantenimiento realizado sobre un equipo.

Conceptualmente:

```text
id
equipmentId
performedBy
maintenanceType
description
cost
maintenanceDate
nextMaintenanceDate
status
createdAt
updatedAt
```

Un mantenimiento puede generar un gasto financiero cuando corresponda.

---

# 31. AuditLog

Representa acciones importantes realizadas dentro del sistema.

Conceptualmente:

```text
id
userId
action
entity
entityId
description
createdAt
```

Debe permitir saber:

```text
Quién
Qué hizo
Sobre qué entidad
Cuándo
Por qué
```

cuando la operación requiera motivo.

---

# 32. Relaciones principales

```text
Person
 ├── Member
 │    ├── Membership
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
 └── PurchaseItem


Supplier
 └── Purchase
      └── PurchaseItem


Sale
 ├── SaleItem
 ├── Payment
 ├── Receipt
 └── FinancialTransaction


Equipment
 └── EquipmentMaintenance


FinancialTransaction
 └── AuditLog
```

---

# 33. Regla para schema.prisma

No crear el schema definitivo solamente copiando este documento.

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

---

# 34. Integridad histórica

No eliminar físicamente:

- Ventas históricas.
- Movimientos financieros.
- Operaciones importantes.
- Historial de acceso.
- Auditoría.

Las correcciones deben utilizar estados, cancelaciones, ajustes y auditoría.