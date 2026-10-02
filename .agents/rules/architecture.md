---
trigger: always_on
description: "Reglas generales de arquitectura del proyecto GYM APP."
---

# Architecture Rules

- La arquitectura oficial es Next.js → NestJS → Prisma → PostgreSQL.
- El frontend no accede directamente a PostgreSQL.
- NestJS concentra las reglas de negocio.
- Prisma se utiliza para acceso a datos.
- Los cambios arquitectónicos deben documentarse en `docs/DECISIONS.md`.
- No introducir nuevas tecnologías principales sin autorización.
- Mantener separación de responsabilidades.
- Evitar acoplamiento innecesario entre frontend, backend y hardware.
- Las integraciones físicas deben utilizar interfaces/adaptadores cuando corresponda.
- Consultar `docs/ARCHITECTURE.md` antes de modificar estructura arquitectónica.