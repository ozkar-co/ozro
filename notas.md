# Notas Técnicas - Proyecto Ozro

## Límites de Variables en Hercules/rAthena

### Problema Resuelto: Card Collector Variable Overflow (Enero 2026)

#### Descripción del Error
```
[Error]: script:set_reg: Value of variable #CARD_FOUND$ is too long: 260! Maximum is 255. Skipping...
```

#### Causa Raíz
El motor de Hercules/rAthena tiene un **límite de 255 caracteres** para variables permanentes de tipo string (`#VARIABLE$`). El script `npc/custom/cardcollector.txt` estaba intentando almacenar todos los IDs de cartas colectadas en una sola variable `#CARD_FOUND$`, lo cual excedía este límite.

#### Análisis Técnico

**Cálculo del espacio por carta:**
- ID de carta típico: 4 dígitos (ej: `4013`, `4121`)
- Separador: 1 carácter (`|`)
- **Total por carta: 5 caracteres**

**Categorías problemáticas:**
- BOSS: 67 cartas × 5 = **335 caracteres** ❌ (excede límite)
- MVP: 55 cartas × 5 = **275 caracteres** ❌ (excede límite)
- Categoría S (la más grande de letras): 44 cartas × 5 = **220 caracteres** ✓ (bajo límite)

#### Solución Implementada

**Estrategia:** División de almacenamiento por categorías

1. **Variables por categoría de letra** (A-Z): Una variable por letra
   - `#CARD_A$`, `#CARD_B$`, `#CARD_C$`, ..., `#CARD_Z$`
   - Ninguna excede 255 caracteres

2. **Variables divididas para categorías grandes:**
   - BOSS: `#CARD_BOSS1$` + `#CARD_BOSS2$`
   - MVP: `#CARD_MVP1$` + `#CARD_MVP2$`
   - **Umbral de división:** 200 caracteres (~40 cartas)

3. **Funciones helper creadas:**
   ```javascript
   GetCardCategory(item_id)        // Identifica categoría de una carta
   IsCardFound(item_id, category)  // Verifica si carta existe (maneja splits)
   AddCardToCategory(item_id, cat) // Añade carta a variable correcta
   ```

#### Lecciones Aprendidas y Reglas para el Futuro

### ⚠️ REGLAS CRÍTICAS PARA VARIABLES PERMANENTES

1. **Límite estricto de 255 caracteres para variables string permanentes (`#VAR$`)**
   - Planear siempre para el peor caso (máximo de elementos)
   - Calcular espacio: `(caracteres por elemento + separador) × cantidad máxima`

2. **Estrategia de división cuando sea necesario:**
   - Si una categoría puede exceder 200 caracteres, dividir en múltiples variables
   - Usar umbral conservador (200 chars) para dar margen de seguridad
   - Nombrar variables con sufijos: `VAR1$`, `VAR2$`, etc.

3. **Cálculo de capacidad máxima segura:**
   ```
   Caracteres por item = longitud_máxima_del_dato + longitud_separador
   Máximo items seguros = 250 / caracteres_por_item
   
   Ejemplo:
   - Card ID: 4 chars + separador 1 char = 5 chars/item
   - Máximo seguro: 250 / 5 = 50 cartas por variable
   ```

4. **Documentar límites en comentarios:**
   ```javascript
   // IMPORTANTE: Esta categoría puede almacenar máximo 50 cartas
   // para no exceder el límite de 255 caracteres
   // Actual: 67 cartas - REQUIERE SPLIT en múltiples variables
   setarray .carta_BOSS, ...;
   ```

5. **Testing preventivo:**
   - Probar con el máximo número de elementos posible
   - Simular casos extremos antes de deployment
   - Verificar longitud de strings antes de asignar a variables permanentes

### 📋 Checklist para Nuevos Scripts con Almacenamiento Persistente

Antes de implementar variables permanentes que almacenan múltiples elementos:

- [ ] ¿Cuál es el número MÁXIMO de elementos que se pueden almacenar?
- [ ] ¿Cuántos caracteres ocupa cada elemento (incluyendo separadores)?
- [ ] ¿El total excede 200 caracteres? → Implementar sistema de división
- [ ] ¿Hay funciones helper para manejar lectura/escritura distribuida?
- [ ] ¿Se documentó el límite y la estrategia en comentarios del código?
- [ ] ¿Se probó con el máximo número de elementos?

### 🔧 Herramientas de Depuración

**Verificar longitud de variable en runtime:**
```javascript
.@len = getstrlen(#CARD_FOUND$);
if (.@len > 200) {
    debugmes "WARNING: Variable approaching limit: " + .@len + "/255 chars";
}
```

**Script de análisis de categorías** (ver `/tmp/count_cards.sh` en historial):
- Cuenta elementos por categoría
- Calcula espacio requerido
- Identifica categorías que necesitan división

### 📚 Referencias

- **Archivo modificado:** `npc/custom/cardcollector.txt`
- **Versión:** 2.0 → 2.1
- **Commits:**
  - `d5cea0b`: Implementación de sistema de almacenamiento por categorías
  - `fcb8bd5`: Actualización de versión y documentación
