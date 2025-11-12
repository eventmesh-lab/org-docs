---
title: "0007 - Documentar y decidir sobre svc_yarp_api-gateway (API Gateway)"
status: Accepted
date: 2025-11-11
---

## Context

El repositorio `Services/eventmesh-frontend/svc_yarp_api-gateway` contiene un API Gateway y proxy de eventos implementado con arquitectura hexagonal en .NET 8.0. Hasta la fecha el repositorio incluía un `README.md` que era originalmente una plantilla; el servicio no estaba documentado en el portal central de `org-docs` ni tenía un ADR que describiera la decisión de exponerlo y documentarlo a nivel organizacional.

## Decision

1. Documentar el servicio `svc_yarp_api-gateway` en el portal central (`docs/services/api-gateway.md`) con la información operativa y de arquitectura mínima (propósito, capas, endpoints, cómo ejecutar y pruebas).
2. Añadir un ADR local al repositorio del servicio (si se requiere) para decisiones futuras sobre su diseño/infraestructura. Mientras tanto, se registra este ADR global (0007) para dejar constancia organizacional.
3. Marcar el estado del servicio en el índice central de servicios como "Documentado (básico)" y mantener sincronización entre el repositorio del servicio y la carpeta `docs/services/` del portal.

## Consequences

- Ventajas:
  - Mejora la trazabilidad y descubribilidad del API Gateway.
  - Facilita la incorporación de desarrolladores al proyecto y la revisión arquitectural.
  - Provee una base para futuras ADRs locales relacionadas con autenticación, escalado o rutas del gateway.

- Obligaciones:
  - Mantener la documentación sincronizada con el repositorio fuente del servicio.
  - Cuando se tomen decisiones relevantes (auth, redes, despliegue), crear un ADR local en el repo del servicio y referenciarlo desde aquí.

## Notes / Next steps

- Añadir un ADR local en `Services/eventmesh-frontend/svc_yarp_api-gateway/docs/adr/` si el equipo del servicio toma decisiones de diseño o despliegue.
- Revisar y ampliar la documentación con diagramas de integración y flujos de eventos.
