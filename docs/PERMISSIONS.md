# GYM APP — Roles and Permissions

## 1. Principio

El sistema separa:

```text
Puesto laboral
        !=
Rol del sistema
        !=
Permiso
```

---

# 2. Puestos laborales

Los puestos laborales pertenecen al empleado.

Ejemplos contemplados:

- Recepción.
- Coach.
- Limpieza.
- Mantenimiento.

El puesto no debe utilizarse directamente como mecanismo de autorización.

---

# 3. Roles del sistema

Roles iniciales:

```text
ADMIN
RECEPTIONIST
COACH
CLEANING
MAINTENANCE
```

---

# 4. Cliente

No existe un rol administrativo `CLIENT`.

El cliente utiliza principalmente:

```text
Fingerprint
    ↓
Member
    ↓
Membership
    ↓
Access
```

---

# 5. Permisos

Permisos documentados inicialmente:

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

MANAGE_CASH

VIEW_AUDIT_LOG
```

El catálogo puede crecer conforme aparezcan nuevas necesidades.

---

# 6. Relación

```text
UserAccount
     |
     v
UserRole
     |
     v
Role
     |
     v
RolePermission
     |
     v
Permission
```

Roles y permisos son relaciones muchos a muchos.

---

# 7. Ejemplo RECEPTIONIST

Puede tener permisos como:

```text
VIEW_CLIENTS
MANAGE_CLIENTS
VIEW_MEMBERSHIPS
MANAGE_MEMBERSHIPS
CREATE_SALE
VIEW_TODAY_MOVEMENTS
MANAGE_CASH
```

---

# 8. Ejemplo ADMIN

Puede tener permisos administrativos y financieros, incluyendo:

```text
VIEW_FINANCIAL_REPORTS
MANAGE_FINANCES
VIEW_AUDIT_LOG
```

---

# 9. Operaciones sensibles

Las siguientes operaciones requieren especial atención:

- Cancelación de venta.
- Operaciones de caja y manejo de sesión de caja (apertura/cierre).
- Corrección de operación.
- Modificación financiera.
- Gestión de inventario.
- Consulta financiera.
- Auditoría.

La autorización definitiva debe basarse en permisos, no únicamente en el nombre del rol.

---

# 10. Regla de seguridad

No asumir:

```text
ADMIN = acceso automático a absolutamente todo
```

sin que el sistema lo modele explícitamente.

La implementación debe mantener una política clara y comprobable de autorización.

---

# 11. Regla para agentes

Cuando se agregue un nuevo permiso:

1. Documentarlo aquí.
2. Determinar qué roles pueden recibirlo.
3. Implementar el permiso.
4. Proteger las operaciones correspondientes.
5. Agregar pruebas.
6. Actualizar `DECISIONS.md` si cambia la política de seguridad.