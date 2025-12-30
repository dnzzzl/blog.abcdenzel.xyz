+++
date = '2025-11-26T11:26:35-04:00'
draft = false
title = 'Detección de Anomalías en Tráfico de Red'
tags = ['AI', 'python', 'algorithms & data structures']
categories = ['tech', 'writeup']
+++

Configurar un laboratorio para Análisis de Tráfico de Red es uno de esos proyectos que ha estado flotando en mi lista de cosas por hacer. Se siente como una mirada tan útil y genuinamente perspicaz a las conexiones con las que una computadora interactúa a lo largo de cada día.

Ser capaz de registrar, analizar, generar alertas y subsecuentemente automatizar análisis e incluso tomar acción proactiva informada en un sistema basado en los datos; estas son habilidades que se sienten infinitamente valiosas para cualquier sistema expuesto al Internet o una Red de Área Local.

Así que finalmente tuve que abordar este desafío.

Me propuse leer, entender y aprender tanto como fuera necesario sobre detección de anomalías, para implementar un algoritmo simple basado en árboles binarios bajo una restricción de tiempo sensata de dos días.

Esto me permite meter los pies en un objetivo más amplio que tengo que es tender puentes entre IA y ciberseguridad usando un enfoque basado en proyectos prácticos.

En esta entrada de blog te guío a través de mi proceso desde cero hasta la implementación de un enfoque de Árboles de Aislamiento (Isolation Tree) para Detección de Anomalías.

## Prerrequisitos

Estos son temas de conocimiento fundamental que necesitan estar solidificados antes de que tentemos nuestra suerte con una implementación.
Y por supuesto, antes de tener un bosque, necesitamos un árbol:

### Árboles binarios

Los árboles binarios son [estructuras de datos]() que organizan datos de una manera jerárquica y traversable.
Los árboles están compuestos de niveles, también llamados profundidad, en cada nivel encontramos nodos que se ramifican en otros dos nodos un nivel por debajo de él, de manera que el nodo superior es la raíz y se ramifica en otros dos nodos abajo.

La relación entre estos nodos es la lógica detrás de la estructura de datos, representa un requisito de que para alcanzar un nodo abajo, se necesita seguir un camino usando las ramas correctas.

En el contexto de Isolation Forest, esto se vuelve perspicaz: la longitud del camino en sí nos dice cuántas decisiones fueron requeridas para aislar un punto.

Para recorrer el árbol binario, un algoritmo necesita evaluar el nodo actual y los nodos debajo de él, tomar una decisión binaria que lo acerque a su destino y, en caso de fallo, ser capaz de retroceder y elegir un camino diferente en un punto de divergencia.

Sin embargo, el recorrido de caminos binarios no es nuestro problema a resolver hoy, la forma en que usaremos los árboles binarios es usando particionamiento recursivo aleatorio para determinar los puntos anómalos. Este particionamiento hace lo que dice en la lata: El nodo raíz es la muestra completa del conjunto de datos, después de que se selecciona una característica aleatoria, los datos se dividen entre los valores mínimo y máximo, y este proceso se repite recursivamente formando una estructura de árbol. Esto sucede hasta que cada rama tiene un nodo o se alcanza una profundidad máxima preconfigurada.

## Aprendizaje

Ahora que entendemos cómo los árboles binarios crean caminos de aislamiento, veamos cómo Isolation Forest usa bosques de estos árboles para detectar anomalías en teoría.

### Algoritmo de Isolation Forest

El enfoque de Isolation Forest para Detección de Anomalías difiere de métodos anteriores que dependían de perfilar la actividad normal vía análisis estadístico para luego identificar valores atípicos, esos métodos producían mayor tasa de falsos positivos, tenían [complejidad temporal]() no lineal limitando su aplicación a conjuntos de datos más pequeños y en términos simples eran simplemente peores. Ver las referencias para fuentes.

Con el enfoque de iForest nos apoyamos en dos características: las anomalías son pocas y diferentes. lo cual nos permite separar puntos de datos anómalos de otros usando menor número de particiones.

Las particiones en este contexto pueden explicarse imaginando un gráfico de dispersión bidimensional de nuestros puntos de datos, donde las anomalías son pocas y distantes entre sí, toma por ejemplo un punto a la extrema derecha, debido a su posición podrías aislarlo dibujando una línea a la izquierda del punto anómalo y tomó 1 partición aislar ese punto. Mientras que para aislar un punto que está rodeado por vecinos, tendrías que dibujar cuatro líneas: arriba, abajo y una a cada lado.

