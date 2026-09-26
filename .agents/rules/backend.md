---
trigger: glob
globs: "backend/**/*.{ts,js}"
description: "Reglas para desarrollo del backend NestJS."
---

# Backend Rules

- El backend utiliza NestJS y TypeScript.
- La API es REST.
- Utilizar módulos de NestJS para separar responsabilidades.
- Controllers reciben y delegan operaciones.
- Services concentran lógica de negocio.
- Utilizar DTOs para entradas de API.
- Utilizar `class-validator` cuando corresponda.
- Proteger rutas administrativas mediante autenticación.
- Validar permisos para operaciones sensibles.
- No acceder a PostgreSQL directamente desde controllers.
- Utilizar Prisma para persistencia.
- Mantener manejo consistente de errores.
- No modificar reglas de negocio sin revisar `docs/BUSINESS-RULES.md`.
- Consultar `docs/API.md` cuando se agreguen o modifiquen endpoints.