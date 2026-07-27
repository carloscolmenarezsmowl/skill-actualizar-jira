# actualizar-jira

Skill del equipo de Smowltech que **actualiza** issues de Jira ya existentes siguiendo las convenciones internas: rellenar descripciones con plantilla, comentar hitos, transicionar estados, ajustar labels o assignee. Tras la actualización, comprueba que la rama git local contiene el ID del issue y, si no, **crea la rama correcta preservando los cambios en curso**.

**No crea issues ni subtasks, ni enlaza issues entre sí.** Esas operaciones quedan como responsabilidad del desarrollador, que conoce mejor el proyecto destino, tipo de issue y campos custom requeridos. La skill se centra exclusivamente en escribir sobre tickets ya creados.

## Dependencia: MCP de Atlassian

Esta skill se apoya en el MCP oficial de Atlassian para operar contra Jira. Si no lo tienes conectado:

- **En Claude Code**: añade el MCP de Atlassian a tu configuración. Documentación: https://www.atlassian.com/platform/remote-mcp-server
- **En Copilot/Kiro**: la skill degrada con elegancia y genera el contenido (descripción, comentario, transición sugerida) en markdown listo para pegar a mano en Jira. No hay automatización pero el contenido sigue siendo útil.

## Cuándo se activa

Disparadores típicos:

- "actualiza el Jira GOT-431 con esta descripción mejor"
- "comenta en el GOT-431 que ya está mergeado"
- "transiciona GOT-431 a Code Review"
- "ponle el label `needs-qa` al GOT-431"
- "completa los criterios de aceptación del GOT-431"
- "el GOT-431 tiene la descripción muy floja, redáctala bien"

Casos que **no** son para esta skill (y la skill te lo dirá):

- "crea un issue en Jira para esto" → fuera de alcance. El desarrollador lo crea a mano; la skill le prepara el resumen y la descripción para pegar.
- "crea la subtask de DoD" → fuera de alcance. La crea el desarrollador a mano y le pega el contenido generado por la skill `generar-dod`.
- "enlaza el GOT-431 con el GOT-500" → fuera de alcance. Los links entre issues los gestiona el desarrollador.

## Instalación

### Claude Code (recomendado — formato nativo)

**User scope** — disponible en todos tus proyectos:

```bash
git clone https://github.com/<TU-USUARIO>/actualizar-jira.git ~/.claude/skills/actualizar-jira
```

**Project scope** — compartida con el equipo vía el repo del proyecto (se commitea):

```bash
# desde la raíz del repo del proyecto donde vayas a usarla
mkdir -p .claude/skills
git clone https://github.com/<TU-USUARIO>/actualizar-jira.git .claude/skills/actualizar-jira
```

Tras clonar, reinicia la sesión de Claude Code para que recoja la skill (`/doctor` ayuda a diagnosticar si no aparece). Asegúrate de tener el MCP de Atlassian conectado en tu configuración de Claude Code, si no la skill operará en modo degradado.

### GitHub Copilot

Copilot no tiene MCPs ni "skills" con activación condicional, así que la parte automatizada de la skill (llamadas a Jira) no aplica. Lo que sí puede hacer Copilot con estas instrucciones es **generar el contenido** (descripción rellena, comentario estructurado, comprobación de rama git) para que tú lo apliques a mano en Jira.

1. En el repo donde quieras que Copilot conozca las convenciones de Jira, crea o edita `.github/copilot-instructions.md`.
2. Copia el contenido del [`SKILL.md`](./SKILL.md) **sin el frontmatter YAML**.
3. Quita o adapta las menciones a tools del MCP (`getJiraIssue`, `editJiraIssue`, etc.) — en Copilot no aplican; cambia "lee con `getJiraIssue`" por "pide al usuario que pegue el contenido actual del issue".
4. Mantén tal cual:
   - La plantilla de descripciones de Story/Task/Bug.
   - El formato de comentarios estructurados.
   - El algoritmo de verificación/creación de rama git (Copilot sí puede ejecutar comandos git en el agent mode).
5. Commitea y empuja.

Considera el archivo de referencia [`references/adf.md`](./references/adf.md) como opcional para Copilot — solo es útil si vas a construir manualmente el JSON para enviar a la API de Jira (poco probable en Copilot).

