# actualizar-jira

Skill del equipo de Smowltech que **actualiza** issues de Jira ya existentes siguiendo las convenciones internas: rellenar descripciones con plantilla, comentar hitos, transicionar estados, ajustar labels. Tras la actualización, comprueba que la rama git local contiene el ID del issue y, si no, **crea automáticamente la rama correcta preservando los cambios en curso**.

**No crea issues ni subtasks.** La creación queda como responsabilidad del desarrollador, que conoce mejor el proyecto destino, tipo de issue y campos custom requeridos. La skill se centra exclusivamente en escribir sobre tickets ya creados.

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

Casos que **no** son para esta skill (y la skill te lo dirá):

- "crea un issue en Jira para esto" → fuera de alcance. El desarrollador lo crea a mano.
- "crea la subtask de DoD" → fuera de alcance. La crea el desarrollador a mano y le pega el contenido generado por la skill `generar-dod`.

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

Tras clonar, en una sesión de Claude Code ejecuta `/reload-plugins`. Asegúrate de tener el MCP de Atlassian conectado en tu configuración de Claude Code, si no la skill operará en modo degradado.

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
5. Consulta transiciones disponibles y mueve a `In Progress`.
6. **Comprueba la rama git local**. Si estás en `dev` o en una rama sin `GOT-431`, crea automáticamente `feature/GOT-431-permitir-login-con-saml` con `git checkout -b`, preservando los cambios sin commitear.

## Convenciones que asume

- **Prefijo de Jira**: `GOT-` (hardcodeado en regex de detección de rama). Cambia para otros proyectos.
- **Estados de Jira**: nombres habituales del flujo del equipo (`In Progress`, `Code Review`, `Ready for QA`, `In QA`, `Done`). La skill consulta las transiciones disponibles vía MCP antes de actuar, así que aguanta variaciones — pero el mapping "momento del flujo → estado destino" está pensado para este flujo.
- **Convención de rama**: `<tipo>/<KEY>-<slug>` donde `<tipo>` se deriva del `issueType` (Story/Task/Improvement → `feature`, Bug → `bugfix`, Hotfix → `hotfix`, resto → `chore`).
- **No crea issues ni subtasks**: decisión deliberada del equipo. Si lo quieres habilitar, edita la lista de tools permitidos en el SKILL.md.

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
# skill-actualizar-jira
