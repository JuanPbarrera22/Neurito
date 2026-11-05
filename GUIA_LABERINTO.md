# Neurito 3.0: aprende a encontrar comida en un laberinto

Neurito busca comida dentro de tres laberintos. Su cerebro debe elegir entre arriba, abajo, izquierda y derecha. La red se construye con NumPy, y Pygame dibuja el escenario, la mascota y una gráfica de la pérdida.

## Cómo aprende

1. BFS calcula rutas cortas y genera ejemplos etiquetados: mapa, posición, meta y movimiento recomendado.
2. Los ejemplos se dividen en entrenamiento (80 %) y prueba (20 %). Los pares posición-meta de prueba no aparecen en entrenamiento.
3. Cada entrada incluye un vector one-hot de tres mapas y dos vectores one-hot de 81 casillas: 165 características.
4. La arquitectura es 165 → 96 → 4. Usa ReLU en la capa oculta y Softmax en las cuatro salidas.
5. El entrenamiento minimiza entropía cruzada multiclase mediante retropropagación y descenso de gradiente con mini-lotes.
6. Durante el juego, el modelo decide sin consultar BFS. BFS es el experto que proporcionó las etiquetas.

Esto es aprendizaje supervisado por imitación, NO aprendizaje por refuerzo.

## Dos modos

- Red pura: toma la salida de mayor probabilidad. Puede chocar con paredes o quedarse en ciclos.
- Asistido: descarta movimientos contra paredes y penaliza casillas visitadas. Son reglas adicionales, no conocimientos aprendidos por la red.

La precisión de clasificación no equivale al porcentaje de laberintos completados.

## Estructura

- laberinto_mundo.py: mapas, movimiento, BFS y navegación asistida.
- laberinto_red.py: red neuronal y entrenamiento con NumPy.
- laberinto.py: juego Pygame, sprites y pérdida.
- test_laberinto.py: pruebas automáticas.
- GUIA_LABERINTO.md: esta guía.

Reutiliza assets/neutral_1.png, neutral_2.png y neutral_3.png. No modifica las versiones anteriores.

## Ejecutar

    uv sync
    uv run laberinto.py

## Controles

| Tecla | Acción |
|---|---|
| 1 | Entrenar una época |
| 2 | Entrenar cien épocas |
| 3 | Entrenamiento continuo |
| A | Alternar red pura y navegación asistida |
| P | Alternar navegación automática y paso a paso |
| Espacio | Avanzar un paso en modo manual |
| N | Cambiar mapa |
| G | Nuevas posiciones y meta |
| C | Reiniciar contadores |
| R | Reiniciar pesos y entrenamiento |
| Esc | Salir |

## Probar automáticamente

    uv run python -m unittest -v test_laberinto.py

## Experimentos

1. Observa Neurito sin entrenar. ¿Por qué se atasca?
2. Entrena 100, 200 y 400 épocas. Compara pérdida, precisión y movimientos.
3. Compara red pura frente a navegación asistida. ¿Qué parte corresponde al modelo y cuál a las reglas?
4. Cambia las 96 neuronas ocultas por 16, 32 o 128.
5. Cambia la tasa de aprendizaje 0.45 y observa si converge.
6. Cambia de mapa. Los mapas forman parte del entrenamiento, pero las combinaciones de inicio y meta usadas para probar no se entrenaron.
7. Haz vibecoding para verificar como mejora o empeora Neurito al cambiar DFS, por BFS y A*

## Alcance y limitaciones

El modelo no interpreta capturas de pantalla: trabaja con características estructuradas. Conoce el identificador de cada mapa. Los tres mapas pertenecen al mismo universo de entrenamiento. La evaluación demuestra generalización a pares de posición-meta no entrenados, no a laberintos totalmente desconocidos. Para eso habría que representar también las paredes de cualquier mapa nuevo y entrenar y evaluar en mapas separados.

# Resultados de los experimentos

Para medir qué tan bien funciona Neurito usé los 716 retos de prueba, es decir, combinaciones de inicio y meta que la red nunca vio en el entrenamiento. En cada reto dejé que Neurito diera hasta 45 movimientos, igual que en el juego, y conté si llegaba a la comida. También comparé cuántos pasos dio contra el camino más corto.

## 1. Neurito sin entrenar

Sin entrenar, la exactitud fue de 27.7 %, prácticamente lo mismo que adivinar entre cuatro direcciones. En modo red pura solo llegó a la comida en el 3.1 % de los retos, y en promedio chocó 36 veces contra una pared de sus 45 intentos.

Se atasca porque la red es determinista: con la misma entrada siempre da la misma salida. Si la dirección con más probabilidad apunta a una pared, Neurito no se mueve, la entrada no cambia y vuelve a elegir la misma pared una y otra vez. Cuando sí logra moverse, a veces cae en dos casillas que se apuntan entre sí y se queda yendo y viniendo entre ellas para siempre.

## 2. Entrenar 100, 200 y 400 épocas

Con 100 épocas la pérdida bajó a 0.135, la exactitud de prueba subió a 84.9 % y Neurito completó el 51.3 % de los retos en red pura. Con 200 épocas la pérdida llegó a 0.038, la exactitud a 90.1 % y completó el 81.3 %. Con 400 épocas la pérdida quedó en 0.011, la exactitud en 91.5 % y completó el 82.7 %.

Cuando llega, lo hace casi por el camino más corto (solo un 1 % de pasos de más), porque imita a BFS. De 200 a 400 épocas la pérdida siguió bajando mucho, pero los retos completados casi no cambiaron. A las 400 épocas la red ya acierta el 100 % de los ejemplos de entrenamiento y el 91.5 % de los de prueba. Esa diferencia muestra que una parte de lo que aprende es de memoria y no se traslada del todo a retos nuevos.

