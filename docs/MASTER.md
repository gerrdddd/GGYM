# GYM APP — Master Project Context

## 1. Propósito del documento

Este documento es el punto de entrada principal para comprender el proyecto GYM APP.

Antes de realizar cambios importantes en el proyecto, el agente debe leer este documento y consultar la documentación especializada correspondiente.

La documentación se divide por responsabilidades para evitar concentrar toda la información del proyecto en un único archivo.

---

# 2. Proyecto

**Nombre:** GYM APP

## Objetivo

Desarrollar un sistema web para administrar integralmente un gimnasio, centralizando:

- Clientes.
- Membresías.
- Control de acceso mediante huella.
- Personal.
- Productos e inventario.
- Ventas.
- Proveedores.
- Equipamiento.
- Finanzas.
- Tickets/comprobantes.
- Auditoría.
- Dashboard administrativo.

El sistema busca reducir errores operativos, facilitar la administración del gimnasio y proporcionar trazabilidad sobre las operaciones realizadas.

---

# 3. Stack tecnológico

## Frontend

- Next.js
- React
- TypeScript

## Backend

- NestJS
- Node.js
- TypeScript
- REST API

## Base de datos

- PostgreSQL

## ORM

- Prisma

## Infraestructura

- Docker
- Docker Compose

## Control de versiones

- Git
- GitHub

## Autenticación

- JWT
- Password hashing

## Validación

- DTOs
- class-validator

---

# 4. Arquitectura

La arquitectura principal es:

```text
Next.js
Frontend
    |
    | REST API
    v
NestJS
Backend
    |
    | Prisma
    v
PostgreSQL
```

El frontend no debe acceder directamente a PostgreSQL.

Toda operación relacionada con datos debe pasar por el backend NestJS.

Consultar:

- `docs/ARCHITECTURE.md`
- `docs/DATABASE.md`

---

# 5. Experiencias principales

El sistema tiene dos experiencias principales.

## 5.1 Estación de acceso

Es una computadora ubicada en recepción.

Flujo general:

```text
Pantalla de espera
        |
        v
Cliente coloca huella
        |
        v
Sistema identifica cliente
        |
        v
Consulta membresía
        |
        v
¿Membresía vigente?
      /       \
    SÍ         NO
    |           |
    v           v
Permitir      Denegar
acceso        acceso
```

La pantalla debe mostrar únicamente información básica para el cliente.

### Acceso permitido

```text
✓ ACCESO PERMITIDO

Bienvenido, Juan.

Membresía válida hasta:
15/10/2026
```

### Membresía vencida

```text
✕ ACCESO DENEGADO

Tu membresía venció:
15/09/2026
```

### Huella desconocida

```text
✕ HUELLA NO REGISTRADA
```

Después de unos segundos, la estación debe regresar automáticamente a la pantalla de espera.

---

# 6. Estación administrativa

La estación administrativa es utilizada por personal autorizado.

Flujo:

```text
Login
  |
  v
Email + contraseña
  |
  v
Autenticación
  |
  v
JWT
  |
  v
Validación de permisos
  |
  v
Dashboard
```

Las rutas administrativas deben estar protegidas.

---

# 7. Conceptos fundamentales

## 7.1 Puesto laboral != Rol del sistema

El puesto laboral de una persona y sus permisos dentro del sistema son conceptos diferentes.

Ejemplo:

```text
Employee
Puesto: RECEPTIONIST

UserAccount
Rol: RECEPTIONIST
```

Pero el sistema no debe asumir que un puesto laboral determina automáticamente todos los permisos.

Los permisos deben gestionarse mediante roles y permisos.

Consultar:

`docs/PERMISSIONS.md`

---

# 8. Cliente != Usuario administrativo

Un cliente del gimnasio no necesita necesariamente una cuenta administrativa.

El cliente utiliza principalmente:

```text
Fingerprint
    |
    v
Member
    |
    v
Membership
    |
    v
Access
```

