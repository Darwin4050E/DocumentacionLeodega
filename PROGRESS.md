# Seguimiento del proyecto — Especificación de Proyecto Final (Leodega)

> Documento de control interno (no forma parte del entregable). Registra qué se ha hecho, qué decisiones se tomaron y por qué, y qué falta, para poder retomar el trabajo sin perder contexto.

**Curso:** SOFG1007 — Ingeniería de Software I (ESPOL/FIEC)
**Proyecto real:** Leodega (marketplace de almacenamiento), continuación de un sistema entregado por un equipo anterior.
**Fuente de la plantilla:** `project_instructions.md`.

---

## 1. Estado por sección

| Sección | Estado | Notas |
|---|---|---|
| Portada | 🟡 Casi completa | Falta la matrícula (`[ID]`) de los 4 integrantes |
| Acta de Conformidad del Cliente | 🔴 Pendiente | Requiere el PDF firmado real; penalidad de -100 si falta |
| Índices (TOC/LOF/LOT) | 🟢 Automáticos | No requieren edición manual |
| Información General del Proyecto | 🟢 Completa | Descripción, objetivos y alcance, tomados de `proyecto1P.tex` |
| Sección A — Gestión de Riesgos | 🟢 Completa | Matriz 5×5 + 9 riesgos evaluados y con plan de respuesta |
| Sección A — Sprint Backlogs | 🟢 Completa | 5 sprints, 38 historias reales | Ver convenciones (§3) sobre "Responsable" y fechas |
| Sección A — Cronograma | 🟢 Completa | Tabla CPM, ruta crítica, AOA y recursos limitados (imágenes reales del usuario) + Gantt generado por Claude |
| Sección B — Casos de uso | 🔴 Pendiente | Placeholders; falta el diagrama y la documentación de cada UC |
| Sección B — Diagrama de clases (SOLID/patrones) | 🔴 Pendiente | Falta el diagrama de clases real |
| Sección B — Objetos/Componentes/Despliegue | 🔴 Pendiente | Placeholders |
| Sección C — Actividad/Secuencia (32 mín.)/Colaboración/Estado | 🔴 Pendiente | Placeholders |
| Contribuciones Individuales | 🔴 Pendiente | Nombres y % de aporte reales |
| Referencias | 🟢 Completa | Sin placeholders |
| Anexos | 🟡 Parcial | Estructura lista, contenido específico pendiente |

Leyenda: 🟢 completo · 🟡 parcial/falta un dato puntual · 🔴 pendiente (trabajo sustancial)

---

## 2. Qué se hizo, en orden

1. **Plantilla LaTeX inicial** generada a partir de `project_instructions.md`: estructura completa (`main.tex` + `secciones/*.tex`), con placeholders en todo lo que es específico del proyecto. Logo ESPOL (no ETS). Corregidos dos bugs de compilación (guion bajo fuera de modo matemático → paquete `underscore`; `\\` seguido de `[` interpretado como argumento óptico → se separó con `\\{}`).
2. **Información General** (Descripción general, Objetivos, Alcance) completada con datos reales de `recursos/informacion_general/proyecto1P.tex` (el ERS del primer parcial).
3. **Gestión de Riesgos** completada con `recursos/riesgos/analisis_de_riesgo.md` (metodología, matriz 5×5, 6 estrategias) y `Estimación de Esfuerzo con COSMIC - Leodega.xlsx` (para justificar R-01/R-02 por concentración de tamaño funcional). 9 riesgos identificados y evaluados (R-01 a R-09), cada uno con estrategia, acciones de mitigación, plan de contingencia y responsable.
4. **Placeholders de portada** llenados: logo real (`imagenes/logo-espol.jpg`), nombre del sistema (Leodega), integrantes del equipo, profesor, cliente ("Leonardo", único nombre disponible en las actas de reunión) y período académico (2026 PAO2).
5. **Preparación del repositorio git** para `https://github.com/Darwin4050E/DocumentacionLeodega.git`: `git init`, `.gitignore` (excluye artefactos de compilación LaTeX y `.claude/settings.local.json`), commit `7d6684c` sobre el `README.md` ya existente en el remoto. **Aún no se ha hecho `git push`** (pendiente de confirmación del usuario, y hay cambios nuevos sin commitear desde entonces — ver `git status`).
6. **Corrección de tablas desbordadas**: la matriz de riesgo (Tabla 2.1) se ajustó con `\resizebox`; la tabla de plan de respuesta (Tabla 2.3) tenía el header "Responsable" desbordando su celda, se corrigió ensanchando la columna y reduciendo `\tabcolsep`.
7. **Sprint Backlogs completados** (5 sprints, 38 historias de usuario reales HUG/HUC/HUA/HUE/HUL, tomadas del capítulo "User Stories" de `proyecto1P.tex`, agrupadas por el campo "Sprint" que ya traía cada historia). Ver convenciones adoptadas en §3.
8. **Centrado de tablas**: se añadieron los tipos de columna reutilizables `M{ancho}` (centrado horizontal+vertical) y `L{ancho}` (alineado a la izquierda, centrado vertical) en `main.tex`, aplicados a **todas** las tablas del documento (Sección A, Sección B, portada, contribuciones) para que headers y contenido dejen de quedar pegados arriba.
9. **Cronograma del Proyecto completado**: tabla de 38 actividades (HST-001 a HST-038) con dependencias, duración, personal, EET/holgura/LET y marca de ruta crítica; texto de ruta crítica (52 días); sección nueva de planificación con recursos limitados (87→85 días, límite 4 personas). Todo tomado del capítulo "Project Planning and Control" de `proyecto1P.tex`.
10. **Imágenes del cronograma insertadas**: el usuario proporcionó las imágenes reales de AOA (`imagenes/activity-on-arrow.png`) y de recursos limitados sin optimizar/optimizado (`imagenes/resoruce-limited-without-optimizatio.png` y `imagenes/resource-limited-optimization.png` — nombres de archivo tal cual los dejó el usuario, con una errata en el primero que no se corrigió porque hay que respetar el nombre real del archivo en disco). No tenía el diagrama de Gantt, así que Claude lo generó programáticamente con Python/Matplotlib (`imagenes/diagrama_gantt.png`) a partir de los mismos datos EET/duración/ruta crítica de la tabla CPM, con la misma paleta de colores por sprint que las otras imágenes para mantener consistencia visual. Se documentó en el propio LaTeX que la herramienta utilizada fue "Python/Matplotlib" en vez de dejar el placeholder de herramienta comercial.

