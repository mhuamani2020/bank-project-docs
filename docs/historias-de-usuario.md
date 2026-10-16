# Historias de Usuario — Sistema Bancario de Microservicios

Este documento describe **cada funcionalidad del sistema** (Partes I, II y III del enunciado)
en formato de **historia de usuario**:

> **Como** &lt;rol&gt;, **quiero** &lt;funcionalidad&gt;, **para** &lt;beneficio&gt;.

Cada historia incluye sus **criterios de aceptación** y la **trazabilidad** con el
microservicio y los endpoints que la implementan (contratos OpenAPI, *contract-first*).

## Roles

| Rol | Descripción |
|---|---|
| **Administrador/Operador del banco** | Gestiona el catálogo de clientes y productos del banco. |
| **Cliente** | Titular de productos bancarios (persona natural o jurídica). |
| **Cliente personal** | Persona natural. |
| **Cliente empresarial** | Persona jurídica. |
| **Cliente VIP** | Cliente personal con perfil preferente. |
| **Cliente PYME** | Cliente empresarial con perfil preferente. |
| **Usuario Yanki** | Persona con monedero móvil (no requiere ser cliente del banco). |
| **Analista de negocio** | Consulta reportes e indicadores. |

## Índice de épicas

1. [Catálogo de clientes](#1-catálogo-de-clientes-customer-service)
2. [Productos pasivos — cuentas bancarias](#2-productos-pasivos--cuentas-bancarias-bank-account-service)
3. [Productos activos — créditos y tarjetas](#3-productos-activos--créditos-y-tarjetas-credit-service)
4. [Perfiles de cliente VIP y PYME](#4-perfiles-de-cliente-vip-y-pyme-parte-ii)
5. [Comisiones, transferencias y reportes](#5-comisiones-transferencias-y-reportes-parte-ii)
6. [Seguridad con JWT](#6-seguridad-con-jwt-parte-iii)
7. [Nuevas reglas de crédito](#7-nuevas-reglas-de-crédito-parte-iii)
8. [Tarjetas de débito](#8-tarjetas-de-débito-parte-iii)
9. [Monedero móvil Yanki](#9-monedero-móvil-yanki-parte-iii)
10. [Requerimientos técnicos y no funcionales](#10-requerimientos-técnicos-y-no-funcionales)

---

## 1. Catálogo de clientes (customer-service)

### US-01 — Registrar un cliente
**Como** administrador del banco, **quiero** registrar los datos de un cliente **para** mantener actualizado el catálogo de clientes.

**Criterios de aceptación**
- Se registra con tipo `PERSONAL` o `BUSINESS` y sus datos de identificación.
- `POST /customers` responde `201 Created` con el cliente creado.
- Los datos inválidos responden `400 Bad Request`.

**Servicio/endpoint:** `customer-service` · `POST /customers`

### US-02 — Consultar clientes
**Como** administrador del banco, **quiero** listar y consultar clientes por su identificador **para** verificar su información antes de operar.

**Criterios de aceptación**
- `GET /customers` devuelve todos los clientes (`200`).
- `GET /customers/{id}` devuelve un cliente o `404` si no existe.

**Servicio/endpoint:** `customer-service` · `GET /customers`, `GET /customers/{id}`

### US-03 — Actualizar un cliente
**Como** administrador del banco, **quiero** modificar los datos de un cliente **para** corregir o ampliar su información.

**Criterios de aceptación**
- `PUT /customers/{id}` actualiza el cliente y responde `200`.
- Si el cliente no existe, responde `404`.

**Servicio/endpoint:** `customer-service` · `PUT /customers/{id}`

### US-04 — Eliminar un cliente
**Como** administrador del banco, **quiero** dar de baja un cliente **para** retirarlo del catálogo.

**Criterios de aceptación**
- `DELETE /customers/{id}` elimina el cliente y responde `204`.
- Si no existe, responde `404`.

**Servicio/endpoint:** `customer-service` · `DELETE /customers/{id}`

---

## 2. Productos pasivos — cuentas bancarias (bank-account-service)

### US-05 — Gestionar (CRUD) cuentas bancarias
**Como** administrador del banco, **quiero** crear, listar, actualizar y eliminar cuentas **para** administrar los productos pasivos de los clientes.

**Criterios de aceptación**
- `POST /bank-accounts` (`201`), `GET /bank-accounts` (`200`), `GET /bank-accounts/{id}` (`200`/`404`), `PUT /bank-accounts/{id}` (`200`/`404`), `DELETE /bank-accounts/{id}` (`204`/`404`).
- Al crear, se valida que el cliente exista vía **customer-service** (con *circuit breaker*).

**Servicio/endpoint:** `bank-account-service` · `/bank-accounts`

### US-06 — Cuenta de ahorro
**Como** cliente personal, **quiero** una cuenta de ahorro **para** guardar mi dinero sin comisión de mantenimiento.

**Criterios de aceptación**
- El producto `SAVINGS` no cobra comisión de mantenimiento.
- Tiene un **límite máximo de movimientos mensuales**; al superarlo se cobra comisión por exceso.

**Servicio/endpoint:** `bank-account-service` · `POST /bank-accounts/{accountId}/transactions`

### US-07 — Cuenta corriente
**Como** cliente, **quiero** una cuenta corriente **para** realizar movimientos sin límite mensual.

**Criterios de aceptación**
- El producto `CURRENT` cobra **comisión de mantenimiento**.
- No tiene límite de movimientos mensuales.

**Servicio/endpoint:** `bank-account-service` · `/bank-accounts`

### US-08 — Cuenta a plazo fijo
**Como** cliente, **quiero** una cuenta a plazo fijo **para** depositar o retirar en una fecha específica del mes.

**Criterios de aceptación**
- El producto `FIXED_TERM` no cobra comisión de mantenimiento.
- Solo permite **un movimiento** (retiro o depósito) en un **día específico del mes**.

**Servicio/endpoint:** `bank-account-service` · `POST /bank-accounts/{accountId}/transactions`

### US-09 — Restricciones de productos por tipo de cliente
**Como** administrador del banco, **quiero** que se apliquen las reglas de productos según el tipo de cliente **para** cumplir la política del banco.

**Criterios de aceptación**
- Un **cliente personal** solo puede tener **una** cuenta de ahorro, **una** corriente o cuentas a plazo fijo.
- Un **cliente empresarial** no puede tener ahorro ni plazo fijo, pero sí **múltiples** cuentas corrientes.

**Servicio/endpoint:** `bank-account-service` · `POST /bank-accounts`

### US-10 — Titulares y firmantes de cuentas empresariales
**Como** cliente empresarial, **quiero** que mi cuenta tenga uno o más titulares y cero o más firmantes autorizados **para** operar de forma corporativa.

**Criterios de aceptación**
- Una cuenta empresarial admite **≥1 titular** y **≥0 firmantes** autorizados.

**Servicio/endpoint:** `bank-account-service` · `POST /bank-accounts`

### US-11 — Depositar y retirar
**Como** cliente, **quiero** hacer depósitos y retiros de mis cuentas **para** gestionar mi dinero.

**Criterios de aceptación**
- `POST /bank-accounts/{accountId}/transactions` registra el movimiento (`DEPOSIT`/`WITHDRAW`).
- El retiro valida que exista **saldo suficiente** (`400` si no).

**Servicio/endpoint:** `bank-account-service` · `POST /bank-accounts/{accountId}/transactions`

### US-12 — Consultar saldo disponible de una cuenta
**Como** cliente, **quiero** consultar el saldo disponible de mis cuentas **para** conocer mi dinero disponible.

**Criterios de aceptación**
- `GET /bank-accounts/{id}/balance` devuelve el saldo disponible (`200`).

**Servicio/endpoint:** `bank-account-service` · `GET /bank-accounts/{id}/balance`

### US-13 — Consultar movimientos de una cuenta
**Como** cliente, **quiero** ver todos los movimientos de mi producto **para** llevar el control de mis operaciones.

**Criterios de aceptación**
- `GET /bank-accounts/{accountId}/transactions` devuelve el historial (`200`).
- CRUD de movimientos disponible en `/bank-accounts/transactions`.

**Servicio/endpoint:** `bank-account-service` · `GET /bank-accounts/{accountId}/transactions`

---

## 3. Productos activos — créditos y tarjetas (credit-service)

### US-14 — Crédito personal (uno por persona)
**Como** cliente personal, **quiero** solicitar un crédito personal **para** financiar mis necesidades.

**Criterios de aceptación**
- Solo se permite **un** crédito personal por persona (`400` si ya tiene uno).

**Servicio/endpoint:** `credit-service` · `POST /credits`

### US-15 — Crédito empresarial (múltiples por empresa)
**Como** cliente empresarial, **quiero** solicitar varios créditos **para** financiar mi negocio.

**Criterios de aceptación**
- Se permite **más de un** crédito empresarial por empresa.

**Servicio/endpoint:** `credit-service` · `POST /credits`

### US-16 — Tarjeta de crédito
**Como** cliente, **quiero** una tarjeta de crédito personal o empresarial **para** realizar consumos.

**Criterios de aceptación**
- Se crea con su límite de crédito.
- `GET /credits/customer/{customerId}/has-credit-card` indica si el cliente posee tarjeta.

**Servicio/endpoint:** `credit-service` · `/credits`, `GET /credits/customer/{customerId}/has-credit-card`

### US-17 — Pagar un producto de crédito
**Como** cliente, **quiero** pagar mi crédito **para** reducir mi deuda.

**Criterios de aceptación**
- `POST /credits/{id}/payments` registra el pago y descuenta del saldo.
- Genera el movimiento correspondiente.

**Servicio/endpoint:** `credit-service` · `POST /credits/{id}/payments`

### US-18 — Cargar consumos a la tarjeta
**Como** cliente, **quiero** cargar consumos a mi tarjeta **para** usarla como medio de pago.

**Criterios de aceptación**
- `POST /credits/{id}/consumptions` valida el **límite de crédito disponible** (`400` si se excede).

**Servicio/endpoint:** `credit-service` · `POST /credits/{id}/consumptions`

### US-19 — Consultar saldo disponible de la tarjeta
**Como** cliente, **quiero** consultar el saldo disponible de mi tarjeta **para** conocer cuánto puedo consumir.

**Criterios de aceptación**
- `GET /credits/{id}/available-balance` devuelve el disponible (`200`).

**Servicio/endpoint:** `credit-service` · `GET /credits/{id}/available-balance`

### US-20 — Consultar movimientos de crédito
**Como** cliente, **quiero** ver los movimientos de mi crédito **para** controlar pagos y consumos.

**Criterios de aceptación**
- `GET /credits/{id}/movements` devuelve el historial (`200`).
- CRUD de movimientos en `/credits/movements`.

**Servicio/endpoint:** `credit-service` · `GET /credits/{id}/movements`

### US-21 — Crédito sin cuenta bancaria
**Como** cliente, **quiero** obtener un crédito sin necesidad de tener una cuenta bancaria **para** acceder al financiamiento.

**Criterios de aceptación**
- No se exige cuenta bancaria para crear el crédito.

**Servicio/endpoint:** `credit-service` · `POST /credits`

---

## 4. Perfiles de cliente VIP y PYME (Parte II)

### US-22 — Monto mínimo de apertura
**Como** administrador del banco, **quiero** definir un monto mínimo de apertura **para** cada cuenta.

**Criterios de aceptación**
- El monto mínimo **puede ser cero**.
- Al crear la cuenta se valida el monto (`400` si es menor al mínimo).

**Servicio/endpoint:** `bank-account-service` · `POST /bank-accounts`

### US-23 — Perfil VIP (personal)
**Como** cliente personal VIP, **quiero** una cuenta de ahorro preferente **para** acceder a mejores condiciones.

**Criterios de aceptación**
- La cuenta de ahorro VIP exige un **monto mínimo de promedio diario mensual**.
- Para solicitarla, el cliente **debe tener una tarjeta de crédito** al momento de la creación (se valida vía `credit-service`).
- El cliente debe estar marcado como `VIP`.

**Servicio/endpoint:** `bank-account-service` · `POST /bank-accounts`

### US-24 — Perfil PYME (empresarial)
**Como** cliente empresarial PYME, **quiero** una cuenta corriente sin comisión de mantenimiento **para** reducir mis costos.

**Criterios de aceptación**
- La cuenta corriente PYME **no cobra comisión de mantenimiento**.
- Para solicitarla, el cliente **debe tener una tarjeta de crédito** al momento de la creación (se valida vía `credit-service`).
- El cliente debe estar marcado como `PYME`.

**Servicio/endpoint:** `bank-account-service` · `POST /bank-accounts`

---

## 5. Comisiones, transferencias y reportes (Parte II)

### US-25 — Comisión por exceso de transacciones
**Como** administrador del banco, **quiero** cobrar comisión al superar el número máximo de transacciones sin costo **para** incentivar el uso eficiente.

**Criterios de aceptación**
- Cada cuenta tiene un **número máximo de transacciones** (depósitos y retiros) sin comisión.
- Superado ese número, se **cobra comisión por cada transacción** adicional.

**Servicio/endpoint:** `bank-account-service` · `POST /bank-accounts/{accountId}/transactions`

### US-26 — Transferencias bancarias
**Como** cliente, **quiero** transferir dinero entre mis cuentas y a terceros del mismo banco **para** mover fondos.

**Criterios de aceptación**
- `POST /bank-accounts/transfers` soporta cuentas **propias** y de **terceros del mismo banco**.
- Valida existencia de las cuentas y **saldo suficiente** (`400` en caso contrario).

**Servicio/endpoint:** `bank-account-service` · `POST /bank-accounts/transfers`

### US-27 — Reporte general por producto
**Como** analista de negocio, **quiero** un reporte general por producto en un intervalo de tiempo **para** analizar la operación del banco.

**Criterios de aceptación**
- `GET /reports/bank-accounts/products-summary?from=&to=` (cuentas) y `GET /reports/credits/products-summary?from=&to=` (créditos) devuelven el resumen por producto.
- El intervalo se recibe en formato `YYYY-MM-DD`.

**Servicio/endpoint:** `bank-account-service` / `credit-service` · `/reports/.../products-summary`

### US-28 — Últimos 10 movimientos de tarjeta
**Como** analista de negocio, **quiero** los últimos 10 movimientos de una tarjeta **para** revisar la actividad reciente.

**Criterios de aceptación**
- `GET /reports/debit-card/{cardId}/last-movements` y `GET /reports/credit-card/{creditId}/last-movements` devuelven hasta **10** movimientos.

**Servicio/endpoint:** `bank-account-service` / `credit-service` · `/reports/.../last-movements`

---

## 6. Seguridad con JWT (Parte III)

### US-29 — Iniciar sesión y obtener el token
**Como** usuario del sistema, **quiero** autenticarme y recibir un JWT **para** acceder a las APIs protegidas.

**Criterios de aceptación**
- `POST /auth/login` valida credenciales y devuelve `accessToken` (`200`).
- Credenciales inválidas responden `401`.
- El token incluye el claim `roles` y expira en 3600 s.

**Servicio/endpoint:** `auth-service` · `POST /auth/login`

### US-30 — Registrar usuario del sistema
**Como** administrador, **quiero** registrar usuarios **para** que puedan autenticarse.

**Criterios de aceptación**
- `POST /auth/register` crea el usuario (`201`); datos inválidos `400`.

**Servicio/endpoint:** `auth-service` · `POST /auth/register`

### US-31 — Proteger las APIs con JWT
**Como** responsable de seguridad, **quiero** que cada microservicio valide el JWT **para** impedir accesos no autorizados.

**Criterios de aceptación**
- Cada servicio de negocio actúa como **OAuth2 Resource Server** y valida la firma y vigencia.
- Sin token o token inválido → `401`; sin permisos → `403`.
- El `api-gateway` **propaga** el token (no lo valida); los servicios REST del banco se invocan con **token relay**.

**Servicio/endpoint:** `customer-service`, `bank-account-service`, `credit-service`, `yanki-service` · `SecurityConfig`

---

## 7. Nuevas reglas de crédito (Parte III)

### US-32 — Bloqueo por deuda vencida
**Como** administrador del banco, **quiero** impedir que un cliente con deuda vencida adquiera productos **para** mitigar el riesgo.

**Criterios de aceptación**
- Antes de crear una cuenta o un crédito se consulta `GET /credits/customer/{customerId}/has-overdue-debt`.
- Si el cliente tiene deuda vencida, la operación responde `400`.

**Servicio/endpoint:** `credit-service` (`has-overdue-debt`) · consumido por `bank-account-service`

### US-33 — Pago de crédito de terceros
**Como** cliente, **quiero** pagar el producto de crédito de un tercero **para** apoyar a otra persona o empresa.

**Criterios de aceptación**
- El pago admite un `payerCustomerId` distinto del titular del crédito.
- `POST /credits/{id}/payments` registra el pago indicando quién paga.

**Servicio/endpoint:** `credit-service` · `POST /credits/{id}/payments`

---

## 8. Tarjetas de débito (Parte III)

### US-34 — Asociar tarjeta de débito a una cuenta
**Como** cliente, **quiero** tener una tarjeta de débito asociada a mi cuenta **para** realizar pagos.

**Criterios de aceptación**
- `POST /debit-cards` crea la tarjeta asociada a una cuenta existente (`201`).
- `GET /debit-cards`, `GET /debit-cards/{id}` y `GET /debit-cards/customer/{customerId}` listan tarjetas.

**Servicio/endpoint:** `bank-account-service` · `/debit-cards`

### US-35 — Pagar con tarjeta de débito
**Como** cliente, **quiero** pagar con mi tarjeta de débito **para** debitar directamente de mi cuenta.

**Criterios de aceptación**
- `POST /debit-cards/{id}/payments` debita la **cuenta principal** asociada.
- Valida saldo y estado de la tarjeta (`400` si no procede).

**Servicio/endpoint:** `bank-account-service` · `POST /debit-cards/{id}/payments`

### US-36 — Ver movimientos de la tarjeta de débito
**Como** cliente, **quiero** ver los movimientos de mi tarjeta **para** controlar los pagos realizados.

**Criterios de aceptación**
- `GET /debit-cards/{id}/movements` devuelve el historial (`200`).

**Servicio/endpoint:** `bank-account-service` · `GET /debit-cards/{id}/movements`

---

## 9. Monedero móvil Yanki (Parte III)

> **Nota:** `yanki-service` es un microservicio **nuevo**; por la Parte III **no puede llamar por REST** a otros
> microservicios. Su comunicación con `bank-account-service` es **solo por Kafka**.

### US-37 — Registrar un monedero sin ser cliente
**Como** usuario Yanki, **quiero** crear un monedero con mis datos **para** usarlo aunque no sea cliente del banco.

**Criterios de aceptación**
- `POST /wallets` registra con **documento (DNI/CEX/Pasaporte), celular, IMEI y correo** (`201`).
- No se exige ser cliente del banco.
- Celular duplicado responde `400`.

**Servicio/endpoint:** `yanki-service` · `POST /wallets`

### US-38 — Enviar un pago a otro monedero por celular
**Como** usuario Yanki, **quiero** enviar dinero a otro monedero usando solo su número de celular **para** pagar de forma simple.

**Criterios de aceptación**
- `POST /wallets/{id}/payments` transfiere saldo al monedero destino identificado por celular.
- Valida saldo suficiente y que el destino exista (`400` si no).

**Servicio/endpoint:** `yanki-service` · `POST /wallets/{id}/payments`

### US-39 — Consultar mi monedero
**Como** usuario Yanki, **quiero** consultar mi monedero por identificador o celular **para** conocer mi saldo.

**Criterios de aceptación**
- `GET /wallets`, `GET /wallets/{id}` y `GET /wallets/by-cellphone/{cellphone}` devuelven el monedero (`200`/`404`).

**Servicio/endpoint:** `yanki-service` · `/wallets`

### US-40 — Asociar el monedero a una tarjeta de débito
**Como** usuario Yanki, **quiero** asociar mi monedero a una tarjeta de débito **para** cargar o acreditar saldo contra mi cuenta principal.

**Criterios de aceptación**
- `POST /wallets/{id}/debit-card` asocia la tarjeta (`200`).
- El saldo se carga/acredita **solo** a la cuenta principal asociada a esa tarjeta.

**Servicio/endpoint:** `yanki-service` · `POST /wallets/{id}/debit-card`

### US-41 — Cargar saldo desde la tarjeta de débito (asíncrono)
**Como** usuario Yanki, **quiero** cargar saldo a mi monedero desde mi tarjeta **para** disponer de dinero en el monedero.

**Criterios de aceptación**
- `POST /wallets/{id}/top-up` responde **`202 Accepted`** con la operación en estado `PENDING`.
- Publica el comando `TOP_UP` en el topic **`wallet-operations`**.
- `bank-account-service` debita la cuenta (`DEBIT_CARD_PAYMENT`) y publica el resultado en **`wallet-operation-results`**.
- Al recibir `COMPLETED`, el monedero **abona** el saldo; si `FAILED`, no altera el saldo.

**Servicio/endpoint:** `yanki-service` ⇄ `bank-account-service` (Kafka)

### US-42 — Retirar saldo hacia la cuenta principal (asíncrono)
**Como** usuario Yanki, **quiero** retirar saldo de mi monedero hacia mi cuenta **para** disponer del dinero en el banco.

**Criterios de aceptación**
- `POST /wallets/{id}/withdraw` responde **`202 Accepted`** con la operación `PENDING`.
- Publica el comando `WITHDRAW` en **`wallet-operations`**.
- `bank-account-service` **abona** la cuenta (`DEPOSIT`) y publica el resultado.
- Al recibir `COMPLETED`, el monedero **debita** el saldo; si `FAILED`, no lo altera.

**Servicio/endpoint:** `yanki-service` ⇄ `bank-account-service` (Kafka)

---

## 10. Requerimientos técnicos y no funcionales

### US-43 — Configuración externalizada
**Como** responsable técnico, **quiero** que la configuración esté fuera del código **para** parametrizar los entornos sin recompilar.
- **Criterios:** los servicios toman su configuración del **Config Server** (`bank-config-repo`); no hay configuraciones *hardcodeadas*.

### US-44 — Registro y descubrimiento de APIs
**Como** responsable técnico, **quiero** un registro de APIs **para** localizar los servicios por nombre.
- **Criterios:** **Eureka** (`discovery-server`, `:8761`) con panel de control; todos los servicios se registran.

### US-45 — API Gateway
**Como** responsable técnico, **quiero** un punto de entrada único **para** enrutar las peticiones a los servicios.
- **Criterios:** **Spring Cloud Gateway** (`:8089`) enruta `/customers/**`, `/bank-accounts/**`, `/credits/**`, `/wallets/**` y `/auth/**`, resuelve por Eureka y propaga el JWT.

### US-46 — Resiliencia (Circuit Breaker)
**Como** responsable técnico, **quiero** que las llamadas entre servicios sean resilientes **para** evitar fallos en cascada.
- **Criterios:** **Resilience4j** (`@CircuitBreaker` + `@TimeLimiter`) con **timeout de 2 s**; *fallback* a `503`.

### US-47 — Persistencia con Spring Data (sin SQL dinámico)
**Como** responsable técnico, **quiero** usar Spring Data sobre MongoDB **para** acceder a los datos sin SQL dinámico.
- **Criterios:** **database-per-service**; sin `@Query`; consultas derivadas; documentos NoSQL.

### US-48 — CRUD y endpoints REST
**Como** responsable técnico, **quiero** operaciones CRUD en cada entidad con endpoints REST **para** cumplir los lineamientos REST.
- **Criterios:** Create, FindAll, Update y Delete expuestos como endpoints REST en cada servicio.

### US-49 — Contratos OpenAPI
**Como** responsable técnico, **quiero** contratos OpenAPI por microservicio **para** definir las APIs *contract-first*.
- **Criterios:** cada servicio tiene su `openapi.yaml` y los controladores implementan las interfaces generadas.

### US-50 — Pruebas unitarias y cobertura
**Como** responsable técnico, **quiero** pruebas unitarias con *mocks* y reporte de cobertura **para** asegurar la calidad.
- **Criterios:** los nuevos métodos públicos tienen pruebas con **mocks**; **JaCoCo** genera el reporte de cobertura.

### US-51 — Estilo de código (Checkstyle)
**Como** responsable técnico, **quiero** un control de estilo **para** homogeneizar el código.
- **Criterios:** plugin **Checkstyle** en el `pom.xml` (modo informativo).

### US-52 — Programación reactiva (RxJava)
**Como** responsable técnico, **quiero** las nuevas funcionalidades con programación funcional y reactiva **para** un modelo no bloqueante.
- **Criterios:** capa de servicio en **RxJava 3**; controladores WebFlux que adaptan a Reactor; uso de *lambdas* y *streams*.

### US-53 — Arquitectura orientada a eventos (Kafka)
**Como** responsable técnico, **quiero** las nuevas funcionalidades orientadas a eventos **para** desacoplar los servicios.
- **Criterios:** **Kafka** como *message broker*; topics `bank-account-transactions`, `credit-movements`, `wallet-operations` y `wallet-operation-results`; mensajes **JSON en texto**.

### US-54 — Caché de datos maestros (Redis)
**Como** responsable técnico, **quiero** acelerar el acceso a datos maestros con caché **para** reducir la latencia.
- **Criterios:** **Redis** en `customer-service` (patrón *cache-aside* sobre el catálogo de clientes), con *fallback* a MongoDB si falla.

### US-55 — Diagramas y documentación
**Como** responsable técnico, **quiero** los diagramas de diseño, secuencia y UML en draw.io **para** documentar la solución.
- **Criterios:** diagrama de diseño de la solución, diagramas de secuencia por microservicio y diagramas UML (clases y casos de uso).

### US-56 — Repositorios Git y Postman
**Como** responsable técnico, **quiero** un repositorio por microservicio y un repositorio con las pruebas de Postman **para** versionar el entregable.
- **Criterios:** cada microservicio en su repositorio Git; colección Postman con los escenarios de éxito y de error.

### US-57 — Contenedores Docker (deseable)
**Como** responsable técnico, **quiero** desplegar cada microservicio en un contenedor independiente **para** facilitar el despliegue.
- **Criterios:** *deseable*; cada servicio empaquetable en su propio contenedor.