Con espacio de datos de dimensiones más altas el principio sigue en pie, y cada dimensión representa un atributo de nuestros datos, precisamente, cada punto de datos recolectado dentro de cada registro conn.log.

Volvamos a nuestra representación de datos de árbol binario y consideremos esto visualmente:

```plaintext
        [raíz]
        /    \
      [A]    [B]
      / \    / \
    [C] [D][E] [F]
```

Para alcanzar el nodo D, debes: ir a la izquierda en la raíz, luego ir a la derecha en A. Este camino (izquierda → derecha) codifica una "dirección" única para esa región de datos. Las anomalías, al estar aisladas, obtienen direcciones cortas. Los puntos normales, al estar agrupados, requieren direcciones más largas para distinguirlos de los vecinos.

#### Constante de Euler, Serie Armónica y el Problema de Comparabilidad

Siéntete libre de saltarte esta sección, sin embargo, pasé un poco demasiado tiempo reuniendo una intuición matemática lo suficientemente buena para implementar nuestro algoritmo, aquí está la matemática expuesta simplemente:

"_Cuando calculas longitudes de camino en árboles de aislamiento, tienes un problema de comparabilidad. Si yo construyo un árbol con 100 muestras y tú construyes uno con 10,000 muestras, tus caminos serán naturalmente más largos—no porque tus anomalías sean menos anómalas, sino simplemente porque hay más datos por los cuales particionar_"

La **longitud de camino esperada** para un punto en un árbol de búsqueda binaria de n muestras sigue:

```LaTeX
c(n) = 2H(n-1) - (2(n-1)/n)
```

Donde H(n) es el número armónico (aproximadamente ln(n) + 0.5772).
Esto nos da una línea base: si la longitud de camino promedio de un punto a través de muchos árboles es mucho más corta que c(n), es anómalo.

La fórmula de puntuación de anomalía:

```LaTeX
s(x, n) = 2^(-E(h(x))/c(n))
```

Donde:

- E(h(x)) = longitud de camino promedio para el punto x a través de todos los árboles
- c(n) = factor de normalización

Puntuación cercana a 1 → anomalía
Puntuación cercana a 0.5 → normal
Puntuación cercana a 0 → muy profundo en clúster normal

## Conclusiones

Las anomalías son pocas, así que no se agrupan.
Las anomalías son diferentes, así que yacen en regiones dispersas.
Por lo tanto: Las particiones aleatorias aíslan anomalías rápidamente mientras que los puntos normales requieren muchas particiones.
La longitud del camino se convierte en la señal de anomalía.

## Implementación

### Conectando conceptos a nuestro conn.log de Zeek

La configuración predeterminada de Zeek produce varios tipos de logs. Los más relevantes para detección de anomalías típicamente incluyen:

- conn.log - Registros de conexión (IPs origen/destino, puertos, duración, bytes transferidos)
- dns.log - Consultas y respuestas DNS
- http.log - Metadatos de solicitud/respuesta HTTP
- ssl.log - Información de handshake SSL/TLS

Para este proyecto, nos enfocaremos en conn.log ya que proporciona información sobre cada conexión que alcanza nuestra Interfaz de Red, y una parte importante de estos datos es numérica, generando características útiles que podemos usar para Isolation Forest.

Aunque dns.log es un conjunto de datos que también me gustaría explorar ya que podría generar alertas para resoluciones DNS anómalas útiles para detección y caza de amenazas.

Cada registro de conexión tiene múltiples características formando un punto de alta dimensión:

| **Característica** | **Rol en Aislamiento**                                       |
|-------------------|-------------------------------------------------------------|
| **Duration**      | Conexiones de larga duración pueden aislarse rápidamente.   |
| **Orig Bytes**    | Tamaños de transferencia inusuales se destacan.             |
| **Resp Bytes**    | Transferencias asimétricas pueden ser anómalas.             |
| **Orig Pkts**     | Anomalías en el número de paquetes intercambiados pueden insinuar patrones de tráfico inusuales o ataques potenciales. Monitorear conteos de paquetes ayuda a diferenciar comportamiento normal de sospechoso. |
| **Conn State**    | Estados inusuales (S0, REJ) pueden indicar problemas.       |

Cuando construimos un árbol de aislamiento, una división aleatoria en orig_bytes podría separar inmediatamente un intento de exfiltración de datos (gigabytes transferidos) de navegación normal (kilobytes). Esa sola división aísla la anomalía—de ahí una longitud de camino corta, de ahí una puntuación de anomalía alta.

### Script de Python

La teoría fue divertida de leer, pero ahora tengo que escribir el código realmente.
Antes de escribir cualquier código, establezcamos lo que necesitamos y justifiquemos cada dependencia.