---

## 3. Decisiones y convenciones adoptadas (el "por qué")

- **Los IDs de historia (HUG/HUC/HUA/HUE/HUL) y los IDs de actividad (HST-XXX) son intencionalmente independientes.** `proyecto1P.tex` los usa como dos numeraciones distintas: HU-XX para las historias de usuario (Sección de Requerimientos) y HST-XXX para las actividades del análisis PERT/CPM. Se verificó que hay correspondencia 1 a 1 entre ambos conjuntos (38 = 38, mismas duraciones/sprint), pero se documentan por separado (con nota al pie en el cronograma) en vez de forzar una fusión, para ser fieles a la fuente.
- **"Responsable" = "Equipo Leodega"** en todas las historias de los Sprint Backlogs. Ninguna historia tiene un integrante asignado individualmente en el documento fuente; en vez de inventar asignaciones, se usó un valor genérico. **Es fácilmente reemplazable** por nombres reales más adelante (es solo texto en la tabla).
- **Fechas de sprint = rangos de "Día N" relativos**, no fechas de calendario. El cronograma PERT/CPM original solo tiene días relativos (0 a 52), sin fechas reales. Se dividieron los 52 días en 5 sprints aproximadamente iguales (11/10/11/10/10 días). El instructivo del curso no exige fechas de calendario exactas, así que esto es válido; **también reemplazable** por fechas reales cuando se definan.
- **Estado de todas las historias = "Pendiente"**, no "Hecho". Corrección importante del usuario: el proyecto fue *empezado por un equipo anterior* (plataforma base ya entregada: auth, RBAC, publicación/moderación de almacenes, búsqueda, reservas, calificaciones, reportes, notificaciones, mensajería, pagos). El equipo actual **extiende** esa base (organizaciones multiempresa, Leodeguita, integración de inventario, cumplimiento de permiso de bomberos, panel de administrador), y en el primer parcial *solo se especificaron* los requerimientos — no se implementó nada todavía. Por eso todas las 38 historias del backlog están "Pendiente".
- **Cliente = "Leonardo"** (fallback explícito pedido por el usuario): es el único nombre de cliente que aparece en el documento fuente (actas de reunión), sin apellido.
- **Cambio de portada no revertido**: en algún momento el logo se agrandó a 14cm, se quitaron las líneas divisorias (`\rule`) alrededor del título "LEODEGA", y el header de la tabla de integrantes pasó de "Matrícula / ID" a "Matrícula". Esto se preservó tal cual (fue un cambio intencional, no un error a corregir).

---

## 4. Lo que yo (Claude) puedo y no puedo hacer desde aquí

**Sí puedo:**
- Redactar documentación de casos de uso, justificación de SOLID/patrones de diseño, y descripciones de diagramas de actividad/secuencia/estado — **si se me da el diagrama ya elaborado** (imagen o descripción) o una lista clara de entidades/flujos del sistema.
- Seguir ajustando formato LaTeX (tablas, estilos, estructura).
- Rellenar cualquier campo puntual (fechas, nombres, porcentajes) en cuanto se me proporcionen.

**No puedo:**
- Generar los diagramas UML en sí (casos de uso, clases, objetos, componentes, despliegue, actividad, secuencia, colaboración, estado) como los pide la rúbrica — estos deben modelarse en una herramienta como Visual Paradigm, StarUML o draw.io y exportarse como imagen a `imagenes/`.
- Inventar datos que no están en ninguna fuente (matrículas, % de aporte individual, fechas de calendario reales, nombre de la herramienta ágil usada) sin que el usuario los confirme.

---

## 5. Pendientes concretos (para retomar)

1. Matrícula de los 4 integrantes (portada).
2. PDF del acta de conformidad firmada por el cliente.
3. Nombre de la herramienta ágil (Jira/Trello/Azure DevOps/etc.) para los Sprint Backlogs (el Gantt ya quedó documentado como generado con Python/Matplotlib).
4. Sección B completa (casos de uso, clases, objetos, componentes, despliegue).
5. Sección C completa (actividad, ≥32 secuencia, colaboración, estado).
6. Contribuciones individuales (nombres y %).
7. `git push` al repositorio remoto — preparado pero no ejecutado; hay cambios sin commitear (`main.tex`, `secciones/*.tex`, `main.pdf`, y las 4 imágenes nuevas en `imagenes/`).
