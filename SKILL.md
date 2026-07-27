---
name: actualizar-jira
description: "Actualiza el contenido de un issue de Jira ya existente siguiendo las convenciones de Smowltech, usando el MCP de Atlassian. Edita descripción, ajusta labels o campos, añade comentarios estructurados y transiciona estados. Tras la actualización, verifica que la rama git local contenga el ID del issue (ej. GOT-431) y, si no, crea automáticamente una rama con el formato correcto preservando los cambios en curso. Úsala siempre que el usuario pida actualizar/editar/comentar/transicionar un ticket de Jira (frases tipo \"actualiza el Jira GOT-XXX\", \"comenta en el Jira\", \"mueve GOT-XXX a Done\", \"ponle estos labels\", \"completa los criterios de aceptación\"), incluso si no nombra explícitamente Jira pero el contexto deja claro que se refiere a un ticket ya creado. No crea issues ni subtasks — la creación es responsabilidad del desarrollador por decisión del equipo."
---

# actualizar-jira — actualizar issues de Jira ya existentes

Esta skill cubre la operación de **escribir sobre tickets que ya existen** en Jira: rellenar/mejorar la descripción, ajustar labels o campos, comentar el progreso y transicionar de estado a medida que la tarea avanza por el flujo del equipo.

**Esta skill no crea issues ni subtasks.** Es una decisión deliberada del equipo: la creación de tickets se hace a mano porque el desarrollador conoce mejor el contexto (proyecto, tipo, epic/sprint, prioridad, criterios de aceptación). Si el usuario pide crear un issue nuevo o una subtask, recuérdaselo educadamente y guíalo para que lo cree él mismo en Jira — luego, una vez exista el `key`, esta skill puede entrar a rellenar/completar su contenido.

Se apoya en el MCP de Atlassian (ya conectado en la cuenta del usuario). Si por algún motivo el MCP no está disponible, degrada con elegancia: genera el contenido (descripción, comentario) listo para pegar a mano y avisa de qué pasos faltarían por hacer manualmente.

## Cuándo usar esta skill

Disparadores claros:
- "actualiza el Jira GOT-431 con esta descripción mejor"
- "comenta en el GOT-431 que ya está mergeado"
- "transiciona GOT-431 a Code Review"
- "ponle el label `needs-qa` al GOT-431"
- "completa los criterios de aceptación del GOT-431"
- "el GOT-431 tiene la descripción muy floja, redáctala bien"

Disparadores que **no** son para esta skill (pero conviene reconocerlos para responder bien):
- "crea un issue en Jira para esto" → fuera de alcance. Indica al usuario que cree el issue manualmente y, una vez tenga el `key`, vuelva a invocarte para rellenar la descripción.
- "crea la subtask de DoD" → idem, fuera de alcance. La subtask la crea él/ella; el contenido del DoD lo genera por separado.

## Información que necesitas

Antes de tocar nada, asegúrate de saber:

1. **Operación**: leer / editar campos / comentar / transicionar.
2. **`key` del issue** (ej. `GOT-431`). Si no lo tienes, pídelo. No intentes adivinar a partir del título.
3. **Qué cambia exactamente** (texto nuevo de descripción, labels a añadir/quitar, comentario a publicar, estado destino).

Si necesitas listar proyectos o consultar transiciones disponibles, los tools del MCP de Atlassian los exponen — úsalos en lugar de adivinar.

## Cómo operar contra Jira

El MCP de Atlassian expone, entre otros, estos tools. Esta skill **solo** debe usar los marcados como permitidos:

**Permitidos en esta skill:**
- `getAccessibleAtlassianResources` — para obtener el `cloudId` del workspace. Llámalo una vez al inicio si aún no lo tienes.
- `getJiraIssue` — para leer la descripción/estado/labels/subtasks actuales antes de tocar nada. Imprescindible si vas a editar para no sobreescribir contenido valioso.
- `editJiraIssue` — para modificar resumen, descripción, labels, prioridad u otros campos editables.
- `addCommentToJiraIssue` — para añadir comentarios.
- `getTransitionsForJiraIssue` — para listar las transiciones disponibles desde el estado actual.
- `transitionJiraIssue` — para mover de estado, usando el id devuelto por la llamada anterior.
- `searchJiraIssuesUsingJql` — para localizar un issue por criterios si el usuario no tiene el `key` a mano.
- `lookupJiraAccountId` — para resolver `assignee`/`reporter` por email cuando haga falta cambiar la asignación.

