# Instrucciones para generar los diagramas UML con Claude (PlantUML)

> Documento de trabajo interno del equipo — no forma parte del entregable.
> Objetivo: que cada integrante pueda pedirle a Claude sus diagramas asignados
> sin ambigüedad, sin que Claude invente clases/flujos que no existen, y con
> resultados consistentes entre todos.

---

## 1. Contexto del proyecto (pégalo o resúmelo al abrir tu sesión con Claude)

- **Proyecto:** Leodega (marketplace de almacenamiento).
- **Situación:** el sistema **ya fue iniciado por un equipo anterior**, que
  entregó la plataforma base: autenticación, RBAC, publicación/moderación de
  almacenes, búsqueda, reservas, calificaciones, reportes, notificaciones,
  mensajería, pagos.
- **Lo que hace este equipo:** *incrementa* esa base con nuevos requerimientos
  funcionales y no funcionales (organizaciones multiempresa, Leodeguita,
  integración de inventario, cumplimiento de permiso de bomberos, panel de
  administrador — confirma la lista exacta contra el ERS antes de empezar).
- **Curso:** SOFG1007 — Ingeniería de Software I (ESPOL/FIEC), entrega del
  segundo parcial.
- **Fuentes de verdad que Claude debe usar:**
  1. El **código real** del proyecto (ruta que te compartirá quien lidera el
     repo de documentación — pídesela si no la tienes).
  2. El **ERS del primer parcial**: `recursos/informacion_general/proyecto1P.tex`
     (requerimientos, alcance, historias de usuario).
  3. El repo de documentación LaTeX actual (este mismo), en especial
     `secciones/seccionB.tex` y `secciones/seccionC.tex`, que ya tienen la
     estructura y los nombres de archivo esperados para cada imagen.

---

## 2. Regla de alcance (IMPORTANTE — decisión ya tomada por el equipo)

Los diagramas deben modelar el **sistema completo** (lo heredado + lo nuevo),
**no solo el incremento**. La rúbrica del curso exige explícitamente "todas
las clases... del sistema" en el diagrama de clases, y equivalente para
componentes y despliegue — modelar solo lo nuevo perdería puntos por
incompletitud.

Pero cada elemento (clase, componente, actividad, etc.) debe quedar marcado
como:

- 🟦 **Heredado** — ya existía en la entrega del equipo anterior.
- 🟩 **Nuevo** — agregado por este equipo en este parcial.

Esto se logra con **color o etiqueta explícita dentro del propio código
PlantUML** (ver plantilla en la sección 5) y con la tabla de trazabilidad
(sección 4). Esa distinción es clave para la sección de **Contribuciones
Individuales** del documento.

**Excepción — Diagramas de Secuencia (32 obligatorios):** ahí sí prioricen
escenarios transaccionales de la funcionalidad **nueva**, porque es donde más
aporta mostrar profundidad del trabajo de este equipo. Completen con algunos
escenarios heredados solo si hace falta para llegar al mínimo de 32 o si el
escenario es crítico para el sistema.

---

## 3. Flujo de trabajo con Claude (paso a paso, por persona)

### Paso 0 — Dale contexto a Claude una sola vez al inicio de tu sesión
Pega el contenido de la sección 1 y 2 de este documento, y dile en qué
directorio está el código y qué tipo(s) de diagrama te tocan (ver sección 6).

### Paso 1 — Que Claude explore el código antes de proponer nada
Pídele explícitamente que **lea el código real** (no que asuma), identifique
los módulos/entidades relevantes para tu tipo de diagrama, y que separe
mentalmente qué es heredado vs. nuevo cruzando contra el ERS.

Ejemplo de prompt:
> "Antes de proponer nada, explora el directorio `<ruta>` y dime qué
> paquetes/módulos existen y cuáles corresponden a funcionalidad nueva según
> `proyecto1P.tex`. No generes diagramas todavía."

### Paso 2 — Pide el LISTADO antes que el diagrama
No le pidas el diagrama completo de una. Primero el listado candidato, con
una frase de justificación por ítem (por qué es representativo).

Ejemplo:
> "Con base en lo anterior, dame un listado de los diagramas de secuencia más
> representativos que deberíamos documentar (necesitamos 32 en total). Para
> cada uno: nombre del escenario, si es nuevo o heredado, y qué archivos de
> código lo sustentan. Todavía no generes el PlantUML."

**Revisa y aprueba (o corrige) ese listado antes de seguir.** Si Claude
propone algo que no reconoces del proyecto, es la señal de alucinación —
pídele que te muestre el archivo/línea exacta que lo sustenta.

### Paso 3 — Genera UNO a la vez
Nunca pidas "genera los 12 diagramas de clase". Pide uno, revísalo, y recién
ahí el siguiente. Esto mantiene la calidad y hace más fácil detectar errores.