#### Bibliotecas

- Pandas: Que ayuda con la carga de datos, preprocesamiento y manipulación de columnas. Los logs de Zeek son datos tabulares estructurados. Pandas proporciona operaciones intuitivas de DataFrame para cargar archivos de entrada JSON/CSV, manejar valores faltantes, y agregar nuestra columna de puntuación de anomalía.

- Numpy: Para manejar operaciones numéricas y manipulación de arrays. Isolation Forest depende fuertemente de muestreo aleatorio y cálculos de recorrido de árboles. NumPy proporciona operaciones vectorizadas que son órdenes de magnitud más rápidas que loops puros de Python.

- SciKitLearn: Incluye implementaciones de algoritmos ya codificadas, es la alternativa ideal para código de producción en lugar de implementar el tuyo propio para contenido de blogpost.

#### Estrategia

Lo primero que viene a la mente cuando resuelvo problemas es Modelado de Dominio. Lo primero que trato de descifrar es un recuento de todas las piezas móviles en nuestra solución, sus propiedades y acciones.

Aterricé en tener una clase `IsolationForest` que puede crear recursivamente otros `IsolationTree`s particionando los datos, un árbol es siempre en sí mismo un nodo y un árbol para otros nodos.
Y un orquestador `IsolationForest` que maneja el proceso general y se encarga del muestreo de datos inicial y otras propiedades como el número total de árboles a crear.

- `IsolationTree`:
  Todas las propiedades del árbol:

    particionamiento
  - La `feature` (característica) seleccionada para particionamiento
  - El `value` (valor) seleccionado para particionamiento
  - Puntero `right` al nodo hijo `IsolationTree` derecho.
  - Puntero `left` al nodo hijo `IsolationTree` izquierdo.
    estructura
  - La `max_height` para limitar la profundidad del árbol
  - La `current_depth` en la que está este árbol.

  Los métodos o acciones que el árbol puede realizar:
  - `fit(X: np.array, nt: int = 1)`:
    Dada una muestra de forma (n_samples, n_features), construir el árbol recursivamente.
  - `calculate_path_length(x: np.array)`:
    Retornar la profundidad de una muestra recorriendo los árboles.

- `IsolationForest`:
  Los métodos o acciones que el árbol puede realizar:
  - `fit(X: np.array, nt: int = 1)`:
    Dada una muestra de forma (n_samples, n_features), construir el árbol recursivamente. Almacenar en self.trees.
  - `predict()`:
    Usando información de profundidad dentro de cada árbol, usar la fórmula de anomalía para calcular la puntuación para cada muestra en X.
  - `_calculate_avg`:
    Calcular el valor promedio de una característica dada a través de todos los árboles, usado para comparación dentro del cálculo de anomalía.

### Integración con Zeek

### Oportunidades de mejora

1. **Ingeniería de Características**: Explorar características adicionales que podrían mejorar la capacidad del modelo para detectar anomalías. Esto podría incluir características basadas en tiempo, como la duración de conexiones o la frecuencia de solicitudes desde una dirección IP particular. Codificar características categóricas parece un paso fácil que puede demostrar ser inmediatamente útil (One Hot Encoding y otros métodos).

2. **Ajuste de Parámetros**: Experimentar con diferentes parámetros para el algoritmo de Isolation Forest en sí, como el número de árboles, la profundidad máxima de los árboles, y el parámetro de contaminación. Esto podría ayudar a mejorar el rendimiento del modelo en conjuntos de datos específicos.

3. **Pruebas y Evaluación del Modelo**: Admitidamente, no soy el mejor en Desarrollo Guiado por Pruebas. Implementar un marco de pruebas y evaluación para evaluar el rendimiento del modelo nos permitiría rastrear la precisión de la detección de anomalías. Esto podría incluir principalmente validación cruzada con conjuntos de datos de control donde las anomalías son conocidas, para entender mejor los compromisos entre falsos positivos y falsos negativos.

44. **Integración con Otras Herramientas**: Esta parte involucra elaborar un sistema ligero de microservicios donde enriquecemos los datos con el cálculo de anomalía antes de ser enviados a una solución de monitoreo como Kibana + Elastic que es lo que personalmente uso. Esto integraría el modelo de Isolation Forest con otras herramientas y marcos de seguridad como un SIEM o IDS para generar alertas o tomar acción. Esto también lograría **Monitoreo en Tiempo Real**.

## Referencias

[Paper de Isolation Forest por Tony Liu et al](https://www.lamda.nju.edu.cn/publication/icdm08b.pdf)
