# Tareas — Manager Library (Laravel)

Cada tarea termina cuando su **evidencia** existe y se puede ejecutar (artículo XII). Estado inicial de todas: **pendiente**.
Mismas tareas y numeración que [`spec-python/tasks.md`](../spec-python/tasks.md); la evidencia se traduce a Laravel ([ADR-005](adr/adr-005-traduccion-a-react-y-laravel.md)).
Cada tarea se entrega en commits propios cuyo mensaje empieza por su número (`T-01: …`).

| # | Tarea | Depende de | Evidencia |
|---|---|---|---|
| T-01 | Esqueleto del proyecto Laravel y `GET /api/v1/health` | — | `GET /api/v1/health` responde `{"status":"ok"}`; el contenedor arranca con un comando |
| T-02 | Migración inicial: seis tablas, FK, índices únicos y las cinco categorías sembradas | T-01 | Las consultas de `data-model.md` §10 devuelven lo documentado; ninguna columna con `DEFAULT` |
| T-03 | Dominio: `Isbn`, `Quantity`, `DateRange` (objetos de valor) | T-01 | Pruebas unitarias Pest: ISBN de 13 dígitos, cantidad > 0, fin ≥ inicio |
| T-04 | Dominio: `Book` (`withdraw`, `restock`, baja lógica) | T-03 | Pruebas: nunca < 0, nunca > total |
| T-05 | Dominio: `Loan` (`addItem`, `returnItem`, estado calculado, 14 días, máx. 3) | T-04 | Pruebas con un `Clock` falso: `open`, `overdue`, `closed` |
| T-06 | Dominio: `Member` y `User` (normalización de nombre, rol válido) | T-03 | `"Ana "` queda `ana`; rol `admin` por API rechazado |
| T-07 | Puertos y casos de uso de **autenticación** (`LoginService`, `CreateLibrarianService`) con fakes | T-06 | HU-01 y HU-02 pasan contra fakes |
| T-08 | Puertos y casos de uso de **libros** (crear, buscar, editar, baja, portada) con fakes | T-04 | HU-03 y HU-04 pasan contra fakes |
| T-09 | Caso de uso **prestar** (`LendBooksService`) con fakes | T-05, T-08 | HU-06 completa: límites, vencidos, sin ejemplares |
| T-10 | Caso de uso **devolver** y **vencidos** | T-09 | HU-07 y HU-09; devolver dos veces → `ITEM_ALREADY_RETURNED` |
| T-11 | Caso de uso **reporte de más prestados** y su puerto de lectura | T-09 | HU-10; el título viejo se conserva tras renombrar (ADR-004) |
| T-12 | Adaptador de persistencia: repositorios de libro, socio, usuario | T-02, T-08 | Pruebas de integración contra PostgreSQL real |
| T-13 | Adaptador de persistencia: repositorio de préstamos + testigo de versión | T-12, T-09 | Dos préstamos del último ejemplar: uno 201, otro 409 (RNF-2) |
| T-14 | Adaptador de seguridad: hash Argon2id y token JWT (`firebase/php-jwt`) | T-07 | La clave nunca aparece en logs ni respuestas (RNF-3) |
| T-15 | Adaptador de portadas (disco) | T-08 | Subir PNG de 2 MB ok; 3 MB → 422; sin portada → `cover_key` nulo |
| T-16 | Adaptador HTTP: rutas de [`api-contract.md`](../spec-python/api-contract.md), cuerpo de error único, autorización por rol | T-10, T-11, T-14, T-15 | Prueba de contrato recorre todas las rutas y todos los códigos de §2 |
| T-17 | Composición (`PortBindingsServiceProvider`) y arranque (admin inicial desde entorno) | T-16 | El sistema arranca con la base vacía y se puede iniciar sesión como admin |
| T-18 | Bajar a la base las invariantes **solo dominio**: `ck_book_total_positive`, `ck_user_role` | T-12 | Un `INSERT` por `psql` que las viole falla; `data-model.md` §4 se actualiza |
| T-19 | Pruebas de arquitectura (artículos I, II y III) | T-17 | Las pruebas `arch()` de Pest pasan y fallan si se introduce una importación prohibida |
| T-20 | `infra`: `docker compose up` levanta motor vacío, API y frontend | T-17 | Un comando; la base arranca sin tablas hasta correr las migraciones |
| T-21 | `tool`: sembrador de demostración **por la API** (≥ 40 libros, 10 socios, 15 préstamos, 3 vencidos) | T-17 | Tras correrlo, `/loans/overdue` devuelve 3 líneas y el reporte tiene filas |
| T-22 | `app`: login, catálogo, socios, prestar, devolver, vencidos, reporte (React) | T-16 | Recorrido manual de punta a punta en la feria |
| T-23 | `page`: sitio público estático de presentación | — | Se abre sin la API; correcto en móvil y escritorio |

## Definición de terminado de la prueba

Quien llegue a T-17 con evidencia tiene el **núcleo** hecho. T-18 a T-23 son lo que separa un trabajo bueno de uno sobresaliente. Se evalúa
el desempeño con SDD, no la cantidad de tareas: una tarea con evidencia vale más que tres a medias.

## Avance

