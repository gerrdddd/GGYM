---
trigger: glob
globs: "backend/**/*.prisma,backend/prisma/**/*"
description: "Reglas para Prisma y PostgreSQL."
---

# Database Rules

- PostgreSQL es la base de datos oficial.
- Prisma es el ORM oficial.
- Consultar `docs/DATABASE.md` antes de modificar el modelo.
- No crear modelos únicamente por conveniencia de implementación.
- Mantener relaciones explícitas.
- Utilizar UUID para entidades principales.
- Utilizar Decimal para dinero.
- No utilizar Float para valores monetarios.
- Utilizar migraciones para cambios estructurales.
- No realizar resets destructivos sin autorización.
- Preservar datos históricos importantes.
- Revisar impacto de cambios en relaciones.
- Revisar índices y restricciones cuando corresponda.
- Ejecutar validaciones de Prisma después de cambios relevantes.
- Si el modelo actual contradice `docs/DATABASE.md`, detenerse y reportar la diferencia.