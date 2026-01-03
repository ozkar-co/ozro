# Notas de Scripting - Hercules Emulator

## Errores Comunes y Soluciones

### 1. Definición de Funciones

**Funciones DENTRO de un NPC:**
```c
function NombreFuncion {
    // código
}
```

**Funciones GLOBALES (fuera de NPCs):**
```c
function	script	NombreFuncion	{
    // código
}
```

❌ **INCORRECTO** (mezclar sintaxis):
```c
// Dentro de un NPC:
function script MiFuncion {  // ❌ NO usar 'script' aquí
```

✅ **CORRECTO**:
```c
// Dentro de un NPC:
function MiFuncion {
    callfunc("OtraFuncion");
}
```

**Importante**: 
- Funciones dentro de NPC: `function Nombre {`
- Funciones globales: `function<TAB>script<TAB>Nombre<TAB>{`
- **Declarar funciones internas al inicio**: `function Func1; function Func2;`

### 2. Sintaxis de Llamadas a Funciones

**Funciones DENTRO del mismo NPC:**
```c
// 1. Declarar al inicio del NPC
function MiFuncion; function OtraFuncion;

// 2. Llamar directamente (sin callfunc)
MiFuncion();
OtraFuncion("argumento");
```

**Funciones GLOBALES (externas):**
```c
callfunc("NombreGlobal");
callfunc("NombreGlobal", arg1, arg2);
```

❌ **INCORRECTO**:
```c
callfunc("MiFuncion");  // ❌ Si MiFuncion está en el mismo NPC
call MiFuncion;  // ❌ 'call' es solo para labels
```

✅ **CORRECTO**:
```c
MiFuncion();  // ✅ Función interna del NPC
callfunc("F_GlobalFunc");  // ✅ Función global externa
```

### 3. Límite de Nombres de Funciones
- **Máximo: 23 caracteres** para nombres de funciones y labels
- ❌ `RegisterCardsFromInventory` (28 chars)
- ✅ `RegCardsInv` (11 chars)
- **Tip**: Usa abreviaciones claras pero cortas

### 4. Asignación de Variables
- ❌ **SINTAXIS ANTIGUA**: `set .@var, valor;`
- ✅ **SINTAXIS MODERNA**: `.@var = valor;`
- Aplica para todos los tipos de variables: `.@temp`, `#account`, `.npc`, `$global`

### 5. Funciones en Concatenación de Strings
El parser NO permite usar `callfunc()` o `getarraysize()` directamente en concatenaciones.

❌ **INCORRECTO**:
```c
mes "Total: " + callfunc("GetCount") + " items";
```

✅ **CORRECTO**:
```c
.@count = callfunc("GetCount");
mes "Total: " + .@count + " items";
```

❌ **INCORRECTO**:
```c
mes "Size: " + getarraysize(.array) + " elementos";
```

✅ **CORRECTO**:
```c
.@size = getarraysize(.array);
mes "Size: " + .@size + " elementos";
```

### 5. Expresiones Lógicas Complejas
El parser tiene problemas con expresiones con múltiples `||` y `&&` anidados.

❌ **INCORRECTO**:
```c
if (.@x == "" || (strlen(.@x) > 1 && .@x != "A" && .@x != "B")) {
    mes "Error";
}
```

✅ **CORRECTO** (dividir en checks secuenciales):
```c
.@len = strlen(.@x);
if (.@x == "") {
    mes "Error";
    close;
}
if (.@len > 1) {
    if (.@x != "A" && .@x != "B") {
        mes "Error";
        close;
    }
}
```

### 6. Debug en OnInit
- ❌ `consolemes()` en `OnInit` causa error: "player not attached"
- ✅ Usa `debugmes()` para logs del servidor sin jugador
- `consolemes()` solo funciona cuando hay un jugador activo (RID attached)

### 7. Variables de Cuenta vs Temporales
- `#VARIABLE` - Account variable (persiste entre personajes)
- `.@variable` - Temporary variable (solo en scope actual)
- `.variable` - NPC variable (persiste en el NPC)
- `$VARIABLE` - Global server variable

### 8. Búsqueda en Strings
Para buscar en strings con delimitadores:
```c
// Formato: "|ID1|ID2|ID3|"
if (strpos(#CARD_FOUND$, "|" + .@id + "|") != -1) {
    // Carta encontrada
}
```
Los delimitadores evitan falsos positivos (ej: buscar "1" no coincide con "10").

### 9. Funciones de String - Nombres Correctos
- ❌ `strlen()` - NO EXISTE en Hercules
- ✅ `getstrlen()` - Función correcta para obtener longitud de string
```c
.@len = getstrlen(.@text$);
```

### 10. Arrays y Copyarray
**Sintaxis correcta**: `copyarray(destino[0], origen[0], tamaño)`

