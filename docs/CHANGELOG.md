# GYM APP — Changelog

Todos los cambios importantes del proyecto deben registrarse aquí.

---

## 2026-09-25

### Documentation

- Se estableció la estructura operativa de documentación.
- Se definió `docs/MASTER.md`.
- Se definieron documentos especializados para arquitectura, base de datos, reglas de negocio, flujos, permisos, API, estado, cambios y decisiones.
- Se definieron instrucciones para agentes mediante `AGENTS.md`.
- Se definió la posibilidad de reglas específicas dentro de `.agents/rules/`.

### Architecture

- Se confirmó la arquitectura:
  - Next.js.
  - NestJS.
  - Prisma.
  - PostgreSQL.
- Se confirmó la separación frontend/backend.
- Se confirmó que el frontend no accede directamente a PostgreSQL.

### Database

- Se confirmó PostgreSQL como base de datos.
- Se confirmó Prisma como ORM.
- Se estableció que el modelo debe revisarse antes de crear el schema definitivo.
- Se estableció el uso de UUID para entidades principales.
- Se estableció el uso de Decimal para valores monetarios.

### Business

- Se confirmó la separación entre cliente y cuenta administrativa.
- Se confirmó la separación entre puesto laboral, rol y permiso.
- Se confirmó que no existe un rol `CLIENT` para acceso mediante huella.
- Se confirmó la conservación del historial de operaciones importantes.
- Se confirmó la necesidad de movimientos del día para corrección de operaciones.

---

## 2026-09-28

### Documentation & Business Decisions

- **Productos:** código de barras opcional (`barcode`), búsqueda manual y por categoría.
- **Ventas:** integración de arquitectura de carrito.
- **Ventas:** soporte para descuentos mediante `subtotal`, `discount` y `total`.
- **Inventario:** se descartó una entidad `Inventory` independiente.
- **Inventario:** el stock se gestiona mediante `ProductBatch.quantity`.
- **Inventario:** se aprobó estrategia FEFO.
- **Ventas:** se aprobó `SaleItemBatch` para rastrear los lotes consumidos.
- **Devoluciones:** se aprobaron `Return` y `ReturnItem`.
- **Caja:** `CashSession` queda asociada al usuario responsable.
- **Caja:** se requiere `MANAGE_CASH`.
- **Caja:** `FinancialTransaction` puede asociarse opcionalmente a `CashSession`.
- **Pagos:** primera versión limitada a efectivo, tarjeta y transferencia.
- **UI:** se aprobaron alertas accionables en Dashboard.
- **Clientes:** se aprobó historial visual.
- **ADRs:** se documentaron las decisiones ADR-018 a ADR-031.

---


## 2026-10-01

### Database & Business Model Update
- Actualización completa del modelo relacional.
- Sincronización completa de toda la documentación.
- Nuevas decisiones documentadas (ADR-042 a ADR-053).
- Contradicciones antiguas corregidas (`Sale.memberId` eliminado a favor de `SaleMember`, `FinancialTransaction.membershipId` eliminado a favor de `paymentId` y `equipmentMaintenanceId`, `Equipment.quantity` eliminado, `AuditLog.actorType` implementado).
- Aclaración explícita de `Membership.saleId` opcional para membresías históricas/administrativas.
- `schema.prisma` todavía NO implementado en este ticket.

## 2026-09-30

### Business Model

- Se definió `Sale` como el checkout comercial completo.
- Se estableció que una misma `Sale` puede contener productos y una membresía.
- Se eliminó conceptualmente la necesidad de asociar `Payment` directamente a `Membership`.
- `Payment` pertenece al checkout representado por `Sale`.
- `Receipt` pertenece al checkout representado por `Sale`.
- Se estableció que una operación combinada de membresía + productos puede generar un único ticket.

### Payments

- Se aprobó soporte para pagos divididos.
- Una `Sale` puede tener múltiples `Payment`.
- La suma de los pagos debe corresponder al total de la `Sale`.
- Cada pago conserva individualmente su método.
- Los pagos en efectivo afectan la `CashSession`.
- Los pagos con tarjeta y transferencia no afectan el efectivo físico.

### Financial Transactions

- Se estableció la trazabilidad entre `Payment` y `FinancialTransaction`.
- En pagos divididos, cada pago puede generar su movimiento financiero correspondiente.
- Se evita registrar nuevamente el total completo como ingreso adicional cuando ya se registraron los movimientos correspondientes a cada pago.

### Memberships

- Se estableció que `Membership` debe conservar `pricePaid`.
- Una membresía originada durante un checkout puede relacionarse con la `Sale` que la originó.
- Se mantiene el historial de membresías anteriores.

### Inventory

- Se aprobó el uso de `STOCK_ADJUSTED` para ajustes de inventario.
- Los ajustes requieren usuario y motivo.
- Los ajustes actualizan `ProductBatch.quantity`.
- Los ajustes deben quedar registrados en `AuditLog`.
- Un ajuste de stock no genera automáticamente un movimiento financiero.

### Product Batches

- `ProductBatch.batchNumber` es opcional.
- Cuando exista, la identificación del lote se considera dentro del producto.
- Se mantiene `Product.purchasePrice` como precio actual/de referencia.
- Se mantiene `ProductBatch.purchasePrice` como costo histórico real del lote.

### Documentation

- Se actualizaron las reglas de negocio para reflejar checkout combinado, pagos divididos, recibos, ajustes de stock y trazabilidad financiera.
- Se actualizaron los flujos principales.
- Se actualizaron las decisiones arquitectónicas.
- Se actualizaron `MASTER.md` y `PROJECT-STATUS.md`.
- Se actualizó este `CHANGELOG.md`.
- Se corrigió la referencia del antiguo ADR-020 que apuntaba a un ADR inexistente.
- Las nuevas decisiones continúan desde `ADR-032`.