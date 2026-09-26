---
trigger: glob
globs: "frontend/**/*.{ts,tsx,js,jsx}"
description: "Reglas para desarrollo del frontend Next.js."
---

# Frontend Rules

- El frontend utiliza Next.js, React y TypeScript.
- El frontend consume la API REST del backend.
- No conectarse directamente a PostgreSQL.
- No implementar reglas de negocio críticas exclusivamente en el frontend.
- Validar formularios antes de enviarlos.
- Mantener separación entre componentes de interfaz y acceso a API.
- Respetar la autenticación administrativa.
- Respetar los permisos entregados por el backend.
- No asumir permisos únicamente por el nombre del rol.
- La estación de acceso y el panel administrativo son experiencias diferentes.
- Consultar `docs/FLOWS.md` antes de modificar flujos principales.