# Microservicio Envio (envio_service)

Microservicio del sistema **EcoMarket SPA** encargado de la gestion de envios y rutas de entrega. Crea envios a partir de pedidos, controla su estado de despacho y administra rutas de entrega que agrupan varios envios.

## Tecnologias

- Java 25
- Spring Boot 4.1.0
- Spring Data JPA
- MySQL (produccion) / H2 (pruebas)
- Lombok
- Maven

## Puerto

```
9095
```

## Estructura del paquete

`com.example.envio_service`

## Modelo de datos

### Entidad: Envio (tabla `envios`)

| Campo | Tipo | Descripcion |
|---|---|---|
| idEnvio | Long | Identificador, autogenerado |
| idPedido | Long | Referencia al pedido, debe ser valido (obligatorio) |
| direccionEnvio | String | Direccion de entrega (se completa desde el Pedido) |
| estadoEnvio | EstadoEnvio | Estado del envio (enum) |
| fechaEnvio | LocalDate | Fecha en que se genera el envio |
| fechaEntrega | LocalDate | Fecha de entrega (se fija al pasar a ENTREGADO) |

### Enum: EstadoEnvio

`PREPARANDO`, `EN_CAMINO`, `ENTREGADO`

### Entidad: RutaEntrega (tabla `rutas_entrega`)

| Campo | Tipo | Descripcion |
|---|---|---|
| idRuta | Long | Identificador, autogenerado |
| comunaDestino | String | Comuna de destino de la ruta (obligatorio) |
| enviosIds | List<Long> | Lista de ids de envios incluidos en la ruta (al menos uno) |

### DTO: PedidoDTO

Datos que se reciben del Pedido: `idPedido`, `nombreCliente` y `direccionEnvio` (objeto que se aplana a un String con calle, numero, comuna y ciudad).

## Endpoints

### Envios — ruta base `/api/v1/envios`

| Metodo | Ruta | Descripcion | Respuestas |
|---|---|---|---|
| POST | `/api/v1/envios` | Crea un envio a partir de un pedido | 200 OK / 404 NOT FOUND / 409 CONFLICT |
| GET | `/api/v1/envios` | Lista todos los envios | 200 OK / 404 NOT FOUND |
| GET | `/api/v1/envios/{id}` | Obtiene un envio por id | 200 OK / 404 NOT FOUND |
| PUT | `/api/v1/envios/{id}/estado?estadoEnvio=` | Actualiza el estado del envio | 200 OK / 404 NOT FOUND |
| DELETE | `/api/v1/envios/{id}` | Elimina un envio | 200 OK / 404 NOT FOUND |

### Rutas de entrega — ruta base `/api/v1/rutas`

| Metodo | Ruta | Descripcion | Respuestas |
|---|---|---|---|
| POST | `/api/v1/rutas` | Crea una ruta de entrega | 200 OK / 409 CONFLICT |
| GET | `/api/v1/rutas` | Lista todas las rutas | 200 OK / 404 NOT FOUND |
| GET | `/api/v1/rutas/{id}` | Obtiene una ruta por id | 200 OK / 404 NOT FOUND |
| PUT | `/api/v1/rutas/{id}` | Actualiza una ruta | 200 OK / 404 NOT FOUND |
| DELETE | `/api/v1/rutas/{id}` | Elimina una ruta | 200 OK / 404 NOT FOUND |

## Logica destacada

- Al crear un envio se consulta el Pedido; si existe, se copia su direccion, se fija la fecha de envio y, si no trae estado, queda en `PREPARANDO`.
- Al actualizar el estado a `ENTREGADO`, se fija automaticamente la `fechaEntrega`.

## Comunicacion con otros microservicios

Mediante `RestTemplate`:

- **Pedidos** (`http://localhost:8093/api/v1/pedidos/{idPedido}`): obtiene los datos del pedido (direccion, cliente) al crear el envio.


## Como ejecutar las pruebas

```bash
./mvnw test
```

Las pruebas usan H2 en memoria (perfil `test`). Incluye pruebas unitarias de los dos services (Mockito), de los dos controllers (`@WebMvcTest`) y de integracion (`@SpringBootTest` con H2).
