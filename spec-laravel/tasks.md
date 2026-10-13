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
| T-20 | en curso (parte 1: motor vacío) | `infra`: `docker compose up -d` levanta PostgreSQL 16 vacío en UTC |
