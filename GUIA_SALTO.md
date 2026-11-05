# Neurito 2.0: aprende a saltar obstáculos

Neurito aprende **cuándo saltar** ante obstáculos que se aproximan con diferentes velocidades. El cerebro se implementa manualmente con NumPy y el aprendizaje puede observarse en Pygame.

## ¿Qué aprende?

La red recibe tres características normalizadas: distancia al obstáculo, velocidad y tiempo estimado hasta alcanzarlo (distancia / velocidad). La última es una característica derivada, no una observación independiente.

La arquitectura es **3 → 12 → 1**: doce neuronas ocultas con ReLU y una salida con sigmoide. La última devuelve una puntuación entre cero y uno que, con el umbral 0.5, determina si Neurito debe saltar cuando está en el suelo.

## ¿Cómo aprende?

Aplicamos **aprendizaje supervisado**, no aprendizaje por refuerzo. Creamos datos sintéticos que incluyen ejemplos donde es conveniente saltar y otros donde debe esperar. Para generar las etiquetas, definimos una ventana de referencia: saltar cuando el tiempo estimado está entre **7 y 13 fotogramas**. No es una regla aprendida por recompensa ni garantiza evitar choques en todas las situaciones.

La red ajusta sus pesos a partir de la **entropía cruzada binaria**, la **retropropagación** y el **descenso de gradiente**:

`W_nuevo = W_anterior - tasa_de_aprendizaje * gradiente`

El conjunto de entrenamiento equilibra aproximadamente las dos decisiones. De otra manera, una red podría aparentar buena precisión diciendo casi siempre «esperar».

La precisión se mide sobre ejemplos distintos de los utilizados en el entrenamiento. **Una precisión alta no garantiza un buen resultado en el juego**, que también depende de la física del salto.

## Archivos

```text
Neurito/
├── main.py                 # Mascota original: comer, dormir, jugar
├── red-sencilla.py         # Ejemplo de una neurona
├── salto_modelo.py         # Red neuronal manual y datos de entrenamiento
├── salto_juego.py          # Física y detección de colisiones
├── saltar.py               # Juego y visualización del entrenamiento
├── GUIA_SALTO.md
└── assets/
    └── jugar_1.png        # Reutilizado en el nuevo juego
```

## Ejecución

```bash
uv sync
uv run saltar.py
```

No se necesitan dependencias nuevas: se reutilizan NumPy y Pygame.

## Controles

| Tecla | Función |
|---|---|
| `1` | Entrenar una época |
| `2` | Entrenar cien épocas |
| `3` | Activar o pausar entrenamiento continuo |
| `A` | Alternar control automático y manual |
| `Espacio` | Saltar manualmente |
| `C` | Reiniciar pista y contadores |
| `R` | Reiniciar pesos y contadores |
| `Esc` | Salir |

## Experimentos

1. Prueba el juego sin entrenar. Anota colisiones y obstáculos esquivados.
2. Entrena de cien en cien épocas; compara la pérdida, la precisión y el comportamiento.
3. Cambia `ocultas=12` por `ocultas=3` o `ocultas=24`; compara generalización.
4. Modifica la tasa de aprendizaje `0.8` a `0.01` o `1.2` y estudia la convergencia.
5. Cambia la ventana de entrenamiento en `etiqueta()` y en la generación de positivos. Comprueba si la política resultante es compatible con la física.

**Pregunta central:** ¿por qué una red puede clasificar ejemplos con buena precisión y aun así chocar?

# Resultados de los experimentos

Para cada prueba dejé el juego corriendo en modo automático unos 100 segundos (3000 fotogramas) y anoté cuántos obstáculos esquivó Neurito y cuántas veces chocó. La precisión la medí con ejemplos de prueba que la red no vio durante el entrenamiento.

## 1. Juego sin entrenar

Sin entrenar, Neurito no saltó ni una vez. Esquivó 0 obstáculos y chocó 20 veces. La precisión fue de 44.9 %, menos que lanzar una moneda. Esto pasa porque los pesos empiezan al azar y la probabilidad de salto nunca llega al 50 %, así que la red casi siempre responde "esperar" y solo acierta en los ejemplos donde no había que saltar.

## 2. Entrenar de cien en cien épocas

