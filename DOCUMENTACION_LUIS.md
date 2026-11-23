Módulos responsables: Billing Service y Report Analyst Service

En este proyecto de arquitectura de microservicios tuve a mi cargo Billing Service, orientado a la gestión y generación de facturas, y Report Analyst Service, enfocado en la consolidación analítica de eventos y en la generación de reportes del sistema.

1. Puertos e interacción con Eureka

Billing Service se ejecuta en el puerto 8085. Se registra como cliente en Eureka Server para su descubrimiento dentro de la red de microservicios y expone endpoints REST mediante InvoiceRestController, además de vistas a través de InvoiceController y la plantilla invoices.html.

Report Analyst Service se ejecuta en el puerto 8086. También se registra en Eureka Server, consume información y eventos del ecosistema y ofrece endpoints analíticos mediante ReportRestController, junto con una vista de reportes implementada en ReportDashboardController y reports.html.

Los puertos definidos son 8085 para Billing Service y 8086 para Report Analyst Service. La configuración correspondiente puede verificarse en backend/services/billing-service/src/main/resources/application.properties, en application-local.properties, en backend/services/report-analyst-service/src/main/resources/application.properties y en docker-compose.yml.

2. Eventos de Kafka

El sistema combina comunicación síncrona mediante REST con comunicación asíncrona basada en eventos usando Apache Kafka, lo que permite desacoplar servicios y mantener actualizadas las vistas analíticas.

En Billing Service, el productor InvoiceEventProducer, configurado en KafkaProducerConfig, emite el evento InvoiceCreatedEvent cuando se genera una factura. Este evento notifica de forma asíncrona a los consumidores interesados, como los servicios de reportes y notificaciones.

En Report Analyst Service, los consumidores escuchan eventos del dominio desde el paquete com.hospital.reports.kafka: AppointmentEventConsumer para AppointmentCreatedEvent, InvoiceEventConsumer para InvoiceCreatedEvent, PatientEventConsumer para PatientRegisteredEvent y PrescriptionEventConsumer para PrescriptionIssuedEvent. Con estos eventos se persisten registros de consulta y análisis como AppointmentRecord, InvoiceRecord, PatientRecord y PrescriptionRecord, los cuales alimentan resúmenes y reportes mediante ReportSummaryResponse.

3. Docker y Docker Compose

Cada servicio se empaqueta como imagen contenedora mediante Dockerfile y entrypoint.sh, lo que permite ejecutarlo de forma aislada y reproducible en cualquier entorno.

La orquestación se realiza con docker-compose.yml, que levanta de manera coordinada los microservicios, Eureka, Kafka, las bases de datos y el frontend o proxy. Además, gestiona redes internas, variables de entorno definidas en .env.example y dependencias entre servicios.

4. Arquitectura MVC aplicada

Ambos servicios siguen una separación por capas inspirada en el patrón Modelo, Vista y Controlador, complementada con capas de servicio, repositorio, DTOs y eventos.

En el modelo se incluyen las entidades JPA, como Invoice en billing-service, y AppointmentRecord, InvoiceRecord, PatientRecord y PrescriptionRecord en report-analyst-service, junto con los repositorios, DTOs y eventos correspondientes.

En la vista se utilizan plantillas HTML servidas desde src/main/resources/templates, como invoices.html en billing-service y reports.html en report-analyst-service, además de respuestas JSON generadas por los controladores REST para el frontend.

En los controladores se implementan componentes web y REST como InvoiceController e InvoiceRestController en billing-service, y ReportDashboardController y ReportRestController en report-analyst-service. Estos reciben las peticiones HTTP, delegan la lógica de negocio y devuelven vistas o respuestas estructuradas.

5. Reflexión sobre Programación de Servicios Web

A lo largo de la materia de Programación de Servicios Web adquirí bases prácticas valiosas para el desarrollo backend moderno. Comprendí la diferencia entre una arquitectura monolítica y una de microservicios, y observé cómo dividir responsabilidades facilita el mantenimiento y el despliegue independiente, aunque exige mayor cuidado con la red, los datos y la coordinación entre servicios.

También aprendí lo básico de Apache Kafka para desacoplar procesos mediante eventos asíncronos y de Eureka para el descubrimiento dinámico de servicios. Reforcé el uso de Docker y Docker Compose para levantar entornos completos de forma consistente, sin depender de configuraciones manuales en cada máquina.

Mejoré mi flujo de trabajo con IntelliJ IDEA para desarrollar y estructurar proyectos Java con Spring, y con Postman para probar endpoints, validar payloads, revisar códigos de estado HTTP y depurar flujos. Finalmente, confirmé la importancia de los microservicios en Java y en la actualidad, ya que el ecosistema Java y Spring sigue siendo clave en sistemas empresariales y críticos donde se buscan escalabilidad, orden en el código y resiliencia.
