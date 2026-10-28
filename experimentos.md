# Experimentos con Neurito

Neurito es una red neuronal pequeña. Recibe tres necesidades (hambre, energía y aburrimiento) y decide si debe comer, dormir o jugar. Con la configuración original (6 neuronas ocultas, 1200 épocas y tasa de aprendizaje 0.1) acierta el 99.4 % de las veces.

## 1. Variar una sola necesidad

Dejé fijas dos necesidades y fui subiendo la tercera. Con hambre y energía en 0.5, Neurito elige comer o dormir mientras el aburrimiento está bajo. Cuando el aburrimiento pasa de 0.5, cambia a jugar.

La acción cambia cuando la necesidad que estoy subiendo se vuelve la más grande de las tres.

## 2. Prioridades casi iguales

Cuando dos o tres necesidades tienen casi el mismo valor, Softmax reparte las probabilidades casi en partes iguales. Por ejemplo, con las tres necesidades en 0.5, el resultado fue 35 % comer, 33 % dormir y 31 % jugar. En esos casos Neurito está indeciso y la decisión se define por muy poco.

## 3. Cambiar el número de neuronas

- 3 neuronas: 96.3 % de precisión. Le falta capacidad para aprender bien.
- 6 neuronas: 99.4 %. Es el mejor resultado.
- 10 neuronas: 99.3 %. No mejora.
- 20 neuronas: 98.8 %. Tampoco mejora.

El problema es sencillo, así que 6 neuronas son suficientes. Agregar más no ayuda.

## 4. Cambiar épocas y tasa de aprendizaje

Con más épocas el entrenamiento mejora: con 100 épocas acierta el 77 % y con 1200 el 99 %. Pasado cierto punto, la mejora es muy pequeña.

Con la tasa de aprendizaje pasa esto:

- 0.01 (muy baja): aprende muy lento y solo llega al 83 %.
- 0.1 (normal): aprende bien y llega al 99 %.
- 5 o 20 (muy alta): el entrenamiento se vuelve inestable y la red se rompe. Siempre responde lo mismo y solo acierta el 34 %.

## 5. Agregar una cuarta acción

Para agregar una acción nueva, por ejemplo bañarse, habría que:

- Agregar una cuarta necesidad a los datos, por ejemplo suciedad.
- Agregar la etiqueta "bañarse" a las respuestas correctas.
- Darle a la red una cuarta salida.
- Tener suficientes ejemplos de las cuatro acciones para entrenarla.

# Explicación de los resultados

## ¿En qué se diferencian pesos y sesgos?

Los pesos indican cuánto importa cada entrada. Los sesgos son un valor que se suma siempre, sin importar la entrada, y ayudan a ajustar cuándo se activa una neurona. En la recta y = w·x + b, el peso sería la pendiente y el sesgo el punto de partida.

## ¿Por qué el cansancio es 1 - energía?

Porque las tres entradas deben medir lo mismo: cuánta necesidad hay. Hambre alto y aburrimiento alto significan que Neurito necesita algo. Energía alta significa lo contrario: que no necesita dormir. Al usar 1 - energía, se convierte en cansancio, y así un valor más alto siempre significa una necesidad más urgente.

## ¿Qué hace ReLU y qué hace Softmax?

ReLU deja pasar los números positivos y convierte los negativos en cero. Esto permite que la red aprenda decisiones más complejas.

Softmax convierte los resultados finales en porcentajes que suman 100 %. Así se puede ver qué tan seguro está Neurito de cada acción.

## ¿Por qué una precisión alta puede ser engañosa?

Porque la precisión solo dice cuántas veces acertó, no qué tan seguro estaba. En los empates Neurito puede acertar por casualidad. Además, si casi todos los ejemplos fueran de una sola acción, una red que siempre responde esa acción tendría una precisión alta sin haber aprendido nada. Por último, el 99 % solo indica que Neurito copia bien la regla con la que se generaron los datos, no que sus decisiones sean buenas en cualquier situación.
