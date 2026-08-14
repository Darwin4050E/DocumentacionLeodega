# Plan de corrección de inconsistencias — Leodega (backend heredado)

> **Estado (2026-08-14): Fases 1 y 2 implementadas en el working tree de `recursos/leodega_codigo/` (repo `LeodegaPruebas`, rama `main`), pendientes de commit y push por el usuario.** La Fase 3 / HUA-03 (§7) sigue sin implementar, tal como se decidió: es trabajo de Sprint 1, no una corrección. Nada de esto se ha commiteado ni pusheado todavía — el usuario prepara y sube los commits.
>
> **Verificación realizada antes de dar por buena la implementación:** suite completa de PHPUnit (109/109, incluyendo 11 tests nuevos), PHPStan (0 errores, 2 falsos positivos de propiedades dinámicas de Eloquent añadidos al baseline existente, mismo criterio que ya usaba el proyecto), PHPMD (0 violaciones), Laravel Pint (124 archivos, estilo conforme), suite de Vitest del frontend (33/33, sin cambios ahí). El fix de `Reports.status` (Fase 1.1) se probó además contra un PostgreSQL 16 real (contenedor Docker desechable), no solo contra SQLite.
>
> **Corrección a este mismo documento:** la Fase 1.1 originalmente decía que el bug de `Reports.status` "funciona por accidente en SQLite" y solo falla en PostgreSQL. Es **incorrecto** — se verificó empíricamente que SQLite sí aplica el `CHECK` constraint del enum igual que PostgreSQL; el `INSERT`/`UPDATE` con `status = 'canceled'` fallaba en ambos motores. También se descubrió, verificándolo contra PostgreSQL real, que `$table->enum(...)->change()` de Laravel 12 genera SQL sintácticamente inválido para PostgreSQL (aunque funciona bien en SQLite) — la migración final usa SQL específico por motor en vez de depender de ese método.
>
> **Origen:** los 5 hallazgos aquí descritos surgieron durante la construcción de los Diagramas de Estado de la Sección C (ver `secciones/seccionC.tex`, figuras `estado_02` a `estado_05`) y de la tabla de Malos Olores de la Sección B (`secciones/seccionB.tex`, §3.2.3). Los 5 son defectos en código **heredado** del equipo anterior (ver justificación en la sección "Atribución" más abajo) — ninguno proviene de los módulos nuevos que este equipo debe construir (organizaciones, Leodeguita, integración de inventario, permiso de bomberos, panel admin, moderación de cuentas con auditoría), porque esos módulos todavía no tienen código.

---

## 1. Atribución: ¿de quién es cada inconsistencia?

El ERS del primer parcial (`recursos/informacion_general/proyecto1P.tex`, línea 240, sección "Existing capabilities") declara explícitamente qué entregó el equipo anterior como funcionalidad **completa**:

> *"User authentication and registration, role-based access control (…), publication and moderation of storage spaces, search and detail views, reservations (…), ratings, **reports**, in-app notifications, conversations and messages, **a payments registry**, and role-based dashboards."*

| # | Hallazgo | Módulo (declarado "existente" en el ERS) |
|---|---|---|
| 1 | `Reports.status` acepta `"canceled"`, valor fuera de su propio `enum` | reports |
| 2 | `in_review` es un estado muerto (declarado, nunca alcanzado) | reports |
| 3 | `StoreRooms.publication_status` y `StoreModeration.status` desincronizados | publication and moderation of storage spaces |
| 4 | `Payments` desacoplado de `ReservationService::confirm()` | a payments registry |
| 5 | `AuthController::login()` no valida `User.state` | authentication and role-based access control |

Los 5 caen dentro de módulos que el equipo anterior declaró terminados. La única matización es el **#5**: el bug es heredado, pero **cerrarlo del todo** (bloqueo/reactivación con auditoría) es responsabilidad de este equipo, porque corresponde al requerimiento nuevo **HUA-03** del ERS actual.