**NO permitidos en esta skill (por diseño):**
- `createJiraIssue` — la creación de issues y subtasks la hace el desarrollador a mano.
- `createIssueLink` — los links entre issues los gestiona el desarrollador.

Si el usuario te pide explícitamente algo de la lista no permitida, no lo hagas: explícale el porqué (decisión del equipo, el desarrollador conoce mejor el contexto), y ofrece ayudarle con el **contenido** para que él lo cree a mano (ej. "te preparo el resumen y la descripción para que la pegues al crear el issue").

Reglas generales:
- **Siempre** confirma con el usuario antes de aplicar cambios. Una edición errónea sobreescribe trabajo; un comentario erróneo es ruido; una transición errónea descoloca el sprint.
- Para descripciones y comentarios, Jira espera ADF (Atlassian Document Format). Si el usuario te da Markdown, conviértelo. Para texto plano, envuelve en el ADF mínimo de un párrafo. Ver `references/adf.md`.
- Antes de editar la descripción, **lee la actual con `getJiraIssue`**. Si vas a reemplazarla, pregunta si conserva algo. Si vas a añadir secciones, mezcla con la existente.
- Nunca inventes `accountId` — resuelve con `lookupJiraAccountId` a partir del email.

## Plantillas de contenido para descripciones

Estas plantillas se usan cuando el usuario te pide **rellenar o mejorar** la descripción de un issue ya creado. La idea es que el contenido sea consistente con el resto de tickets del equipo.

### Story / Task (issue padre estándar)

```markdown
## Contexto

<Una o dos frases con el porqué del trabajo: problema de usuario, decisión de producto,
necesidad técnica.>

## Objetivo

<Una frase que describe el resultado esperado, no la implementación.>

## Alcance

- <Qué entra.>
- <Qué NO entra (explícito si hay riesgo de scope creep).>

## Criterios de aceptación

- [ ] <Criterio observable y verificable.>
- [ ] <Otro criterio.>

## Notas técnicas

<Opcional. Decisiones de implementación, dependencias, riesgos.>

## Referencias

- <Enlaces a Figma, Slack, docs, otros issues.>
```

### Bug

```markdown
## Resumen

<Qué falla, en una frase.>

## Pasos para reproducir

1. <Paso a paso, con datos concretos.>
2.
3.

## Resultado actual

<Qué hace el sistema hoy.>

## Resultado esperado

<Qué debería hacer.>

## Entorno

- Entorno: <pro | pre | local>
- Versión / commit: <hash o tag>
- Usuario afectado: <email u organización>
- Navegador / SO: <si aplica>

## Evidencia

<Capturas, vídeo, logs, traza de error.>

## Impacto

<A cuántos usuarios afecta, si hay workaround, prioridad sugerida.>
```

Cuando rellenes una de estas plantillas sobre un issue existente:
- Si la descripción actual ya tiene algunas de estas secciones, **respétalas** y completa solo las que faltan o estén vacías.
- Si la descripción actual está vacía o es una sola línea, reemplázala por la plantilla rellena.
- Si el usuario te indica que parta de cero, confirma una vez más antes de sobreescribir.

## Comentarios "buenos"

Cuando el usuario te pida comentar en un Jira, evita comentarios huecos tipo "ya está hecho". Usa esta estructura corta:

```markdown
**<Hito>**: <una línea con lo que ha pasado>

- PR: <enlace>
- Notas: <opcional, decisiones, cambios de alcance, dudas pendientes>
```

Ejemplos:
- `**PR abierto**: pendiente de review.\n- PR: https://github.com/.../pull/431\n- Notas: incluye migración menor, ver descripción.`
- `**Mergeado a develop**: listo para QA.\n- Subtask DoD: GOT-431-1`
- `**Bloqueado**: necesito decisión de producto sobre flujo de error.\n- Pregunta concreta: ...`

## Transiciones de estado del flujo base

