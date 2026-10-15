# Sistema Bancario de Microservicios — Entregables

Repositorio de **artefactos y entregables** del proyecto bancario (Parte I y Parte II):
diagramas de diseño y la colección de pruebas manuales de Postman.

El código de cada microservicio vive en **su propio repositorio** (ver más abajo), tal como
exige el enunciado; aquí solo se versionan la documentación y los artefactos de prueba.

## Contenido de este repositorio

| Ruta | Descripción |
|---|---|
| `docs/diagrams/` | Diagramas en formato **draw.io (XML)**: diseño de la solución, diagramas de secuencia por microservicio y diagramas UML (clases y casos de uso). |
| `postman/` | Colección de **Postman** con los escenarios de éxito y de validación de reglas de negocio de los tres servicios. |

## Repositorios de la solución

| Repositorio | Puerto | Descripción |
|---|---|---|
| `discovery-server` | 8761 | Servidor **Eureka** con panel de control y registro de todas las APIs. |
| `config-server` | 8888 | **Config Server** que sirve la configuración centralizada desde el repositorio `bank-config-repo`. |
| `api-gateway` | 8089 | **API Gateway** (Spring Cloud Gateway): enruta `/customers/**`, `/bank-accounts/**` y `/credits/**`. |
| `customer-service` | 8081 | Clientes del banco (PERSONAL / BUSINESS, con perfiles VIP y PYME). |
| `bank-account-service` | 8082 | Productos **Pasivos**: cuentas de ahorro, corriente y plazo fijo, sus movimientos y transferencias. |
| `credit-service` | 8083 | Productos **Activos**: créditos personales y empresariales, tarjetas de crédito, pagos y consumos. |
| `bank-resilience-commons` | — | Librería compartida con el estándar de resiliencia (Resilience4j) y el manejo uniforme de errores. |
| `bank-config-repo` | — | Configuración externalizada consumida por el Config Server. |

## Documentación

- **[Diagramas draw.io](docs/diagrams/README.md)** — guía de apertura e índice de los 8 diagramas.
- **[Colección Postman](postman/)** — incluye cada caso de éxito y de error de las reglas de negocio.

## Orden de arranque de la solución

```text
1. MongoDB                        (puerto 27017)
2. config-server                  (puerto 8888)
3. discovery-server               (puerto 8761)
4. customer-service               (puerto 8081)
5. bank-account-service           (puerto 8082)
6. credit-service                 (puerto 8083)
7. api-gateway                    (puerto 8089)
```

Los microservicios toman su configuración del Config Server y se registran en Eureka.

## Arquitectura en una imagen

```text
                         ┌──────────────────┐
        Cliente  ──────► │   api-gateway    │ :8089
                         └────────┬─────────┘
                                  │  (enruta /customers, /bank-accounts, /credits)
             ┌────────────────────┼────────────────────┐
             ▼                    ▼                    ▼
    ┌────────────────┐  ┌────────────────────┐  ┌────────────────┐
    │customer-service│  │bank-account-service│  │ credit-service │
    │     :8081      │◄─┤       :8082        │  │     :8083      │
    └───────┬────────┘  └─────────┬──────────┘  └───────┬────────┘
            │                     │  └──────────────►   │
            ▼                     ▼                     ▼
      customer_db         bank_account_db         credit_db
            ▲                     ▲                     ▲
            └────────────┬────────┴──────────┬──────────┘
                         │                   │
                 ┌───────┴──────┐    ┌───────┴────────┐
                 │discovery-srv │    │ config-server  │
                 │   :8761      │    │     :8888      │
                 └──────────────┘    └───────┬────────┘
                                             ▼
                                    bank-config-repo (Git)
```

Las llamadas entre servicios (`bank-account-service` → `customer-service` / `credit-service`
y `credit-service` → `customer-service`) están protegidas con **circuit breaker** y
**time limiter** de Resilience4j, con *fallback* a HTTP 503 y timeout de 2 segundos.

> Cada servicio usa su propia base de datos (**database per service**): `customer_db`,
> `bank_account_db` y `credit_db`, configuradas en `bank-config-repo` mediante
> `spring.data.mongodb.uri`.

## Pruebas manuales con Postman

1. Importar `postman/Bank-System-Microservices.postman_collection_v2.json`.
2. Revisar la colección: incluye escenarios de éxito y de error para las reglas de negocio
   de clientes, cuentas, movimientos, transferencias, créditos, pagos, consumos y saldos.
3. Ejecutar las carpetas en orden (1 → 5); hay variables de colección con las URL y los
   identificadores usados por los escenarios.
