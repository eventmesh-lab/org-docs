# Guía: Contexto de negocio para plataformas de eventos

Las plataformas de eventos conectan diversos tipos de usuarios: por un lado, los organizadores (empresas, promotores, artistas, dueños de recintos) que crean y gestionan eventos; por otro, los asistentes/consumidores que buscan y compran entradas. Por ejemplo, Ticketmaster opera en un modelo B2B2C asociándose con recintos, promotores y artistas (B2B) para vender entradas a los aficionados (B2C). En Eventbrite, los usuarios se dividen también en organizadores de eventos (desde conferencias corporativas hasta festivales) y asistentes que compran boletos. Además hay proveedores y patrocinadores externos (servicios de catering, transporte, merchandising, medios) que ofrecen productos adicionales; y administradores del sistema que supervisan usuarios, eventos y finanzas.

## Operaciones fundamentales
Los sistemas de gestión de eventos cubren todo el flujo de un evento en vivo. Entre las operaciones principales están:

- Creación y configuración de eventos: definir el evento con nombre, descripción, categorías, fecha(s) y lugar/aforo; configurar plantillas de evento reutilizables y mapas de asientos si aplica. Por ejemplo, Ticketmaster permite editar el plano del recinto, asignar niveles de precios y usar plantillas predefinidas para agilizar nuevos eventos.
- Gestión de entradas y reservas: crear tipos de entradas (por ejemplo “general” o “VIP”) con precios y estados (disponible, reservado, vendido). El usuario puede seleccionar asientos disponibles y agregar boletos al carrito. La plataforma debe bloquear brevemente la disponibilidad al iniciar compra (para evitar doble reserva) y luego confirmar o liberar el asiento según el pago. Las reservas pueden expirar automáticamente si no se paga en tiempo.
- Procesamiento de pagos y facturación: integrar pasarelas de pago para cobrar tarjetas, plataformas online, etc. Los pagos suelen procesarse de forma asíncrona (confirmando el asiento solo al recibir la transacción). Se generan comprobantes/facturas digitales y se aplican comisiones (por entrada, por servicio). La conciliación financiera y gestión de reembolsos es parte clave en este flujo.
- Promociones y marketing: ofrecer códigos de descuento, promociones por redes sociales y campañas de email. Por ejemplo, Eventbrite integra widgets sociales y envía invitaciones o recordatorios por correo. Se envían newsletters automáticos (p. ej. “Event Picks”) con eventos sugeridos y notificaciones si amigos confirman asistencia. Estos mecanismos ayudan a difundir eventos y aumentar ventas.
- Validación de acceso: el día del evento, los asistentes presentan su ticket digital (código QR o enlace). La plataforma ofrece apps o dispositivos de escaneo que verifican la entrada en tiempo real. Por ejemplo, Ticketmaster reporta el uso de códigos dinámicos para evitar fraude.
- Servicios adicionales: muchas plataformas permiten vender complementos: comida/bebida, estacionamiento, merchandising, incluso transmisión en vivo. Algunas soluciones permiten planificar la demanda de food & beverage y vender paquetes junto con el ticket.
- Reportes y análisis: generación de informes de ventas, asistencia, ingresos, ROI de campañas, etc. Esto apoya la toma de decisiones posteriores (por ejemplo, métricas avanzadas de marketing y medición).

## Entidades del dominio y relaciones clave
En el modelo de dominio suelen aparecer entidades como:

- Evento: atributos como nombre, descripción, lugar, fecha/hora, aforo, etc.
- Usuario: datos del asistente o del organizador.
- Entrada / Boleto: vinculada a un evento, con tipo, precio y estado.
- Reserva / Pedido: un conjunto de entradas reservado o comprado por un usuario.
- Pago: monto, método, estado, vinculado a la reserva.
- Servicios complementarios: ofrecimientos ligados al evento, como food, parking, merch o streaming.

Relaciones típicas: un organizador posee varios eventos; un asistente puede tener múltiples reservas; una reserva consiste en uno o varios boletos; cada pago corresponde a una reserva. El “Evento” puede relacionarse con una “Ubicación” y una “Fecha”, mientras que cada “Boleto” tiene un precio y un estado (vendido, reservado, cancelado).

## Contextos de negocio (bounded contexts)
Estos dominios se agrupan en contextos independientes en el software. Algunos ejemplos:

- Gestión de eventos: incluye todo lo relacionado con la definición del evento – creación, edición, configuración de ubicaciones y asientos, plantillas de evento – con entidades como Evento, Escenario, Asiento.
- Ventas / Reservas de entradas: abarca la compra de boletos, la verificación de disponibilidad en tiempo real y la gestión de colas/espera. Aquí operan entidades Pedido/Reserva y Entrada, y se evita la doble asignación de asientos mediante bloqueos temporales.
- Pagos y facturación: contempla el procesamiento de pagos externos, la generación de facturas electrónicas y la conciliación financiera. Se implementan lógicas de retry y manejo de fallos, y se registra cada transacción con su estado.
- Marketing y promociones: cubre la creación y aplicación de códigos promocionales, newsletters, anuncios en redes sociales y análisis de campañas. En este contexto se gestionan descuentos y estadísticas de conversión.
- Comunicación y notificaciones: envíos automáticos de correos (confirmaciones, recordatorios) y mensajes en tiempo real a asistentes y organizadores.
- Acceso al evento: se encarga de validar los tickets en la entrada (por QR o similar) y de gestionar la lista de asistentes en el día del evento.
- Servicios adicionales: venta de addons (comidas, estacionamiento, merchandising, streaming), cada uno con sus propios flujos de pedido y pago.
- Administración / general: gestión de usuarios y roles, seguridad, reportes del sistema, etc.

Estos contextos delimitados permiten implementar la plataforma con claridad: por ejemplo, un contexto “Reservas” se integraría con “Pagos” mediante eventos de dominio (una reserva pagada dispara la emisión del ticket), mientras “Marketing” opera por separado usando datos de eventos y usuarios.

## Fuentes y lectura adicional
Documentación de plataformas y estudios de caso reales que respaldan las características de negocio descritas:

- https://vizologi.com/es/lienzo-de-estrategia-empresarial/lienzo-del-modelo-de-negocio-de-Ticketmaster/
- https://es.scribd.com/document/503360560/CASO-MODULO-8-Transformacion-Digital
- https://business.ticketmaster.com/solutions/event-creation-management/
- https://www.designgurus.io/blog/design-ticketing-system
- https://systemdesignschool.io/problems/ticketmaster/solution
- https://www.businessmodelzoo.com/exemplars/eventbrite/
- https://business.ticketmaster.com.mx/
- https://www.ticketmundo.com/vende-con-nosotros
- https://virtualeduca.org/mediacenter/caso-de-estudio-como-la-universidad-de-los-andes-revoluciono-su-estrategia-de-eventos-con-eventtia/

---
_Este documento sintetiza el contexto de negocio para plataformas de venta y gestión de eventos y sirve como guía para diseñar los bounded contexts y entidades del dominio en la arquitectura del proyecto._
