# GYM APP — Project Status

## Última actualización

2026-10-01

---

# 1. Estado general

El proyecto se encuentra en etapa de preparación técnica y organización de documentación.

La arquitectura general y el modelo conceptual están definidos.

La implementación funcional completa todavía no ha comenzado.

---

# 2. Repositorio

Repositorio:

```text
GGYM
```

Rama principal:

```text
main
```

El proyecto utiliza Git y GitHub.

---

# 3. Frontend

Tecnología:

```text
Next.js
React
TypeScript
```

Estado:

```text
Configuración inicial realizada.
```

---

# 4. Backend

Tecnología:

```text
NestJS
Node.js
TypeScript
REST API
```

Estado:

```text
Configuración inicial realizada.
```

---

# 5. PostgreSQL

PostgreSQL está configurado mediante Docker.

Base de datos de desarrollo:

```text
gym_db
```

Usuario:

```text
gym_user
```

Puerto local:

```text
5432
```

---

# 6. Docker

Docker y Docker Compose están configurados.

El servicio PostgreSQL se ejecuta mediante Docker Compose.

---

# 7. Prisma

Prisma está instalado y configurado.

Versiones utilizadas actualmente:

```text
Prisma 7.10.0
@prisma/client 7.10.0
```

El proyecto debe mantener las versiones alineadas.

El schema definitivo todavía no debe considerarse terminado.

---

# 8. Base de datos

Estado:

```text
Base de datos vacía / sin tablas de dominio implementadas.
```

No debe realizarse `db pull` contra la base vacía.

Primero se debe completar y revisar el modelo de datos.

Después:

```text
DATABASE.md
   ↓
schema.prisma
   ↓
prisma validate
   ↓
migration
```

---

# 9. Documentación

Documento maestro:

```text
GGYM Documento Maestro.docx
```

Documentación operativa:

```text
docs/
```

La documentación operativa contiene actualmente las decisiones sobre:

- checkout combinado;
- pagos divididos;
- recibos;
- inventario;
- FEFO;
- devoluciones;
- ajustes de stock;
- caja;
- auditoría;
- historial.

---

# 10. Estado funcional

Todavía no se consideran implementados completamente:

- Autenticación.
- Autorización.
- Clientes.
- Membresías.
- Biometría.
- Inventario.
- Ventas.
- Finanzas.
- Caja.
- Tickets.
- Equipamiento.
- Dashboard.
- Auditoría.

---

# 11. Decisiones recientes documentadas

Se han tomado y documentado las siguientes decisiones clave antes de la implementación:

- **Modelo relacional:** modelo cerrado y revisado.
- **Restricciones:** restricciones de integridad revisadas.
- **Documentación:** documentación sincronizada.
- **Inventario:** no existe una entidad `Inventory` independiente.
- **Stock:** se gestiona mediante `ProductBatch.quantity`.
- **Consumo:** se utiliza FEFO.
- **Lotes:** `batchNumber` es opcional y se distingue por producto.
- **Costos:** `Product.purchasePrice` representa el precio actual/de referencia y `ProductBatch.purchasePrice` conserva el costo histórico real.
- **Ventas:** utilizan arquitectura de carrito.
- **Checkout:** una `Sale` puede contener productos y una membresía.
- **Pagos:** una `Sale` puede tener múltiples `Payment`.
- **Métodos de pago:** únicamente efectivo, tarjeta y transferencia.
- **Finanzas:** los movimientos derivados de pagos mantienen trazabilidad individual.
- **Tickets:** `Receipt` pertenece al checkout representado por `Sale`.
- **Devoluciones:** se utilizan `Return` y `ReturnItem`.
- **Trazabilidad de lotes:** se utiliza `SaleItemBatch`.
- **Ajustes de stock:** se auditan mediante `STOCK_ADJUSTED`.
- **Caja:** `CashSession` está asociada al usuario responsable.
- **Caja:** requiere `MANAGE_CASH`.
- **UI:** existen alertas accionables en Dashboard.
- **Clientes:** existe historial visual.

---

# 12. Próximo objetivo

Preparar y validar:

1. `schema.prisma` (diseño e implementación).
2. `prisma validate`.
3. Primera migración.
4. Validación de Prisma.
5. Estructura inicial de módulos NestJS.

---

# 13. Regla

Este archivo debe actualizarse después de avances significativos.

No utilizar este documento para registrar cada pequeño cambio de código.

Para cambios detallados utilizar:

```text
CHANGELOG.md
```