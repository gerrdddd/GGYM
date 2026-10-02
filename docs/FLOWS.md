# GYM APP — System Flows

## 1. Cliente nuevo

```text
Recepcionista
     |
     v
Registrar cliente
     |
     v
Tomar fotografía
     |
     v
Registrar huella
     |
     v
Seleccionar membresía
     |
     v
Abrir checkout
     |
     ├──> Membresía
     ├──> Productos opcionales
     |
     v
Calcular total
     |
     v
Registrar uno o varios pagos
     |
     v
Confirmar Sale
     |
     ├──> Membership
     ├──> SaleItems
     ├──> Payment(s)
     ├──> FinancialTransaction(s)
     ├──> Receipt
     └──> AuditLog
     |
     v
Cliente activo
```

---

# 2. Acceso de cliente habitual

```text
Cliente
  |
  v
Huella
  |
  v
Identificación
  |
  v
Member
  |
  v
Membership
  |
  v
¿Vigente?
 /       \
Sí       No
|         |
v         v
Permitir  Denegar
acceso    acceso
```

Después:

```text
Resultado
   |
   v
Registrar AccessLog
   |
   v
Mostrar mensaje
   |
   v
Esperar unos segundos
   |
   v
Pantalla de espera
```

---

# 3. Acceso con huella desconocida

```text
Huella
  |
  v
Buscar referencia
  |
  v
¿Existe?
  |
 NO
  |
  v
HUELLA NO REGISTRADA
  |
  v
Registrar AccessLog
  |
  v
Regresar a espera
```

---

# 4. Login administrativo

```text
Email + contraseña
       |
       v
Validar credenciales
       |
       v
Generar JWT
       |
       v
Solicitar recurso
       |
       v
Validar JWT
       |
       v
Validar permisos
       |
       v
Ejecutar operación
```

---

# 5. Registrar venta con carrito

```text
Recepcionista
     |
     v
Nueva Sale
     |
     v
Abrir carrito
     |
     v
Buscar / Escanear producto
     |
     v
Agregar al carrito
     |
     v
Modificar cantidades (+/-)
     |
     v
Eliminar productos si corresponde
     |
     v
Validar stock
     |
     v
Validar productos vencidos
     |
     v
Calcular subtotal
     |
     v
Aplicar descuento opcional
     |
     v
Recalcular total
     |
     v
Seleccionar uno o varios métodos de pago
     |
     v
Confirmar
     |
     v
Registrar Sale
     |
     ├──> SaleItems
     |
     ├──> ProductBatch decrease
     |
     ├──> SaleItemBatch
     |
     ├──> Payment(s)
     |
     ├──> FinancialTransaction(s)
     |
     ├──> Receipt
     |
     └──> AuditLog
```

Las operaciones relacionadas deben ejecutarse manteniendo consistencia transaccional.

---

# 6. Checkout de membresía + productos

El sistema debe permitir que una membresía y productos sean adquiridos dentro de la misma operación.

Ejemplo:

```text
Cliente
   |
   v
Nueva Sale
   |
   ├──> Membresía $500
   |
   ├──> Proteína $850
   |
   └──> Bebida $30
   |
   v
Subtotal
   |
   v
Descuento
   |
   v
Total
   |
   v
Payment(s)
   |
   ├──> $500 CASH
   └──> $880 CARD
   |
   v
Confirmar
   |
   ├──> Sale
   ├──> Membership
   ├──> SaleItems
   ├──> Payment(s)
   ├──> FinancialTransaction(s)
   └──> Receipt
```

No se deben crear dos operaciones comerciales independientes solamente por combinar una membresía y productos.

---

# 7. Pago dividido

```text
Sale
 |
 | Total = $1,300
 |
 +── Payment #1
 |      $500 CASH
 |
 +── Payment #2
        $800 CARD
```

Reglas:

```text
SUM(Payments.amount) = Sale.total
```

Los movimientos financieros derivados de cada pago deben conservar su método.

```text
Payment CASH
   |
   └──> FinancialTransaction
             |
             └──> CashSession

Payment CARD
   |
   └──> FinancialTransaction
             |
             └──> No CashSession
```

