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

---

# 3. Membresías

- Una membresía tiene fecha de inicio.
- Una membresía tiene fecha de finalización.
- El acceso depende de la vigencia de la membresía.
- Una membresía vencida no permite acceso.
- Una renovación debe quedar registrada.
- El historial anterior no debe sobrescribirse.

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

---

# 5. Biometría

- La huella no debe tratarse como contraseña.
- El sistema debe utilizar una referencia biométrica.
- La integración con hardware debe estar desacoplada.
- No asumir que el dominio necesita almacenar directamente una imagen de huella.

---

# 6. Ventas

Una venta confirmada debe:

1. Registrar la venta.
2. Registrar sus productos.
3. Validar disponibilidad.
4. Descontar inventario.
5. Registrar el pago.
6. Registrar el ingreso financiero correspondiente.
7. Generar ticket cuando corresponda.
8. Registrar auditoría cuando corresponda.

---

# 7. Venta de múltiples productos

Una venta puede contener múltiples productos.

```text
Sale
 ├── SaleItem
 ├── SaleItem
 └── SaleItem
```

El total debe calcularse a partir de los conceptos correspondientes.

---

# 8. Productos vencidos

Un producto vencido no puede venderse.

El sistema debe detectar:

- Productos vencidos.
- Productos próximos a vencer.
- Productos con stock bajo.

---

# 9. Inventario

Las entradas y salidas de inventario deben conservar trazabilidad.

Las ventas deben disminuir inventario.

Las compras a proveedores deben aumentar inventario.

Las correcciones de ventas deben ajustar inventario cuando corresponda.

---

# 10. Proveedores y compras

El flujo conceptual es:

```text
Proveedor
 ↓
Compra
 ↓
PurchaseItems
 ↓
Entrada de inventario
 ↓
Gasto financiero
```

---

# 11. Finanzas

Una venta genera un ingreso.

Una compra a proveedor genera un gasto.

Un mantenimiento con costo puede generar un gasto.

Los movimientos financieros deben conservarse.

---

# 12. Dashboard financiero

El dashboard operativo general muestra:

```text
INGRESOS DE HOY
```

Los reportes financieros más amplios requieren permisos.

Los reportes contemplados son:

- Hoy.
- Esta semana.
- Este mes.
- Este año.
- Rango personalizado.

---

# 13. Movimientos del día

El recepcionista debe poder consultar las operaciones realizadas durante el día para detectar errores.

Puede incluir:

- Ventas.
- Membresías.
- Renovaciones.
- Otros ingresos.

Debe poder consultar el detalle y, según sus permisos, corregir o cancelar operaciones.

---

# 14. Correcciones

Una corrección debe:

1. Identificar la operación original.
2. Mantener la operación histórica.
3. Registrar el motivo.
4. Registrar al usuario responsable.
5. Realizar los ajustes necesarios.
6. Mantener trazabilidad.
7. Registrar auditoría.

No eliminar silenciosamente la operación original.

---

# 15. Cancelaciones

Las cancelaciones deben conservar historial.

Una cancelación puede requerir:

- Motivo.
- Usuario.
- Fecha/hora.
- Ajuste de inventario.
- Ajuste financiero.
- Actualización del ticket/comprobante.
- Registro de auditoría.

---

# 16. Tickets

El sistema puede generar tickets internos.

Un ticket interno no significa automáticamente CFDI fiscal.

La facturación fiscal mexicana deberá tratarse como una integración/módulo independiente si posteriormente se requiere.

---

# 17. Seguridad

- Las rutas administrativas requieren autenticación.
- Las operaciones sensibles requieren permisos.
- Las contraseñas deben almacenarse como hashes.
- Los cambios importantes deben quedar auditados.
- Los clientes no requieren necesariamente una cuenta administrativa.

---

# 18. Historial

El sistema debe priorizar trazabilidad.

No se deben eliminar físicamente operaciones importantes solamente para corregir errores.

---

# 19. Regla para agentes

Si una implementación parece contradecir una regla de este documento:

1. Detener la implementación relacionada.
2. Identificar la regla.
3. Explicar el conflicto.
4. Revisar `DECISIONS.md`.
5. Solicitar autorización para cambiar la regla.

No modificar reglas de negocio silenciosamente.