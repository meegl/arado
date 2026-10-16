# Decisiones de Arado

Registro de por qué el programa es como es. Si cambias algo de lo que aquí se describe, actualiza este archivo.

- **Origen:** tercer programa de la familia de [Mesa de tangentes](https://github.com/meegl/mesa-de-tangentes) y [Goma y clavos](https://github.com/meegl/goma-y-clavos) (octubre de 2026). Comparte con ellos estilo, interfaz y forma de exportar.
- **Nombre:** *Arado* por los surcos paralelos que deja en la tierra. Dentro del programa, cada haz de líneas es un **peine** y cada línea, una **púa**.

## 1. Para qué sirve

Hacer la base geométrica de carteles hechos con haces de líneas paralelas que se curvan, se abren en abanico, dan quiebros o parecen cintas en 3D. Referencias: el cartel del 11.º Festival de Cine por los Derechos de las Mujeres (Seúl, 2017), *Downward root*, la 10.ª Bienal Internacional del Cartel de China, la exposición de Alvar Aalto (Arlington, 1980) y un cartel amarillo de flechas en perspectiva.

**Fuera de alcance:** color, textos y capas de diseño. El programa hace solo las formas, que se exportan a Illustrator o Affinity para acabar allí el diseño.

## 2. El modelo: una guía y un peine que la sigue

- La **guía** es una polilínea de nodos. Cada nodo interior tiene un **radio** que redondea su esquina con un arco tangente.
- El **peine** son N púas paralelas a la guía, centradas en ella: la púa k está a una distancia (k − (N−1)/2) · separación.
- Todo sale en **rectas y arcos exactos** (comandos `L` y `A` de SVG), sin aproximar con puntos. En Illustrator llegan trazados limpios.

### Dos tipos de esquina (`Esquinas`)

- **Concéntricas** (por defecto). La paralela de un arco es otro arco con el mismo centro, así que los radios crecen hacia fuera y menguan hacia dentro, como en el cartel de Aalto. Si a una púa interior le toca un radio de cero o menos, hace **esquina viva**: es el corte de las dos rectas vecinas. Con radio 0 en la guía, el lado de fuera queda redondeado y el de dentro, vivo.
- **Mismo radio.** Cada púa es la polilínea paralela con esquinas vivas, redondeada después con el mismo radio que la guía. Se ve como un haz de cintas iguales.

### Geometría (módulo `AR` en `index.html`)

- `fillet(P, R)`: devuelve tramos alternos (recta, esquina, recta…). En cada esquina, la tangente se separa del vértice t = r · tan(φ/2), donde φ es el ángulo de giro.
- **Reparto de los tramos:** si dos esquinas piden más longitud de la que tiene el tramo que las une, cada una se recorta en proporción a lo que pide. El radio efectivo puede quedar más pequeño que el pedido.
- `offsetD(els, d)`: hace la paralela. En una esquina, el radio nuevo es r − s·d, donde s es el lado del centro.
- **Aviso de pliegue:** si una recta de una púa queda del revés respecto a la guía, la púa se pliega sobre sí misma. La barra de estado lo avisa en rojo, en los dos modos.
- Los nodos repetidos seguidos se ignoran (`clean`), y `ix` guarda su índice original para las asas.

## 3. Interacción

- **Mover (`V`):**
  - Arrastrar un nodo lo mueve.
  - Arrastrar el cuerpo del peine mueve el peine entero (con el imán, en pasos de celda).
  - El **punto azul** de cada esquina cambia su radio. Está en la bisectriz, 14 px más allá del punto medio del arco, para que se pueda coger aunque el radio sea 0.
- **Añadir (`A`):**
  - Clic en un hueco: añade un nodo al final del peine seleccionado, o al principio si está seleccionado el primer nodo. Un círculo marca el extremo por el que se sigue.
  - Sin peine seleccionado, el clic empieza un peine nuevo.
  - Clic sobre un tramo de la guía: inserta un nodo en ese punto.
- Cuando un nodo deja de ser extremo, recibe un radio por defecto: medio ancho del peine más una celda. Así las púas interiores no se pliegan.
- **Nuevo peine (`P`)** quita la selección y pasa a Añadir. El peine nuevo copia los ajustes del seleccionado o del último. **Esc** quita la selección.
- **Mesa:** alrededor del papel hay un margen (`PB` = 72 unidades) donde se pueden poner nodos, para que las líneas **sangren**. Lo que cae fuera del papel se ve velado.

## 4. Capas y exportación

- Casillas de **Capas visibles:** Fondo de papel, Rejilla y Guías de edición (`G`). Se exporta lo que se ve, **salvo las guías**, que nunca se exportan.
- Capas del SVG: `fondo`, `rejilla` y un grupo por peine: `peine-1`, `peine-2`… Dentro de cada grupo, una púa por trazado: `peine-1-pua-01`…
- Las púas se exportan como **trazos abiertos** (`fill="none"`), con el grosor y el remate en el grupo. Así se editan en Illustrator o Affinity. Lo que se sale del papel también va en el SVG, y el programa de vectores lo recorta con la mesa de trabajo.
- Archivos: `arado.svg` y `arado.png`. El PNG solo incluye el papel.

## 5. Almacenamiento

- Clave `localStorage`: **`arado-v1`**. No se leen las claves de los otros programas, aunque comparten origen en `meegl.github.io`.
- Con el almacenamiento vacío, arranca con un peine de ejemplo en forma de S, al estilo Aalto.

## 6. Fases pendientes

La fase 1 (este documento) es el peine básico. Las siguientes, en el orden propuesto:

2. **Extremos escalonados:** cada púa empieza y acaba a una altura distinta según un perfil (diagonal, curva, ondas o aleatorio con semilla), con puntitos o guiones opcionales al final. Referencias: cartel coreano y *Downward root*.
3. **Abanico y fuga:** ancho del peine por nodo, para que las púas se junten o se abran por el camino. Opción de converger en un punto. Referencias: cartel coreano y Bienal de China.
4. **Quiebros:** en un nodo, el peine se desplaza de lado, todo junto o en escalera púa a púa. Referencia: los escalones del cartel de Aalto.
5. **Giro 3D y cruces:**
   - Ángulo de giro por nodo, como una cinta: el ancho visible es ancho · |cos θ| y el orden se invierte al pasar de 90°.
   - Cuando un peine cruza otro, hueco en el de abajo.
   - Referencias: Bienal de China y cartel amarillo.
6. **Perspectiva:** punto de fuga global; grosor y separación menguan con la profundidad. Referencia: cartel amarillo.

## 7. Limitaciones conocidas e ideas

- En **Mismo radio**, una curva en U más estrecha que el peine da formas raras. Se avisa, pero no se corrige.
- El cuerpo del peine se detecta por su distancia al polígono de control, no a las curvas reales.
- No hay rotación ni simetría de peines.
- No hay aún **Aleatorio**.

## 8. Cómo se trabaja en el proyecto

- **Un único `index.html`**, sin dependencias ni compilación. JavaScript ES5 (`var`, `function`), igual que en Goma y clavos. Interfaz, comentarios y commits en español.
- Para probar, levanta un servidor estático en la carpeta y abre `http://localhost:8765`:

  ```bash
  python3 -m http.server 8765
  ```

- Commits en español, con una primera línea corta que describe el cambio.
