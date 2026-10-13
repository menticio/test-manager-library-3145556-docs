# ADR-007 · Cómo se leen en Laravel los puntos que el contrato deja abiertos

- **Estado:** aceptada. **Se registra después de implementada** (T-02, T-08 y T-16 de `api`); este documento deja escritas
  decisiones que hasta ahora solo estaban en el código y en los mensajes de commit.
- **Contexto:** [`api-contract.md`](../../spec-python/api-contract.md) y [`data-model.md`](../../spec-python/data-model.md) no se
  modifican ([ADR-005](adr-005-traduccion-a-react-y-laravel.md)). Algunos casos no están dichos de forma explícita y había que
  elegir **sin agregar endpoints, campos ni códigos de negocio**.
- **Decisiones:**

| # | Caso | Decisión | Por qué |
|---|---|---|---|
| 1 | 401 sin token, token inválido o vencido | Código `INVALID_CREDENTIALS` con otro mensaje | Es el único código 401 de §2; la tabla de estados agrupa los cuatro casos en el 401 |
| 2 | Ruta inexistente o método no permitido | 404 `NOT_FOUND` con el cuerpo único | La tabla de estados no tiene 405; «no existe» es el 404 |
| 3 | Error inesperado del servidor | 500 con `code = INTERNAL_ERROR` y el cuerpo único, sin trazas | RNF-4: todo error usa el mismo cuerpo; no es un error de negocio, por eso no está en §2 |
| 4 | `size` mayor que 50, `page` menor que 1 | 422 `VALIDATION_ERROR` | «`size` máximo 50» es una regla de entrada; no se recorta en silencio |
| 5 | `GET /categories` | `{ "items": [ { "id", "name" } ] }` | Misma forma que `GET /loans/overdue`, el otro listado sin paginación |
| 6 | Cantidad de líneas de un préstamo | Se valida 1 a 3 (`VALIDATION_ERROR`) antes de las reglas del socio | CA-06.1 describe el préstamo; CA-06.4 solo aplica a un préstamo bien formado |
| 7 | `GET /books/{id}/cover` | Requiere token como cualquier otra ruta | §3.4 lo nombra para pintar la portada; CA-01.3 exige token en todo salvo login y health |
| 8 | Tabla de control `migrations` de Laravel | Vive en el esquema `public` | `data-model.md` §2: el esquema `library` tiene seis tablas; Alembic hace lo mismo con su tabla de versión |
| 9 | Validación de portada (tipo por contenido, 2 MB) | En la aplicación (`CoverImage`), no en el adaptador de disco | Artículo IV (OCP): cambiar disco por S3 no toca las reglas |

- **Alternativas descartadas:** crear códigos nuevos como `UNAUTHORIZED` o `METHOD_NOT_ALLOWED` (amplía el contrato); recortar `size`
  a 50 (oculta un error de entrada); devolver las categorías como arreglo suelto (forma distinta del otro listado).
- **Cómo se verifica:** `tests/Contract/ApiContractTest.php` y `tests/Feature/HealthTest.php` en `api` (la lista de rutas es exactamente la
  del contrato).
