# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris clásico en JavaScript vanilla + HTML5 Canvas + CSS. Sin dependencias, sin build, sin package.json.

## Running

No hay build ni test suite. Abrir directo o servir estático:

```bash
xdg-open index.html          # o open (macOS) / start (Windows)
python3 -m http.server 8000  # alternativa: npx serve .
```

Testear cambios abriendo `index.html` en navegador y jugando (teclado: flechas mover/rotar, espacio hard drop, P pausa).

## Architecture

Tres archivos, todo vive en `game.js` (~300 líneas, sin módulos):

- **`index.html`** — DOM: `<canvas id="board">` (300×600, 10×20 celdas de 30px), canvas de preview `next-canvas`, panel de score/lines/level, overlay pausa/game-over.
- **`style.css`** — tema dark/retro arcade.
- **`game.js`** — toda la lógica, estado global en variables `let` top-level (`board, current, next, score, lines, level, paused, gameOver, ...`).

Flujo:

```
init() → createBoard() + randomPiece() + spawn() → requestAnimationFrame(loop)
loop(ts) → acumula dt → si dt ≥ dropInterval baja pieza o lockPiece() → draw() → recurse
lockPiece() → merge() al board → clearLines() → spawn() (spawn colisiona → endGame())
```

Puntos clave para modificar sin romper nada:

- **Tablero**: matriz `ROWS × COLS`, celda = `0` (vacía) o índice 1–7 (tipo de pieza, indexa `COLORS`).
- **Piezas** (`PIECES`): matrices cuadradas fijas. Rotación vía `rotateCW` (transposición + reverso), wall kicks en `tryRotate` probando offsets `[0,-1,1,-2,2]`.
- **Colisión** (`collide(shape, ox, oy)`): única fuente de verdad para bordes y solapamiento; se reusa en movimiento, rotación, ghost y game loop — no duplicar esta lógica.
- **Velocidad/nivel**: `level = floor(lines/10)+1`; `dropInterval = max(100, 1000-(level-1)*90)`.
- **Puntuación**: `LINE_SCORES = [0,100,300,500,800]` × nivel; hard drop = 2 pts/celda, soft drop = 1 pt/fila.
- Si se cambia `COLS`, `ROWS` o `BLOCK` en `game.js`, hay que ajustar también el `width`/`height` del `<canvas id="board">` en `index.html` para que coincidan (`COLS×BLOCK`, `ROWS×BLOCK`).