- **Documentación Hercules:** [String Variables](https://github.com/HerculesWS/Hercules/wiki/Variables)

### 🎯 Otros Límites Importantes del Motor

- **Variables permanentes de cuenta (`##VAR`):** Mismo límite de 255 chars
- **Variables temporales (`.@var`):** Sin límite estricto, pero usar con moderación
- **Arrays:** Límite de 128 elementos por array en algunas versiones
- **Nombre de variables:** Máximo 32 caracteres

### 💡 Mejores Prácticas Generales

1. **Usar arrays cuando sea posible** en lugar de strings concatenados
2. **Normalizar longitud de datos** (usar IDs de longitud fija)
3. **Implementar migraciones** si cambias estructura de almacenamiento
4. **Logging preventivo** para detectar problemas antes de que ocurran
5. **Comentar decisiones técnicas** en el código para el futuro

## Estilo y Preferencias de Código

### Preferencias para NPCs

1. **No usar decoradores ni colores en texto de NPCs**
   - Evitar códigos de color como `^FF6B00`, `^00FF00`, etc.
   - Mantener texto simple y legible sin formato especial
   - El texto debe ser claro por sí mismo sin depender de colores

2. **Mantener NPCs simples**
   - Evitar loops infinitos (`while(1)`) en menús
   - Diseñar flujos lineales: mostrar menú → ejecutar acción → salir
   - Si el jugador quiere hacer otra acción, que vuelva a hablar con el NPC
   - Esto previene bugs de interacción y simplifica el código

3. **Uso correcto de `next;` y `close;`**
   - `next;` espera que el jugador haga clic para ver MÁS diálogo
   - `close;` termina la conversación con el NPC
   - **NUNCA** usar `next;` seguido de `return;` - esto bloquea al jugador
   - Toda función que muestra diálogo debe terminar con `close;` o retornar a un `close;`

### ⚠️ Problema Resuelto: Bloqueo de Diálogo en Card Collector (Enero 2026)

#### Síntomas
El NPC se bloqueaba al seleccionar "Register Cards" o "Check Progress", dejando al jugador esperando indefinidamente.

#### Causa Raíz
Funciones terminaban con `next;` seguido de `return;`, dejando el diálogo esperando más contenido que nunca llegaba.

**Código problemático:**
```javascript
function ShowProgress {
    mes "Progress data...";
    next;  // ❌ Espera más diálogo
    return; // ❌ Termina la función sin mostrar nada más
}
```

#### Solución
1. **Remover `next;` innecesarios** antes de `return;` en funciones
2. **Agregar `close;` terminal** después del switch para cerrar el diálogo

**Código correcto:**
```javascript
// En funciones
function ShowProgress {
    mes "Progress data...";
    return; // ✓ Retorna al menú principal
}

// En el menú principal
switch(select(...)) {
    case 1: ShowProgress(); break;
    case 2: ViewCards(); break;
}
close; // ✓ CRÍTICO: Cierra el diálogo después del switch
```

#### Regla General
- **Switch con funciones**: Siempre agregar `close;` después del switch
- **Funciones de diálogo**: Solo usar `next;` si hay más contenido que mostrar
- **Multi-página**: Usar `next;` entre páginas, NO antes del `return;` final

---

## Clarificaciones v4.0 (Enero 2026)

### Deprecación de `debugmes`
- `debugmes` está deprecado y puede ser removido.
- Para mensajes al jugador, usar `dispbottom`.
- Para logs de servidor, preferir herramientas del servidor o `logmes` según versión.

### Lectura de inventario (`getinventorylist`)
- Uso correcto:
   ```javascript
   getinventorylist;
   for (.@i = 0; .@i < @inventorylist_count; .@i++) {
         .@item_id = @inventorylist_id[.@i];
         // ...
   }
   ```
- No se llama `getinventorylist(.@i)`; se invoca sin argumentos y luego se iteran los arrays `@inventorylist_*`.

### Invocación de funciones de usuario
- Declaración de función:
   ```javascript
   function GetCardCategory {
         // ...
         return .@cat$;
   }
   ```
- Llamada correcta:
   ```javascript
   .@cat$ = callfunc("GetCardCategory", .@item_id);
   ```
- Evitar llamar como `GetCardCategory(.@item_id)` en asignaciones.
- Límite de nombre de función: ≤ 23 caracteres; mantener nombres cortos.

### Tabla de búsqueda con `setd/getd`
- Preparar datos en `OnInit` con `setd ".card_cat_<ID>$", "<CAT>"`.
- Consulta en runtime con `getd(".card_cat_" + .@id + "$")` (O(1)).
- Evita loops en tiempo de ejecución para resolución de categoría.

### Concatenación segura
- No mezclar llamadas a funciones dentro de concatenaciones de strings.
- Construir primero la cadena necesaria y luego usarla en `getd/setd`.

**Última actualización:** 10 Enero 2026
**Actualizado por:** @copilot
**Revisión necesaria:** Anual o al añadir nuevas categorías de cartas
