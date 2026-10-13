# Metodología — SDD, DDD y TDD en Manager Library (Laravel)

Cómo se aplican los tres enfoques en la traducción a React + Laravel ([ADR-005](adr/adr-005-traduccion-a-react-y-laravel.md)).
Cada afirmación apunta a algo que se puede revisar: un documento, una carpeta, una prueba o un commit.

## 1. SDD — Desarrollo guiado por especificación

| Práctica | Dónde se ve |
|---|---|
| La especificación va **antes** que el código | Los commits de `docs` (ADR-005, constitución, plan, arquitectura, tareas) son anteriores al primer commit de código en `api` |
| El negocio no se reescribe: una sola fuente de verdad | `spec.md`, `data-model.md`, `api-contract.md` y ADR-001 a ADR-004 se **referencian** desde `spec-python/`; no se copian ni se modifican |
| Trazabilidad: cada cambio dice qué parte del spec cumple | Cada commit empieza por su tarea (`T-xx`) o su historia (`HU-xx`) y cita los criterios (`CA-xx.x`), reglas (`RN-x`) y ADR que cubre |
| Una tarea termina cuando su **evidencia** existe (artículo XII) | Tabla **Avance** de [`tasks.md`](tasks.md): cada tarea terminada indica su commit y la prueba o consulta que la demuestra |
| El contrato no se amplía | Una prueba en `api` falla si se registra una ruta que no está en `api-contract.md` |

## 2. DDD — Diseño guiado por el dominio

| Concepto | Spec | Código en `api` |
|---|---|---|
| **Lenguaje ubicuo** | Glosario de `data-model.md` §1 | Las clases y métodos usan esos términos: `Book`, `Loan`, `LoanItem`, `Member`, `User`, `withdraw`, `restock`, `retire` |
| **Agregados** y sus invariantes | `architecture.md` §3 | `app/Domain/Book.php` (0 ≤ disponibles ≤ total), `app/Domain/Loan.php` (1 a 3 líneas, sin repetidos, 14 días), `Member`, `User` |
| **Objetos de valor** (sin identidad) | `plan.md` D-07 | `app/Domain/ValueObjects/`: `Isbn`, `Quantity`, `DocumentNumber`, `DateRange` (`final readonly`) |
| **Entidad dentro del agregado** | `Loan` contiene `LoanItem` | `app/Domain/LoanItem.php`; solo cambia `returned_at` |
| **Dominio aislado** | Artículo I | `app/Domain` no importa Laravel ni otras capas |
| **Puertos definidos por quien los consume** | Artículo II, DIP | `app/Application/Ports/Outbound/` (repositorios, `Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage`, lectores de reporte y vencidos) |
| **Servicios de aplicación**: un caso de uso cada uno | Artículo IV (SRP) | `app/Application/Services/` (`LendBooksService`, `ReturnItemService`, …) |
| **Errores del dominio** con el código del contrato | `architecture.md` §5 | `app/Domain/Errors/DomainError.php`; solo el adaptador HTTP los traduce a estado HTTP |

## 3. TDD — Desarrollo guiado por pruebas

| Práctica | Dónde se ve |
|---|---|
| Cada criterio de aceptación tiene al menos una prueba que lo nombra | Los nombres de las pruebas citan su `CA-xx.x`, `RN-x` o ADR (`tests/Domain`, `tests/Application`) |
| Pirámide de pruebas de `plan.md` §3 | Dominio sin dependencias; aplicación con **fakes** de los puertos (`tests/Fakes`); integración contra PostgreSQL real |
| Reloj falso para el tiempo | `tests/Fakes/FixedClock.php`: vencimientos y retrasos sin esperar 14 días |

**Ciclo rojo → verde en el historial.** De T-01 a T-11 (más HU-05 y HU-08) cada commit trae juntas la prueba y el código que la
satisface: la prueba es la evidencia de la tarea, pero el historial no muestra el orden en que se escribieron. **Desde T-12** cada tarea
se entrega en dos commits visibles:

1. `T-xx (red): …` — solo la prueba nueva, que falla.
2. `T-xx (green): …` — el código mínimo que la hace pasar, con toda la suite en verde.