Ejemplo:
> "Genera el PlantUML del diagrama #1 del listado (`Gestión de Almacenes`).
> Usa el formato de la sección 5 de INSTRUCCIONES_DIAGRAMAS.md, incluyendo
> la tabla de trazabilidad."

### Paso 4 — Exige trazabilidad en cada diagrama
Todo diagrama debe venir acompañado de la tabla de la sección 4. Si Claude no
la incluye, pídesela explícitamente antes de dar el diagrama por bueno.

### Paso 5 — Verifica contra el código antes de aceptar
Antes de marcar un diagrama como terminado, abre tú mismo (o pide a Claude que
cite) el archivo/clase/función real que sustenta cada elemento no trivial del
diagrama. No aceptes un diagrama solo porque "se ve bien".

---

## 4. Tabla de trazabilidad obligatoria (pégala después de cada diagrama)

| Elemento del diagrama | Fuente (archivo / clase / método) | Estado |
|---|---|---|
| `Almacen` | `src/.../Almacen.java` | 🟦 Heredado |
| `Leodeguita` | `src/.../Leodeguita.java` | 🟩 Nuevo |
| `SolicitudPermisoBomberos` | ERS §X.X (aún sin código) | 🟩 Nuevo (planificado) |

Si un elemento nuevo todavía no tiene código (solo está en el ERS), márcalo
igual como 🟩 Nuevo y anota que la fuente es el ERS, no inventes una ruta de
archivo que no existe.

---

## 5. Formato de entrega esperado por diagrama

1. **Código PlantUML** (`.puml`), en un archivo por diagrama.
2. **Convención de color** dentro del propio `.puml` para distinguir estado
   (ejemplo con `skinparam` y notas — ajusta a tu tipo de diagrama):

   ```plantuml
   @startuml
   skinparam class {
     BackgroundColor<<Nuevo>> #D5F5E3
     BackgroundColor<<Heredado>> #D6EAF8
   }

   class Almacen <<Heredado>>
   class Leodeguita <<Nuevo>>
   @enduml
   ```

3. **Tabla de trazabilidad** (sección 4) inmediatamente después del bloque de
   código.
4. **Nombre de archivo de imagen exportada**: debe coincidir EXACTO con lo que
   ya espera `secciones/seccionB.tex` / `seccionC.tex` (ver sección 6). Si
   generas más diagramas de los que hay placeholders (p. ej. objetos,
   actividad, comunicación, estado — donde el `.tex` solo trae 2-3 de
   ejemplo), sigue la numeración consecutiva (`_03`, `_04`, ...).
5. Exporta el `.puml` a PNG (con la extensión de PlantUML, el jar oficial, o
   `plantuml.com/plantuml`) y coloca el PNG en `imagenes/` con ese nombre
   exacto.

---

## 6. División de trabajo y nombres de archivo esperados

| Responsable | Diagramas | Depende de | Archivos esperados en `imagenes/` |
|---|---|---|---|
| Persona 1 | Casos de uso + documentación de cada UC | Solo del ERS | `diagrama_casos_uso.png` |
| Persona 1 | Actividad (todos los procesos) | Casos de uso ya listados | `actividad_01.png`, `actividad_02.png`, ... |
| Persona 2 | Clases (SOLID + patrones) | Solo del ERS/código | `diagrama_clases.png` |
| Persona 2 | Objetos | Diagrama de clases (Persona 2, mismo) | `objetos_01.png`, `objetos_02.png`, ... |
| Persona 2 | Estado | Diagrama de clases | `estado_01.png`, `estado_02.png`, ... |
| Persona 3 | Componentes | Arquitectura general del código | `diagrama_componentes.png` |
| Persona 3 | Despliegue | Diagrama de componentes | `diagrama_despliegue.png` |
| Persona 4 | Secuencia (mínimo 32, repartibles entre todos una vez listas Casos de Uso + Clases) | Casos de uso + Clases | `secuencia_01.png` ... `secuencia_32.png` |
| Persona 4 | Colaboración/Comunicación | Diagramas de secuencia ya hechos | `comunicacion_01.png`, `comunicacion_02.png`, ... |

**Orden crítico:** Casos de Uso y Clases deben quedar aprobados primero (son
la base de casi todo lo demás). Una vez estén listos, el resto — y en
particular los 32 de secuencia — se puede repartir entre los 4 en paralelo.

Ajusten los nombres de "Persona 1-4" por los nombres reales del equipo.

---

## 7. Checklist rápido antes de dar un diagrama por terminado

- [ ] Cada clase/actor/nodo del diagrama existe en el código real o en el ERS
      (no inventado).
- [ ] Marcado como 🟦 Heredado / 🟩 Nuevo.
- [ ] Tabla de trazabilidad incluida y completa.
- [ ] Nombre de archivo PNG coincide exactamente con el que espera el `.tex`.
- [ ] Sin errores de notación UML (la rúbrica penaliza hasta -40 pts por esto).
- [ ] El diagrama es legible en una sola página (la rúbrica penaliza hasta
      -50 pts por mala organización/legibilidad).
