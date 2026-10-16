# Sistema Bancario de Microservicios — Entregables

Repositorio de **artefactos y entregables** del proyecto bancario (**Parte I, Parte II y
Parte III**): diagramas de diseño y la colección de pruebas manuales de Postman.

El código de cada microservicio vive en **su propio repositorio** (ver más abajo), tal como
exige el enunciado; aquí solo se versionan la documentación y los artefactos de prueba.

## Contenido de este repositorio

| Ruta | Descripción |
|---|---|
| `docs/diagrams/` | Diagramas en formato **draw.io (XML)**: diseño de la solución, diagramas de secuencia por microservicio y diagramas UML (clases y casos de uso). |
| `postman/` | Colección de **Postman** con los escenarios de éxito y de validación de reglas de negocio de todos los servicios, incluido el login JWT. |

## Repositorios de la solución

| Repositorio | Puerto | Descripción |
|---|---|---|
| `discovery-server` | 8761 | Servidor **Eureka** con panel de control y registro de todas las APIs. |
| `config-server` | 8888 | **Config Server** que sirve la configuración centralizada desde el repositorio `bank-config-repo`. |
| `api-gateway` | 8089 | **API Gateway** (Spring Cloud Gateway): enruta `/customers/**`, `/bank-accounts/**`, `/credits/**`, `/wallets/**` y `/auth/**`, y **propaga el JWT**. |
| `auth-service` | 8084 | **Autenticación/autorización JWT** (Parte III): emite el token que el resto de servicios valida. |
| `customer-service` | 8081 | Clientes del banco (PERSONAL / BUSINESS, con perfiles VIP y PYME). Datos maestros cacheados en **Redis**. |
| `bank-account-service` | 8082 | Productos **Pasivos**: cuentas de ahorro, corriente y plazo fijo, movimientos, transferencias y **tarjetas de débito**. Publica eventos en **Kafka**. |
| `credit-service` | 8083 | Productos **Activos**: créditos personales y empresariales, tarjetas de crédito, pagos y consumos. Publica eventos en **Kafka**. |
| `yanki-service` | 8085 | **Monedero móvil Yanki** (Parte III): registro sin ser cliente, pagos entre monederos por celular y carga/retiro de saldo contra la tarjeta de débito. Se comunica por **Kafka** (nunca por REST). |
| `bank-resilience-commons` | — | Librería compartida con el estándar de resiliencia (Resilience4j) y el manejo uniforme de errores. |
| `bank-config-repo` | — | Configuración externalizada consumida por el Config Server. |

## Documentación

- **[Historias de usuario](docs/historias-de-usuario.md)** — cada funcionalidad del sistema (Partes I, II y III) descrita como historia de usuario con criterios de aceptación y endpoints.
- **[Diagramas draw.io](docs/diagrams/README.md)** — guía de apertura e índice de los diagramas.
- **[Colección Postman](postman/)** — incluye cada caso de éxito y de error de las reglas de negocio.

## Orden de arranque de la solución

```text
1.  MongoDB                        (puerto 27017)
2.  Redis                          (puerto 6379)  — caché de datos maestros (Parte III)
3.  Kafka (broker)                 (puerto 9092)  — arquitectura orientada a eventos (Parte III)
4.  config-server                  (puerto 8888)
5.  discovery-server               (puerto 8761)
6.  auth-service                   (puerto 8084)
7.  customer-service               (puerto 8081)
8.  bank-account-service           (puerto 8082)
9.  credit-service                 (puerto 8083)
10. yanki-service                  (puerto 8085)
11. api-gateway                    (puerto 8089)
```

Los microservicios toman su configuración del Config Server y se registran en Eureka.

## Arquitectura en una imagen

```text
                             ┌──────────────────┐
            Cliente  ──────► │   api-gateway    │ :8089   (propaga el JWT)
                             └────────┬─────────┘
                                      │  /customers /bank-accounts /credits /wallets /auth
        ┌───────────────┬────────────┼────────────┬───────────────┐
        ▼               ▼            ▼            ▼               ▼
 ┌────────────┐ ┌──────────────┐ ┌──────────┐ ┌───────────┐ ┌──────────────┐
 │auth-service│ │customer-svc  │ │bank-acct │ │credit-svc │ │yanki-service │
 │   :8084    │ │   :8081      │ │  :8082   │ │  :8083    │ │    :8085     │
 └─────┬──────┘ └──────┬───────┘ └────┬─────┘ └─────┬─────┘ └──────┬───────┘
       │               │              │             │              │
       ▼               ▼              ▼             ▼              ▼
   auth_db        customer_db   bank_account_db  credit_db     yanki_db
       ▲               ▲  ▲           ▲  ▲
       │               │  │           │  │            ┌───────────────────┐
       │         Redis │  └───────────┘  └──────────► │  Kafka :9092      │
       │        (caché) │   REST + circuit breaker    │ wallet-operations │
       │                 │                             │ wallet-operation- │
       │                 │                             │ results, etc.     │
       │                 │                             └───────────────────┘
       │                 │            (yanki-service ↔ bank-account-service: SOLO Kafka)
       │                 │
 ┌─────┴──────┐   ┌──────┴───────┐
 │discovery   │   │ config-server│
 │ :8761      │   │    :8888     │────► bank-config-repo (Git)
 └────────────┘   └──────────────┘
```

