![Hercules NPC Syntax Highlighting Extension](https://img.shields.io/badge/Hercules-NPC%20Syntax-blue)

# Hercules NPC Script Syntax Highlighting

Proporciona resalte de sintaxis para archivos NPC de Hercules/Athena en VS Code.

## Características

✅ **Comandos destacados**: `mes`, `next`, `close`, `getd`, `setd`, `setarray`, etc.  
✅ **Strings con comillas**: Texto entre comillas resaltado en verde  
✅ **Variables de color**: `#variable`, `@variable`, `.@variable` resaltadas  
✅ **Comentarios**: `//` y `/* */` con colores apropiados  
✅ **Números**: Constantes numéricas resaltadas  
✅ **Control de flujo**: `if`, `else`, `for`, `switch`, `case` destacados  
✅ **Eventos NPC**: `OnInit`, `OnTalk`, `OnTouch`, etc. resaltados  

## Instalación

Esta extensión está en `.vscode/extensions/hercules-npc-syntax/` y se carga automáticamente en este workspace.

## Uso

Simplemente abre archivos `.txt` de NPCs y verás el resalte de sintaxis automáticamente.

## Personalización

Para personalizar los colores, añade reglas en tu `settings.json`:

```json
"editor.tokenColorCustomizations": {
  "comments": "#90EE90",
  "[Tu Theme]": {
    "comments": "#76D776"
  }
}
```

## Comandos soportados

- Funciones: `mes`, `next`, `close`, `callsub`, `return`
- Variables: `getd`, `setd`, `setarray`
- Inventario: `getinventorylist`, `getiteminfo`, `getitemname`
- SQL: `query_sql`
- Arrays: `getarraysize`
- Strings: `strtoupper`, `substr`, `atoi`
- Control: `select`, `input`, `switch`, `case`
- Debugging: `consolemes`
- Información: `getcharid`
