# Decisiones de Arado

Registro de por qué el programa es como es. Si cambias algo de lo que aquí se describe, actualiza este archivo.

- **Web:** https://meegl.github.io/arado/
- **Repositorio:** https://github.com/meegl/arado
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
  - Arrastrar el cuerpo de la pasada la mueve entera (con el imán, en pasos de celda). Cada nodo conserva todos sus valores (radio, apertura, grosor, quiebro…), y el desplazamiento se frena en el borde de la mesa para que la forma no se deforme.
    - *Un fallo anterior copiaba solo la posición y el radio, y al mover la pasada se perdían la apertura, el grosor y los quiebros.*
  - El **punto azul** de cada esquina cambia su radio. Está en la bisectriz, 14 px más allá del punto medio del arco, para que se pueda coger aunque el radio sea 0.
- **Añadir (`A`):**
  - Clic en un hueco: añade un nodo al final de la pasada seleccionada, o al principio si está seleccionado el primer nodo. Un círculo marca el extremo por el que se sigue.
  - Sin pasada seleccionada, el clic empieza una pasada nueva.
  - Clic sobre un tramo de la guía: inserta un nodo en ese punto. El nodo hereda la media de apertura y grosor de sus dos vecinos, para que la pasada no cambie de golpe.
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

Hechas: **fase 1** (pasada básica), **fase 2** (extremos escalonados, §7), **fase 3** (abanico y fuga, §8), **fase 4** (quiebros, §8), **fase 5** (giro 3D y cruces, §8) y **fase 6** (perspectiva, §8). Todas las fases propuestas están hechas.


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
- **Ajustar a la curva** (pestaña Fuga, `fitPasada`):
  1. **Sube el radio** de cada curva donde no cabe la pasada, lo justo para que quepa entera: medio ancho · apertura aparente + medio grosor, con un 4 % de margen.
  2. Si los tramos no dan para ese radio (el radio efectivo se recorta), **baja la apertura** solo hasta lo que cabe (97 %), repitiendo hasta cuatro veces.
  - *Antes solo bajaba la apertura, y en curvas de radio pequeño dejaba la pasada casi en una línea (por ejemplo, al 8 %). El usuario lo notó. Con el radio primero, en el mismo caso queda al 100 % si hay sitio, o al 61 % con un tramo muy corto.*
  - El aleatorio usa el mismo ajuste.
- Origen: una pasada de 85 mm de ancho en un giro de 136° cuyo radio real se quedaba en 13 mm. Los surcos de dentro no cabían y salían pinchos, ganchos y trozos perdidos. El usuario lo había resuelto a mano bajando la apertura en ese nodo, y el botón hace lo mismo.

### Grosor variable

Hay dos maneras de variar el grosor, y se pueden combinar:

- **Entre surcos** (`pasada.wl`, en la pestaña Pasada), con un **reparto** (`pasada.wp`): wk = w · (1 + (wl − 1) · v).
  - **Lineal:** v = u. El control se llama «Último surco (%)»: grosor del último surco respecto al primero.
  - **Punta:** v = 1 − |2u − 1|, en forma de tienda de campaña.
  - **Curva:** v = sin(π·u).
  - Con Punta y Curva, el control se llama «Surco central (%)»: cambia el centro y los dos bordes quedan con el grosor base. Lo pidió el usuario para poder dejar los dos lados iguales. Los surcos siguen siendo trazos, cada uno con su `stroke-width` (el del grupo es el del primero).
- **A lo largo de la guía** (`nd.g`, «Grosor (%)» en el panel del nodo): se interpola igual que la apertura, fijo en cada arco y con la misma transición.
  - El SVG no tiene trazos de grosor variable, así que cada surco se exporta como **forma rellena**: su contorno (`outline`), con los dos bordes a ± grosor/2 de la línea central, suavizado con Bézier.
  - Las terminaciones: recta (cierra con una recta), redonda (arcos SVG de radio grosor/2) y cuadrada (alarga los bordes grosor/2).
  - El factor de grosor viaja con cada punto como tercer valor ([x, y, g]) a través del filtrado, `unloop` y el recorte de los extremos.
  - Los remates (puntos y guiones) usan el grosor del surco en ese extremo.
  - La pestaña Pasada avisa de que se exporta como formas, y tiene un botón para volver los grosores de los nodos al 100 %.
- El cálculo de si la pasada cabe en la curva usa el grosor mayor (primero o último surco) por el grosor del nodo.
- Esto ya prepara las fases 5 (la cinta que gira se estrecha) y 6 (en perspectiva, el grosor mengua con la profundidad).

