# Diagramas — Arquitectura de Microservicios Bancarios

Diagramas de la **Parte I, Parte II y Parte III** del proyecto. Todos los archivos están en formato **draw.io (XML)**. La segunda sección reúne los **10 diagramas de secuencia por requerimiento**, que cubren los 58 endpoints de los 5 servicios.

## Cómo abrirlos

- **[app.diagrams.net](https://app.diagrams.net/)** → *File → Open from → Device* y seleccionar el `.drawio`.
- **draw.io Desktop** (plugin de VS Code o app independiente): abrir el archivo directamente.
- También son editables con Lucidchart (importación) o con la extensión *Draw.io Integration* de VS Code.

## Índice de diagramas

| Archivo | Contenido |
|---|---|
| `01-architecture-design.drawio` | Diseño/despliegue de la solución: API Gateway (:8089), discovery-server Eureka (:8761), config-server (:8888) con repo Git `bank-config-repo`, customer-service (:8081), bank-account-service (:8082), credit-service (:8083), MongoDB (:27017, database-per-service) y módulo compartido `bank-resilience-commons`. Incluye los componentes de la **Parte III**: auth-service (:8084), yanki-service (:8085), Redis (:6379) y Kafka (:9092). |
| `02-sequence-api-gateway.drawio` | Secuencia del API Gateway: enrutamiento por patrones de ruta (`/customers/**`, `/bank-accounts/**`, `/credits/**`, `/wallets/**`, `/auth/**`), resolución vía Eureka/LoadBalancer, propagación del JWT, CORS y fallback 503. |
| `03-sequence-customer-service.drawio` | Secuencia de customer-service: CRUD de clientes (PERSONAL / BUSINESS) sobre MongoDB. |
| `04-sequence-bank-account-service.drawio` | Secuencia de bank-account-service: crear cuenta (valida cliente en customer-service y tarjeta en credit-service para VIP/PYME), reglas de producto, transferencias y movimientos. |
| `05-sequence-credit-service.drawio` | Secuencia de credit-service: créditos y tarjetas, consumos contra el límite, pagos, saldo disponible y movimientos. |
| `06-sequence-discovery-config.drawio` | Secuencia combinada de discovery-server (Eureka: registro, heartbeat, evicción) y config-server (configuración centralizada desde Git). |
| `07-uml-class-diagram.drawio` | Diagrama de clases: `bank-resilience-commons` (ServiceNames, ServiceFallbacks, ServiceUnavailableException, ApiError, StandardApiExceptionHandler), adapters con Resilience4j y clases/enum de dominio. |
| `08-uml-use-case-diagram.drawio` | Diagrama de casos de uso: cliente y sistemas externos, casos de negocio e incluidos/extendidos (validar cliente, validar tarjeta, comisión por exceso). |
| `09-sequence-security-jwt.drawio` | **Parte III** — Secuencia de seguridad JWT: login en auth-service (validación de credenciales y firma del token HS256), propagación del token por el gateway y validación en cada microservicio como OAuth2 Resource Server (incluye *token relay* en las llamadas REST salientes). |
| `10-sequence-yanki-kafka.drawio` | **Parte III** — Secuencia del monedero Yanki: registro sin ser cliente, pagos entre monederos por celular y operaciones asíncronas de carga/retiro (`202 Accepted`) mediante los topics `wallet-operations` y `wallet-operation-results` (sin llamadas REST entre servicios nuevos). |
| `11-testing-flow.drawio` | **Flujo de pruebas y verificación (según los requisitos de la documentación)** — Dos vías: **(A) Pruebas unitarias y cobertura de código** (Parte II, líneas 63 y 68; Parte III, líneas 94–95): desarrollo → pruebas unitarias de los métodos públicos nuevos con mocks → `mvnw verify` con **Checkstyle** → **Surefire** → **JaCoCo** → reporte por módulo → presentar el coverage. **(B) Verificación funcional de las APIs con Postman** (Parte I, línea 46; Parte III, línea 112): repositorio Postman → preparar entorno → autenticación (JWT `{{token}}`) → carpetas 1→7 (éxito y error) → validación en la pestaña *Tests* → Yanki asíncrono por Kafka. Incluye los **entregables** (un repo por microservicio + repo Postman + reporte de cobertura + diagramas y contratos OpenAPI). |
| `12-endpoint-flow.drawio` | **Flujo de los endpoints (cadena end-to-end)** — Orden de invocación de las APIs por fase (método + ruta): Auth → Clientes → Créditos → Cuentas → Movimientos/Transferencias → Tarjetas de débito/Reportes → Yanki (monedero y operaciones asíncronas), con el recurso que crea cada endpoint, las **dependencias entre microservicios** (validación de cliente y de tarjeta/deuda por REST) y el tramo **Kafka** de Yanki (sin REST entre servicios nuevos). |

## Diagramas de secuencia por requerimiento (endpoints)

Los 10 diagramas siguientes cubren las **58 operaciones HTTP** de los 5 servicios, agrupadas por épica de las historias de usuario. Cada uno recorre la **ruta completa**: cliente → API Gateway → JWT (Resource Server) → microservicio → MongoDB / Redis / Kafka, con los códigos de estado y las reglas de negocio reales.

| Archivo | Contenido |
|---|---|
| `13-sequence-gateway-resilience.drawio` | **Ruta completa y resiliencia (US-31, US-44 a US-46)** — cliente → API Gateway (:8089) → Eureka (`lb://`) → microservicio como *Resource Server* (firma HS256, expiración y roles) → MongoDB; incluye `401 Unauthorized` y el *fallback* `503` de Resilience4j (timeout 2 s). |
| `14-sequence-auth-jwt.drawio` | **Autenticación y JWT (US-29 a US-31)** — `POST /auth/login` (BCrypt, token HS256 con `sub`, `roles`, `iss` y `exp`) y `POST /auth/register` (rol por defecto USER), más el uso del `Bearer` en una ruta protegida. **2 endpoints.** |
| `15-sequence-customers-crud.drawio` | **CRUD de clientes (US-01 a US-04, US-54)** — `/customers` con caché Redis `customer:{id}` (TTL 300 s, *best-effort*): GET 200, POST 201, GET/{id} 200/404, PUT 200/404 y DELETE 204. **5 endpoints.** |
| `16-sequence-bank-accounts-crud.drawio` | **Cuentas bancarias (US-05 a US-12, US-22 a US-24)** — validación del cliente en `customer-service`, deuda vencida y tarjeta de crédito en `credit-service`, reglas por producto y perfil VIP/PYME, monto mínimo de apertura y consulta de saldo. **6 endpoints.** |
| `17-sequence-transactions-transfers.drawio` | **Movimientos, comisiones y transferencias (US-11, US-13, US-25, US-26)** — depósitos/retiros con mantenimiento y comisión 3.50 por exceso, historial y CRUD de movimientos, y transferencia entre cuentas (`TRANSFER_OUT` / `TRANSFER_IN`) con evento en Kafka. **7 endpoints.** |
| `18-sequence-credits-crud.drawio` | **Créditos (US-14 a US-21, US-32, US-33)** — validación de cliente, bloqueo por deuda vencida, reglas PERSONAL/BUSINESS, consumos contra el límite, pagos, saldo disponible, movimientos y `has-credit-card` / `has-overdue-debt`. **16 endpoints.** |
| `19-sequence-debit-cards.drawio` | **Tarjetas de débito (US-34 a US-36)** — emisión, CRUD, consulta por cliente, pago que debita la cuenta asociada (con comisiones) y movimientos. **8 endpoints.** |
| `20-sequence-reports.drawio` | **Reportes (US-27, US-28)** — resumen por producto de cuentas y créditos (`Single.zip` de RxJava 3) y últimos 10 movimientos de tarjeta de débito y de crédito. **4 endpoints.** |
| `21-sequence-wallet-crud.drawio` | **Monedero Yanki (US-37 a US-40)** — registro sin ser cliente, consultas por id y celular, actualización de datos maestros, DELETE, asociación de tarjeta y pago entre monederos por celular. **8 endpoints.** |
| `22-sequence-wallet-async-kafka.drawio` | **Carga y retiro asíncronos (US-41, US-42, US-53)** — `202 Accepted` con operación `PENDING`, topics `wallet-operations` y `wallet-operation-results`, y cierre del saldo por el consumidor de Yanki. **2 endpoints.** |

## Colores (capas)

- **Gris**: infraestructura (Eureka, Config Server, repo Git).
- **Azul**: microservicios y clases de dominio.
- **Verde**: bases de datos MongoDB.
- **Naranja**: clientes / actores.
- **Rojo**: caché Redis (datos maestros).
- **Morado**: broker Kafka y mensajería orientada a eventos.
- **Amarillo**: notas de reglas de negocio y resiliencia.
