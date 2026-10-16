# Arado

Editor de pasadas: haces de surcos (líneas paralelas) que siguen una guía con esquinas redondeadas, con extremos escalonados. Exporta SVG por capas y PNG.

- Antes de cambiar el comportamiento, lee **[DECISIONES.md](DECISIONES.md)**: explica el modelo, la interacción y las **fases pendientes** (§6). Si cambias algo de lo que describe, actualízalo.
- Todo vive en `index.html`, sin dependencias ni compilación. Hay tres bloques: `AR` (geometría de los surcos), `AX` (exportador SVG) y la aplicación. Usa ES5, como el resto del archivo. La interfaz, los comentarios y los commits van en español.
- Para probar, levanta `python3 -m http.server 8765` en esta carpeta y abre `http://localhost:8765`. Comprueba en el navegador cualquier cambio visible.
- Clave de `localStorage`: `arado-v1`. No leas las claves de Mesa de tangentes ni de Goma y clavos: comparten origen en `meegl.github.io`.