### Quiebros (fase 4)

- Cada nodo interior tiene tres valores, en el panel del nodo:
  - **Quiebro** (`nd.q`): un salto de lado de la pasada, positivo hacia la normal y negativo hacia el otro lado.
  - **Escalera** (`nd.qe`): cuánto se adelanta el salto de cada surco respecto al anterior a lo largo de la guía. El surco k salta en el centro del nodo más (k − centro) · escalera.
  - **Largo** (`nd.ql`): lo que ocupa el salto. Con 0 es un escalón en ángulo recto; con más, una rampa suave (smoothstep).
- Escalera y Largo solo aparecen cuando el quiebro no es 0. En los extremos no hay quiebro.
- Los saltos se suman al desplazamiento de cada surco: d · apertura + Σ quiebros. Para que el escalón quede exacto, se insertan puntos justo en cada salto (`spineAt` interpola la guía en cualquier longitud). El filtro de retrocesos no toca esos puntos, porque un escalón recto no avanza.
- Un quiebro hace que la pasada use la geometría muestreada (Bézier), como la apertura.
- Funciona mejor en un nodo sobre un tramo recto: con «Añadir», un clic sobre la guía mete el nodo. En una esquina, el salto cae a mitad del arco.
- Referencia: los escalones del cartel de Alvar Aalto.

### Giro 3D y cruces (fase 5)

- **Giro** (`nd.t`, en grados, de −360 a 360, en el panel del nodo): la pasada es una cinta que gira sobre su guía.
  - Vista de lado, la distancia de cada surco a la guía se multiplica por cos(giro). A 90° la cinta se ve de canto y todos los surcos se juntan; pasado 90°, cos es negativo y la cinta se da la vuelta (los surcos cambian de lado).
  - El giro se interpola entre nodos como la apertura (fijo en cada arco, transición suave o lineal).
  - **Adelgazar los surcos al girar** (`pasada.thin`, activado por defecto, en la pestaña Pasada): el grosor también se multiplica por |cos(giro)| (mínimo 4 %). Por eso, con giro y adelgazar, los surcos se exportan como formas rellenas.
  - Con giro, los avisos usan la apertura aparente (apertura · |cos giro|) y no se avisa de cruces entre surcos vecinos, porque al darse la vuelta se cruzan a propósito.
- **Cruces con hueco** (`pasada.hueco`, «Hueco al cruzar (mm)» en la pestaña Pasada):
  - Una pasada con hueco **corta de verdad** los surcos de las pasadas que tiene debajo: quita todo lo que queda a menos de medio grosor de cualquiera de sus surcos más el hueco (`AR.cut`). No es una mancha de color papel encima, así que en Illustrator o Affinity se puede cambiar el fondo.
  - Los surcos cortados pasan a ser varias piezas en un mismo trazado (varios `M`). Los extremos del corte van rectos; si la terminación es redonda o cuadrada, el hueco se agranda medio grosor para compensar.
  - El orden de las pasadas es el orden de dibujo: la última de la lista va encima. **Subir** y **Bajar** (pestaña Pasada) la mueven.
  - Una pasada que se cruza consigo misma no se corta; queda pendiente.
- Referencias: Bienal de China (cinta que gira) y cartel amarillo (cruces).

### Alejarse: perspectiva de cada pasada

- Al probar la perspectiva de suelo, el usuario vio que no era lo que buscaba: aplastaba todo el dibujo sobre el suelo y dejaba vacía la mitad de arriba del papel. Lo que pedía, como en el cartel amarillo, es que **cada pasada se aleje por sí misma** a lo largo de su guía.
- **Tamaño al fondo (%)** (`pasada.lejos`, pestaña Pasada; 100 = no se aleja): el tamaño que tiene la pasada al final de su guía. Separación y grosor se encogen juntos.
- **Ley de perspectiva real:** si la pasada se aleja a velocidad constante, su tamaño aparente es m = 1 / (1 + α·u), con u de 0 a 1 a lo largo de la guía y α = 1/e − 1, para que al final mida e. Encoge deprisa al principio y cada vez más despacio, no en línea recta.
- **Empezar al fondo** (`pasada.lejosIni`) invierte el sentido (u → 1 − u): la pasada viene del fondo hacia ti.
- Se aplica dentro de `fanMaker`, multiplicando apertura y grosor de cada muestra. Por eso la pasada usa la geometría muestreada y se exporta como formas rellenas.
- La guía se dibuja y se edita como siempre, sin proyección. La perspectiva de suelo (abajo) sigue como opción aparte en Papel.