---

## 2. Resumen y fases recomendadas

| Fase | Issue | Esfuerzo | Estado |
|---|---|---|---|
| **1 — Quick wins** | #1 / #2 — Enum de `Reports.status` | S (1 migración) | ✅ Implementado (ver §3) |
| **1 — Quick wins** | #5-A — Guard mínimo de `state` en el login | S (una condición, alcance acotado — ver §4) | ✅ Implementado (ver §4) |
| **2 — Refactor con servicio** | #3 — `StoreRooms` ↔ `StoreModeration` | M (nuevo servicio + endpoint dedicado) | ✅ Implementado, variante no invasiva (ver §5) |
| **2 — Refactor con servicio** | #4 — `Payments` ↔ `Reservations` | M (nuevo servicio + transacción + policy) | ✅ Implementado (ver §6) |
| **(fuera de este plan)** | #5-B — Bloqueo/reactivación con auditoría | — | Sin implementar, a propósito: **HUA-03, Sprint 1** del Sprint Backlog (Sección A). Ver nota en §7. |

Orden sugerido: Fase 1 primero (bajo riesgo, cierra el gap de seguridad más urgente), luego Fase 2 (introduce los servicios que faltan, siguiendo el mismo patrón que ya usa `ReservationService`). La implementación completa de HUA-03 se planifica y ejecuta como parte del Sprint 1, no desde este documento.

> **Nota de proceso:** este plan originalmente incluía una "Fase 3" con pasos de implementación para HUA-03. Se eliminó esa sección al notar que HUA-03 **ya está en el Sprint Backlog** (Sprint 1, 5 puntos, estado "Pendiente" — `secciones/seccionA.tex`, tabla `tab:sprint1`). Mantener dos planes de implementación distintos para la misma historia —uno aquí y otro en el Sprint Backlog— es exactamente el mismo problema de "dos fuentes de verdad desincronizadas" que se documenta como hallazgo #3 más abajo. La planificación detallada de HUA-03 debe vivir únicamente en el Sprint Backlog / la ceremonia de sprint planning correspondiente.

Contexto adicional verificado en `routes/api.php` y en los tests existentes (no estaba en el análisis original y afina el plan):

- `StoreModerationController` (`/store_moderation`, protegida `role:admin`) **no es el camino real** por el que hoy se aprueban bodegas: el test `StoreRoomModerationTest::admin_can_approve_a_store_room` aprueba vía `PUT /storeRooms/{id}` (CRUD genérico de `StoreRoomsController`). La tabla `store_moderation` está prácticamente huérfana en el flujo real.
- `POST` / `PUT /payments` solo exige `auth.api:sanctum` (cualquier usuario autenticado, no necesariamente el tenant dueño de la reserva) — es una brecha de autorización adicional, ligada al mismo punto de código que hay que tocar para el fix #4.

---

## 3. Fase 1.1 — `Reports.status`: enum inconsistente + estado muerto (#1, #2) ✅

**Decisión de diseño:** el código y `UpdateReportStatusRequest` ya usan `resolved` / `canceled` en toda la lógica real; nadie usa `in_review`. Es más barato y menos riesgoso alinear el **esquema** a lo que el **código** ya hace, en vez de diseñar un flujo de "en revisión" que el ERS no pide.

**Implementación real** (`backend/database/migrations/2026_08_14_000001_fix_reports_status_enum.php`): se verificó contra un PostgreSQL 16 real (contenedor Docker) que `$table->enum(...)->change()` de Laravel 12 genera SQL inválido para ese motor (`ALTER COLUMN ... TYPE ... check(...)` no es sintaxis válida en PostgreSQL), aunque sí funciona nativamente en SQLite. La migración final detecta el driver (`Schema::getConnection()->getDriverName()`) y usa `->change()` en SQLite, SQL crudo (`DROP CONSTRAINT` / `ADD CONSTRAINT`) en cualquier otro driver — validado exitosamente contra ambos motores reales, no solo deducido.