El flujo del equipo usa, como mínimo, estos estados (los nombres reales pueden variar — usa `getTransitionsForJiraIssue` para descubrirlos al primer uso y guárdalos en la conversación):

| Momento del flujo | Estado destino sugerido |
| --- | --- |
| Empiezo a analizar / aplicar la solución | `In Progress` |
| Abro el PR y asigno reviewers | `In Review` / `Code Review` |
| Reviewers aprueban y se mergea | `Ready for QA` |
| Asigno testers en la subtask de DoD | `In QA` |
| QA valida | `Done` |

Antes de transicionar:
1. Llama a `getTransitionsForJiraIssue` con el `issueIdOrKey`.
2. Empareja la transición buscada por **nombre**, no por id (los ids cambian entre proyectos).
3. Si la transición no existe (ej. saltar de `Open` a `Ready for QA` sin pasar por estados intermedios), avisa al usuario y propón la cadena correcta.

## Patrones habituales (recetas rápidas)

### Rellenar la descripción de un issue que llegó vacío

1. `getJiraIssue` para leer el estado actual: resumen, descripción, labels, tipo.
2. Decide qué plantilla aplica (Story/Task o Bug) según el `issueType`.
3. Pregunta al usuario los huecos imprescindibles (contexto, criterios de aceptación, pasos de reproducción si es bug).
4. Convierte a ADF la descripción final.
5. `editJiraIssue` cambiando solo el campo `description`.
6. Ejecuta el paso de **post-acción: alinear la rama git con el issue** (sección de más abajo).
7. Devuelve un breve resumen de lo modificado, el enlace al issue y, si aplica, el cambio de rama.

### Comentar un hito del flujo

1. Confirma el hito (PR abierto, PR mergeado, bloqueado, etc.) y los datos asociados (enlace al PR, motivo del bloqueo…).
2. Genera el comentario siguiendo el formato corto.
3. `addCommentToJiraIssue`.
4. Si el hito indica que el usuario está empezando a trabajar el ticket (ej. `In Progress`, `PR abierto`), ejecuta el paso de **post-acción: alinear la rama git con el issue**. Si el hito es de cierre (`Mergeado`, `Listo para QA`), normalmente la rama ya estará bien — comprueba igualmente, no cuesta nada.
5. Devuelve el enlace al comentario si el MCP lo expone y, si aplica, el cambio de rama.

### Transicionar al cerrar el PR (o al pasar a QA)

1. `getTransitionsForJiraIssue` para listar lo disponible desde el estado actual.
2. Confirma con el usuario el estado destino — léelo del nombre devuelto por el MCP, no asumas.
3. `transitionJiraIssue` con el id de la transición elegida.
4. Opcional: `addCommentToJiraIssue` con un comentario que explique el porqué del cambio de estado.
5. Si la transición es a un estado activo (`In Progress`, `In Review`, `Code Review`), ejecuta el paso de **post-acción: alinear la rama git con el issue**.

### Petición fuera de alcance: "créame el issue"

1. Recuerda al usuario que la creación se hace a mano (decisión del equipo).
2. Ofrécele preparar el **contenido** (resumen + descripción rellena con la plantilla) en un bloque markdown listo para pegar al crear el issue.
3. Una vez exista el issue y tenga `key`, esta skill puede entrar a actualizarlo si hace falta.

## Post-acción: alinear la rama git con el issue

Después de una acción **de escritura** sobre el issue (editar, comentar un hito de trabajo, transicionar a un estado activo como `In Progress` o `In Review`), el flujo del equipo pide que la rama local de git tenga el `key` del issue en su nombre, para mantener trazabilidad entre código y ticket. Esta skill comprueba esto automáticamente y, si la rama no encaja, crea una rama nueva con el formato correcto **sin perder los cambios en curso**.

No ejecutes este paso si la acción fue solo de lectura (`getJiraIssue`, búsquedas, listar transiciones) — solo tiene sentido cuando el usuario está realmente trabajando sobre el issue.

### Algoritmo