❌ **INCORRECTO**:
```c
copyarray .@temp, .carta_A;
```

✅ **CORRECTO**:
```c
copyarray(.@temp[0], .carta_A[0], getarraysize(.carta_A));
```

Útil para manipular datos sin modificar el array original. Requiere:
- Índice `[0]` en origen y destino
- Tamaño como tercer parámetro
- Paréntesis alrededor de todos los parámetros

### 11. Consideraciones de Performance
- **strpos()** es O(n) - Aceptable para ~500 búsquedas
- Si necesitas más de 1000 elementos, considera:
  - Múltiples variables por categoría
  - Sistema de bitfields
  - Arrays numéricos en lugar de strings

## Checklist Pre-Compilación
- [ ] Nombres de funciones ≤ 23 caracteres
- [ ] Usar `callfunc("Nombre")` para funciones
- [ ] Pre-evaluar todas las funciones antes de concatenar strings
- [ ] Simplificar expresiones lógicas complejas
- [ ] Usar sintaxis moderna: `var = valor` en lugar de `set var, valor`
- [ ] No usar `consolemes()` en OnInit sin jugador

## Mensajes de Error Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| `function not found` | Falta `script` en definición | Usar `function script Nombre` |
| `expect command, missing function name` | Usar `call` en vez de `callfunc` | Cambiar a `callfunc("Nombre")` |
| `label name longer than 23 chars` | Nombre de función muy largo | Acortar a ≤23 caracteres |
| `not enough arguments, expected ','` | Función inline en concatenación | Pre-evaluar en variable |
| `unmatched ')'` | Expresión lógica compleja | Dividir en checks simples |
| `need ';'` | Sintaxis `set var, value` | Cambiar a `var = value` |
| `player not attached` | `consolemes()` en OnInit | Usar `debugmes()` o remover |
| `need ';'` (con función) | Usar `strlen()` en vez de `getstrlen()` | Cambiar a `getstrlen()` |
| `not enough arguments, expected ','` | Sintaxis incorrecta de `copyarray` | Usar `copyarray(dest[0], src[0], size)` |

## Casos de Uso Exitosos

### 12. Sistema de Tracking de MVP (Enero 2026)
**Descripción**: Sistema global que registra kills de MVPs por jugador usando variables de personaje.

**Archivos creados**:
- `npc/custom/mvp/mvp_tracker.txt` - Script global con OnNPCKillEvent
- `npc/custom/mvp/mvp_journal.txt` - NPC para consultar estadísticas

**Técnicas aplicadas**:
1. **Variables dinámicas con getd/setd**: Se crearon variables con nombres dinámicos basados en el ID del MVP (`MVP_KILL_1115`, etc.) para rastrear kills individuales.
   ```c
   .@var_name$ = "MVP_KILL_" + .@mvp_id;
   .@current_count = getd(.@var_name$);
   setd .@var_name$, .@current_count + 1;
   ```

2. **OnNPCKillEvent**: Se usó el evento global para capturar todas las muertes de NPCs/monstruos.
   ```c
   -	script	MVP_Tracker	-1,{
   OnNPCKillEvent:
       if (strmobinfo(1, killedrid) != 1) end;
       // procesamiento...
   }
   ```

3. **Evitar funciones en concatenaciones**: Todas las llamadas a funciones (strmobinfo, getd, getarraysize) se evaluaron primero en variables antes de concatenar strings.
   ```c
   // ❌ INCORRECTO: mes "Killed: " + strmobinfo(2, .@id);
   // ✅ CORRECTO:
   .@name$ = strmobinfo(2, .@id);
   mes "Killed: " + .@name$;
   ```

4. **Loops con for en lugar de while**: Para mejor legibilidad y menos errores.
   ```c
   for (.@i = 0; .@i < .@array_size; .@i = .@i + 1) {
       // código
   }
   ```

**Notas importantes**:
- strmobinfo(1, id) retorna 1 si el mob es MVP, 0 si no lo es
- killedrid contiene el ID del mob asesinado en OnNPCKillEvent
- Las variables de personaje (sin prefijo especial) persisten entre sesiones
- bc_all en announce envía el mensaje a todos los jugadores online
- Importante: Al crear menús con arrays, asegurar que el tamaño del array coincida con las opciones del menú
- Los arrays de IDs deben estar libres de duplicados para evitar lookups incorrectos

**Correcciones aplicadas**:
- v1.1: Corregido array de IDs en menu de MVPs específicos (eliminados duplicados, ajustado tamaño)
- v1.1: Separada función CheckSpecificMVP para mejor organización del código

---
**Última actualización**: Enero 2026  
**Proyecto**: Hercules rAthena Custom NPCs
