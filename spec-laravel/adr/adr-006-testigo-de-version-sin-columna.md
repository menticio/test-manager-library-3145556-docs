# ADR-006 · El testigo de versión del ADR-002 es `available_copies`, sin columna nueva

- **Estado:** aceptada. **Se registra después de implementada** (T-13, commits `f190454` → `dba9fcf` de `api`); este documento
  la deja escrita para que la decisión no viva solo en el código.
- **Contexto:** [ADR-002](../../spec-python/adr/adr-002-concurrencia-optimista.md) pide control optimista con «un testigo de versión»
  por libro. [`data-model.md`](../../spec-python/data-model.md) §3 lista las columnas de `book` y **no** incluye `version`; el mismo
  documento dice que si algo lo contradice, gana él, y el artículo X dice que gana el motor.
- **Decisión:** el testigo es el propio `available_copies` tal como se leyó. Al prestar, el adaptador escribe
  `UPDATE book SET available_copies = :nuevo WHERE id = :id AND available_copies = :leido`; si no afecta una fila, lanza
  `ConcurrencyConflict`, la transacción completa se revierte y el servicio reintenta **una vez** (ADR-002); si sigue fallando
  responde **409 `BOOK_NOT_AVAILABLE`**. Al devolver se usa el mismo testigo, y la línea solo se escribe mientras `returned_at` es nula.
  La última barrera sigue siendo `ck_book_available_range`. No hay `SELECT … FOR UPDATE`.
- **Alternativas descartadas:** agregar una columna `version` (contradice `data-model.md` §3 y obligaría a cambiar un documento del
  negocio); bloqueo pesimista (descartado por ADR-002).
- **Consecuencias:** cualquier cambio concurrente del número de ejemplares de un libro (otro préstamo, una devolución, un cambio de
  total) invalida la lectura y provoca el reintento: es más conservador que una versión dedicada, pero nunca presta lo que no hay.
- **Cómo se verifica:** `tests/Integration/LoanPersistenceTest.php` en `api`: con una lectura vieja → `ConcurrencyConflict` y no se
  guarda nada; dos préstamos del último ejemplar → uno registrado y el otro `BOOK_NOT_AVAILABLE` (RNF-2).
