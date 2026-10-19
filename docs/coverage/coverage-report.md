# Reporte de cobertura de pruebas — JaCoCo

> **Consolidado de los 9 módulos Maven** del sistema bancario. Cada microservicio es un
> repositorio y un proyecto independiente (no existe un `pom.xml` raíz), por lo que este
> informe agrega los reportes individuales que genera el plugin `jacoco-maven-plugin`
> en cada módulo durante la fase `test`.

**Generado:** 2026-10-19 15:08 UTC  
**Plugin:** `jacoco-maven-plugin` 0.8.11  
**Comando por módulo:** `mvnw clean verify` (commons: `mvnw clean install`)  
**Reporte HTML individual:** `<modulo>/target/site/jacoco/index.html`

---

## 1. Resumen global

| Métrica | Cubierto | No cubierto | Total | Cobertura |
|---|---:|---:|---:|---:|
| **Líneas** | 1,291 | 161 | 1,452 | **88.9%** |
| **Ramas** | 255 | 105 | 360 | **70.8%** |
| Instrucciones | 6,064 | 751 | 6,815 | 89.0% |
| Métodos | 407 | 58 | 465 | 87.5% |
| Complejidad ciclomática | 490 | 155 | 645 | 76.0% |

**Pruebas ejecutadas:** 269 · **fallos:** 0 · **errores:** 0

## 2. Cobertura por módulo

| Módulo | Tipo | Tests | Clases | Líneas | Ramas | Métodos | Instrucciones |
|---|---|---:|---:|---:|---:|---:|---:|
| `bank-resilience-commons` | Librería compartida | 5 | 5 | 88.6% | — | 75.0% | 87.1% |
| `discovery-server` | Infraestructura | 1 | 1 | 33.3% | — | 50.0% | 37.5% |
| `config-server` | Infraestructura | — | — | sin pruebas | sin pruebas | sin pruebas | sin pruebas |
| `auth-service` | Microservicio | 12 | 8 | 42.1% | 69.2% | 50.0% | 45.6% |
| `customer-service` | Microservicio | 27 | 8 | 93.8% | 50.0% | 93.2% | 93.9% |
| `bank-account-service` | Microservicio | 119 | 24 | 95.2% | 73.1% | 94.4% | 94.6% |
| `credit-service` | Microservicio | 67 | 15 | 94.3% | 66.2% | 92.9% | 93.2% |
| `yanki-service` | Microservicio | 37 | 10 | 84.8% | 72.9% | 81.4% | 85.1% |
| `api-gateway` | Infraestructura | 1 | 1 | 33.3% | — | 50.0% | 37.5% |

> `config-server` **no tiene carpeta `src/test`**: es un módulo de infraestructura (solo
> expone la configuración centralizada) y aporta **0 líneas instrumentables con pruebas**,
> por eso aparece como *sin pruebas* en lugar de un porcentaje.

## 3. Detalle por módulo

### `bank-resilience-commons`

**Total:** 88.6% de líneas · — de ramas · 75.0% de métodos

| Paquete | Clase | Líneas cub. / tot. | Líneas % | Ramas % | Métodos % |
|---|---|---:|---:|---:|---:|
| `com.nttdata.commons.resilience.exception` | `ServiceUnavailableException` | 4 / 4 | 100.0% | — | 100.0% |
| `com.nttdata.commons.resilience.client` | `ServiceFallbacks` | 1 / 1 | 100.0% | — | 100.0% |
| `com.nttdata.commons.resilience.autoconfigure` | `ResilienceCommonsAutoConfiguration` | 0 / 2 | 0.0% | — | 0.0% |
| `com.nttdata.commons.resilience.error` | `StandardApiExceptionHandler` | 16 / 16 | 100.0% | — | 88.9% |
| `com.nttdata.commons.resilience.error` | `ApiError` | 10 / 12 | 83.3% | — | 66.7% |

### `discovery-server`

**Total:** 33.3% de líneas · — de ramas · 50.0% de métodos

| Paquete | Clase | Líneas cub. / tot. | Líneas % | Ramas % | Métodos % |
|---|---|---:|---:|---:|---:|
| `com.nttdata.discovery_server` | `DiscoveryServerApplication` | 1 / 3 | 33.3% | — | 50.0% |

### `config-server`

_Sin reporte de cobertura (módulo sin pruebas)._

### `auth-service`

**Total:** 42.1% de líneas · 69.2% de ramas · 50.0% de métodos

| Paquete | Clase | Líneas cub. / tot. | Líneas % | Ramas % | Métodos % |
|---|---|---:|---:|---:|---:|
| `com.nttdata.auth_service.api` | `ApiUtil` | 0 / 6 | 0.0% | — | 0.0% |
| `com.nttdata.auth_service.mapper` | `AuthMapper` | 5 / 5 | 100.0% | — | 100.0% |
| `com.nttdata.auth_service.security` | `JwtService` | 1 / 18 | 5.6% | 0.0% | 50.0% |
| `com.nttdata.auth_service.exception` | `InvalidCredentialsException` | 2 / 2 | 100.0% | — | 100.0% |
| `com.nttdata.auth_service.service.impl` | `AuthServiceImpl` | 28 / 28 | 100.0% | 81.8% | 100.0% |
| `com.nttdata.auth_service.controller` | `AuthController` | 9 / 9 | 100.0% | — | 100.0% |
| `com.nttdata.auth_service.config` | `DataInitializer` | 0 / 13 | 0.0% | 0.0% | 0.0% |
| `com.nttdata.auth_service.config` | `SecurityConfig` | 0 / 26 | 0.0% | — | 0.0% |

