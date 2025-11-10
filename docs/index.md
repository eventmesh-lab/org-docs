# Portal de documentación de la Organización

Bienvenido. Este sitio agrega la documentación por servicio y la documentación organizacional.

Contenido:
- Arquitectura global y lineamientos
- ADRs globales
- Índice por servicio (cada servicio tiene su propio subtree bajo "Servicios")
- Catálogo de APIs y eventos (OpenAPI / AsyncAPI)
- Guías de contribución y plantillas

Estructura:
- Las docs de cada servicio se sincronizan desde los repositorios listados en `repos.json`.
- Las ADRs locales a cada servicio están en `docs/adr/` dentro del servicio y se indexan aquí.

## Origen y gobernanza

- Este repositorio agrega contenido automáticamente; evita editar documentos sincronizados en `docs/services/**`.
- Cada página de servicio debe enlazar su repositorio de origen para que los lectores tengan contexto y puedan contribuir.
- Revisa `repos.json` y los flujos de GitHub Actions para añadir nuevos servicios o actualizar tokens.
- Puedes copiar los workflows de ejemplo desde `templates/` para estandarizar la notificación y validación en cada microservicio.