No se debe crear un rol `CLIENT` únicamente para controlar el acceso mediante huella.

---

# 9. Módulos principales

El sistema contempla:

1. Configuración inicial.
2. Autenticación y autorización.
3. Clientes.
4. Control biométrico y acceso.
5. Membresías.
6. Personal.
7. Productos e inventario.
8. Proveedores.
9. Ventas.
10. Finanzas.
11. Corte de caja.
12. Tickets y comprobantes.
13. Equipamiento.
14. Dashboard.
15. Auditoría.

---

# 10. Prioridad general

La implementación debe seguir aproximadamente este orden:

## Fase 1 — Base técnica

- GitHub.
- NestJS.
- Next.js.
- Docker.
- PostgreSQL.
- Prisma.
- Estructura inicial.

## Fase 2 — Seguridad

- Usuarios.
- Login.
- JWT.
- Roles.
- Permisos.

## Fase 3 — Clientes y membresías

- Clientes.
- Planes.
- Membresías.
- Pagos.

## Fase 4 — Acceso

- Huella.
- Identificación.
- Validación de membresía.
- Pantalla de acceso.

## Fase 5 — Inventario

- Productos.
- Stock.
- Proveedores.
- Compras.
- Vencimientos.

## Fase 6 — Ventas

- Ventas.
- Pagos.
- Tickets.
- Movimientos del día.
- Correcciones.

## Fase 7 — Finanzas

- Ingresos.
- Gastos.
- Corte de caja.
- Reportes.

## Fase 8 — Equipamiento

- Equipos.
- Mantenimiento.
- Historial.

## Fase 9 — Dashboard y auditoría

- Dashboard.
- Auditoría.
- Indicadores.

---

# 11. Regla de trabajo

El desarrollo debe seguir:

```text
EPIC
  |
  v
USER STORY
  |
  v
TASKS
  |
  v
IMPLEMENTACIÓN
  |
  v
PRUEBAS
  |
  v
REVISIÓN
  |
  v
DONE
```

El agente no debe modificar funcionalidades fuera del ticket asignado sin indicarlo primero.

Si una historia requiere modificar varias entidades o módulos relacionados, puede hacerlo cuando sea necesario para completar correctamente el ticket, pero debe explicar el motivo.

---

# 12. Documentación relacionada

| Documento | Propósito |
|---|---|
| `ARCHITECTURE.md` | Arquitectura técnica |
| `DATABASE.md` | Modelo de datos y reglas de persistencia |
| `BUSINESS-RULES.md` | Reglas de negocio |
| `FLOWS.md` | Flujos principales |
| `PERMISSIONS.md` | Roles y permisos |
| `API.md` | API REST |
| `PROJECT-STATUS.md` | Estado actual |
| `CHANGELOG.md` | Cambios importantes |
| `DECISIONS.md` | Decisiones arquitectónicas |

---

# 13. Regla de precedencia

Cuando exista una diferencia entre documentación y código:

1. No asumir automáticamente que el código es correcto.
2. Identificar la diferencia.
3. Revisar el ticket que originó el cambio.
4. Revisar `DECISIONS.md`.
5. Informar la inconsistencia.
6. Resolverla antes de implementar cambios que dependan de ella.

---

# 14. Principios generales

El proyecto prioriza:

- Seguridad.
- Trazabilidad.
- Separación de responsabilidades.
- Validación de datos.
- Código modular.
- Mantenibilidad.
- Historial de operaciones.
- Integridad financiera.
- Integridad del inventario.
- Separación frontend/backend.

No introducir tecnologías o funcionalidades fuera del alcance sin autorización.

---

# 15. Fuente de referencia

El archivo:

`GGYM Documento Maestro.docx`

es el documento maestro original del proyecto.

Los archivos Markdown dentro de `docs/` representan la documentación operativa utilizada durante el desarrollo.

Cuando se tome una nueva decisión importante, debe documentarse en `docs/DECISIONS.md` y actualizar el documento especializado correspondiente.