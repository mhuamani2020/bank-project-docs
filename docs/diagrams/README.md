# Diagramas — Arquitectura de Microservicios Bancarios

Diagramas de la **Parte I** del proyecto. Todos los archivos están en formato **draw.io (XML)**.

## Cómo abrirlos

- **[app.diagrams.net](https://app.diagrams.net/)** → *File → Open from → Device* y seleccionar el `.drawio`.
- **draw.io Desktop** (plugin de VS Code o app independiente): abrir el archivo directamente.
- También son editables con Lucidchart (importación) o con la extensión *Draw.io Integration* de VS Code.

## Índice de diagramas

| Archivo | Contenido |
|---|---|
| `01-architecture-design.drawio` | Diseño/despliegue de la solución: API Gateway (:8089), discovery-server Eureka (:8761), config-server (:8888) con repo Git `bank-config-repo`, customer-service (:8081), bank-account-service (:8082), credit-service (:8083), MongoDB (:27017, database-per-service) y módulo compartido `bank-resilience-commons`. |
| `02-sequence-api-gateway.drawio` | Secuencia del API Gateway: enrutamiento por patrones de ruta (`/customers/**`, `/bank-accounts/**`, `/credits/**`), resolución vía Eureka/LoadBalancer, CORS y fallback 503. |
| `03-sequence-customer-service.drawio` | Secuencia de customer-service: CRUD de clientes (PERSONAL / BUSINESS) sobre MongoDB. |
| `04-sequence-bank-account-service.drawio` | Secuencia de bank-account-service: crear cuenta (valida cliente en customer-service y tarjeta en credit-service para VIP/PYME), reglas de producto, transferencias y movimientos. |
| `05-sequence-credit-service.drawio` | Secuencia de credit-service: créditos y tarjetas, consumos contra el límite, pagos, saldo disponible y movimientos. |
| `06-sequence-discovery-config.drawio` | Secuencia combinada de discovery-server (Eureka: registro, heartbeat, evicción) y config-server (configuración centralizada desde Git). |
| `07-uml-class-diagram.drawio` | Diagrama de clases: `bank-resilience-commons` (ServiceNames, ServiceFallbacks, ServiceUnavailableException, ApiError, StandardApiExceptionHandler), adapters con Resilience4j y clases/enum de dominio. |
| `08-uml-use-case-diagram.drawio` | Diagrama de casos de uso: cliente y sistemas externos, casos de negocio e incluidos/extendidos (validar cliente, validar tarjeta, comisión por exceso). |

## Colores (capas)

- **Gris**: infraestructura (Eureka, Config Server, repo Git).
- **Azul**: microservicios y clases de dominio.
- **Verde**: bases de datos MongoDB.
- **Naranja**: clientes / actores.
- **Amarillo**: notas de reglas de negocio y resiliencia.