Con 100 épocas la pérdida bajó a 0.26 y la precisión subió a 91.2 %. Neurito ya esquivó 22 obstáculos y solo chocó una vez. Desde las 200 épocas ya no volvió a chocar: esquivó los 24 obstáculos que aparecieron.

Seguí entrenando y la pérdida siguió bajando (0.21 en 300 épocas, 0.14 en 500 y 0.06 en 1000), mientras que la precisión llegó a 97.8 %. Pero en el juego ya no se notó ninguna diferencia. Lo que mejora con más épocas son los ejemplos que están justo en el borde de la ventana, y esos casi no afectan a si Neurito esquiva o no.

## 3. Cambiar el número de neuronas ocultas

Con 3 neuronas la red se estancó en 93.7 % de precisión y la pérdida no bajó de 0.20, aunque igual logró esquivar todo. Le falta capacidad para marcar bien los dos bordes de la ventana.

Con 12 neuronas llegó a 98.8 % y con 24 a 98.7 %. Con 24 aprende un poco más rápido al principio, pero al final queda igual que con 12, así que agregar más neuronas no ayuda.

En los tres casos la precisión con los datos de entrenamiento y con los de prueba fue casi la misma, así que la red generaliza bien y no se aprende de memoria los ejemplos.

## 4. Cambiar la tasa de aprendizaje

Con 0.01 la red aprende muy lento. Después de 2000 épocas la pérdida seguía en 0.35 y la precisión en 87.5 %, y en el juego chocó siempre. La red todavía no distingue bien entre "el obstáculo está lejos" y "es el momento de saltar", así que salta demasiado pronto y cae antes de llegar al obstáculo. Lo curioso es que con 100 épocas esquivó 18 obstáculos, pero fue de casualidad, porque estaba indecisa y sus probabilidades estaban todas cerca de 0.5.

Con 0.8, la tasa original, la red converge bien y no choca.

Con 1.2 converge un poco más rápido que con 0.8 y llega a 99.1 % sin volverse inestable. En este problema todavía es una tasa segura. Si se subiera mucho más, los pasos serían tan grandes que la pérdida empezaría a oscilar en vez de bajar.

## 5. Cambiar la ventana de entrenamiento

Primero revisé la física del salto. El salto dura unos 37 fotogramas y sube hasta 110 píxeles. Probando, vi que un salto esquiva el obstáculo si se hace entre 4 y 14.5 fotogramas antes cuando el obstáculo va lento (4.5 px por fotograma), y entre 4 y 22.5 fotogramas antes cuando va rápido (8.5 px por fotograma).

Después cambié la ventana y entrené 2000 épocas con cada una. Con la ventana original de 7 a 13, y también con las de 2 a 5 y de 4 a 9, Neurito esquivó todo. Con la ventana de 15 a 22 y con la de 25 a 35 chocó siempre.

La explicación es que el obstáculo se va acercando, así que Neurito salta apenas entra en la ventana, o sea, en su límite superior. Si ese límite queda por debajo de 14.5 fotogramas, el salto funciona con cualquier velocidad. Si es mayor, Neurito salta demasiado pronto y cae antes de pasar el obstáculo.

Lo más interesante es que la ventana de 25 a 35 tuvo la mejor precisión de todas, 99.7 %, y aun así fue la peor en el juego. La red aprendió muy bien una regla que no tiene sentido con la física.

## Respuesta a la pregunta central

Una red puede tener buena precisión y aun así chocar porque la precisión solo mide qué tan bien copia la regla con la que se hicieron las etiquetas, no si esa regla sirve en el juego. La red no sabe nada de gravedad ni de alturas, solo imita la ventana que le damos. Si la ventana está mal elegida, como la de 25 a 35, la red la aprende casi perfecto y aun así choca con todos los obstáculos.

Además, la precisión cuenta todos los errores igual, pero en el juego no pesan lo mismo. Equivocarse cuando el obstáculo está muy lejos no pasa nada, pero equivocarse justo cuando hay que saltar provoca un choque. Por eso con la tasa de 0.01 la red tenía 87.5 % de precisión y no esquivó ningún obstáculo: sus errores estaban justo en el momento importante.

En conclusión, que la red tenga buena precisión significa que aprendió bien la regla, pero para que Neurito no choque también hace falta que la regla esté bien pensada para la física del juego.
