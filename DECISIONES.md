# Decisiones de Arado

Registro de por qué el programa es como es. Si cambias algo de lo que aquí se describe, actualiza este archivo.

- **Origen:** tercer programa de la familia de [Mesa de tangentes](https://github.com/meegl/mesa-de-tangentes) y [Goma y clavos](https://github.com/meegl/goma-y-clavos) (octubre de 2026). Comparte con ellos estilo, interfaz y forma de exportar.
- **Nombre:** *Arado*, por los surcos paralelos que deja en la tierra. Dentro del programa, cada línea es un **surco** y cada haz de surcos, una **pasada** (una pasada del arado). En la primera versión se llamaban «peine» y «púa». Se cambiaron para que el vocabulario fuera el del nombre.

## 1. Para qué sirve

Hacer la base geométrica de carteles hechos con haces de líneas paralelas que se curvan, se abren en abanico, dan quiebros o parecen cintas en 3D. Referencias: el cartel del 11.º Festival de Cine por los Derechos de las Mujeres (Seúl, 2017), *Downward root*, la 10.ª Bienal Internacional del Cartel de China, la exposición de Alvar Aalto (Arlington, 1980) y un cartel amarillo de flechas en perspectiva.

**Fuera de alcance:** color, textos y capas de diseño. El programa hace solo las formas, que se exportan a Illustrator o Affinity para acabar allí el diseño.

## 2. El modelo: una guía y una pasada que la sigue

- La **guía** es una polilínea de nodos. Cada nodo interior tiene un **radio** que redondea su esquina con un arco tangente.
- La **pasada** son N surcos paralelos a la guía, centrados en ella: el surco k está a una distancia (k − (N−1)/2) · separación.
- Todo sale en **rectas y arcos exactos** (comandos `L` y `A` de SVG), sin aproximar con puntos. En Illustrator llegan trazados limpios.

### Dos tipos de esquina (`Esquinas`)

- **Concéntricas** (por defecto). La paralela de un arco es otro arco con el mismo centro, así que los radios crecen hacia fuera y menguan hacia dentro, como en el cartel de Aalto. Si a un surco interior le toca un radio de cero o menos, hace **esquina viva**: es el corte de las dos rectas vecinas. Con radio 0 en la guía, el lado de fuera queda redondeado y el de dentro, vivo.
- **Mismo radio.** Cada surco es la polilínea paralela con esquinas vivas, redondeada después con el mismo radio que la guía. Se ve como un haz de cintas iguales.

### Geometría (módulo `AR` en `index.html`)

- `fillet(P, R)`: devuelve tramos alternos (recta, esquina, recta…). En cada esquina, la tangente se separa del vértice t = r · tan(φ/2), donde φ es el ángulo de giro.
- **Reparto de los tramos:** si dos esquinas piden más longitud de la que tiene el tramo que las une, cada una se recorta en proporción a lo que pide. El radio efectivo puede quedar más pequeño que el pedido.
- **Radio sin tope.** Ni el slider ni el asa limitan el radio pedido. Pedir más de lo que cabe sirve, porque decide cómo se reparten el tramo dos esquinas vecinas (así salen las S suaves). El panel indica el radio efectivo cuando es menor que el pedido.
- **Slider del radio progresivo:** va de 0 a 1000 y el radio es máx · (s/1000)², con máx = el doble del lado mayor del papel (594 mm en A4), o el radio actual si es mayor. Así hay rango para curvas amplias y precisión en los radios pequeños. La casilla numérica acepta cualquier valor en mm.
  - *Error que no se debe repetir:* una versión limitó el slider y el asa al radio que «cabía» sin quitar sitio a las vecinas. Con una vecina de radio grande, el rango se quedaba en unos pocos milímetros y el usuario perdía el control que tenía arrastrando el asa.
- `offsetSegs(els, d)`: hace la paralela como lista de tramos con su principio y su final. En una esquina, el radio nuevo es r − s·d, donde s es el lado del centro; el centro es el mismo.
- **Aviso de pliegue:** si una recta de un surco queda del revés respecto a la guía, el surco se pliega sobre sí mismo. La barra de estado lo avisa en rojo, en los dos modos.
- Los nodos repetidos seguidos se ignoran (`clean`), y `ix` guarda su índice original para las asas.

## 3. Interacción

- **Mover (`V`):**
  - Arrastrar un nodo lo mueve.
  - Arrastrar el cuerpo de la pasada la mueve entera (con el imán, en pasos de celda).
  - El **punto azul** de cada esquina cambia su radio. Está en la bisectriz, 14 px más allá del punto medio del arco, para que se pueda coger aunque el radio sea 0.
- **Añadir (`A`):**
  - Clic en un hueco: añade un nodo al final de la pasada seleccionada, o al principio si está seleccionado el primer nodo. Un círculo marca el extremo por el que se sigue.
  - Sin pasada seleccionada, el clic empieza una pasada nueva.
  - Clic sobre un tramo de la guía: inserta un nodo en ese punto.
- Cuando un nodo deja de ser extremo, recibe un radio por defecto: medio ancho de la pasada más una celda. Así los surcos interiores no se pliegan.
- **Nueva pasada (`P`)** quita la selección y pasa a Añadir. La pasada nueva copia los ajustes (también los extremos) de la seleccionada o de la última. **Esc** quita la selección.
- El **primer nodo** de la guía se dibuja redondo: marca el **inicio**, para saber qué extremo es cuál.
- **Mesa:** alrededor del papel hay un margen (`PB` = 72 unidades) donde se pueden poner nodos, para que las líneas **sangren**. Lo que cae fuera del papel se ve velado.

## 4. Capas y exportación

- Casillas de **Capas visibles:** Fondo de papel, Rejilla y Guías de edición (`G`). Se exporta lo que se ve, **salvo las guías**, que nunca se exportan.
- Capas del SVG: `fondo`, `rejilla` y un grupo por pasada: `pasada-1`, `pasada-2`… Dentro de cada grupo, un surco por trazado (`pasada-1-surco-01`…) y, si hay remates, un subgrupo `pasada-1-remates` con los guiones (trazos) y los puntos (círculos rellenos).
- Los surcos se exportan como **trazos abiertos** (`fill="none"`), con el grosor y el remate en el grupo. Así se editan en Illustrator o Affinity. Lo que se sale del papel también va en el SVG, y el programa de vectores lo recorta con la mesa de trabajo.
- Archivos: `arado.svg` y `arado.png`. El PNG solo incluye el papel.

## 5. Almacenamiento

- Clave `localStorage`: **`arado-v1`**. No se leen las claves de los otros programas, aunque comparten origen en `meegl.github.io`.
- Con el almacenamiento vacío, arranca con una pasada de ejemplo en forma de S, al estilo Aalto.

## 6. Fases pendientes

Hechas: **fase 1** (pasada básica), **fase 2** (extremos escalonados, §7) y **fase 3** (abanico y fuga, §8). Pendientes, en el orden propuesto:

4. **Quiebros:** en un nodo, la pasada se desplaza de lado, toda junta o en escalera surco a surco. Referencia: los escalones del cartel de Aalto.
5. **Giro 3D y cruces:**
   - Ángulo de giro por nodo, como una cinta: el ancho visible es ancho · |cos θ| y el orden se invierte al pasar de 90°.
   - Cuando una pasada cruza otra, hueco en la de abajo.
   - Referencias: Bienal de China y cartel amarillo.
6. **Perspectiva:** punto de fuga global; grosor y separación menguan con la profundidad. Referencia: cartel amarillo.

## 7. Extremos escalonados (fase 2)

Cada extremo (**Inicio** y **Fin**) tiene sus propios ajustes, que se editan de uno en uno con el selector del panel. Están en `pasada.ext.ini` y `pasada.ext.fin`.

- **Escalonado (mm):** lo máximo que se recorta un surco desde su extremo, medido **a lo largo del surco**. Así el corte puede entrar en una curva.
- **Perfil:** cuánto se recorta cada surco según su posición u (0 = primero, 1 = último):
  - Recto: 0.
  - Diagonal: u.
  - Punta: |2u − 1|. El surco del centro queda más largo.
  - Curva: (2u − 1)². Una punta redondeada.
  - Ondas: (1 − cos(2π · ondas · u)) / 2.
  - **Invertir** cambia f por 1 − f.
- **Variación al azar (%):** mezcla el perfil con un azar repetible (`rnd(semilla, surco)`). Con perfil recto y variación al 100 %, los cortes son aleatorios. **Otra variación** cambia la semilla. Inicio y fin usan azares distintos.
- **Remate:** puntos o guiones después del extremo, en la dirección de la tangente, separados por el **Hueco**. **Cantidad** dice cuántos.
  - El punto tiene el diámetro del grosor.
  - El guion mide tres grosores y usa la misma terminación que los surcos.
- Si el recorte de los dos extremos supera la longitud del surco, el surco desaparece.
- **Copiar al otro extremo** copia todos los ajustes del extremo que se está editando.
- La terminación de los surcos (recta, redonda o cuadrada) se llama **Terminación**, para no confundirla con los remates.

## 8. Abanico y fuga (fase 3)

- Cada nodo tiene una **apertura** (`nd.a`, en %, 100 por defecto, de 0 a 400). Es el factor por el que se multiplica la distancia de cada surco a la guía cuando la pasada pasa por ese nodo.
- **Dónde está cada nodo en la guía:** los extremos, al principio y al final; un nodo con esquina ocupa **todo su arco**; uno sin esquina, la unión de sus tramos. La apertura es fija mientras dura el arco y solo cambia en las rectas de entre medias, con una **transición** suave (smoothstep, por defecto) o lineal (`pasada.tr`).
  - *Antes el nodo era el punto medio del arco y la apertura cambiaba también dentro de la curva. Donde la guía pasa de recta a curva, el surco de dentro doblaba de golpe y hacía un codo. Con la apertura fija en el arco y la transición suave (que empieza y acaba sin pendiente), el empalme queda sin codos. Con la transición lineal aún puede verse un codo pequeño.*
- **Converger en un punto** es poner la apertura de un nodo a 0 %. Hay un botón para volver todo al 100 %.
  - *Había botones «Converger al final / al inicio», pero solo ponían a 0 % el primer o el último nodo, que ya se hace con su barra. Se quitaron.*
- **Geometría (`fanMaker`):**
  - Solo se usa si algún nodo no está al 100 %; si no, los surcos siguen siendo rectas y arcos exactos.
  - Se muestrea la guía (cada 4 unidades en las rectas y cada 3° en los arcos, también en las esquinas de radio 0), y cada muestra se desplaza por su normal: distancia × apertura.
  - **Esquinas cerradas sin aberraciones.** Cuando un surco interior queda más lejos de la guía que el radio de la curva, su paralela retrocede y hace ganchos y rizos (una «cola de golondrina»). Se arregla en dos pasos:
    1. Se quitan los puntos cuyo tramo va hacia atrás respecto a la tangente de la guía.
    2. `unloop` corta donde se cruzan la rama que llega y la que sale, y deja una esquina viva. Solo quita trozos de hasta 2π · |d| · apertura máxima + 4 muestras. *Antes no tenía límite: cuando un surco se cruzaba consigo mismo lejos (las dos piernas de una curva en U donde la pasada no cabe), perdía un trozo enorme.*
    - *Un primer intento comparaba cada punto con el último guardado: dejaba una punta en cada esquina, en zigzag. Se descartó.*
  - Después se recorta por los extremos (fase 2), se simplifica con Ramer-Douglas-Peucker (tolerancia de 0,2 unidades) y se convierte en curvas Bézier. Las tangentes salen de los vecinos; donde el giro supera 35°, la esquina se respeta.
- Con aperturas distintas, **Esquinas** no tiene efecto: siempre se comporta como «Concéntricas».

### ¿Cabe la pasada en la curva?

- En cada esquina se calcula la **apertura máxima que cabe** (`res.tight`, con el nodo y `amax`):
  - **Con radio:** todos los surcos deben girar en arco, es decir, d + grosor/2 ≤ radio efectivo.
  - **Sin radio (esquina viva):** la esquina del surco de dentro debe caer dentro de sus tramos: d ≤ tramo libre / tan(φ/2).
  - No se comprueba en «Mismo radio» sin apertura, que tiene su propio aviso de pliegue.
- Si un nodo pide más apertura de la que cabe:
  - En el lienzo se marca con un **anillo rojo** (aunque la pasada no esté seleccionada).
  - La barra de estado dice qué pasada, qué nodo y cuánta apertura cabe.
  - En la pestaña Fuga, su fila sale en rojo con «máx X %».
- **Ajustar a la curva** (pestaña Fuga) baja la apertura de esos nodos al 97 % de lo que cabe. Repite hasta cuatro veces por si al cambiar uno cambia otro.
- Origen: una pasada de 85 mm de ancho en un giro de 136° cuyo radio real se quedaba en 13 mm. Los surcos de dentro no cabían y salían pinchos, ganchos y trozos perdidos. El usuario lo había resuelto a mano bajando la apertura en ese nodo, y el botón hace lo mismo.

## 9. Panel en pestañas

- Al crecer los controles, el panel dejó de caber. Ahora:
  - Cada control ocupa **una sola fila**: nombre, barra y casilla.
  - Hay **pestañas**: Pasada, Extremos, Fuga y Papel. Papel incluye el formato, la rejilla y las capas visibles.
  - El **nodo seleccionado** (X, Y, radio y apertura) va fijo encima de las pestañas, con borde azul, y solo aparece si hay un nodo seleccionado.
  - **Exportar** es un solo botón en la barra que abre un menú: Descargar SVG, Descargar PNG (con tamaño y transparencia en el mismo menú) y Copiar SVG. Se cierra al elegir algo, al pulsar fuera o con Esc. Los mensajes salen debajo del lienzo.
    - *Antes eran tres botones sueltos (SVG en azul, PNG y Copiar) y las opciones del PNG estaban en la pestaña Papel. Al usuario no le gustaba, y otra pestaña habrían sido demasiadas.*
- El slider del radio es progresivo: el radio es máx · (s/1000)², donde máx es el doble del lado mayor del papel, o el radio actual si es mayor. Ni el slider ni el asa ponen tope.
  - *Una versión limitó el radio al que «cabía» sin quitar sitio a las vecinas, y le quitaba rango al asa. Se descartó.*

## 10. Limitaciones conocidas e ideas

- En **Mismo radio**, una curva en U más estrecha que la pasada da formas raras. Se avisa, pero no se corrige.
- El cuerpo de la pasada se detecta por su distancia al polígono de control, no a las curvas reales.
- No hay rotación ni simetría de pasadas.
- Los remates siguen la tangente en línea recta, no la curva.
- La apertura solo se edita con sliders; falta un asa en el lienzo para arrastrarla.
- No hay aún **Aleatorio**.

## 11. Cómo se trabaja en el proyecto

- **Un único `index.html`**, sin dependencias ni compilación. JavaScript ES5 (`var`, `function`), igual que en Goma y clavos. Interfaz, comentarios y commits en español.
- Para probar, levanta un servidor estático en la carpeta y abre `http://localhost:8765`:

  ```bash
  python3 -m http.server 8765
  ```

- Commits en español, con una primera línea corta que describe el cambio.
