# GYM APP — Business Rules

## 1. Propósito

Este documento contiene reglas de negocio que no deben cambiarse arbitrariamente durante la implementación.

---

# 2. Clientes

- Un cliente puede tener historial de membresías.
- Un cliente puede tener una referencia biométrica.
- Un cliente no necesita cuenta administrativa.
- Los datos históricos deben conservarse.
- La baja de un cliente debe evaluarse preferentemente como baja lógica.
- **Historial visual:** El sistema debe permitir consultar visualmente información histórica relevante:
  - información personal;
  - membresías anteriores y actuales;
  - historial de acceso;
  - información relevante de recepción.

---

# 3. Membresías

- Una membresía tiene fecha de inicio.
- Una membresía tiene fecha de finalización.
- El acceso depende de la vigencia de la membresía.
- Una membresía vencida no permite acceso.
- Una renovación debe quedar registrada.
- El historial anterior no debe sobrescribirse.
- El precio pagado por una membresía debe conservarse históricamente mediante `pricePaid`.
- Una membresía originada mediante un checkout debe poder relacionarse con la `Sale` que originó la operación.
- Una membresía puede formar parte del mismo checkout que una o varias compras de productos.

Ejemplo:

```text
Cliente
   ↓
Sale
   ├── Membership
   ├── SaleItem → Producto
   ├── Payment
   └── Receipt
```

La membresía sigue siendo una entidad independiente para controlar vigencia, historial y acceso.

---

# 4. Acceso

El flujo es:

```text
Huella
  ↓
Identificación
  ↓
Member
  ↓
Membership
  ↓
Validación de vigencia
  ↓
Permitir / Denegar
```

Resultados principales:

```text
ACCESS_GRANTED
MEMBERSHIP_EXPIRED
FINGERPRINT_NOT_REGISTERED
```

La implementación puede utilizar otros enums internos, pero debe preservar estos significados.

Después de mostrar el resultado, la estación debe regresar a la pantalla de espera.

---

# 5. Biometría

- La huella no debe tratarse como contraseña.
- El sistema debe utilizar una referencia biométrica.
- La referencia debe ser adecuada para búsquedas rápidas.
- La referencia biométrica debe ser única dentro del sistema.
- La integración con hardware debe estar desacoplada.
- No asumir que el dominio necesita almacenar directamente una imagen de huella.
- La implementación inicial puede utilizar una referencia simulada mientras no exista integración con hardware real.

---

# 6. Ventas y Carrito

- El proceso de venta debe utilizar un **carrito de compras**.
- Debe permitir:
  - agregar productos;
  - modificar cantidades;
  - eliminar productos.
- El sistema debe recalcular el subtotal.
- Puede aplicar descuentos.
- El sistema debe recalcular el total después del descuento.
- Una venta confirmada debe conservar `subtotal`, `discount` y `total`.

Una `Sale` representa el **checkout comercial completo**.

Una misma `Sale` puede contener:

- cero o más `SaleItem`;
- opcionalmente una membresía adquirida o renovada;
- uno o más `Payment`;
- un `Receipt`.

Por lo tanto, una operación puede ser:

```text
Sale
 ├── Productos
 └── Membresía
```

o solamente:

```text
Sale
 └── Productos
```

o:

```text
Sale
 └── Membresía
```

Una venta confirmada debe:

1. Registrar la venta.
2. Registrar sus productos cuando existan.
3. Registrar la membresía cuando corresponda.
4. Validar disponibilidad.
5. Descontar inventario cuando existan productos.
6. Registrar uno o varios pagos.
7. Validar que la suma de los pagos corresponda al total de la operación.
8. Registrar los movimientos financieros correspondientes.
9. Generar el ticket interno.
10. Registrar auditoría cuando corresponda.

Las operaciones relacionadas deben ejecutarse manteniendo consistencia transaccional.

---

# 7. Venta de múltiples productos

Una venta puede contener múltiples productos.

```text
Sale
 ├── SaleItem
 ├── SaleItem
 └── SaleItem
```

El total debe calcularse a partir de los conceptos correspondientes y considerando posibles descuentos.

---

# 8. Membresía + productos en el mismo checkout

El sistema debe permitir que un cliente:

1. compre o renueve una membresía;
2. compre uno o más productos;
3. pague todo dentro de una misma `Sale`.

Ejemplo:

```text
Membresía       $500
Proteína        $850
Bebida           $30
--------------------
Subtotal       $1,380
Descuento          $80
--------------------
Total          $1,300
```

