# Sistema Bancario de Microservicios — Entregables

Repositorio de **artefactos y entregables** del proyecto bancario (**Parte I, Parte II y
Parte III**): diagramas de diseño y la colección de pruebas manuales de Postman.

El código de cada microservicio vive en **su propio repositorio** (ver más abajo), tal como
exige el enunciado; aquí solo se versionan la documentación y los artefactos de prueba.

## Contenido de este repositorio

| Ruta | Descripción |
|---|---|
| `docs/diagrams/` | Diagramas en formato **draw.io (XML)**: diseño de la solución, diagramas de secuencia por microservicio y por requerimiento (58 endpoints) y diagramas UML (clases y casos de uso). |
| `docs/historias-de-usuario.md` | Cada funcionalidad del sistema descrita como **historia de usuario** con sus criterios de aceptación. |
| `bank-postman/` | Repositorio dedicado con la colección de **Postman**: escenarios de éxito y de validación de reglas de negocio de todos los servicios, incluido el login JWT. |
| `docker-compose.yml` | Orquestación de **todos los contenedores**: infraestructura (MongoDB, Kafka, Redis), plataforma (discovery, config, gateway) y microservicios de negocio. |
| `deploy/config-repo/` | Configuración externalizada **específica del entorno Docker** (hostnames de la red), servida por el `config-server` en modo `native`. |

> Cada repositorio de microservicio incluye su **propio `Dockerfile`** (build multi-etapa), tal
> como exige la Parte II: “cada microservicio deberá estar en un contenedor independiente”.

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
| `bank-postman` | — | Repositorio dedicado con la **colección Postman** (requisito de la Parte III: “mantener un repositorio Postman”). |

## Documentación

- **[Historias de usuario](docs/historias-de-usuario.md)** — cada funcionalidad del sistema (Partes I, II y III) descrita como historia de usuario con criterios de aceptación y endpoints.
- **[Diagramas draw.io](docs/diagrams/README.md)** — guía de apertura e índice de los diagramas.
- **[Colección Postman](bank-postman/)** — incluye cada caso de éxito y de error de las reglas de negocio.

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

## Despliegue con Docker (Parte II)

Los microservicios pueden levantarse en **contenedores independientes** (requerimiento
“deseable” de la Parte II). Todo el sistema se orquesta con `docker-compose.yml`:

```bash
# Construir las imágenes y levantar todo (infraestructura + plataforma + negocio)
docker compose up -d --build

# Ver el estado de los contenedores
docker compose ps

# Seguir los logs de un servicio concreto
docker compose logs -f bank-account-service

# Detener todo (y borrar los volúmenes de datos con -v)
docker compose down
```

### Contenedores

| Contenedor | Imagen / build | Puerto host | Rol |
|---|---|---|---|
| `bank-mongodb` | `mongo:7.0` | 27017 | Base de datos **documental** (una base por servicio). |
| `bank-redis` | `redis:7-alpine` | 6379 | **Caché** de datos maestros (Parte III). |
| `bank-kafka` | `confluentinc/cp-kafka:7.6.1` | 9092 | **Message broker** en modo **KRaft** (sin ZooKeeper). |
| `bank-discovery-server` | `discovery-server/Dockerfile` | 8761 | **Eureka** (registro de APIs). |
| `bank-config-server` | `config-server/Dockerfile` | 8888 | **Config Server** en perfil `native`. |
| `bank-api-gateway` | `api-gateway/Dockerfile` | 8089 | **API Gateway** (enruta y propaga el JWT). |
| `bank-auth-service` | `auth-service/Dockerfile` | 8084 | Autenticación **JWT**. |
| `bank-customer-service` | `customer-service/Dockerfile` | 8081 | Clientes (con caché **Redis**). |
| `bank-bank-account-service` | `bank-account-service/Dockerfile` | 8082 | Productos pasivos + tarjetas de débito. |
| `bank-credit-service` | `credit-service/Dockerfile` | 8083 | Productos activos. |
| `bank-yanki-service` | `yanki-service/Dockerfile` | 8085 | Monedero móvil **Yanki**. |

### Cómo se resuelve la configuración

- El `config-server` se ejecuta con el perfil **`native`** y sirve el repositorio
  `deploy/config-repo/` (montado en `/config-repo`), que contiene los **mismos valores** de
  `bank-config-repo` pero con los **hostnames de la red Docker** (`mongodb`, `kafka`, `redis`,
  `discovery-server`, `customer-service`, `credit-service`) en lugar de `localhost`.
- Cada microservicio apunta al config-server con
  `SPRING_CONFIG_IMPORT=optional:configserver:http://config-server:8888` y a Eureka con
  `EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://discovery-server:8761/eureka/`.
- `api-gateway` también obtiene su configuración (puerto y rutas de Spring Cloud Gateway) del
  config-server mediante `SPRING_CONFIG_IMPORT`; solo la URL de Eureka se ajusta por variable
  de entorno.
- El arranque se ordena con `depends_on` + *healthchecks* (`service_healthy`), de modo que los
  servicios esperan a MongoDB/Redis/Kafka y a la plataforma antes de iniciar.

> **Requisito:** el *build* multi-etapa compila cada servicio con Maven e instala antes la
> librería compartida `bank-resilience-commons`; por eso el **contexto de build** de cada
> `Dockerfile` es la **raíz** del proyecto (`docker compose` ya lo configura así).

## Pruebas manuales con Postman

1. Importar `bank-postman/Bank-System-Microservices.postman_collection_v2.json`.
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

Además existe un **[reporte de cobertura consolidado](docs/coverage/coverage-report.md)**
que agrega los 9 módulos (no hay un `pom.xml` raíz, así que cada proyecto se compila por
separado). Resumen:

| Métrica | Resultado |
|---|---|
| **Pruebas ejecutadas** | **296** · 0 fallos · 0 errores |
| **Líneas cubiertas** | **93.2%** (1.379 / 1.480) |
| **Ramas cubiertas** | **72.2%** (273 / 378) |
| **Métodos cubiertos** | **91.3%** (430 / 471) |
| Clases analizadas | 70 |

| Módulo | Tests | Líneas |
|---|---:|---:|
| `bank-account-service` | 119 | 95.2% |
| `auth-service` | 26 | 94.4% |
| `credit-service` | 67 | 94.3% |
| `customer-service` | 27 | 93.8% |
| `bank-resilience-commons` | 5 | 88.6% |
| `yanki-service` | 48 | 86.8% |
| `discovery-server` | 1 | n/a |
| `api-gateway` | 1 | n/a |
| `config-server` | 2 | n/a |

> Los **9 módulos declaran las mismas exclusiones** (`**/dto/**`, `**/model/**`,
> `**/*Application*` y `com/sun/tools/**`), de modo que los porcentajes son comparables
> entre sí. Por eso `discovery-server`, `config-server` y `api-gateway` figuran como
> **`n/a`**: sus tests corren y validan, pero su única clase es la principal
> `*Application` (plumbing de Spring Boot), excluida en todos los módulos.

> Los datos por clase están en [`docs/coverage/jacoco-consolidado.csv`](docs/coverage/jacoco-consolidado.csv),
> y el informe metodológico (exclusiones, cómo regenerar) en
> [`docs/coverage/coverage-report.md`](docs/coverage/coverage-report.md).