### Perspectiva de suelo (fase 6)

- Es un ajuste de toda la composición (`S.persp`, en la pestaña Papel):
  - **Ver en perspectiva**
  - **Ancho abajo** (`B`): ancho del dibujo en el borde de abajo, en proporción al papel.
  - **Ancho arriba** (`T`): ancho en el borde de arriba.
  - **Horizonte** (`hz`): automático por defecto (`hz` = null), o fijo donde se quiera, también dentro del papel, para dejar cielo arriba. La casilla «Horizonte automático» lo devuelve a su sitio.
  - **Fuga X** (`vx`): hacia dónde convergen.
- **Con el horizonte automático, el borde de abajo de la composición queda siempre abajo del papel, y el de arriba, arriba.** Con horizonte manual, el de abajo sigue abajo y el de arriba se acerca al horizonte. El usuario lo pidió después de probar los dos anchos.
  - *La primera versión tenía Horizonte, Profundidad y Escala, con el borde de abajo fijo a tamaño real. Agrandarla la sacaba por abajo y, sin agrandar, aplastaba el dibujo y dejaba vacía la mitad de arriba. El usuario propuso controlar «ampliar abajo» y «achatar arriba», y así se rehízo.*
- **El modelo:** la homografía que lleva el rectángulo del papel a un trapecio. Con v = y/H (0 arriba, 1 abajo):
  - λ(v) = 1/(p + q·v), con p = 1/T y q = 1/B − 1/T; así λ(0) = T y λ(1) = B.
  - X = vx + (x − vx)·λ e Y = H·v·λ/B; así Y(0) = 0 e Y(H) = H.
  - Las rectas siguen siendo rectas, y el escorzo vertical sale solo de la diferencia entre los dos anchos.
  - En general, Y = hz + (H − hz)·λ/B. El horizonte automático es hz = H/(B·q), el que hace que Y(0) = 0; si arriba es más estrecho, queda por encima del papel. Con horizonte manual se usa P.hz y la parte de arriba de la composición cae en hz + (H − hz)·T/B.
  - Si los dos anchos son iguales (q = 0) no hay horizonte y el manual se ignora.
  - Con los dos anchos al 100 % no hay proyección.
- Al quitar la perspectiva se ve la composición plana tal cual: los nodos se guardan siempre en el plano.
- **Se edita sobre el dibujo proyectado:**
  - `toWorld` deshace la proyección del puntero (`Ij`). Los nodos, las asas, el polígono de control y los círculos de las esquinas se dibujan proyectados.
  - Los nodos y las asas se buscan en pantalla. La tolerancia de los tramos se divide por la escala local λ.
  - Con perspectiva, el suelo no tiene el tope del papel: los nodos pueden ir hasta 50 anchos a cada lado, 50 alturas hacia el fondo y 5 alturas por debajo. *El límite de abajo era el del papel; como el nodo inicial suele estar ya ahí, al arrastrar la pasada entera no se podía bajar.*
  - En el lienzo se dibujan el horizonte (discontinuo) y una cruz en el punto de fuga, si caen cerca del papel.
- **Dibujo (`projectBuilt`)**, después de los cortes de los cruces:
  - Cada surco (o cada pieza cortada) pasa a ser su contorno en el plano (`AR.ring`, con sus terminaciones), se proyecta punto a punto y se suaviza (`AR.ringD`). Por eso, con perspectiva, todo se exporta como **formas rellenas**.
  - Los puntos se proyectan por el centro y escalan su radio por λ. Los guiones, por sus extremos.
  - La rejilla cubre un suelo mucho mayor que el papel (de −4 a 5 anchos y hasta 4 alturas de fondo), se proyecta y así se exporta, si está visible.
- **Doblar hacia arriba** (`fold`): a cierta altura del papel, el suelo se curva y sube como una pared vertical (referencia: el cartel del 11.º Festival de Cine por los Derechos de las Mujeres, 2017).
  - **Doblez a (mm)** (`fd`): altura del papel, desde abajo, a la que empieza el doblez. **Radio del doblez (mm)** (`fr`): 0 es un codo seco.
  - Los dos se dan **como se ven en el papel** y se pasan al plano con la inversa del suelo (`pFloorY`) y la escala en el doblez. *En una primera versión se daban en el plano y, con perspectiva fuerte, el doblez caía fuera del papel.*
  - Un punto a una distancia t más allá del doblez queda a profundidad ye y altura h: en el arco, ye = yf − R·sin(t/R) y h = R·(1 − cos(t/R)); después, pared (ye fijo, h crece).
  - En pantalla, Y = (Y del suelo a esa profundidad) − h·λ: la altura sube con la misma escala que el ancho, así que en la pared los surcos son verticales y paralelos.
  - Con doblez no hay inversa cerrada: `Ij` busca por bisección (Y crece siempre con y).