| # | Estado | Commit / evidencia |
|---|---|---|
| T-01 | terminada | `api` cbd280f: `GET /api/v1/health` → `{"status":"ok"}`; `vendor/bin/pest` (3 pruebas en verde); única ruta registrada |
| T-02 | terminada | `api` 05c2b9b: consultas de `data-model.md` §10 coinciden; 0 columnas con `DEFAULT`; 6 tablas en `library`; `ck_book_total_positive` y `ck_user_role` siguen pendientes (T-18) |
| T-03 | terminada | `api` 5b4a134: `tests/Domain/ValueObjectsTest.php` (ISBN de 13 dígitos, cantidad > 0, fin ≥ inicio) |
| T-04 | terminada | `api` 8c93840: `tests/Domain/BookTest.php` (nunca < 0, nunca > total, baja lógica) |
| T-05 | terminada | `api` 1a4768f: `tests/Domain/LoanTest.php` con `Clock` falso: `open` → `overdue` → `closed` |
| T-06 | terminada | `api` e2e95d4: `tests/Domain/MemberAndUserTest.php` (`"Ana "` → `ana`; rol `admin` por API rechazado) |
| T-07 | terminada | `api` 02ab652: `tests/Application/AuthenticationTest.php` (HU-01 y HU-02 contra fakes; rol `admin` → 422) |
| T-08 | terminada | `api` e0c49f1: `tests/Application/BooksTest.php` (HU-03 y HU-04 contra fakes) |
| HU-05 | terminada (sin tarea propia en la tabla) | `api` 7ead9a0: `tests/Application/MembersTest.php` |
| T-09 | terminada | `api` 32395ac: `tests/Application/LendBooksTest.php` (límites, vencidos, sin ejemplares, reintento ADR-002) |
| T-10 | terminada | `api` f97ecfb: `tests/Application/ReturnAndOverdueTest.php` (devolver dos veces → `ITEM_ALREADY_RETURNED`) |
| T-11 | terminada | `api` 92d7ee6: `tests/Application/MostBorrowedReportTest.php` (título viejo tras renombrar, ADR-004) |
| HU-08 | terminada (sin tarea propia en la tabla) | `api` fc46835: `tests/Application/LoansQueryTest.php` |
| T-12 | terminada (rojo 0b3501d → verde 37254df) | `tests/Integration/PersistenceRepositoriesTest.php` contra PostgreSQL real |
| T-13 | terminada (rojo f190454 → verde dba9fcf) | `tests/Integration/LoanPersistenceTest.php`: dos préstamos del último ejemplar → uno registrado, otro `BOOK_NOT_AVAILABLE` (RNF-2) |
| T-14 | terminada (rojo 6d5f776 → verde c6aa799) | `tests/Integration/SecurityAdaptersTest.php`; la clave no aparece en logs ni respuestas (RNF-3) |
| T-15 | terminada (rojo d52299c → verde 26483f4) | `tests/Integration/CoverStorageTest.php`: PNG de 2 MB ok, 3 MB → 422, sin portada → `null` |
| T-16 | terminada (rojo c73656c → verde 60cf9de) | `tests/Contract/ApiContractTest.php` recorre las 21 rutas y los códigos de §2; la lista de rutas es exactamente la del contrato. Incluye `PortBindingsServiceProvider` porque la prueba necesita la API levantada |
| T-17 | terminada (rojo 4dcd2da → verde a18eef4) | Base vacía → `docker/start.sh` → login como admin (`role: admin`); 133 pruebas en verde |
| T-19 | terminada | `api` 5a11ea3: pruebas `arch()` de Pest (artículos I, II, III y IV); se comprobó que fallan al meter `Illuminate` en el dominio o un adaptador fuera de la composición |
| T-20 | terminada | `infra` b0bdae5 + e005d94 + b2ae8a5 + fd70400, `app` 4dba4eb: `docker compose up -d` levanta el motor vacío, la API (migra y crea el admin) y el frontend en 5173 |
| T-21 | terminada (rojo e8a2799 → verde 222e4ff en `api`; `tool` 5fe61fa) | Sembrador Node.js solo por la API: 40 libros, 10 socios, 15 préstamos; `/loans/overdue` devuelve 3 líneas y el reporte 12 filas |
| T-22 | terminada | `app` 2610be8: recorrido manual ingresar → crear libro → crear socio → prestar → devolver → reporte |
| T-23 | terminada | `page` a500cd7: HTML y CSS estáticos, sin API ni código; revisada a 375 px y en escritorio |
| T-18 | no se hace | Su evidencia exige reclasificar `ck_book_total_positive` y `ck_user_role` en `data-model.md` §4, que está en `spec-python/` y no se modifica (ADR-005). Crear los `CHECK` sin actualizar ese documento violaría el artículo X. Queda pendiente para quien pueda cambiar el documento del negocio |
| RNF-1 | cumplido | `tool` `node seed.mjs load`: 5 000 libros creados por la API; 20 búsquedas, mediana 19 ms y máxima 25 ms (< 300 ms) |
| RNF-2 | cumplido | `tool` `node seed.mjs race` con la API en 4 workers: dos préstamos simultáneos del último ejemplar → uno 201 y otro 409 `BOOK_NOT_AVAILABLE`, 0 disponibles |
| RNF-3 | cumplido | La clave del admin no aparece en los logs de la API (revisado en T-17 y T-20); ninguna respuesta incluye `password` ni `password_hash` (prueba de contrato) |
| RNF-4 | cumplido | `tests/Contract/ApiContractTest.php`: todo error, incluidos 404 y 500, usa `{code, message, details}` |
| RNF-5 | cumplido | `docker compose up -d` en `infra`; la base arranca sin tablas hasta que la API migra |
