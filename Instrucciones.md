## Escenario

La tienda crece: crea dos microservicios nuevos y conectalos al gateway existente.

Categorías -> 40 pts
Reseñas (Se comunica con productos) -> 50 pts
Pruebas y entrega -> 10 pts 


## Reglas y requisitos

- Material permitido: tus proyectos anteriores (products-ms, orders-ms, client-gateway) y tu tutotial.
- Configuración (puerto, DATABASE_URL, host y puerto de otros servicios) en .env, validada como en tus proyectos anteriores.
- Los errores se lanzan con new RpcException({ status, message }) y el gateway los devuelve al cliente como respuesta HTTP con ese código.
- Los nombres de los patrones de mensaje (`{ cmd: '...' }`) deben ser exactamente los indicados.

# PARTE 1 · Microservicio de Categorías (40 pts)

## Modelo Prisma `Category`

| Campo         | Tipo       | Restricciones                   |
| ------------- | ---------- | ------------------------------- |
| `id`          | `Int`      | Clave primaria, autoincremental |
| `name`        | `String`   | Único                           |
| `description` | `String?`  | Opcional                        |
| `createdAt`   | `DateTime` | Por defecto, fecha actual       |

## Patrones de mensaje

| Patrón                           | Payload                    | Respuesta                                   |
| -------------------------------- | -------------------------- | ------------------------------------------- |
| `{ cmd: 'create_category' }`     | `name`, `description?`     | La categoría creada                         |
| `{ cmd: 'find_all_categories' }` | `page?`, `limit?`          | `{ data, meta: { total, page, lastPage } }` |
| `{ cmd: 'find_one_category' }`   | `id`                       | La categoría                                |
| `{ cmd: 'update_category' }`     | `id` + campos a modificar  | La categoría actualizada                    |
| `{ cmd: 'remove_category' }`     | `id`                       | La categoría eliminada                      |

## Reglas de validación

- `name`: texto de 3 a 50 caracteres**. 
- `description`: opcional, máximo 200 caracteres.
- Paginación: el mismo pagination que en products-ms y orders-ms
- `update_category`: todos los campos opcionales, pero `id` es obligatorio (entero positivo).

## Errores (con `RpcException`)

| Situación                                           | `status` |
| ----------------------------------------------------| -------- |
| Crear o actualizar con un `name` que ya existe      | `409`    | (CONFLICT)
| Buscar, actualizar o eliminar un `id` que no existe | `400`    | (BAD_REQUEST)

## Conexión con el gateway

Registra un cliente TCP para Categorías (con su propio host y puerto en `.env`) y crea el controlador REST:

| Método   | Ruta                         | Descripción           |
| -------- | ---------------------------- | --------------------- |
| `POST`   | `/categories`                | Crear                 |
| `GET`    | `/categories?page=1&limit=5` | Listar con paginación |
| `GET`    | `/categories/:id`            | Obtener una           |
| `PATCH`  | `/categories/:id`            | Actualizar            |
| `DELETE` | `/categories/:id`            | Eliminar              |

---

## PRUEBAS Y ENTREGA (10 pts)

1. Levanta products-ms, categorias-ms, reseñas-ms y client-gateway al mismo tiempo.
2. Prueba todos los endpoints con Postman. Guarda la colección en la raíz como evidencia, incluyendo al menos un caso de error `404`, uno `409` y uno `400`.
3. Sigue los pasos de `README.md` para la entrega