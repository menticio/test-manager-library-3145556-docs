# Plan técnico — Manager Library (Laravel)

Cómo se construye lo que dicen [`spec.md`](../spec-python/spec.md) y [`constitution.md`](constitution.md) con **React + PHP (Laravel)**,
según [ADR-005](adr/adr-005-traduccion-a-react-y-laravel.md). El esquema **no** se describe aquí: vive en
[`data-model.md`](../spec-python/data-model.md). El contrato HTTP vive en [`api-contract.md`](../spec-python/api-contract.md).
La forma del sistema, en [`architecture.md`](architecture.md).

## 1. Decisiones técnicas

| # | Decisión | Elección (Laravel) | Por qué |
|---|---|---|---|
| D-01 | Lenguaje y runtime | PHP 8.3 | Stack del reto |
| D-02 | Framework HTTP | Laravel, rutas en `routes/api.php` con prefijo `/api/v1` | Contrato en [`api-contract.md`](../spec-python/api-contract.md) |
| D-03 | Persistencia | Eloquent / Query Builder **solo en adaptadores** + migraciones de Laravel sobre PostgreSQL 16, esquema `library` | El esquema es de las migraciones (ADR-001) |
| D-04 | Concurrencia | Testigo de versión por libro + `CHECK` de rango ([ADR-002](../spec-python/adr/adr-002-concurrencia-optimista.md)) | Último ejemplar (RNF-2) |
| D-05 | Fechas | Instantes UTC (`timestamptz`, `APP_TIMEZONE=UTC`); el reloj es un puerto (`Clock`) | Probar vencimientos sin esperar 14 días |
| D-06 | Reporte | Una consulta agregada por un puerto de lectura ([ADR-004](../spec-python/adr/adr-004-reporte-agregado-y-congelado.md)) | Estable y barato |
| D-07 | Objetos de valor | `Isbn`, `Quantity`, `DocumentNumber`, `DateRange`: clases `final readonly` que viven dentro de la fila de su dueño, sin tabla | No tienen identidad |
| D-08 | Portadas | Binario en almacenamiento externo (disco local en `COVERS_DIR`); la base guarda solo `cover_key` | Clave opaca, nunca ruta |
| D-09 | Contraseñas | Argon2id (`password_hash(..., PASSWORD_ARGON2ID)`) detrás del puerto `PasswordHasher`; el dominio nunca ve la clave en claro | Artículo IX |
| D-10 | Arranque | La aplicación crea el admin inicial desde `ADMIN_USERNAME` / `ADMIN_PASSWORD` (comando de arranque, no semilla SQL) | La base no puede producir el hash |
| D-11 | Autenticación | JWT HS256 (`firebase/php-jwt`) detrás del puerto `TokenIssuer`, 8 horas, clave en `JWT_SECRET` | Simple y sin estado |
| D-12 | Pruebas | Pest, con fakes para los puertos y una base real solo en pruebas de integración | Pirámide de pruebas |
| D-13 | Frontend | React + Vite en el repositorio `app`; consume solo `/api/v1` | Stack del reto |

**Convenciones de Laravel que ceden ante el spec** (ADR-005): sin `timestamps`, sin `->default()` en migraciones, tablas en singular
dentro del esquema `library`, UUID generados por el dominio, y el cuerpo de error único para **todos** los errores (incluida la validación,
404 de ruta y 401/403).

## 2. Configuración por entorno

| Variable | Obligatoria | Ejemplo |
|---|---|---|
| `DB_CONNECTION` / `DB_HOST` / `DB_PORT` / `DB_DATABASE` / `DB_USERNAME` / `DB_PASSWORD` | sí | `pgsql` / `db` / `5432` / `library` / `library` / *(del `.env`)* |
| `DB_SEARCH_PATH` | sí | `library` |
| `JWT_SECRET` | sí | cadena larga aleatoria |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | sí | definidos en `.env`, nunca en el repo |
| `COVERS_DIR` | no | `storage/app/covers` |
| `CORS_ORIGIN` | sí | `http://localhost:5173` |

## 3. Estrategia de pruebas

| Nivel | Qué prueba | Contra qué |
|---|---|---|
| Dominio | Invariantes: máximo 3, vencimiento a 14 días, disponibles en rango | Nada: objetos puros |
| Aplicación | Casos de uso: prestar, devolver, reporte | Fakes de los puertos |
| Integración | Repositorios y consulta del reporte | PostgreSQL real |
| Contrato | Cuerpo de error único y códigos de estado | La API levantada |
| Arquitectura | Artículos I, II y III | Pruebas `arch()` de Pest en `tests/Architecture/` |
| Concurrencia | Dos préstamos del último ejemplar | PostgreSQL real |

## 4. Orden de construcción

Esqueleto y salud → migración inicial con semilla → dominio → casos de uso con fakes → adaptadores → API → pruebas de contrato y concurrencia.
El detalle, con evidencia, está en [`tasks.md`](tasks.md).