Otra cosa importante es que con 91.5 % de exactitud solo completa el 82.7 % de los laberintos. Un reto necesita muchos movimientos seguidos, y basta con equivocarse en uno para desviarse o quedarse en un ciclo.

## 3. Red pura frente a navegación asistida

Con 400 épocas, la red pura completó el 82.7 % de los retos y la navegación asistida el 100 %. Pero la asistencia no es inteligencia de la red. Incluso sin entrenar, el modo asistido completó el 87.8 % de los retos, aunque dando unas cuatro veces más pasos de los necesarios. Esto pasa porque las reglas ya evitan las paredes y castigan volver a casillas visitadas, así que Neurito termina recorriendo el laberinto casi a ciegas hasta encontrar la comida.

La parte que corresponde al modelo es la de elegir la dirección correcta hacia la comida, y eso se nota en que con la red entrenada el modo asistido solo da un 19 % de pasos de más. La parte que corresponde a las reglas es no chocar con paredes y no repetir casillas. En red pura, de los retos que fallaron, casi todos fueron por ciclos (16.9 %) y muy pocos por chocar con paredes (0.4 %). Es justo lo que la asistencia corrige.

## 4. Cambiar el número de neuronas ocultas

Entrené 400 épocas con cada tamaño. Con 16 neuronas la exactitud de prueba fue de 87.2 % y completó el 74.3 % de los retos. Con 32 neuronas llegó a 89.2 % y 78.4 %. Con 96, la original, a 91.5 % y 82.7 %. Con 128 a 90.8 % y 85.3 %.

Con pocas neuronas la red tiene menos capacidad para aprender todas las combinaciones de mapa, posición y meta. A partir de 96 la mejora es pequeña: con 128 completó algunos retos más, pero la exactitud de prueba fue incluso un poco menor. En los cuatro casos la exactitud de entrenamiento fue de casi 100 %, así que la diferencia está en cuánto generaliza cada una a retos nuevos.

## 5. Cambiar la tasa de aprendizaje

Con 0.05 aprende muy lento. A las 400 épocas la pérdida seguía en 0.28 y solo completaba el 46.1 % de los retos.

Con 0.45, la original, converge bien: pérdida de 0.011 y 82.7 % de retos.

Con 1.0 y con 2.0 converge todavía más rápido. Con 2.0, por ejemplo, en solo 100 épocas ya tenía 92.5 % de exactitud. A las 400 épocas las dos llegaron a 93.3 % de exactitud y alrededor de 85.5 % de retos, un poco mejor que la original. Funcionan porque la red se entrena por mini-lotes de 256 ejemplos y la entrada one-hot solo activa unas pocas neuronas por ejemplo.

Con 5.0 no converge. La pérdida se quedó en 1.39 desde el principio, que es lo mismo que no saber nada entre cuatro opciones, y la exactitud bajó a 23.6 %. Los pasos son tan grandes que las neuronas ocultas se "apagan" (ReLU da cero para todo) y la red deja de aprender. Siempre responde lo mismo y en red pura solo completó el 4.5 % de los retos.

## 6. Cambiar de mapa

Con la red entrenada 400 épocas, en el mapa 1 la exactitud de prueba fue de 92.4 % y completó el 85.7 % de los retos en red pura. En el mapa 2 fue de 90.0 % y 78.5 %. En el mapa 3 fue de 92.2 % y 84.3 %. Con asistencia completó el 100 % en los tres.

El mapa 2 es el más difícil, porque tiene más pasillos que se parecen entre sí y más bifurcaciones, así que es más fácil equivocarse en una y quedar en un ciclo. Aun así, los resultados son parecidos en los tres mapas. Es lo esperado, porque los tres mapas se usaron en el entrenamiento: lo único nuevo en la prueba son las combinaciones de inicio y meta. Esto no demuestra que Neurito funcione en un laberinto que nunca vio.

## 7. Cambiar el maestro: DFS, BFS y A*

Hice un script aparte que genera las etiquetas con tres maestros distintos y entrené una red de 400 épocas con cada uno.

Con DFS la red aprendió las etiquetas casi perfecto, con 99.9 % de exactitud de prueba, pero en el juego solo completó el 7.5 % de los retos en red pura y el 44.8 % con asistencia. El problema no es la red sino el maestro. DFS no busca el camino más corto: se mete por el primer pasillo que encuentra hasta el fondo. Además, la red solo ve la posición actual, así que cada etiqueta sale de un DFS distinto, empezando en esa casilla. Desde una casilla DFS puede decir "derecha" y desde la casilla de la derecha decir "izquierda", y Neurito se queda yendo y viniendo. Incluso el propio maestro DFS, seguido paso a paso así, solo llegó a la comida en el 7.7 % de los retos.

Con BFS, el original, la red tuvo 91.5 % de exactitud, completó el 82.7 % de los retos en red pura y el 100 % con asistencia.

Con A* la red tuvo 92.6 % de exactitud, completó el 86.3 % de los retos en red pura y el 100 % con asistencia, siendo el que menos pasos de más dio (16 %). A* también encuentra el camino más corto, igual que BFS, y por eso el 96.4 % de las etiquetas son iguales. La diferencia está en cómo desempata cuando hay dos caminos igual de cortos: A* prefiere el que se acerca a la comida en línea recta, y eso hace que las etiquetas sean más fáciles de aprender para la red.

En conclusión, BFS y A* son buenos maestros porque dan caminos cortos y consistentes, y A* es un poco mejor. DFS empeora mucho a Neurito, aunque la red lo imite casi perfecto. Esto muestra que en el aprendizaje por imitación la red solo puede ser tan buena como el maestro que copia.