### Kiro

```bash
# Workspace (solo este proyecto)
mkdir -p .kiro/steering
cp <ruta-a-este-repo>/SKILL.md .kiro/steering/actualizar-jira.md
cp <ruta-a-este-repo>/references/adf.md .kiro/steering/actualizar-jira-adf.md

# Global (todos tus workspaces)
mkdir -p ~/.kiro/steering
cp <ruta-a-este-repo>/SKILL.md ~/.kiro/steering/actualizar-jira.md
```

Kiro soporta MCPs, así que si conectas el MCP de Atlassian en tu configuración de Kiro la skill funcionará al 100%. Si no, opera en modo degradado igual que en Copilot.

**Si usas custom agents en Kiro**, añade `"file://.kiro/steering/**/*.md"` a `resources` del agente.

## Uso (ejemplos)

> El GOT-431 está en `Open` y la descripción está vacía. Es una Story sobre login con SAML. Rellénala y pásala a `In Progress`.

La skill:
1. Lee el issue actual con el MCP (si está disponible).
2. Detecta el tipo (`Story`) y aplica la plantilla correspondiente con secciones Contexto, Objetivo, Alcance, Criterios de aceptación, Notas técnicas y Referencias.
3. Confirma la edición.
4. Llama a `editJiraIssue` para actualizar la descripción.
5. Consulta transiciones disponibles con `getTransitionsForJiraIssue` (empareja por nombre, no por id) y mueve a `In Progress`.
6. **Comprueba la rama git local** y la alinea con el issue (ver abajo).

## Alineamiento de rama git

Solo se ejecuta tras acciones de **escritura** sobre el issue (editar, comentar un hito de trabajo, transicionar a un estado activo) — nunca tras una simple lectura. La skill mira la rama actual y decide:

| Rama actual | Comportamiento |
| --- | --- |
| Ya contiene el key del issue | No hace nada. |
| Rama base (`develop` / `main` / `master`) | Crea la rama nueva **sin pedir confirmación** y lo notifica. |
| Contiene **otro** key de Jira | **Pregunta primero** — puede que estés trabajando a propósito ahí. |
| Rama sin key y no base (`wip-pruebas`, `temp-debug`) | **Pregunta primero.** |

Usa `git checkout -b`, que arrastra los cambios sin commitear a la rama nueva — deliberadamente **no** usa `git stash`. Se detiene sin tocar nada si hay un merge/rebase/cherry-pick a medias o estás en detached HEAD, y nunca borra la rama anterior sin confirmación explícita.

## Convenciones que asume

- **Prefijo de Jira**: los ejemplos usan `GOT-`, pero la detección de rama se hace con el key real del issue que se está actualizando, así que funciona con cualquier prefijo de proyecto.
- **Estados de Jira**: nombres habituales del flujo del equipo (`In Progress`, `In Review`/`Code Review`, `Ready for QA`, `In QA`, `Done`). La skill consulta las transiciones disponibles vía MCP antes de actuar, así que aguanta variaciones — pero el mapping "momento del flujo → estado destino" está pensado para este flujo.
- **Ramas base del equipo**: `develop`, `main`, `master`.
- **Convención de rama**: `<tipo>/<KEY>-<slug>` donde `<tipo>` se deriva del `issueType` (Story/Task/Improvement/New Feature → `feature`, Bug → `bugfix`, Hotfix → `hotfix`, resto → `chore`) y `<slug>` sale del `summary` sin acentos, recortado a ~6 palabras o ~50 caracteres.
- **Descripciones y comentarios en ADF**: Jira Cloud no acepta Markdown; la skill convierte siguiendo [`references/adf.md`](./references/adf.md).
- **No crea issues, subtasks ni links entre issues**: decisión deliberada del equipo. Si lo quieres habilitar, edita la lista de tools permitidos en el SKILL.md.

## Estructura del repo

```
actualizar-jira/
├── SKILL.md              # La skill propiamente dicha (formato Claude Code)
├── references/
│   └── adf.md            # Conversión Markdown → ADF para la API de Jira
├── README.md             # Este archivo
└── .gitignore
```

## License

Internal use — Smowltech, S.L. No redistribuir fuera del equipo sin autorización.
