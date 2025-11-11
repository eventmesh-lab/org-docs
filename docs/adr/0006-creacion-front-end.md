# ADR-0006: Integración de la documentación del Front-end como punto base

## Estado
Propuesto

## Fecha
2025-11-11

## Contexto
El equipo ha desarrollado una aplicación Front-end en el repositorio `eventmesh-frontend`. Para centralizar la documentación del proyecto y facilitar el acceso a los equipos, se ha decidido importar la carpeta `docs/` del repositorio frontend dentro del portal central de documentación (`org-docs`).

Se requiere dejar constancia de esta decisión como un ADR que explique el propósito y las consecuencias de integrar la documentación del front-end como referencia y punto base para el desarrollo de la UI de la plataforma.

## Decisión
Se integrará la documentación del front-end en `docs/front-end/` del repositorio `org-docs` y se considerará como el punto base (referencia inicial) para todas las decisiones de diseño de la UI. La documentación puede adaptarse progresivamente al modelo de documentación central (rutas, enlaces y convenciones de MkDocs).

## Consecuencias
- Positivas:
  - Centraliza la documentación para equipos que trabajan en backend y frontend.
  - Facilita la discovery y el mantenimiento de contratos de API y contratos de UI.
  - Permite enlazar el front-end desde la página principal del sitio de documentación.

- Negativas:
  - Puede requerir trabajo de adaptación para alinear la estructura y enlaces con el portal central.
  - Riesgo de divergencia si la documentación original evoluciona en el repo frontend y no se sincroniza.

## Notas
- Se recomienda definir una estrategia de sincronización (manual o automatizada) si la documentación del frontend cambia frecuentemente.
- Esta ADR documenta únicamente la acción de integrar la documentación como punto base; cambios posteriores en el contenido deberían documentarse según el proceso de ADR existente.
