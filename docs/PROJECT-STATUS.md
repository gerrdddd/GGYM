# GYM APP — Project Status

## Última actualización

2026-09-25

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

Versión utilizada actualmente:

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

El siguiente paso no es realizar `db pull`.

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

Documentación maestra:

```text
GGYM Documento Maestro.docx
```

Documentación operativa:

```text
docs/
```

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

# 11. Próximo objetivo

Preparar y validar:

1. Documentación.
2. Modelo relacional.
3. `schema.prisma`.
4. Primera migración.
5. Validación de Prisma.
6. Estructura de módulos NestJS.

---

# 12. Regla

Este archivo debe actualizarse después de avances significativos.

No utilizar este documento para registrar cada pequeño cambio de código.

Para cambios detallados utilizar:

`CHANGELOG.md`