1. **Detecta si hay un repo git accesible**. Lanza `git rev-parse --is-inside-work-tree`. Si falla (no hay repo, no hay git, o el cwd no es el repo del usuario), salta toda esta sección y avisa: "no he podido verificar la rama, hazlo a mano".
2. **Comprueba que no hay operaciones en curso**. Si existen `.git/MERGE_HEAD`, `.git/rebase-merge`, `.git/rebase-apply` o `.git/CHERRY_PICK_HEAD`, hay un merge/rebase/cherry-pick a medias. **No toques nada**: avisa al usuario y deja que termine la operación antes de cambiar de rama.
3. **Lee la rama actual** con `git branch --show-current`. Si devuelve cadena vacía estás en detached HEAD — avisa y no cambies de rama.
4. **Comprueba si el nombre contiene el `key`** (regex case-insensitive `\b<KEY>\b`, ej. `\bGOT-431\b`). Si lo contiene → todo correcto, no hacer nada más y seguir con la salida normal.
5. **Si contiene un `key` distinto** (ej. estás en `feature/GOT-100-...` y acabas de actualizar `GOT-200`): **no cambies de rama automáticamente**. Avisa: "Estás en `feature/GOT-100-...` pero acabas de actualizar `GOT-200`. ¿Quieres que cree una rama nueva para `GOT-200`, o estás trabajando intencionalmente sobre la rama de `GOT-100`?". Espera respuesta.
6. **Si no contiene ningún `key` de Jira** (típicamente estás en `develop`/`main`/`master` o en una rama temporal): procede a crear la rama nueva.

### Construir el nombre de rama

Formato: `<tipo>/<KEY>-<slug>`

- **`<tipo>`** — derivado del `issueType` que devuelve `getJiraIssue`:
  - `Story` / `Task` / `Improvement` / `New Feature` → `feature`
  - `Bug` → `bugfix`
  - `Hotfix` (si existe como tipo en el proyecto) → `hotfix`
  - Cualquier otro → `chore`
- **`<KEY>`** — el key tal cual, en mayúsculas (`GOT-431`).
- **`<slug>`** — derivado del `summary` del issue:
  - Pasa a minúsculas.
  - Quita acentos y `ñ` (`ñ` → `n`, `á` → `a`, etc.) para evitar problemas en shells / remotos.
  - Quita prefijos decorativos del resumen tipo `[Bug]`, `[Hotfix]`, etc.
  - Sustituye espacios y signos de puntuación por guiones.
  - Colapsa guiones repetidos.
  - Recorta a ~6 palabras significativas o ~50 caracteres, lo que sea más corto. Si tienes que cortar, hazlo en un guion (no en medio de una palabra).

**Ejemplos:**

| Issue | Tipo | Summary | Rama |
| --- | --- | --- | --- |
| `GOT-431` | Story | `Permitir login con SAML` | `feature/GOT-431-permitir-login-con-saml` |
| `GOT-512` | Bug | `[Bug] El export tarda más de 30s` | `bugfix/GOT-512-el-export-tarda-mas-de-30s` |
| `GOT-204` | Task | `Actualizar dependencias de mayo` | `chore/GOT-204-actualizar-dependencias-de-mayo` |

### Crear la rama sin perder cambios

Usa **`git checkout -b <nueva-rama>`**. Este comando:
- Crea la rama nueva apuntando al `HEAD` actual.
- Lleva consigo cualquier cambio sin commitear (staged y unstaged) automáticamente.
- No descarta commits previos; si la rama actual tenía commits no presentes en la base, esos commits también pasan a estar en la rama nueva porque ambas parten del mismo `HEAD`.

**No uses** `git stash` + `git checkout` + `git stash pop`: añade fricción y puede generar conflictos al popear en archivos complejos. `checkout -b` es la opción limpia para este caso.

**Secuencia exacta:**

1. Comprueba si la rama destino ya existe: `git rev-parse --verify --quiet <nueva-rama>`.
   - Si existe **y apunta al mismo commit que `HEAD`** (`git rev-parse <nueva-rama>` == `git rev-parse HEAD`): solo conmuta con `git checkout <nueva-rama>`.
   - Si existe **y apunta a otro commit**: avisa al usuario, no fuerces. Sugiere un sufijo (`-v2`, fecha, etc.) o que la borre/renombre él.
   - Si no existe: continúa.
