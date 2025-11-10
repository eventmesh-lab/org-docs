# Plataforma Integral para Gestion de Eventos, Reservas y Servicios con Procesamiento Asincrono

**Universidad Catolica Andres Bello**  
**Facultad de Ingenieria**  
**Escuela de Ingenieria Informatica**  
**Asignatura:** Desarrollo del Software  
**Periodo:** Septiembre 2025 - Enero 2026  
**Profesor:** Bismarck Ponce

## 1. Introduccion
En la actualidad, la organizacion de eventos como conciertos, conferencias, talleres o festivales requiere plataformas digitales que cubran el ciclo completo de gestion. El objetivo del proyecto es disenar y desarrollar una aplicacion web con una API REST robusta y segura, basada en Arquitectura Hexagonal y Domain-Driven Design (DDD). La solucion integrara procesos asincronos mediante colas de mensajeria, trabajos en background y un API Gateway que centralice la seguridad y la autorizacion.

## 2. Descripcion general
### 2.1 Capacidades para usuarios finales
- Registro y autenticacion segura.
- Reserva de entradas y consulta del estado de las operaciones.
- Pago de servicios y seguimiento de transacciones.
- Recepcion de notificaciones en tiempo real sobre confirmaciones, cancelaciones y recordatorios.
- Acceso a historiales de reservas y facturas.

### 2.2 Capacidades para organizadores y administradores
- Creacion y administracion de eventos con escenarios y asientos.
- Control de aforos y disponibilidad en tiempo real.
- Supervision de pagos, facturacion y conciliacion.
- Consulta de reportes y metricas analiticas.

### 2.3 Componentes clave
- Modulos independientes que se comunican via RabbitMQ.
- Seguridad gestionada con Keycloak y un API Gateway.
- Notificaciones en tiempo real mediante SignalR/WebSockets.

## 3. Actores principales
1. **Usuario final:** Persona que accede a la plataforma para registrarse, reservar entradas, contratar servicios y pagar.
2. **Organizador:** Actor responsable de crear y administrar eventos, gestionar la disponibilidad y monitorear ventas.
3. **Administrador:** Encargado de la operacion global de la plataforma, supervisando usuarios, eventos, pagos, reportes y auditoria.
4. **Sistema externo de pagos (simulado):** Pasarela encargada de procesar pagos electronicos.
5. **Servicios externos complementarios:** Proveedores de transporte, catering o merchandising integrados mediante RabbitMQ.

## 4. Requerimientos funcionales
### 4.1 Modulos principales
#### Autenticacion y autorizacion
- Integracion con Keycloak.
- Manejo de roles: Usuario, Organizador, Administrador y Soporte.
- Validacion de accesos a traves del API Gateway.

#### Gestion de usuarios
- Creacion y mantenimiento de perfiles.
- Asociacion de usuarios con reservas, pagos y servicios.
- Historial de actividad registrado para auditoria.

#### Gestion de eventos
- Alta, baja y modificacion de eventos.
- Configuracion de categorias, fechas y aforos.
- Asociacion de archivos en Blob Storage (imagenes, folletos).

#### Escenarios y asientos
- Configuracion de escenarios con asientos numerados.
- Control de disponibilidad en tiempo real.
- Liberacion automatica de asientos no pagados.

#### Reservas
- Creacion, confirmacion y cancelacion de reservas.
- Emision de eventos de negocio en RabbitMQ.
- Expiracion automatica mediante trabajos programados.

#### Pagos y facturacion
- Procesamiento asincrono de pagos.
- Reintentos automaticos en caso de error.
- Generacion de facturas digitales.
- Job de conciliacion financiera.

#### Servicios complementarios
- Contratacion de transporte, catering o merchandising.
- Integracion con proveedores externos via colas.
- Confirmacion y actualizacion en tiempo real.

#### Notificaciones y comunicacion
- Notificaciones en tiempo real mediante SignalR/WebSockets.
- Recordatorios de eventos y confirmaciones de pago.
- Envio de correos ante fallos criticos.

#### Reportes y analitica
- Reportes de ventas, asistencia y cancelaciones.
- Metricas de desempeno de eventos y servicios.
- Jobs automaticos para reportes diarios o semanales.

#### Administracion y panel de control
- Dashboard con metricas en tiempo real.
- Gestion centralizada de usuarios, eventos, pagos y servicios.
- Control de accesos y monitoreo del sistema.