---

# 8. Compra a proveedor

```text
Proveedor
   |
   v
Nueva compra
   |
   v
Productos
   |
   v
Cantidades
   |
   v
Costo
   |
   v
Lote
   |
   v
Confirmar
   |
   ├──> Purchase
   |
   ├──> PurchaseItems
   |
   ├──> ProductBatch increase
   |
   └──> FinancialTransaction EXPENSE
```

El costo histórico del lote debe conservarse en `ProductBatch.purchasePrice`.

---

# 9. Consumo FEFO

```text
Producto
   |
   v
Buscar ProductBatch
   |
   v
Excluir lotes vencidos
   |
   v
Ordenar por expirationDate ASC
   |
   v
Seleccionar lote más próximo a vencer
   |
   v
¿Cantidad suficiente?
   /             \
 Sí               No
 |                 |
 v                 v
Consumir       Consumir lote
cantidad       completa
                 |
                 v
              Siguiente lote
                 |
                 v
              Continuar
```

Cada consumo debe quedar registrado mediante:

```text
SaleItem
   |
   v
SaleItemBatch
   |
   v
ProductBatch
```

---

# 10. Ajuste de stock

```text
Usuario autorizado
       |
       v
Seleccionar lote
       |
       v
Indicar cantidad
       |
       v
Indicar motivo
       |
       v
Validar permiso
       |
       v
Actualizar ProductBatch.quantity
       |
       v
Registrar AuditLog
       |
       └──> action = STOCK_ADJUSTED
```

Ejemplos:

```text
Producto roto
Producto vencido
Producto perdido
Producto inutilizable
Corrección de conteo
```

El ajuste no genera automáticamente un movimiento financiero.

---

# 11. Corrección y devolución de venta

```text
Movimientos del día / Historial
       |
       v
Seleccionar operación
       |
       v
Ver detalle
       |
       v
Corregir / Cancelar / Devolución
       |
       v
Introducir motivo
       |
       v
Validar permisos
       |
       v
Realizar ajuste
       |
       ├──> Return / ReturnItem
       ├──> ProductBatch
       ├──> FinancialTransaction
       ├──> Receipt cuando corresponda
       └──> AuditLog
```

La venta original debe conservarse físicamente.

---

# 12. Renovación de membresía

La renovación forma parte de un checkout.

```text
Cliente
   |
   v
Seleccionar plan
   |
   v
Nueva Sale
   |
   v
Registrar Membership
   |
   v
Registrar uno o varios Payment
   |
   v
Confirmar Sale
   |
   ├──> Membership
   ├──> Payment(s)
   ├──> FinancialTransaction(s)
   └──> Receipt
```

El historial de membresías anteriores debe conservarse.

El precio pagado debe conservarse en la nueva membresía.

---

# 13. Corte de caja

Las operaciones de caja están estrictamente asociadas al usuario responsable y requieren `MANAGE_CASH`.

```text
Usuario abre caja
(MANAGE_CASH)
       |
       v
Registrar apertura
       |
       v
Registrar operaciones CASH
       |
       v
Calcular efectivo esperado
       |
       v
Contar efectivo real
       |
       v
Comparar
       |
       v
Diferencia
       |
       v
Usuario cierra caja
(MANAGE_CASH)
```

Conceptualmente:

```text
difference =
actualAmount - expectedAmount
```

La fórmula definitiva debe implementarse de forma consistente con las reglas de caja.

---

# 14. Dashboard y Alertas Accionables

El dashboard operativo puede mostrar:

```text
Clientes activos
Membresías por vencer
Productos por vencer
Stock bajo
Equipos en mantenimiento
Ingresos de hoy
```

Las alertas deben permitir navegar directamente a la sección correspondiente cuando la funcionalidad esté implementada.

Los reportes financieros completos dependen de permisos.

---

# 15. Auditoría

Las operaciones importantes deben producir información suficiente para responder:

```text
¿Quién?
¿Qué hizo?
¿Cuándo?
¿Sobre qué?
¿Por qué?
```

cuando el tipo de operación requiera motivo.

Ejemplos de acciones auditables:

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