### `customer-service`

**Total:** 93.8% de líneas · 50.0% de ramas · 93.2% de métodos

| Paquete | Clase | Líneas cub. / tot. | Líneas % | Ramas % | Métodos % |
|---|---|---:|---:|---:|---:|
| `com.nttdata.customer_service.mapper` | `CustomerMapper` | 13 / 14 | 92.9% | 50.0% | 66.7% |
| `com.nttdata.customer_service.cache` | `CustomerCache` | 22 / 22 | 100.0% | — | 100.0% |
| `com.nttdata.customer_service.config` | `SecurityConfig` | 22 / 22 | 100.0% | — | 100.0% |
| `com.nttdata.customerservice.api` | `ApiUtil` | 0 / 6 | 0.0% | — | 0.0% |
| `com.nttdata.customer_service.service.impl` | `CustomerServiceImpl` | 23 / 23 | 100.0% | 50.0% | 100.0% |
| `com.nttdata.customer_service.controller` | `CustomerController` | 19 / 19 | 100.0% | — | 100.0% |
| `com.nttdata.customer_service.enums` | `Profile` | 4 / 4 | 100.0% | — | 100.0% |
| `com.nttdata.customer_service.enums` | `CustomerType` | 3 / 3 | 100.0% | — | 100.0% |

### `bank-account-service`

**Total:** 95.2% de líneas · 73.1% de ramas · 94.4% de métodos

