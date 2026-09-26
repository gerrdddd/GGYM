# GYM APP — Agent Instructions

## 1. Lectura obligatoria

Antes de realizar cambios importantes:

1. Leer `docs/MASTER.md`.
2. Leer el documento especializado relacionado con el ticket.
3. Revisar el código existente.
4. Revisar las reglas aplicables dentro de `.agents/rules/`.
5. Revisar `docs/DECISIONS.md` cuando el cambio pueda afectar arquitectura.

---

# 2. Trabajar por tickets

El agente debe trabajar únicamente sobre el ticket solicitado.

No implementar funcionalidades no relacionadas.

Si una modificación adicional es técnicamente necesaria:

1. Informarla.
2. Explicar por qué.
3. Indicar qué archivos se verán afectados.
4. Esperar autorización cuando el cambio exceda claramente el alcance.

---

# 3. Plan antes de implementación

Antes de modificar archivos importantes, presentar brevemente:

- Qué se va a hacer.
- Qué archivos se modificarán.
- Qué archivos se crearán.
- Por qué son necesarios.
- Cómo se validará el resultado.

No comenzar cambios complejos sin presentar el plan.

---

# 4. Arquitectura obligatoria

La arquitectura principal es:

```text
Next.js
   ↓
NestJS
   ↓
Prisma
   ↓
PostgreSQL
```

El frontend nunca debe acceder directamente a PostgreSQL.

---

# 5. Base de datos

- Utilizar PostgreSQL.
- Utilizar Prisma.
- Utilizar migraciones.
- Respetar `docs/DATABASE.md`.
- No realizar cambios destructivos sin autorización.
- No borrar información histórica importante.
- Utilizar `Decimal` para dinero.
- Utilizar UUID para entidades principales.

---

# 6. Reglas de negocio

Antes de implementar funcionalidades de:

- Clientes.
- Membresías.
- Acceso.
- Inventario.
- Ventas.
- Finanzas.
- Caja.
- Tickets.
- Auditoría.

consultar:

`docs/BUSINESS-RULES.md`

---

# 7. Seguridad

Las rutas administrativas requieren autenticación.

Las operaciones sensibles requieren autorización mediante permisos.

Las contraseñas deben almacenarse mediante hash.

La huella no debe tratarse como contraseña.

---

# 8. Historial

No eliminar silenciosamente:

- Ventas.
- Movimientos financieros.
- Membresías históricas.
- Accesos.
- Auditoría.

Las correcciones deben conservar trazabilidad.

---

# 9. Validación

Después de implementar:

- Ejecutar pruebas disponibles.
- Ejecutar validaciones.
- Ejecutar build cuando corresponda.
- Revisar errores.
- Comprobar que no se rompió funcionalidad existente.

---

# 10. Reporte final

Después de cada tarea importante informar:

```text
Archivos creados:
-

Archivos modificados:
-

Archivos eliminados:
-

Pruebas ejecutadas:
-

Resultado:
-

Problemas encontrados:
-

Decisiones tomadas:
-
```

---

# 11. Documentación

Actualizar documentación cuando un cambio importante lo requiera.

Actualizar:

- `PROJECT-STATUS.md`
- `CHANGELOG.md`
- `DECISIONS.md`

cuando corresponda.

---

# 12. No inventar requisitos

Si la documentación no define una decisión:

No asumirla silenciosamente.

Indicar:

```text
Decisión pendiente:
...
```

y solicitar aclaración o proponer alternativas antes de implementar algo que pueda afectar arquitectura o negocio.

---

# 13. Código

Priorizar:

- Modularidad.
- Legibilidad.
- Separación de responsabilidades.
- Tipado fuerte.
- Validación.
- Manejo consistente de errores.
- Mantenibilidad.

Evitar complejidad innecesaria.

---

# 14. Git

Mantener commits relacionados con cambios coherentes.

Evitar mezclar múltiples funcionalidades no relacionadas en un mismo commit.

No ejecutar acciones destructivas de Git sin autorización.

---

# 15. Regla fundamental

El agente implementa.

El usuario decide.

La documentación define el contexto.

Cuando exista una contradicción importante entre código, documentación y ticket:

detenerse, informar y resolver la contradicción antes de continuar.