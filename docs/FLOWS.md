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
Registrar pago
     |
     v
Generar ticket
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

# 5. Registrar venta

```text
Recepcionista
      |
      v
Nueva venta
      |
      v
Seleccionar productos
      |
      v
Seleccionar cantidades
      |
      v
Validar stock
      |
      v
Validar productos vencidos
      |
      v
Calcular total
      |
      v
Seleccionar método de pago
      |
      v
Confirmar
      |
      v
Registrar Sale
      |
      ├──> SaleItems
      |
      ├──> Inventory decrease
      |
      ├──> Payment
      |
      ├──> FinancialTransaction
      |
      ├──> Receipt
      |
      └──> AuditLog
```

Las operaciones relacionadas deben mantener consistencia.

---

# 6. Compra a proveedor

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
Confirmar
   |
   ├──> Purchase
   |
   ├──> PurchaseItems
   |
   ├──> Inventory increase
   |
   └──> FinancialTransaction EXPENSE
```

---

# 7. Corrección de venta

```text
Movimientos del día
       |
       v
Seleccionar operación
       |
       v
Ver detalle
       |
       v
Corregir / cancelar
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
       ├──> Inventario
       ├──> Finanzas
       ├──> Ticket
       └──> Auditoría
```

La operación original debe conservarse.

---

# 8. Renovación de membresía

```text
Cliente
  |
  v
Seleccionar plan
  |
  v
Registrar renovación
  |
  v
Registrar pago
  |
  v
Crear/actualizar nueva Membership
  |
  v
Registrar FinancialTransaction
  |
  v
Generar Receipt
```

El historial de membresías anteriores debe conservarse.

---

# 9. Corte de caja

```text
Abrir caja
   |
   v
Registrar operaciones
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
Cerrar caja
```

Conceptualmente:

```text
difference =
actualAmount - expectedAmount
```

La fórmula definitiva debe implementarse de forma consistente con las reglas de caja.

---

# 10. Dashboard

El dashboard operativo puede mostrar:

```text
Clientes activos
Membresías por vencer
Productos por vencer
Stock bajo
Equipos en mantenimiento
Ingresos de hoy
```

Los reportes financieros completos deben depender de permisos.

---

# 11. Auditoría

Las operaciones importantes deben producir información suficiente para responder:

```text
¿Quién?
¿Qué hizo?
¿Cuándo?
¿Sobre qué?
¿Por qué?
```

cuando el tipo de operación requiera motivo.