### 4.2 Extensiones obligatorias
#### Auditoria y logs
- Registro de actividad en una base documental (MongoDB o ElasticSearch).
- Consultas rapidas para debugging y monitoreo.

#### Recomendaciones inteligentes
- Algoritmo de sugerencias basado en el historial del usuario.
- Filtros por intereses y categorias de eventos.

#### Integracion con proveedores externos
- APIs simuladas para disponibilidad de transporte y catering.
- Procesamiento asincrono via RabbitMQ, con manejo de errores y reintentos.

### 4.3 Catalogo de requerimientos funcionales
1. **Autenticacion y autorizacion**
   - Integracion con Keycloak como servidor de identidad.
   - Manejo de roles y permisos granulares.
   - Validacion de tokens en el API Gateway.
2. **Gestion de usuarios**
   - Registro y actualizacion de datos personales.
   - Consulta del historial de reservas, pagos y servicios.
   - Asociacion de acciones a usuarios para auditoria.
3. **Gestion de eventos**
   - Alta, baja y modificacion de eventos.
   - Configuracion de fechas, categorias, aforo y precios.
   - Gestion de material asociado en Blob Storage.
4. **Escenarios y asientos**
   - Definicion de escenarios con asientos numerados.
   - Reserva y bloqueo de asientos en tiempo real.
   - Liberacion automatica al expirar o no pagar una reserva.
5. **Reservas**
   - Creacion, confirmacion y cancelacion de reservas.
   - Publicacion de eventos de negocio en RabbitMQ.
   - Expiracion automatica mediante trabajos en background.
6. **Pagos y facturacion**
   - Procesamiento mediante pasarela simulada.
   - Manejo de confirmaciones asincronas y reintentos.
   - Generacion de facturas PDF y conciliaciones programadas.
7. **Servicios complementarios**
   - Contratacion de transporte, catering y merchandising.
   - Publicacion de solicitudes a proveedores via RabbitMQ.
   - Confirmaciones automatizadas o manuales, con notificaciones al usuario.
8. **Notificaciones y comunicacion**
   - Notificaciones push mediante SignalR/WebSockets.
   - Envio de recordatorios y correos de respaldo.
   - Reintentos automaticos ante fallas.
9. **Reportes y analitica**
   - Reportes de ventas, cancelaciones y asistencia.
   - Metricas de desempeno como tiempos de confirmacion e ingresos por evento.
   - Generacion automatica de reportes y paneles de visualizacion.
10. **Administracion y panel de control**
   - Consola web para administradores y organizadores.
   - Dashboard con indicadores clave en tiempo real.
   - Monitoreo del estado de colas, trabajos y notificaciones.
11. **Auditoria y logs**
   - Registro de operaciones criticas en MongoDB o ElasticSearch.
   - Consultas rapidas para monitoreo de seguridad.
12. **Recomendaciones inteligentes**
   - Algoritmo basado en historial del usuario e intereses.
   - Filtrado por categorias y ubicacion geografica.
13. **Integracion con proveedores externos**
   - Simulacion de APIs para transporte y catering.
   - Comunicacion asincrona via RabbitMQ con manejo de errores.
14. **Modulo de gestion de archivos y multimedia**
   - Almacenamiento en Blob Storage (Azure, AWS S3, MinIO o Firebase).
   - Gestion de imagenes promocionales, programas en PDF y comprobantes.
   - Control de acceso seguro conforme a roles.
15. **Modulo de localizacion y multilenguaje**
   - Soporte multilenguaje (por ejemplo, espanol e ingles).
   - Adaptacion de precios y horarios al formato local.
16. **Modulo de marketing y promociones**
   - Generacion de codigos de descuento.
   - Validacion de cupones durante el pago.
   - Reportes de uso de promociones.
17. **Modulo de encuestas y feedback**
   - Encuestas vinculadas a reservas finalizadas.
   - Registro de respuestas para el modulo de reportes.
   - Visualizacion de metricas de satisfaccion.
18. **Modulo de streaming**
   - Enlaces unicos y seguros para transmisiones en vivo.
   - Control de accesos basado en reservas confirmadas.
   - Opcion de grabacion y almacenamiento de sesiones.
