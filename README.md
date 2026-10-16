# Arado

Editor visual de **peines**: haces de líneas paralelas que siguen una guía con esquinas redondeadas. Sirve para hacer la base de carteles de líneas que se curvan y dan quiebros, y acabarlos en Illustrator o Affinity. Exporta a **SVG por capas** o **PNG**.

Es de la misma familia que [Mesa de tangentes](https://github.com/meegl/mesa-de-tangentes) y [Goma y clavos](https://github.com/meegl/goma-y-clavos).

## Uso

- **Nuevo peine** (`P`): haz clic en el lienzo para poner los nodos de la guía, uno tras otro.
- **Añadir** (`A`):
  - Un clic en un hueco alarga el peine seleccionado por su extremo.
  - Un clic sobre un tramo de la guía inserta un nodo.
- **Mover** (`V`):
  - Arrastra nodos o el peine entero.
  - El punto azul de cada esquina cambia su radio.
- **Peine:** número de púas, separación, grosor y remate (recto, redondo o cuadrado).
- **Esquinas:**
  - *Concéntricas:* los arcos comparten centro y las púas interiores pueden quedar vivas.
  - *Mismo radio:* cada púa se redondea igual.
- Los nodos pueden salir del papel para que las líneas sangren.
- Deshacer y rehacer (`Ctrl+Z`), quitar selección (`Esc`), borrar nodo o peine (`Supr`), guías (`G`).

## Exportar

SVG en capas con nombre: `fondo`, `rejilla` y un grupo por peine (`peine-1`, `peine-2`…), con una púa por trazado. Las púas son **trazos editables**, así que puedes cambiar su grosor y color en el programa de vectores.

## Cómo está hecho

Un único `index.html`, sin dependencias ni paso de compilación. Las púas se calculan de forma exacta, con rectas y arcos. Más detalles en [DECISIONES.md](DECISIONES.md).

Licencia: MIT.
