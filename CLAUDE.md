# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla + HTML5 Canvas. No hay `package.json`, build, linter ni tests: son solo tres archivos estáticos (`index.html`, `style.css`, `game.js`). El README y la UI están en español; mantén ese idioma en textos visibles al usuario.

## Ejecutar

Abrir `index.html` directamente o servir el directorio con cualquier servidor estático:

```bash
python3 -m http.server 8000   # luego http://localhost:8000
```

No existe comando de test; la verificación es manual en el navegador.

## Arquitectura (`game.js`, ~300 líneas, un solo archivo)

- **Estado global mutable** declarado en una sola línea con `let` (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …). Todas las funciones lo leen/escriben directamente; no hay clases ni módulos. `init()` lo reinicia todo y sirve también como handler del botón "Reiniciar".
- **Modelo**: `board` es una matriz `ROWS × COLS` con `0` (vacío) o un índice 1–7 que indexa a la vez `COLORS` y `PIECES`. Las piezas son matrices cuadradas; `rotateCW` (transposición + reverso) crea una nueva matriz y `tryRotate` aplica kicks horizontales `[0, -1, 1, -2, 2]`.
- **Ciclo de una pieza**: `spawn()` (promueve `next`, genera nueva `next`, llama a `endGame()` si colisiona al aparecer) → movimiento → `lockPiece()` = `merge()` + `clearLines()` + `spawn()`. `clearLines` también recalcula `level` y `dropInterval` (`max(100, 1000 − (level−1)·90)`).
- **Bucle**: `loop(ts)` con `requestAnimationFrame` acumula `dropAccum` y baja una fila al superar `dropInterval`; `draw()` redibuja cada frame (grid, tablero, ghost con alpha 0.2, pieza actual). Pausa y game over usan `cancelAnimationFrame(animId)`.
- **Entrada**: un único listener `keydown` sobre `document` (usa `e.code`: flechas, `KeyX`, `Space`, `KeyP`). `softDrop` suma 1 punto por fila, `hardDrop` 2 por celda.
- **DOM acoplado por IDs**: `game.js` obtiene `board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn` al cargar; si renombras un id en `index.html`, actualiza `game.js`.

## Puntos a tener en cuenta

- El tamaño del canvas está fijado en `index.html` (`#board` 300×600, `#next-canvas` 120×120). Si cambias `COLS`, `ROWS` o `BLOCK` en `game.js`, ajusta `width`/`height` a `COLS·BLOCK × ROWS·BLOCK`. La vista previa asume un área de 4×4 bloques de 30 px.
- `togglePause()` muestra el overlay al pausar, pero al reanudar no le añade la clase `hidden`; solo `init()` lo oculta. Tenlo presente si tocas la pausa.
- `spawn()` llama a `endGame()` (que hace `cancelAnimationFrame`) desde dentro de `loop`, y `loop` reprograma su siguiente frame después; comprueba `gameOver` si modificas ese flujo.