- **Tope bajo el papel:** con perspectivas fuertes, justo por debajo del borde de abajo el suelo crecería sin límite; λ se limita a 2·B (queda fuera del papel).
- **Corte exacto en el doblez (`splitFold`):** el contorno de cada surco se corta justo en la línea donde empieza el doblez (y donde acaba la curva, si hay radio). Con radio 0, ese punto se marca como esquina viva (`p.k`): `rdp` lo conserva y `smoothCmds` no lo suaviza. *Antes el codo caía entre dos muestras, cada borde lo redondeaba a su manera y salía un bulto.*
- **Densificar antes de proyectar:** un surco recto solo guarda sus dos extremos, así que `dense` lo rellena de puntos cada 5 unidades antes de proyectarlo. *Sin esto, el doblez y la convergencia desaparecían y el surco salía como una recta.*
- Para que una sola pasada se aleje, sin proyectar todo, está **Alejarse** (arriba).

### Avisos de aberraciones

El usuario pidió que, si algo de lo que toca rompe el dibujo, el control se ponga en naranja. El programa no sabe si algo es feo, pero sí detecta los problemas geométricos que suelen dar aberraciones (`issues` en `AR`). Cada aviso lleva el nodo que lo provoca (o −1 si es de toda la pasada), los controles que hay que tocar (`ctl`) y un mensaje:

| Problema | Controles en naranja |
|---|---|
| La pasada no cabe en la curva (`tight`) | Radio y Apertura del nodo |
| Esquina sin radio que gira más de 100° con surcos más anchos que su grosor (pinchos) | Radio |
| Grosor ≥ separación: los surcos se tocan | Separación y Grosor |
| En un nodo con apertura ≥ 15 %, separación · apertura ≤ grosor · grosor del nodo: los surcos se funden | Apertura y Grosor del nodo |
| Surcos que desaparecen por el escalonado | Escalonado (Extremos) |
| Pliegue en «Mismo radio» o en esquinas exactas | Radio |
| Surcos vecinos que se cruzan (solo con geometría muestreada; `firstCross` usa una rejilla). Se atribuye al nodo más cercano: si tiene quiebro, a Quiebro y Escalera; si no, a Apertura | Quiebro y Escalera, o Apertura |

- La **convergencia buscada** (apertura < 15 % en el nodo) no avisa.
- En la interfaz:
  - Las filas de los controles tienen `data-ctl` (y `data-scope="nodo"` las del panel del nodo); `markIssues` les pone la clase `bad`.
  - También se marcan en naranja las pestañas que contienen un control afectado y las filas de Fuga.
  - El nodo lleva un anillo naranja en el lienzo, y su panel muestra el mensaje.
  - La barra de estado enumera los dos primeros avisos («y N más»).
- Con una escalera en el sentido del salto, los surcos no se cruzan; con la escalera al revés, sí, y el aviso propone darle la vuelta.

### Aleatorio y Variar

- En la barra de arriba están **Aleatorio** (`R`), **Variar** (`Mayús+R`) y un menú ▾ con **Receta**, **Locura (%)** y **Aberraciones (%)** (`S.rnd`; sus cambios no van al historial).
- **Libre** (receta por defecto): el usuario prefirió algo más aleatorio que los estilos. Sin cartel de referencia:
  - De una a varias pasadas (más con locura), finas o gruesas, con guías que salen de fuera del papel por cualquier lado y giran al azar (más con locura), y radios variados.
  - Cualquier combinación de apertura, grosor por nodo, degradado entre surcos, quiebros, giro 3D, alejarse, hueco en los cruces, escalonados con remates y, a veces, perspectiva con doblez. La probabilidad de cada cosa sube con la locura.
  - El filtro de aberraciones sigue igual, y es lo que evita que salga una maraña rota.
  - Un estado guardado con «Cualquiera» pasa a «Libre» al cargarse.