**Corrección importante:** se verificó empíricamente que, contrario a lo que este documento afirmaba originalmente, **SQLite sí aplica el `CHECK` constraint del enum** (falla igual que PostgreSQL al insertar `'canceled'`). El bug no era "silencioso en dev, explota en producción" — era un fallo garantizado en cualquier entorno; simplemente ningún test lo ejercitaba todavía.

Test de regresión agregado: `tests/Feature/ReportsTest.php::test_admin_can_cancel_report_with_status_outside_original_enum`.

No requirió tocar el frontend (ya enviaba `resolved` / `canceled`).

---

## 4. Fase 1.2 — `User.state` sin guard en el login (#5-A, versión mínima y acotada) ✅

**Implementación real** (`backend/app/Services/AuthService.php`, `AuthController.php`, `AccountBlockedException.php`): tal cual se planeó, sin desviaciones. Tests agregados: `blocked_user_cannot_login` y `pending_user_can_still_login` (este último deja constancia explícita, vía test que pasa, de que `pending` sigue sin bloquear el login hoy).

**Alcance deliberadamente reducido:** como el diseño completo de bloqueo/reactivación es HUA-03 (Sprint 1, todavía no iniciado), este fix **no debe decidir por adelantado** cómo se van a usar `pending` ni cómo se van a poblar `blocked` — eso es justamente lo que HUA-03 tiene que definir. Por eso el guard se limita a lo mínimo seguro hoy: impedir el login de una cuenta ya marcada `blocked`, sin tocar el significado de `pending`.

**Pasos:**

1. En `AuthController::login()`, después de `Auth::attempt()` y antes de emitir el token:
   ```php
   if ($user->state === 'blocked') {
       return response()->json(['message' => 'Cuenta bloqueada'], 403);
   }
   ```
   Nota: se compara explícitamente contra `'blocked'`, **no** contra `!== 'active'` — así el guard no bloquea accidentalmente a los usuarios `pending` mientras ese estado no tenga un significado definido.
2. Extraerlo a `AuthService::ensureUserIsNotBlocked(User $user): void`, lanzando una excepción de dominio nueva (`AccountBlockedException`), manteniendo el mismo estilo que `ReservationConflictException` / `DuplicateRatingException`.
3. **Lo que este fix explícitamente NO decide** (queda para el sprint planning de HUA-03, no para este documento): qué significa `pending`, si el `default` de la migración debe cambiar, cómo se llega a `blocked` con auditoría, invalidación de tokens, etc. Este guard solo cubre el caso ya alcanzable hoy por CRUD genérico (`UserController::update()` puede poner `state = blocked`), sin adelantar el diseño de la historia.
4. Test: `AuthTest.php` — casos `blocked_user_cannot_login` (bloqueado no entra) y `pending_user_can_login` (para dejar constancia explícita, vía test, de que este fix no cambia el comportamiento de `pending`).

---

## 5. Fase 2.1 — `StoreRooms.publication_status` ↔ `StoreModeration` desincronizados (#3) ✅

**Pasos originales:** ver historial de este documento; el plan original proponía un endpoint dedicado (`PATCH /storeRooms/{id}/moderate`) y deprecar `publication_status` del CRUD genérico.

**Implementación real (variante no invasiva, distinta del plan original):** en vez de deprecar `PUT /storeRooms/{id}`, se detectó que ese seguía siendo el único endpoint que el frontend y los tests usan para aprobar/rechazar (`StoreRoomModerationTest::admin_can_approve_a_store_room` ya lo ejercitaba). Se decidió **no romper ese contrato**: `StoreRoomsController::update()` (`backend/app/Http/Controllers/StoreRoomsController.php`) ahora detecta si el payload trae un cambio real de `publication_status` a `approved`/`rejected` y, solo en ese caso, lo enruta por `StoreModerationService::moderate()` (nuevo, `backend/app/Services/StoreModerationService.php`) antes de delegar el resto de campos al CRUD genérico heredado. Se agregó además un chequeo de rol (`admin`) específico a esa rama, cerrando el hueco de autorización que tenía el endpoint (cualquier autenticado, incluido el propio landlord, podía autoaprobarse). `StoreModerationController::store()` (`/store_moderation`) también se enrutó por el mismo servicio, para que ambos caminos de escritura queden sincronizados sin eliminar ninguno.