19. **Modulo de comunidad y foros**
   - Foros asociados a cada evento.
   - Publicacion y respuesta moderada por organizadores.
   - Chat asincrono para participantes.

## 5. Conclusion
El proyecto integra diez modulos principales y diez extensiones, fomentando el trabajo colaborativo del equipo. Se cubren procesos sincronos y asincronos, notificaciones en tiempo real, seguridad distribuida, trabajos programados y comunicacion mediante colas, evitando soluciones tipo CRUD basicas y promoviendo la construccion de una plataforma empresarial compleja con DDD y arquitectura hexagonal.

## 6. Desarrollo
- El proyecto debe ser ejecutado por una empresa conformada por tres personas.
- Se debe entregar un documento ERS completo basado en este enunciado.
- Cada grupo detallara requerimientos mediante casos de uso, diagramas, requerimientos funcionales y no funcionales.
- La aplicacion se desarrollara con programacion orientada a objetos.
- Se aplicaran estandares de programacion definidos por el grupo.
- Se utilizara GitHub para el control de versiones.
- Cada entrega debe incluir pruebas unitarias.
- Cada miembro es responsable de integrar y mantener consistencia con el resto del sistema.
- El sistema debe estar plenamente integrado y desplegado en servidores o dispositivos segun corresponda; no se aceptan entregas dependientes del entorno de desarrollo.

## 7. Reglas y evaluacion
- La colaboracion entre los integrantes es obligatoria.
- Se evaluaran cualidades del software junto con las tecnicas vistas en clase.
- El diseno del sistema es responsabilidad de todo el equipo.
- La evaluacion incluye aspectos grupales e individuales.
- Todos los miembros garantizan la homogenizacion, consistencia e integracion de los componentes.
- Las tareas deben asignarse sin ambiguedades.
- Las entregas fuera del plazo no seran evaluadas.
- Todos los errores o incidencias deben manejarse adecuadamente.
- Todos los miembros deben aportar la misma cantidad de trabajo.
- Tecnologias obligatorias: Microservicios, .NET Core 8, Arquitectura Hexagonal, DDD, MediatR, Hangfire, RabbitMQ, Postgres, MongoDB, Firebase Storage, Keycloak, React, YARP, SignalR/WebSockets.
- El lider del proyecto debe identificar debilidades del equipo y tomar acciones correctivas.
- Se utilizara Trello para la gestion y control del proyecto.

## 8. Entregas
### Primera entrega
1. Sistema totalmente integrado y funcional.
2. Uso correcto del paradigma POO.
3. Cumplimiento de estandares de programacion.
4. Documentacion de codigo.
5. Manejo adecuado de errores.
6. Pruebas unitarias con 90 % de cobertura.
7. Control de versiones del codigo fuente.
8. ERS entregado una semana antes de la fecha limite.
9. Despliegue fuera del entorno de desarrollo.

### Segunda entrega
1. Sistema integrado con servicios de integracion.
2. Plataforma robusta, correcta y completa.
3. Arquitectura implementada siguiendo patrones establecidos.
4. Uso correcto del paradigma POO.
5. Estandares de programacion.
6. Documentacion de codigo.
7. Manejo adecuado de errores.
8. Pruebas unitarias con 90 % de cobertura.
9. Control de versiones.
10. ERS actualizado.
11. Despliegue fuera del entorno de desarrollo.

## 9. Rubricas
| Criterio | Puntos | Descripcion |
| --- | --- | --- |
| Completitud | 2 | El sistema implementa todas las funcionalidades requeridas. |
| Integracion | 2 | Los modulos del sistema estan correctamente integrados. |
| Programacion orientada a objetos | 3 | Se sigue un paradigma orientado a objetos con buen diseno de clases y reutilizacion de codigo. |
| Estandares de programacion | 1 | Buenas practicas en estructura, nomenclatura y organizacion del codigo. |
| Documentacion | 1 | Documentacion clara del codigo y del sistema. |
| Uso del control de versiones | 2 | Gestion adecuada de commits, pull requests, rebases y amend. |
| Manejo de excepciones | 1 | Tratamiento correcto de excepciones para evitar fallos inesperados. |
| Pruebas unitarias | 2 | Cobertura de pruebas unitarias del 90 %. |
| Defensa individual | 6 | Cada miembro demuestra conocimiento del codigo y la arquitectura. |