- **Cualquier estilo** elige entre las recetas de los carteles:
  - **Aalto:** una pasada ancha con una o dos curvas grandes y, a veces, un quiebro en escalera (en el sentido que no cruza).
  - **Cortina coreana:** una a tres pasadas anchas que suben, con finales escalonados (ondas, curva o punta) y puntos. A veces lleva perspectiva de suelo con doblez; entonces la guía y el escalonado se calculan con `Ij` para que los finales caigan dentro del papel.
  - **Abanico (Bienal):** una pasada que se estrecha en el centro y se abre a los lados, o que gira como una cinta.
  - **Cintas que se alejan (cartel amarillo):** dos a cinco cintas que suben, se alejan (Tamaño al fondo) y a veces hacen una onda; las de encima dejan hueco.
  - **Raíces (*Downward root*):** una o dos pasadas finas que bajan y se abren hacia fuera, con finales escalonados.
- **Locura** (L): amplía los márgenes y la probabilidad de extras (más pasadas o surcos, quiebros, giros). Por encima del 60 %, a veces mezcla otra receta sin perspectiva.
- **Aberraciones** (A): cuántos avisos se toleran (`round(A·3)`; con el 100 %, todos).
  - Cada candidata se ajusta primero a las curvas, como «Ajustar a la curva» (`fitAndCount`), y se cuentan sus avisos.
  - Se prueban hasta 30 candidatas y se queda la primera que cumple o, si ninguna cumple, la que menos avisos tiene, con un mensaje.
- **Variar** parte de la composición actual y cambia un poco las posiciones, radios, aperturas, quiebros, giros, número de surcos, separación, grosor, escalonado y perspectiva; además, siempre cambia el azar de los extremos. Usa la locura como intensidad (mínimo 8 %) y el mismo filtro de aberraciones.
- Aleatorio y Variar ocupan una sola entrada del historial, así que un Cmd+Z vuelve a lo anterior.
- Las recetas sin perspectiva la desactivan; Variar la conserva.

## 9. Panel en pestañas

- Al crecer los controles, el panel dejó de caber. Ahora:
  - Cada control ocupa **una sola fila**: nombre, barra y casilla.
  - Hay **pestañas**: Pasada, Extremos, Fuga y Papel. Papel incluye el formato, la rejilla y las capas visibles.
  - El **nodo seleccionado** (X, Y, radio y apertura) va fijo encima de las pestañas, con borde azul, y solo aparece si hay un nodo seleccionado.
  - **Exportar** es un solo botón en la barra que abre un menú: Descargar SVG, Descargar PNG (con tamaño y transparencia en el mismo menú) y Copiar SVG. Se cierra al elegir algo, al pulsar fuera o con Esc. Los mensajes salen debajo del lienzo.
    - *Antes eran tres botones sueltos (SVG en azul, PNG y Copiar) y las opciones del PNG estaban en la pestaña Papel. Al usuario no le gustaba, y otra pestaña habrían sido demasiadas.*
- **Doble clic en una barra** la vuelve a su valor por defecto (atributo `data-def`): 100 % en apertura, grosor y último surco; 0 en quiebro, escalera, largo y azar.
- Con el foco en una barra o una casilla de verificación, Cmd/Ctrl+Z y Cmd/Ctrl+Y siguen deshaciendo y rehaciendo. En los campos de texto y los desplegables, no.
- El slider del radio es progresivo: el radio es máx · (s/1000)², donde máx es el doble del lado mayor del papel, o el radio actual si es mayor. Ni el slider ni el asa ponen tope.
  - *Una versión limitó el radio al que «cabía» sin quitar sitio a las vecinas, y le quitaba rango al asa. Se descartó.*

## 10. Limitaciones conocidas e ideas

- En **Mismo radio**, una curva en U más estrecha que la pasada da formas raras. Se avisa, pero no se corrige.
- El cuerpo de la pasada se detecta por su distancia al polígono de control, no a las curvas reales.
- No hay rotación ni simetría de pasadas.
- Los remates siguen la tangente en línea recta, no la curva.
- La apertura solo se edita con sliders; falta un asa en el lienzo para arrastrarla.
- Los cruces de una pasada consigo misma no dejan hueco.
- Con perspectiva, el asa del radio y los avisos se calculan en el plano, no en lo que se ve.
- No hay asa en el lienzo para mover el horizonte ni el punto de fuga; se hace con las barras.
- No hay aún **Aleatorio**.

## 11. Cómo se trabaja en el proyecto

- **Un único `index.html`**, sin dependencias ni compilación. JavaScript ES5 (`var`, `function`), igual que en Goma y clavos. Interfaz, comentarios y commits en español.
- GitHub Pages publica la rama `main` desde la raíz.
- Para probar, levanta un servidor estático en la carpeta y abre `http://localhost:8765`:

  ```bash
  python3 -m http.server 8765
  ```

- Commits en español, con una primera línea corta que describe el cambio.