La operación debe conservar un único checkout:

```text
Sale #125
 ├── Membership
 ├── SaleItem → Proteína
 ├── SaleItem → Bebida
 ├── Payment
 └── Receipt
```

No se debe obligar al sistema a crear dos ventas independientes solamente porque la operación contiene una membresía y productos.

---

# 9. Pagos

Una `Sale` puede tener uno o varios pagos.

Ejemplo:

```text
Total de Sale: $1,300

Payment #1
$500 EFECTIVO

Payment #2
$800 TARJETA
```

Reglas:

- La suma de los pagos debe corresponder al total de la `Sale`.
- Cada pago debe conservar su método.
- Los métodos iniciales son:
  - `CASH`;
  - `CARD`;
  - `TRANSFER`.
- Un pago en efectivo afecta la `CashSession` correspondiente.
- Un pago con tarjeta o transferencia no afecta el efectivo físico.
- Cada pago debe poder identificarse individualmente.

La capacidad de dividir un pago no debe provocar que se registre dos veces el ingreso total.

---

# 10. Productos y Códigos de Barras

- El código de barras en un producto es **opcional**.
- Si existe un código de barras, debe ser único.
- El sistema debe permitir la venta mediante:
  - escaneo;
  - ingreso manual;
  - búsqueda por nombre;
  - búsqueda por categoría.
- Un producto vencido no puede venderse.

El sistema debe detectar:

- productos vencidos;
- productos próximos a vencer;
- productos con stock bajo.

---

# 11. Inventario

- **No existirá una entidad `Inventory` independiente.**
- El stock se gestiona mediante `ProductBatch.quantity`.
- Las entradas y salidas deben conservar trazabilidad.
- Las compras a proveedores deben aumentar inventario.
- Las ventas deben disminuir inventario.
- Las devoluciones deben incrementar inventario cuando corresponda.
- Los ajustes manuales deben conservar usuario, motivo y trazabilidad.
- Las correcciones no deben modificar silenciosamente el historial.

## FEFO

Cuando una venta u operación necesite descontar stock:

1. Buscar lotes disponibles.
2. Excluir lotes vencidos.
3. Ordenar lotes válidos por fecha de vencimiento ascendente.
4. Consumir primero el lote cuyo vencimiento sea más próximo.
5. Si no existe cantidad suficiente, continuar con el siguiente lote.
6. Registrar exactamente qué cantidad salió de cada lote mediante `SaleItemBatch`.

---

# 12. Ajustes de inventario

El sistema debe permitir ajustes de stock cuando exista una causa operacional que no corresponda a una venta o compra.

Ejemplos:

- producto roto;
- producto echado a perder;
- producto vencido;
- producto perdido;
- producto inutilizable;
- corrección de conteo físico.

El ajuste debe:

1. Identificar el lote afectado.
2. Registrar la cantidad ajustada.
3. Registrar el usuario responsable.
4. Registrar el motivo.
5. Actualizar `ProductBatch.quantity`.
6. Registrar `AuditLog` con la acción `STOCK_ADJUSTED`.

Un ajuste de stock no representa automáticamente un movimiento financiero.

---

# 13. Proveedores y compras

El flujo conceptual es:

```text
Proveedor
   ↓
Compra
   ↓
PurchaseItems
   ↓
ProductBatch
   ↓
Incremento de stock
   ↓
Gasto financiero
```

Una compra debe conservar el costo histórico real de adquisición.

---

# 14. Precios de productos y lotes

`Product.purchasePrice` representa el precio de compra actual o de referencia del producto.

`ProductBatch.purchasePrice` representa el costo real de adquisición de ese lote.

Para cálculos históricos relacionados con inventario o costo de adquisición debe utilizarse el precio almacenado en `ProductBatch`.

Esto evita perder el costo histórico cuando el proveedor cambia sus precios.

---

# 15. Lotes

`ProductBatch.batchNumber` es opcional.

Cuando exista:

- identifica el lote proporcionado;
- no necesita ser globalmente único;
- debe poder distinguirse dentro del producto correspondiente.

La combinación lógica será:

```text
(productId, batchNumber)
```

cuando `batchNumber` exista.

---

# 16. Finanzas y Caja