2. Comprueba el estado del working tree con `git status --porcelain` para saber si hay cambios sin commitear (útil solo para informar al usuario, no para decidir).
3. Ejecuta `git checkout -b <nueva-rama>`.
4. Informa al usuario:
   - Rama anterior → rama nueva.
   - Si había cambios sin commitear, recuérdale que se llevaron consigo a la nueva rama.
   - Si la rama anterior era una **rama base del equipo** (`develop`, `main`, `master`), aclara que esa rama base quedó intacta (es siempre así con `checkout -b`, pero al usuario le tranquiliza confirmarlo).
   - Si la rama anterior era **otra rama que no es base** (huérfana, prototipo, scratch), pregunta si quiere borrarla con `git branch -d <antigua>` o `git branch -D <antigua>` si tenía commits sin mergear. **Nunca** la borres sin confirmación explícita.

### Confirmación

- Si la rama actual es la base del equipo (`develop`/`main`/`master`) y no hay otro `key` de Jira en juego, **procede sin confirmación previa**: el usuario espera este comportamiento automático. Notifica el cambio en la salida final.
- Si la rama actual tiene **otro key** de Jira, **pregunta primero** (caso 5 del algoritmo).
- Si la rama actual no es base pero tampoco tiene un key (ej. `wip-pruebas`, `temp-debug`), **pregunta primero** antes de crear la nueva — puede que el usuario quisiera mantener ese trabajo aparte.

### Degradación si no hay git accesible

Si no hay repo git, git no está disponible, o no tienes permisos para ejecutar comandos:
1. No reintentes.
2. Avisa al usuario: "no he podido verificar la rama".
3. Sugiérele el comando exacto para que lo ejecute él:
   ```bash
   git checkout -b feature/GOT-431-permitir-login-con-saml
   ```
   (con el nombre derivado según las reglas anteriores).

### Errores comunes en este paso

- **Cambiar de rama sin avisar cuando ya había otro `GOT-X` en juego**: corre el riesgo de llevar trabajo a la rama equivocada. Siempre pregunta en ese caso.
- **Usar `stash`**: complica el flujo y puede provocar conflictos. `checkout -b` ya conserva los cambios.
- **Borrar la rama antigua sin permiso**: aunque parezca "huérfana", puede contener trabajo que el usuario quiere preservar. Pregunta siempre.
- **Forzar el rename de una rama que ya existía con otro commit**: nunca uses `-B` para sobreescribir. Pide al usuario que decida.

## Salida al usuario

Mantén la salida muy compacta:
- Confirma qué hiciste en Jira (con el `key` y la URL del issue cuando aplique).
- Si hubo varias acciones (editar + comentar + transicionar), lístalas como bullets cortos.
- Si se ejecutó el paso de alineamiento de rama, indícalo en una línea: rama anterior → rama nueva, y si hubo cambios en curso que viajaron con el checkout.
- Si algo falló o quedó pendiente (ej. no había transición directa, el usuario pidió algo fuera de alcance, o no se pudo verificar la rama), dilo explícitamente y propón el siguiente paso.

## Degradación si el MCP de Atlassian no está

Si las llamadas al MCP fallan o no están autorizadas:
1. No reintentes en bucle.
2. Genera el contenido (descripción rellena, comentario, estado destino sugerido) en un bloque markdown listo para pegar.
3. Avisa al usuario de los pasos manuales que faltan ("pega esto en el campo Description del issue GOT-XXX y mueve a estado Y").

## Errores comunes a evitar

- **Sobreescribir descripción sin leer la actual**: lee primero con `getJiraIssue`, decide si reemplazas o completas, y solo entonces edita.
- **Crear cosas porque parece la opción rápida**: aunque el MCP exponga `createJiraIssue`, esta skill no lo usa. Recuérdaselo al usuario.
- **Asumir el `accountId` de los assignees**: siempre resolver por email.
- **Adivinar transiciones**: usa siempre `getTransitionsForJiraIssue` antes de transicionar.
- **Inventar campos custom**: si el proyecto requiere un campo custom (Sprint, Epic Link, Story Points, etc.) y no lo conoces, pregunta antes de editar.
