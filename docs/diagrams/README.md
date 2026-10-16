# Diagramas — Arquitectura de Microservicios Bancarios

Diagramas de la **Parte I, Parte II y Parte III** del proyecto. Todos los archivos están en formato **draw.io (XML)**.

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

## Colores (capas)

- **Gris**: infraestructura (Eureka, Config Server, repo Git).
- **Azul**: microservicios y clases de dominio.
- **Verde**: bases de datos MongoDB.
- **Naranja**: clientes / actores.
- **Rojo**: caché Redis (datos maestros).
- **Morado**: broker Kafka y mensajería orientada a eventos.
- **Amarillo**: notas de reglas de negocio y resiliencia.