| Paquete | Clase | Líneas cub. / tot. | Líneas % | Ramas % | Métodos % |
|---|---|---:|---:|---:|---:|
| `com.nttdata.bank_account_service.config` | `WebClientConfig` | 6 / 15 | 40.0% | 16.7% | 66.7% |
| `com.nttdata.bank_account_service.config` | `KafkaConfig` | 3 / 3 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.config` | `SecurityConfig` | 22 / 22 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.mapper` | `BankAccountMapper` | 21 / 21 | 100.0% | 50.0% | 100.0% |
| `com.nttdata.bank_account_service.mapper` | `TransactionMapper` | 17 / 17 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.mapper` | `DebitCardMapper` | 18 / 18 | 100.0% | 50.0% | 100.0% |
| `com.nttdata.bank_account_service.enums` | `CustomerType` | 3 / 3 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.enums` | `WalletOperationType` | 3 / 3 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.enums` | `DebitCardStatus` | 3 / 3 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.enums` | `TransactionType` | 6 / 6 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.enums` | `WalletOperationStatus` | 3 / 3 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.enums` | `AccountType` | 6 / 6 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.messaging` | `WalletOperationConsumer` | 49 / 50 | 98.0% | 68.8% | 93.8% |
| `com.nttdata.bank_account_service.messaging` | `KafkaEventPublisher` | 13 / 14 | 92.9% | 50.0% | 100.0% |
| `com.nttdata.bank_account_service.client.impl` | `CreditClientAdapter` | 21 / 21 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.client.impl` | `CustomerClientAdapter` | 1 / 10 | 10.0% | — | 25.0% |
| `com.nttdata.bank_account_service.service.impl` | `BankAccountServiceImpl` | 89 / 92 | 96.7% | 74.1% | 100.0% |
| `com.nttdata.bank_account_service.service.impl` | `ReportServiceImpl` | 42 / 42 | 100.0% | 66.7% | 92.3% |
| `com.nttdata.bank_account_service.service.impl` | `DebitCardServiceImpl` | 63 / 63 | 100.0% | 71.4% | 100.0% |
| `com.nttdata.bank_account_service.service.impl` | `TransactionServiceImpl` | 109 / 109 | 100.0% | 85.7% | 100.0% |
| `com.nttdata.bank_account_service.api` | `ApiUtil` | 0 / 6 | 0.0% | — | 0.0% |
| `com.nttdata.bank_account_service.controller` | `ReportController` | 6 / 6 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.controller` | `DebitCardController` | 27 / 27 | 100.0% | — | 100.0% |
| `com.nttdata.bank_account_service.controller` | `BankAccountController` | 48 / 48 | 100.0% | — | 100.0% |

### `credit-service`

**Total:** 94.3% de líneas · 66.2% de ramas · 92.9% de métodos

| Paquete | Clase | Líneas cub. / tot. | Líneas % | Ramas % | Métodos % |
|---|---|---:|---:|---:|---:|
| `com.nttdata.credit_service.api` | `ApiUtil` | 0 / 6 | 0.0% | — | 0.0% |
| `com.nttdata.credit_service.service.impl` | `ReportServiceImpl` | 42 / 42 | 100.0% | 66.7% | 92.3% |
| `com.nttdata.credit_service.service.impl` | `CreditServiceImpl` | 131 / 133 | 98.5% | 76.2% | 100.0% |
| `com.nttdata.credit_service.config` | `SecurityConfig` | 22 / 22 | 100.0% | — | 100.0% |
| `com.nttdata.credit_service.config` | `KafkaConfig` | 3 / 3 | 100.0% | — | 100.0% |
| `com.nttdata.credit_service.config` | `WebClientConfig` | 6 / 15 | 40.0% | 16.7% | 66.7% |
| `com.nttdata.credit_service.enums` | `CustomerType` | 3 / 3 | 100.0% | — | 100.0% |
| `com.nttdata.credit_service.enums` | `MovementType` | 3 / 3 | 100.0% | — | 100.0% |
| `com.nttdata.credit_service.enums` | `CreditType` | 4 / 4 | 100.0% | — | 100.0% |
| `com.nttdata.credit_service.client.impl` | `CustomerClientAdapter` | 10 / 10 | 100.0% | — | 75.0% |
| `com.nttdata.credit_service.mapper` | `CreditMapper` | 15 / 16 | 93.8% | 50.0% | 66.7% |
| `com.nttdata.credit_service.mapper` | `CreditMovementMapper` | 18 / 18 | 100.0% | 75.0% | 100.0% |
| `com.nttdata.credit_service.controller` | `CreditController` | 57 / 57 | 100.0% | — | 100.0% |
| `com.nttdata.credit_service.controller` | `ReportController` | 6 / 6 | 100.0% | — | 100.0% |
| `com.nttdata.credit_service.messaging` | `KafkaEventPublisher` | 12 / 14 | 85.7% | 50.0% | 100.0% |

### `yanki-service`

**Total:** 84.8% de líneas · 72.9% de ramas · 81.4% de métodos

| Paquete | Clase | Líneas cub. / tot. | Líneas % | Ramas % | Métodos % |
|---|---|---:|---:|---:|---:|
| `com.nttdata.yanki_service.api` | `ApiUtil` | 0 / 6 | 0.0% | — | 0.0% |
| `com.nttdata.yanki_service.enums` | `WalletOperationType` | 3 / 3 | 100.0% | — | 100.0% |
| `com.nttdata.yanki_service.enums` | `WalletOperationStatus` | 4 / 4 | 100.0% | — | 100.0% |
| `com.nttdata.yanki_service.service.impl` | `WalletServiceImpl` | 92 / 92 | 100.0% | 71.4% | 100.0% |
| `com.nttdata.yanki_service.controller` | `WalletController` | 30 / 32 | 93.8% | — | 94.1% |
| `com.nttdata.yanki_service.config` | `KafkaConfig` | 0 / 3 | 0.0% | — | 0.0% |
| `com.nttdata.yanki_service.config` | `SecurityConfig` | 0 / 22 | 0.0% | — | 0.0% |
| `com.nttdata.yanki_service.messaging` | `WalletOperationResultConsumer` | 38 / 38 | 100.0% | 83.3% | 100.0% |
| `com.nttdata.yanki_service.messaging` | `KafkaEventPublisher` | 13 / 15 | 86.7% | 50.0% | 100.0% |
| `com.nttdata.yanki_service.mapper` | `WalletMapper` | 16 / 16 | 100.0% | — | 100.0% |

### `api-gateway`

**Total:** 33.3% de líneas · — de ramas · 50.0% de métodos

| Paquete | Clase | Líneas cub. / tot. | Líneas % | Ramas % | Métodos % |
|---|---|---:|---:|---:|---:|
| `com.nttdata.api_gateway` | `ApiGatewayApplication` | 1 / 3 | 33.3% | — | 50.0% |

---

## 4. Notas metodológicas

- **Agregación por CSV.** Cada módulo ejecuta `jacoco:report` en la fase `test` y escribe
  `target/site/jacoco/jacoco.csv`. Este informe suma esas filas. No se fusionaron los
  archivos `.exec` con la CLI de JaCoCo porque cada módulo declara **exclusiones
  propias** y el reporte fusionado incluiría clases que el build real excluye, dando
  porcentajes distintos a los verdaderos.
- **Exclusiones aplicadas por módulo** (en sus respectivos `pom.xml`):
  `com/sun/tools/**`, `**/dto/**`, `**/model/**`, `**/*Application*`.
  `bank-account-service` y `credit-service` afinan además sus propias exclusiones.
- **Database per service:** las clases `*Application`, DTOs y modelos se excluyen porque
  son *plumbing* de framework y mapeo, sin lógica de negocio verificable.
- **Regenerar:** ejecutar `mvnw clean verify` en cada módulo (o
  `C:\Users\mhuamani\AppData\Local\Temp\opencode\build-cobertura.ps1`) y después
  `node generar-reportecobertura.js`.

## 5. Artefactos

| Archivo | Contenido |
|---|---|
| `coverage-report.md` | Este informe consolidado. |
| `jacoco-consolidado.csv` | Datos por clase de los 8 módulos con pruebas (para gráficas o importar a Excel). |

