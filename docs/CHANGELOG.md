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
  - Next.js
  - NestJS
  - Prisma
  - PostgreSQL
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
- Se confirmó que no existe un rol CLIENT para acceso mediante huella.
- Se confirmó la conservación del historial de operaciones importantes.
- Se confirmó la necesidad de movimientos del día para corrección de operaciones.