- Una venta genera ingresos.
- Una compra a proveedor genera un gasto.
- Un mantenimiento con costo puede generar un gasto.
- Los movimientos financieros deben conservarse.
- Los movimientos financieros derivados de pagos deben conservar el método de pago correspondiente.
- Un pago en efectivo debe poder asociarse a la `CashSession` correspondiente.
- Los pagos con tarjeta o transferencia pueden existir sin `CashSession`.

## Caja

Toda operación de caja está estrictamente asociada al usuario que maneja la sesión.

Se debe conocer:

- quién abrió;
- quién cerró;
- apertura;
- efectivo esperado;
- efectivo real;
- diferencia;
- timestamps.

El manejo de caja requiere explícitamente:

```text
MANAGE_CASH
```

Tener:

```text
CREATE_SALE
```

no otorga automáticamente permiso para administrar caja.

---

# 17. Dashboard financiero y Alertas

El dashboard operativo general muestra:

```text
INGRESOS DE HOY
```

También puede mostrar:

- productos con stock bajo;
- productos próximos a vencer;
- membresías próximas a vencer;
- equipos en mantenimiento.

Las alertas deben ser accionables y permitir navegar a la sección relevante cuando esa funcionalidad esté implementada.

Los reportes financieros más amplios requieren permisos.

Reportes contemplados:

- hoy;
- esta semana;
- este mes;
- este año;
- rango personalizado.

---

# 18. Movimientos del día

El recepcionista debe poder consultar las operaciones realizadas durante el día.

Puede incluir:

- ventas;
- membresías;
- renovaciones;
- otros ingresos.

Debe poder consultar el detalle y, según sus permisos, corregir o cancelar operaciones.

---

# 19. Correcciones y Devoluciones

## Devoluciones

Las devoluciones son una funcionalidad del módulo de ventas.

Se implementan mediante:

```text
Return
ReturnItem
```

La venta original no debe modificarse físicamente para representar la devolución.

Cada `ReturnItem` debe identificar el `batchId` afectado.

Una devolución puede generar:

1. incremento de stock;
2. ajuste financiero;
3. actualización del comprobante cuando corresponda;
4. auditoría obligatoria.

La auditoría debe identificar:

```text
Quién
Qué
Sobre qué entidad
Cuándo
Por qué
```

## Correcciones

Una corrección genérica debe:

1. Identificar la operación original.
2. Mantener la operación histórica.
3. Registrar el motivo.
4. Registrar al usuario responsable.
5. Realizar los ajustes necesarios.
6. Mantener trazabilidad.
7. Registrar auditoría.

---

# 20. Cancelaciones

Las cancelaciones deben conservar historial.

Una cancelación puede requerir:

- motivo;
- usuario;
- fecha/hora;
- ajuste de inventario;
- ajuste financiero;
- actualización del ticket;
- auditoría.

Nunca se debe eliminar silenciosamente una operación importante.

---

# 21. Tickets

El sistema puede generar tickets internos.

Una `Receipt` corresponde al checkout representado por una `Sale`.

Por lo tanto:

```text
Sale
  ↓
Receipt
```

Una operación que incluya productos y membresía puede tener un solo ticket:

```text
Sale
 ├── Membership
 ├── SaleItems
 ├── Payments
 └── Receipt
```

Un ticket interno no significa automáticamente CFDI fiscal.

La facturación fiscal mexicana deberá tratarse como una integración/módulo independiente si posteriormente se requiere.

---

# 22. Seguridad y Auditoría

- Las rutas administrativas requieren autenticación.
- Las operaciones sensibles requieren permisos.
- Las operaciones de caja requieren `MANAGE_CASH`.
- Las contraseñas deben almacenarse como hashes.
- Las cancelaciones deben auditarse.
- Las devoluciones deben auditarse.
- Los ajustes de inventario deben auditarse.
- Las modificaciones financieras sensibles deben auditarse.
- Los clientes no requieren necesariamente una cuenta administrativa.

---

# 23. Historial

El sistema debe priorizar trazabilidad.

No se deben eliminar físicamente operaciones importantes solamente para corregir errores.

Esto incluye, entre otras:

- ventas;
- pagos;
- movimientos financieros;
- accesos;
- membresías;
- devoluciones;
- auditoría;
- operaciones de caja.

---

# 24. Regla para agentes

Si una implementación parece contradecir una regla de este documento:

1. Detener la implementación relacionada.
2. Identificar la regla.
3. Explicar el conflicto.
4. Revisar `DECISIONS.md`.
5. Solicitar autorización para cambiar la regla.

No modificar reglas de negocio silenciosamente.