Las llamadas **REST** entre servicios de las Partes I/II (`bank-account-service` →
`customer-service` / `credit-service` y `credit-service` → `customer-service`) están protegidas
con **circuit breaker** y **time limiter** de Resilience4j, con *fallback* a HTTP 503 y timeout
de 2 segundos. En esas llamadas el servicio **propaga el JWT** recibido (token relay).

> Cada servicio usa su propia base de datos (**database per service**): `auth_db`,
> `customer_db`, `bank_account_db`, `credit_db` y `yanki_db`, configuradas en
> `bank-config-repo` mediante `spring.data.mongodb.uri`.

## Seguridad con JWT (Parte III)

1. El cliente se autentica en **`POST /auth/login`** (a través del gateway) y obtiene un **JWT**.
2. Envía ese token en la cabecera `Authorization: Bearer <token>` en todas las llamadas.
3. El **api-gateway** no valida el token: solo **enruta y lo propaga**.
4. Cada microservicio de negocio actúa como **OAuth2 Resource Server** y valida el JWT con la
   misma clave simétrica (`app.jwt.secret`, externalizada en `bank-config-repo`).
5. Los roles del claim `roles` se resuelven como autoridades `ROLE_*`.

Por defecto `bank-config-repo` crea el usuario **`admin` / `admin123`** (rol `ADMIN`) al
arrancar `auth-service`.

## Comunicación por eventos con Kafka (Parte III)

Las **nuevas funcionalidades** usan una arquitectura **orientada a eventos** con Kafka:

| Topic | Productor | Consumidor | Contenido |
|---|---|---|---|
| `bank-account-transactions` | bank-account-service | — | Evento de cada movimiento de cuenta. |
| `credit-movements` | credit-service | — | Evento de cada movimiento de crédito. |
| `wallet-operations` | yanki-service | bank-account-service | Comando de carga (`TOP_UP`) o retiro (`WITHDRAW`) contra la tarjeta de débito. |
| `wallet-operation-results` | bank-account-service | yanki-service | Resultado (`COMPLETED` / `FAILED`) de la operación de monedero. |

Los mensajes viajan como **JSON en texto**, de modo que los contratos no dependen de las clases
Java de cada repositorio. La publicación es **best-effort**: un fallo del broker se registra en
el log y no interrumpe la operación de negocio.

> **Regla de la Parte III:** los microservicios nuevos (`yanki-service`) **no invocan APIs por
> REST**; toda su comunicación con `bank-account-service` es por Kafka.

## Caché de datos maestros con Redis (Parte III)

El **catálogo de clientes** (dato maestro) se acelera con **Redis** en `customer-service`,
usando el patrón *cache-aside*: al consultar un cliente se busca primero en Redis y, si no
está, se lee de MongoDB y se refresca la caché; al actualizar o eliminar se invalida la
entrada. El TTL y la conexión se externalizan en `bank-config-repo` (`app.cache.*`,
`spring.data.redis.*`). Los fallos de Redis son **best-effort**: el servicio sigue respondiendo
desde MongoDB.

## Pruebas manuales con Postman

1. Importar `postman/Bank-System-Microservices.postman_collection_v2.json`.
2. Ejecutar primero **`POST /auth/login`**: guarda el token en la variable de colección
   `{{token}}` y el resto de peticiones lo usan en `Authorization: Bearer {{token}}`.
3. Revisar la colección: incluye escenarios de éxito y de error para clientes, cuentas,
   movimientos, transferencias, **tarjetas de débito**, **reportes**, créditos, pagos,
   consumos, saldos y **monedero Yanki**.
4. Ejecutar las carpetas en orden; hay variables de colección con las URL y los identificadores
   usados por los escenarios.

## Cobertura de pruebas

Cada microservicio incluye **pruebas unitarias** de los nuevos métodos públicos con *mocks*.
El reporte de cobertura se genera con **JaCoCo** al ejecutar `mvnw test` y queda en
`target/site/jacoco/index.html` de cada repositorio.