Se agregaron `NotificationType::STORE_APPROVED` / `STORE_REJECTED`.

Tests agregados en `StoreRoomModerationTest.php`: creación del registro de auditoría + notificación al landlord, exigencia de motivo al rechazar, 403 para no-admins, y que un landlord sigue pudiendo editar otros campos de su propia bodega sin disparar el chequeo de moderación.

**Deuda pendiente, fuera de este pase:** `StoreModerationController::update()` / `destroy()` siguen siendo CRUD genérico sin sincronizar (no había test que los cubriera; tocarlos sin cobertura habría sido más riesgo que beneficio en este pase).

---

## 6. Fase 2.2 — `Payments` desacoplado de `ReservationService` (#4) ✅

**Implementación real** (`backend/app/Services/PaymentService.php`, `app/Policies/PaymentsPolicy.php`, `PaymentsController.php`): igual al plan, con dos precisiones:

- `PaymentService::process()` solo invoca `ReservationService::confirm()` si la reserva **no está ya** `confirmed`, para no reenviar la notificación `RESERVATION_CONFIRMED` de forma redundante en llamadas repetidas.
- `PaymentsPolicy::create()` (autorización de ownership) sí se implementó; la regla "un solo pago `paid` por reserva" (punto 4 del plan original) **no** se implementó — quedó fuera de este pase por no ser parte de las 5 inconsistencias documentadas y para no ampliar el alcance sin pedirlo explícitamente.
- **Coordinación con frontend (punto 5 del plan original):** se verificó por `grep` que el frontend actual **no tiene ninguna llamada a `/payments`** — el flujo de pago simulado del ERS todavía no está conectado del lado del cliente. Cero riesgo de romper algo ahí; también significa que este fix no es visible para ningún usuario todavía, solo queda disponible para cuando se conecte esa pantalla.

Tests agregados en `PaymentsTest.php`: 403 para quien no es el tenant dueño de la reserva, que un pago `paid` confirma la reserva, y que un pago `pending` no la confirma. Se tuvo que ajustar `test_store_creates_payment_when_authenticated` (y otro test de validación), que antes usaban un `$caller` sin relación con la reserva — exactamente el comportamiento inseguro que se estaba corrigiendo.

---

## 7. `User.state`: bloqueo/reactivación con auditoría (#5-B) — remitido a HUA-03, no planificado aquí

Este hallazgo **no se detalla en este documento** porque ya tiene dueño: es **HUA-03** en el Sprint Backlog (Sección A, Sprint 1, `secciones/seccionA.tex`, tabla `tab:sprint1`, 5 puntos, estado "Pendiente"). Duplicar aquí una implementación paso a paso sería planificar la misma historia dos veces en dos documentos que pueden desincronizarse — el mismo problema que el hallazgo #3 detecta entre `StoreRooms` y `StoreModeration`.

Lo único que este documento deja como referencia, para que quien tome HUA-03 en el sprint no lo pierda de vista:

- El campo y el diseño de la clase `RegistroAuditoria` ya están en el Diagrama de Clases (Sección B) — es el punto de partida natural para el modelo/migración de esa historia.
- El guard mínimo de la Fase 1.2 (§4) queda **intencionalmente acotado** a `state === 'blocked'` para no invadir decisiones de diseño de HUA-03 (qué significa `pending`, cómo se llega a `blocked`, invalidación de tokens, auditoría). Quien planifique HUA-03 debe revisar y probablemente reemplazar/extender ese guard como parte de la historia, no asumir que ya está resuelto.
- `HUA-01` (validación del permiso de bomberos, también Sprint 1) toca el mismo campo que el hallazgo #3 (`StoreRooms.publication_status`); vale la pena que quien planifique esa historia revise la Fase 2.1 (§5) antes de diseñarla, para no construir la validación de permisos sobre una base con dos fuentes de verdad desincronizadas.

---

## 8. Decisiones tomadas durante la implementación (revisar antes de aprobar el commit)

Estas dos decisiones estaban abiertas en el plan original y se resolvieron al implementar. **El usuario debe revisarlas** antes de dar por buenos los commits — no son irreversibles, pero cambian el comportamiento del sistema:

1. **Reports:** se eliminó `in_review` del enum (no se implementó como estado real), por ser el camino de menor esfuerzo y consistente con lo que el ERS pide. Si el equipo prefiere implementarlo como estado real más adelante, es una migración adicional, no un revert de esta.
2. **StoreRooms:** **no** se deprecó `publication_status` del CRUD genérico — se optó por la variante no invasiva descrita en §5 (interceptar el cambio de estado dentro del mismo endpoint) precisamente para no romper el contrato de `PUT /storeRooms/{id}`. Esto significa que la pregunta original ("¿deprecar o no?") queda resuelta como "no, por ahora"; si el equipo prefiere forzar un endpoint dedicado más adelante (p. ej. al planificar HUA-01), es un cambio adicional sobre esta base, no algo que haya que deshacer.

## 9. Qué NO cubre este plan (a propósito)

- **HUA-03 (bloqueo/reactivación de cuentas con auditoría)** — ver §7. Ya está en el Sprint Backlog (Sprint 1, Sección A); se planifica y detalla en esa ceremonia, no en este documento, para no duplicar la planificación de la misma historia en dos lugares. **No se implementó ni se implementará desde aquí.**
- Los demás módulos nuevos del ERS (organizaciones, Leodeguita, integración de inventario, panel admin/métricas) — no tienen inconsistencias porque no tienen código todavía; su planificación vive en el Sprint Backlog de cada sprint correspondiente (Sección A), no aquí.
- No modifica ningún diagrama UML ni la documentación LaTeX ya entregada (Secciones B y C) — este plan es la contraparte de implementación de esos hallazgos, no un reemplazo de la documentación.

## 10. Qué se tocó realmente en `recursos/leodega_codigo/` (Fases 1 y 2, 2026-08-14)

Todo en el working tree del repo `LeodegaPruebas` (rama `main`), **sin commitear ni pushear** — eso queda a cargo del usuario.

**Nuevos:**
`backend/app/Exceptions/AccountBlockedException.php` · `backend/app/Policies/PaymentsPolicy.php` · `backend/app/Services/PaymentService.php` · `backend/app/Services/StoreModerationService.php` · `backend/database/migrations/2026_08_14_000001_fix_reports_status_enum.php`

**Modificados:**
`backend/app/Enums/NotificationType.php` · `backend/app/Http/Controllers/{AuthController,PaymentsController,StoreModerationController,StoreRoomsController}.php` · `backend/app/Services/AuthService.php` · `backend/phpstan-baseline.neon` (2 entradas nuevas, mismo criterio que las 33 ya existentes) · `backend/phpunit.xml` (se agregó un `APP_KEY` de solo-testing; sin esto ningún test corría, ni siquiera antes de este trabajo) · 4 archivos de test (`AuthTest`, `PaymentsTest`, `ReportsTest`, `StoreRoomModerationTest`).

**No tocado:** `PLAN_MAESTRO_REFACTORIZACION.md` seguía apareciendo como borrado en `git status` desde antes de empezar esta sesión — no se restauró ni se volvió a tocar, es trabajo previo del usuario ajeno a este pase.
