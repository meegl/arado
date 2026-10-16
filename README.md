# Arado

Editor visual de **pasadas**: haces de **surcos** (líneas paralelas) que siguen una guía con esquinas redondeadas. Sirve para hacer la base de carteles de líneas que se curvan y dan quiebros, y acabarlos en Illustrator o Affinity. Exporta a **SVG por capas** o **PNG**.

Es de la misma familia que [Mesa de tangentes](https://github.com/meegl/mesa-de-tangentes) y [Goma y clavos](https://github.com/meegl/goma-y-clavos).

## Uso

- **Nueva pasada** (`P`): haz clic en el lienzo para poner los nodos de la guía, uno tras otro.
- **Añadir** (`A`):
  - Un clic en un hueco alarga la pasada seleccionada por su extremo.
  - Un clic sobre un tramo de la guía inserta un nodo.
- **Mover** (`V`):
  - Arrastra nodos o la pasada entera.
  - El punto azul de cada esquina cambia su radio.
- **Pasada:** número de surcos, separación, grosor y terminación (recta, redonda o cuadrada).
- **Esquinas:**
  - *Concéntricas:* los arcos comparten centro y los surcos interiores pueden quedar vivos.
  - *Mismo radio:* cada surco se redondea igual.
- **Extremos** (inicio y fin por separado): escalonado con perfil recto, diagonal, punta, curva u ondas, variación al azar y remates de puntos o guiones. El inicio es el nodo redondo.
- **Grosor variable:** en degradado de un surco a otro, o a lo largo de la guía con un grosor por nodo (entonces los surcos se exportan como formas rellenas).
- **Fuga** (apertura por nodo): los surcos se juntan, se abren en abanico o convergen en un punto.
- Los nodos pueden salir del papel para que las líneas sangren.
- Deshacer y rehacer (`Ctrl+Z`), quitar selección (`Esc`), borrar nodo o pasada (`Supr`), guías (`G`).

## Exportar

SVG en capas con nombre: `fondo`, `rejilla` y un grupo por pasada (`pasada-1`, `pasada-2`…), con un surco por trazado y los remates en un subgrupo. Los surcos son **trazos editables**, así que puedes cambiar su grosor y color en el programa de vectores.

## Cómo está hecho

Un único `index.html`, sin dependencias ni paso de compilación. Los surcos se calculan de forma exacta, con rectas y arcos. Más detalles en [DECISIONES.md](DECISIONES.md).

Licencia: MIT.
