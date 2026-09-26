# GYM APP — REST API

## 1. Propósito

Documentar la API REST del backend NestJS.

Este documento debe mantenerse sincronizado con la implementación.

No documentar endpoints como implementados si todavía no existen.

---

# 2. Arquitectura

```text
Next.js
   |
   | HTTP / REST
   v
NestJS
   |
   v
Prisma
   |
   v
PostgreSQL
```

---

# 3. Convenciones

Los endpoints deben seguir una estructura REST coherente.

Ejemplos conceptuales:

```text
GET    /clients
POST   /clients
GET    /clients/:id
PATCH  /clients/:id
DELETE /clients/:id
```

La estructura definitiva debe reflejar la implementación real.

---

# 4. Autenticación

Las rutas administrativas protegidas deben utilizar autenticación mediante JWT.

Conceptualmente:

```text
Authorization: Bearer <token>
```

---

# 5. Validación

Las entradas de API deben validarse mediante DTOs y `class-validator` cuando corresponda.

---

# 6. Errores

La API debe proporcionar respuestas de error consistentes.

Como mínimo debe distinguirse entre:

- Error de validación.
- No autenticado.
- No autorizado.
- Recurso no encontrado.
- Conflicto de negocio.
- Error interno.

Los códigos HTTP definitivos deben seguir las convenciones de NestJS y las necesidades del dominio.

---

# 7. Módulos previstos

La API podrá incluir módulos para:

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

---

# 8. Regla de documentación

Cuando se complete un ticket que agregue o modifique endpoints:

Actualizar:

- Método HTTP.
- Ruta.
- Autenticación requerida.
- Permiso requerido.
- Parámetros.
- Body.
- Respuesta.
- Errores relevantes.

---

# 9. Regla para agentes

No crear endpoints únicamente para "llenar" este documento.

Primero se implementa la necesidad funcional.

Después se documenta la API real.