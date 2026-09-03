# SPEC 01 — Cuatro fantasmas con personalidades distintas

> **Estado:** Implementado
> **Depende de:** ninguna
> **Fecha:** 2026-09-03
> **Objetivo:** El juego tiene cuatro fantasmas con `kind` distintos (`hunter`, `ambusher`, `flanker`, `shy`) y `hunter` persigue a Pac-Man con Manhattan greedy.

## Scope

**In:**

- Ampliar `GHOST_STARTS` en `src/js/maze.js` de 2 a 4 entradas, todas en la fila 14 de la pen.
- Sustituir el `kind: 'random'` por tres personalidades nuevas en `decideGhost` (`src/js/game.js`).
- Conservar la regla actual: no giro de 180° salvo callejón sin salida.
- Conservar la salida inmediata de la pen (sin timers).
- Conservar `GHOST_COLORS` por índice en `src/js/render.js` (rojo, cian, rosa, naranja).

**Out of scope (for future specs):**

- Power pellets y modo frightened.
- Ciclos scatter/chase del arcade.
- Velocidades distintas por fantasma.
- Salida escalonada por tiempo.
- Nombres visibles (Blinky/Pinky/Inky/Clyde) en HUD o overlay.
- Reordenar colores para coincidir con el arcade.

## Data model

`GHOST_STARTS` en `src/js/maze.js` queda así:

```js
const GHOST_STARTS = [
  { x: 12, y: 14, kind: "hunter" },
  { x: 13, y: 14, kind: "ambusher" },
  { x: 14, y: 14, kind: "flanker" },
  { x: 15, y: 14, kind: "shy" },
];
```

Cada fantasma en `createGame` sigue siendo:

```js
{ x, y, dir: 'up', speed: GHOST_SPEED, kind }
```

Constantes nuevas en `src/js/game.js`:

```js
const AMBUSHER_OFFSET = 4;
const SHY_RANGE = 8;
```

Celdas objetivo (solo para elegir dirección; no tienen que ser transitables):

- `hunter`: `(round(pac.x), round(pac.y))` — igual que ahora.
- `ambusher`: 4 celdas en la dirección actual de Pac-Man: `(px + 4 * DIRS[p.dir].x, py + 4 * DIRS[p.dir].y)`.
- `flanker`: espejo horizontal `(grid[0].length - 1 - px, py)`.
- `shy`: si la Manhattan al Pac-Man redondeado es `>= 8`, mismo objetivo que `hunter`; si no, elección aleatoria entre `choices` (el `else` actual).

`src/js/render.js` no cambia. El color sigue siendo `GHOST_COLORS[i]`.

## Implementation plan

1. En `src/js/maze.js`, reemplazar `GHOST_STARTS` por las 4 entradas del modelo. `createGame` ya mapea el array entero: al abrir el juego se ven 4 fantasmas. Los `kind` aún no reconocidos caen en el `else` (random). Comprobar a ojo: cuatro colores, no se apilan.
2. En `src/js/game.js`, extraer la elección greedy hacia un punto `(tx, ty)` a un helper usado por `hunter`. `hunter` no cambia de comportamiento. Comprobar: el rojo sigue persiguiendo.
3. Añadir `AMBUSHER_OFFSET` y el caso `ambusher` en `decideGhost`. Comprobar: el cian se adelanta a Pac-Man en vez de ir a su celda.
4. Añadir el caso `flanker` (objetivo espejo). Comprobar: el rosa cruza hacia el lado opuesto del laberinto.
5. Añadir `SHY_RANGE` y el caso `shy`. Comprobar: el naranja persigue de lejos y vaga al acercarse (~8 celdas).

## Acceptance criteria

- [ ] Al iniciar una partida hay exactamente 4 fantasmas.
- [ ] Posiciones iniciales: hunter (12,14), ambusher (13,14), flanker (14,14), shy (15,14).
- [ ] Colores por índice: rojo, cian, rosa, naranja.
- [ ] Los 4 salen de la pen en cuanto pueden, sin espera.
- [ ] `hunter` elige la dirección legal (sin 180° salvo callejón) que minimiza Manhattan a Pac-Man.
- [ ] `ambusher` elige la dirección legal que minimiza Manhattan a 4 celdas delante de Pac-Man.
- [ ] `flanker` elige la dirección legal que minimiza Manhattan a `(ancho - 1 - px, py)`.
- [ ] `shy` persigue como `hunter` si Manhattan a Pac-Man `>= 8`; si no, elige al azar entre las direcciones legales.
- [ ] Ningún fantasma hace giro de 180° salvo callejón sin salida.
- [ ] `GHOST_SPEED` no cambia. Pac-Man no pasa la puerta (tile 3); los fantasmas sí.
- [ ] No hay power pellets, frightened, scatter, ni nombres en el HUD.

## Decisions

- **Sí:** cuatro `kind` clásicos simplificados (`hunter` / `ambusher` / `flanker` / `shy`). Encajan en `decideGhost` sin timers ni targeting entre fantasmas.
- **No:** targeting fiel de Inky (vector desde Blinky). Acopla dos actores y es más frágil.
- **Sí:** umbral de `shy` = 8 celdas Manhattan, como Clyde.
- **Sí:** objetivo de `flanker` = espejo horizontal de Pac-Man. Distinto de hunter y ambusher, sin depender de otro fantasma.
- **Sí:** los 4 nacen en y=14 (x 12–15) y salen a la vez. Igual que el juego actual; no introduce relojes.
- **Sí:** identidad solo por `kind` + color por índice. Sin nombres en pantalla.
- **No:** reordenar `GHOST_COLORS` al arcade (Pinky rosa, Inky cian). El usuario eligió los colores actuales.
- **No:** power pellets, frightened, scatter/chase, velocidades distintas. Cada uno es otra spec.
- **Sí:** el `else` de `decideGhost` sigue siendo random, como fallback si llega un `kind` desconocido.

## Risks

| Risk                                                   | Mitigation                                                                  |
| ------------------------------------------------------ | --------------------------------------------------------------------------- |
| Cuatro fantasmas en la misma fila se bloquean al salir | Celdas distintas (x 12–15). La pen tiene hueco; `canMove` ya evita paredes. |
| Objetivo de ambusher/flanker fuera del mapa o en muro  | Solo sirve para comparar Manhattan; no se camina a esa celda.               |
| `shy` parece random casi siempre si el umbral es bajo  | Umbral 8 en este laberinto 28×31; se nota el cambio de modo.                |

## What is **not** in this spec

- Power pellets y modo frightened.
- Ciclos scatter/chase.
- Velocidades distintas por fantasma.
- Salida escalonada.
- Nombres clásicos en la UI.
- Reordenar colores al arcade.

Cada uno de esos, si entra, va en su propia spec.
