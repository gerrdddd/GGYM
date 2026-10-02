# GYM APP — Architecture

## 1. Objetivo

Definir la arquitectura técnica del sistema GYM APP y las responsabilidades principales de cada componente.

---

# 2. Arquitectura general

```text
┌─────────────────────────────┐
│           Next.js           │
│          Frontend           │
└──────────────┬──────────────┘
               │
               │ REST API
               ▼
┌─────────────────────────────┐
│           NestJS            │
│           Backend           │
└──────────────┬──────────────┘
               │
               │ Prisma
               ▼
┌─────────────────────────────┐
│         PostgreSQL          │
└─────────────────────────────┘
```

---

# 3. Frontend

Tecnologías:

- Next.js
- React
- TypeScript

Responsabilidades:

- Interfaz de usuario.
- Navegación.
- Formularios.
- Validaciones de interfaz.
- Consumo de la API REST.
- Manejo de sesión del usuario.
- Presentación de información.
- Estación de acceso.
- Panel administrativo.

El frontend no debe contener acceso directo a PostgreSQL.

---

# 4. Backend

Tecnologías:

- NestJS.
- Node.js.
- TypeScript.
- REST API.

Responsabilidades:

- Reglas de negocio.
- Autenticación.
- Autorización.
- Validación.
- Acceso a datos mediante Prisma.
- Gestión de módulos.
- Operaciones transaccionales.
- Auditoría.
- Integración con dispositivos cuando corresponda.

---

# 5. Base de datos

Tecnología:

- PostgreSQL.

El acceso a PostgreSQL se realiza mediante Prisma.

El frontend nunca debe conectarse directamente a PostgreSQL.

---

# 6. ORM

Prisma es el ORM oficial del proyecto.

Responsabilidades:

- Modelado de datos.
- Consultas.
- Relaciones.
- Migraciones.
- Integración entre NestJS y PostgreSQL.

Los cambios de estructura de base de datos deben realizarse mediante migraciones.

---

# 7. Docker

Docker Compose administra el entorno local.

El entorno debe permitir ejecutar los servicios necesarios para desarrollo de manera reproducible.

Actualmente PostgreSQL forma parte del entorno Docker.

---

# 8. Separación de responsabilidades

La estructura conceptual es:

```text
Frontend
    ↓
Controllers
    ↓
Services
    ↓
Prisma
    ↓
PostgreSQL
```

Los controllers no deben contener toda la lógica de negocio.

Los services deben concentrar la lógica correspondiente al dominio.

El acceso a datos debe mantenerse encapsulado mediante Prisma.

---

# 9. Modularidad

NestJS debe organizarse por módulos funcionales.

Ejemplos:

```text
auth
users
clients
memberships
access
staff
products
inventory
suppliers
purchases
sales
finance
cash
tickets
equipment
dashboard
audit
```

La estructura definitiva puede evolucionar conforme se implementen los tickets.

No crear módulos sin necesidad funcional.

---

# 10. Seguridad

Las operaciones administrativas deben utilizar:

- Autenticación.
- JWT.
- Roles.
- Permisos.

Las operaciones sensibles deben validar permisos específicos.

---

# 11. Datos sensibles

Las contraseñas nunca deben almacenarse en texto plano.

Debe almacenarse un hash de contraseña.

La huella no debe tratarse como una contraseña.

La aplicación debe almacenar una referencia biométrica compatible con el dispositivo, no asumir que debe almacenar directamente la imagen de una huella.

---

# 12. Historial

Las operaciones importantes deben conservar historial.

No se deben eliminar físicamente operaciones financieras o ventas únicamente para corregir errores.

Deben utilizarse:

- Cancelaciones.
- Ajustes.
- Correcciones.
- Auditoría.

---

# 13. Cambios arquitectónicos

Cualquier modificación que cambie:

- Stack.
- Arquitectura.
- Flujo de datos.
- Seguridad.
- Modelo de persistencia.
- Integración externa.

debe registrarse en:

`docs/DECISIONS.md`

---

# 14. Regla para agentes

Antes de modificar arquitectura:

1. Revisar `MASTER.md`.
2. Revisar este documento.
3. Revisar el ticket.
4. Revisar `DECISIONS.md`.
5. Explicar el impacto.
6. Obtener autorización cuando el cambio esté fuera del alcance del ticket.