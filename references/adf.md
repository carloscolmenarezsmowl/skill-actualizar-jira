# Conversión rápida Markdown → ADF

Jira Cloud almacena la descripción y los comentarios en **ADF** (Atlassian Document Format), no en Markdown. El MCP de Atlassian acepta ADF como un objeto JSON estructurado. Esta referencia recoge los patrones que cubren ~95% de lo que escribimos en el flujo del equipo.

Si el contenido es texto plano (sin formato), envuélvelo en el ADF mínimo de un párrafo.

## Documento mínimo

```json
{
  "version": 1,
  "type": "doc",
  "content": [
    { "type": "paragraph", "content": [ { "type": "text", "text": "Hola" } ] }
  ]
}
```

## Encabezados (`## Título`)

```json
{
  "type": "heading",
  "attrs": { "level": 2 },
  "content": [ { "type": "text", "text": "Título" } ]
}
```

`level` va de 1 a 6.

## Párrafo con énfasis

`Texto **negrita** y *cursiva* e `inline code``

```json
{
  "type": "paragraph",
  "content": [
    { "type": "text", "text": "Texto " },
    { "type": "text", "text": "negrita", "marks": [ { "type": "strong" } ] },
    { "type": "text", "text": " y " },
    { "type": "text", "text": "cursiva", "marks": [ { "type": "em" } ] },
    { "type": "text", "text": " e " },
    { "type": "text", "text": "inline code", "marks": [ { "type": "code" } ] }
  ]
}
```

## Bullet list

```json
{
  "type": "bulletList",
  "content": [
    { "type": "listItem", "content": [ { "type": "paragraph", "content": [ { "type": "text", "text": "Primer item" } ] } ] },
    { "type": "listItem", "content": [ { "type": "paragraph", "content": [ { "type": "text", "text": "Segundo item" } ] } ] }
  ]
}
```

## Numbered list

Idéntico a bulletList pero `"type": "orderedList"`.

## Checklist (taskList)

Útil para "Criterios de aceptación" y casos de QA.

```json
{
  "type": "taskList",
  "attrs": { "localId": "ac-list-1" },
  "content": [
    {
      "type": "taskItem",
      "attrs": { "localId": "ac-1", "state": "TODO" },
      "content": [ { "type": "text", "text": "Criterio observable y verificable" } ]
    },
    {
      "type": "taskItem",
      "attrs": { "localId": "ac-2", "state": "TODO" },
      "content": [ { "type": "text", "text": "Otro criterio" } ]
    }
  ]
}
```

`state` puede ser `"TODO"` o `"DONE"`. `localId` puede ser cualquier string único dentro del documento.

## Code block

```json
{
  "type": "codeBlock",
  "attrs": { "language": "bash" },
  "content": [ { "type": "text", "text": "git checkout -b feature/GOT-431" } ]
}
```

## Enlace

```json
{
  "type": "text",
  "text": "ver PR",
  "marks": [
    { "type": "link", "attrs": { "href": "https://github.com/org/repo/pull/431" } }
  ]
}
```

## Bloque de cita (panel "info" / "warning")

```json
{
  "type": "panel",
  "attrs": { "panelType": "info" },
  "content": [
    { "type": "paragraph", "content": [ { "type": "text", "text": "Nota importante" } ] }
  ]
}
```

`panelType` puede ser `info`, `note`, `warning`, `success`, `error`.

## Tip de conversión

Si la descripción es larga, **no escribas el ADF a mano**: parsea tu Markdown nodo a nodo o, si el MCP lo acepta, prueba primero a enviar el Markdown como texto plano dentro de un solo párrafo y, si Jira lo rechaza, escala a la conversión completa.

Para `editJiraIssue`, recuerda que estás sustituyendo el documento entero, no haciendo un diff: lee primero la descripción actual con `getJiraIssue` si quieres preservar contenido previo.
