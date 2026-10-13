# ADR-005 · El reto se traduce a React y PHP (Laravel) sin tocar el negocio

- **Estado:** aceptada
- **Contexto:** el spec se entrega en dos versiones (`spec-python/` y `spec-.net/`) que describen el **mismo producto** y solo cambian
  las decisiones de tecnología. El reto se desarrolla con **React** (frontend) y **PHP con Laravel** (backend), un stack que ninguna de
  las dos versiones cubre. Hace falta decidir qué se traduce y qué se respeta tal cual.
- **Decisión:**
  1. **Se respeta sin cambios** todo lo que define el negocio: [`spec.md`](../../spec-python/spec.md),
     [`data-model.md`](../../spec-python/data-model.md), [`api-contract.md`](../../spec-python/api-contract.md) y los
     ADR-001 a ADR-004. No se copian: se referencian, para que exista **una sola fuente de verdad**.
  2. **Se traduce** solo lo tecnológico, en `spec-laravel/`: `constitution.md`, `plan.md`, `architecture.md` y `tasks.md`.
     Cada artículo, decisión y tarea conserva su número y su intención; cambia la herramienta con la que se cumple y se verifica.
  3. Equivalencias:

| Concepto del spec | Python | .NET | **Laravel / React (reto)** |
|---|---|---|---|
| Lenguaje | Python 3.12 | C# | **PHP 8.3** |
| HTTP | FastAPI | ASP.NET | **Laravel (rutas `routes/api.php`, prefijo `/api/v1`)** |
| Persistencia | SQLAlchemy 2 | EF Core | **Eloquent / Query Builder, solo en adaptadores** |
| Migraciones (ADR-001) | Alembic | Migraciones EF Core | **Migraciones de Laravel** |
| Dominio puro | `dataclass(frozen=True)` | `record` | **Clases PHP `final` con propiedades `readonly`** |
| Composición única | `port_bindings.py` | `PortBindings.cs` | **`app/Bootstrap/PortBindingsServiceProvider.php`** |
| Pruebas de arquitectura | `import-linter` | NetArchTest | **Pruebas de arquitectura de Pest (`arch()`)** |
| Pruebas | pytest | xUnit | **Pest / PHPUnit** |
| Hash de claves | Argon2id | Argon2id | **Argon2id con `password_hash(PASSWORD_ARGON2ID)` detrás del puerto** |
| Token | JWT HS256 | JWT HS256 | **JWT HS256 con `firebase/php-jwt` detrás del puerto** |
| Frontend | React | React | **React + Vite** |

- **Alternativas descartadas:** reescribir `spec.md`, `data-model.md` o `api-contract.md` para Laravel (segunda fuente de verdad que
  puede divergir); usar las convenciones por defecto de Laravel que contradicen el modelo de datos (ver consecuencias).
- **Consecuencias:** las convenciones de Laravel **ceden ante el spec** cuando chocan:
  - Sin `created_at`/`updated_at` (`data-model.md` §3 y §8): los modelos declaran `$timestamps = false` y las migraciones no los crean.
  - Ninguna columna con `DEFAULT` (`data-model.md` §3): las migraciones no usan `->default()` y los UUID los genera el dominio.
  - Tablas en **singular** y en el esquema `library` (`data-model.md` §0), no en el plural que asume Eloquent.
  - El cuerpo de error único (`api-contract.md` §1) reemplaza las respuestas de error por defecto de Laravel, incluida la validación.
  - No se agrega ningún endpoint ni campo que el contrato no tenga (ni rutas de ejemplo de Laravel).
- **Cómo se verifica:** las consultas de `data-model.md` §10 contra la base migrada por Laravel devuelven lo documentado; la prueba de
  contrato recorre solo las rutas de `api-contract.md`; las pruebas `arch()` de Pest cumplen los artículos I a III de
  [`constitution.md`](../constitution.md).
