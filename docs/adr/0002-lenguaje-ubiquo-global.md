# ADR 0002 - Adoptar lenguaje ubicuo global

## Status
Accepted

## Date
2025-11-10

## Context
Los equipos de producto, arquitectura y desarrollo necesitaban un lenguaje comun para describir el dominio de gestion y publicacion de eventos. La ausencia de terminos alineados estaba generando ambiguedades en documentos, backlog y conversaciones tecnicas, lo que dificultaba aplicar DDD de manera consistente en todos los servicios.

## Decision
Adoptamos el lenguaje ubicuo documentado en `docs/lenguaje-ubicuo.md` como referencia central para la plataforma. Los terminos y definiciones ahi descritos se consideran fuente oficial para analistas, desarrolladores y expertos de dominio al modelar casos de uso, escribir historias, definir limites de contexto y nombrar artefactos de codigo.

## Consequences
- Los documentos de arquitectura, ADRs y modelos deben referenciar los terminos definidos en el lenguaje ubicuo.
- Se actualizara la documentacion de servicios y repositorios para mantener consistencia terminologica.
- Cualquier nuevo termino o cambio debe discutirse en un foro de arquitectura y registrarse en el mismo documento junto con una actualizacion de esta ADR.
