# Actividad 3: De una gramática libre de contexto a un analizador descendente LL en C
**Matrícula:** 10240049
**Nombre:** Daniel Hernández Martínez
**Grupo:** 10SC151
**Objetivo:** Construir un programa independiente en C que aplique una Gramática Libre de Contexto (GLC) para aceptar o rechazar secuencias de tokens mediante descenso recursivo (Parser LL).

## Gramáticas
**Gramática Base:**
E  -> T E'
E' -> + T E' | - T E' | epsilon
T  -> F T'
T' -> * F T' | / F T' | epsilon
F  -> ID | NUM | (E)

**Gramática Modificada:**
(Solo cambia F para admitir menos unario)
F  -> - F | ID | NUM | (E)

## Conceptos
* **Terminal:** Símbolos físicos que aparecen en la entrada (ID, NUM, +, -, *, /, (, ), $).
* **No Terminal:** Categorías abstractas definidas por la gramática (E, E', T, T', F).
* **Token:** La categoría abstracta reconocida por el lexer (ej. TOK_ID).
* **Lexema:** El texto real que conforma al token (ej. "id", "+").
* **Epsilon (ε):** Producción vacía que no consume tokens ni mueve el cursor.
* **Lookahead:** El token actual de anticipación que permite decidir qué regla de producción seguir.

## Preguntas de Razonamiento
**1. ¿Por qué E -> E + T | T produciría recursión infinita en un descenso recursivo directo?**
Porque la función `expresion` se llamaría a sí misma en la primera instrucción (para resolver la E de la derecha) sin haber consumido ningún token previo, creando un ciclo infinito (Stack Overflow) antes de poder avanzar el cursor en la memoria.

**2. Checkpoint 9, caso 1: ¿qué líneas demuestran que num * id se procesa dentro de T antes de regresar a E'?**
En la traza, justo después de `[consume] NUM "num"`, el parser llama a `T' -> * F T'` (consumiendo el POR y el ID) para completar toda la evaluación multiplicativa del término actual, resolviéndolo antes de que la regla externa `E' -> epsilon` dé por terminada la expresión global.

**3. Checkpoint 9, caso 2: ¿qué producción y qué línea de traza muestran que la suma interna se procesa antes de la multiplicación externa?**
Ocurre en la producción `F -> (E)`. La traza muestra que al procesar el paréntesis izquierdo, se llama a `E' -> + T E'` resolviendo completamente la suma interna, para luego consumir `)` mediante `[consume] PAR_DER ")"`, todo esto *antes* de que el `T'` externo llame a `T' -> * F T'` para la multiplicación.

**4. Checkpoints 5 y 6: ¿por qué epsilon no consume token ni mueve posicion y con qué lookahead puede elegirse?**
No consume token porque representa una producción vacía (el fin lógico de una cadena de reglas). En la gramática, epsilon se elige en `E'` si el lookahead es `)` o `$`, y en `T'` si el lookahead es `+`, `-`, `)` o `$`.

**5. Checkpoint 10, caso 5: indica posición, lookahead y función que detectan el error.**
En `pos=2`, el lookahead es `POR "*"`. La función `factor` es la encargada de detectar el error sintáctico, ya que esperaba obligatoriamente un ID, NUM o PAR_IZQ para continuar la expresión.

**6. Checkpoint 10, casos 7 y 8: ¿qué comprobación rechaza id) $ y cuál impide aceptar el prefijo id $ cuando todavía queda otro token?**
El caso 7 (`id) $`) lo rechaza la función `analizar` porque la comprobación `consumir(&p, TOK_FIN)` falla al toparse con un paréntesis en lugar de `$`. El caso 8 (`id $ id`) lo rechaza la comprobación `p.posicion != p.cantidad` al final de `analizar`, la cual asegura que no haya basura después de procesar el símbolo de fin de archivo.

**7. Modificación: ¿por qué - id * num $ es válido y - $ no lo es?**
Porque en el primer caso, la regla modificada `- F` se llama recursivamente y encuentra el token `ID`, resolviendo la gramática con éxito. En el segundo caso, la llamada a `- F` topa directamente con `$`, el cual no es un inicio de factor válido, detonando un error sintáctico por falta de operando.

**8. Cierre: ¿por qué aceptar una expresión no demuestra que sus identificadores existan ni que sus tipos sean compatibles?**
Porque el analizador sintáctico (Parser CFG) es ciego a la semántica; su única función matemática es garantizar que el orden y la jerarquía de los tokens formen una estructura válida (que un operador esté entre dos operandos), pero no consulta tablas de símbolos ni verifica reglas de tipos de datos.

## Conclusión
El analizador sintáctico LL demuestra cómo la estructura de una GLC puede traducirse directamente a funciones recursivas en C, utilizando la pila de llamadas del sistema operativo para simular el árbol de derivación. Sin embargo, este análisis queda estrictamente limitado a la validación geométrica de los tokens, dejando la resolución matemática, validación de variables (si existen o no) y compatibilidad de tipos fuera de su alcance, reservándolas para la fase de análisis semántico.

## URL del repositorio
https://github.com/dan-1hm/actividad03_parser_ll.git
 
  
   
    
