# SPEC 03 — Power pellets y modo frightened

> **Estado:** implementado
> **Depende de:** SPEC 02
> **Fecha:** 2026-09-03
> **Objetivo:** Hay 4 power pellets (tile 4) que activan 360 frames de frightened: Pac-Man come fantasmas (200 pts) y estos reaparecen en `GHOST_STARTS`.

## Scope

**In:**

- En `src/js/maze.js`, char `'o'` → tile 4 en las 4 esquinas arcade: (1,3), (26,3), (1,23), (26,23).
- Comer tile 4: celda a 0, +50 puntos, `dotsRemaining--`, `frightenedFrames = 360` (recarga si ya estaba activo).
- `createGame` cuenta tile 2 y tile 4 en `dotsRemaining`. Hay que comer los 4 pellets para ganar.
- Timer global `game.frightenedFrames`, un frame por `update`. Sin `dt`.
- Al activar frightened, `g.dir = OPPOSITE[g.dir]` en cada fantasma.
- Mientras `frightenedFrames > 0`, `decideGhost` usa el branch random (el `else` actual). Misma regla de no 180° salvo callejón.
- Colisión con `frightenedFrames > 0`: +200, ese fantasma vuelve a `GHOST_STARTS[i]` con `dir: 'up'`. El timer no se corta. Sigue comible.
- Colisión con timer a 0: igual que ahora (vida / `resetPositions` / `lost`).
- En `src/js/render.js`, tile 4 es un círculo más grande que el dot. Fantasmas azul sólido si `frightenedFrames > 0`.
- `GHOST_SPEED` no cambia. `main.js` no cambia.

**Out of scope (for future specs):**

- Eyes / volver a la pen.
- Parpadeo de aviso al final del timer.
- Combo 200/400/800/1600.
- Ciclos scatter/chase.
- Velocidad distinta en frightened.
- Invulnerabilidad tras respawn.
- Indicador de frightened en el HUD.
- Sonido.

## Data model

Tile nuevo en `src/js/maze.js`:

```js
// parseTile: 'o' → 4
```

`MAZE_STR` filas 3 y 23:

```js
'#o####.#####.##.#####.####o#', // 3
'#o..##................##..o#', // 23
```

Constantes en `src/js/game.js`:

```js
const FRIGHTENED_FRAMES = 360;
const PELLET_SCORE = 50;
const GHOST_EAT_SCORE = 200;
```

Estado de partida: un campo nuevo, timer global (no hay flag por fantasma):

```js
frightenedFrames: 0;
```

Cada fantasma sigue siendo `{ x, y, dir, speed, kind }`.

`createGame` cuenta collectibles así:

```js
if (v === 2 || v === 4) dots++;
```

En `src/js/render.js`:

```js
const FRIGHTENED_COLOR = "#2121ff";
// tile 4: mismo DOT_COLOR, radio 6 (el dot sigue en 2.5)
```

Orden en `update`: decrementar `frightenedFrames` si `> 0`; `movePacman` (puede poner el timer a 360); `moveGhost`; colisiones.

## Implementation plan

1. En `src/js/maze.js`, `'o'` → 4 en `parseTile` y las 4 celdas de `MAZE_STR`. Tile 4 no es muro. Comprobar: las 4 esquinas ya no tienen dot pequeño; se puede pasar por ellas.
2. En `src/js/render.js`, dibujar tile 4 (radio 6, `DOT_COLOR`) dentro de `drawDots` o un `drawPellets` llamado desde `draw`. Comprobar: 4 pellets grandes en las esquinas.
3. En `src/js/game.js`, contar tile 2 y 4; al aligned, si la celda es 4: `= 0`, `score += 50`, `dotsRemaining--`. Comprobar: el pellet desaparece, +50, no se gana si queda alguno.
4. Añadir `frightenedFrames` y `FRIGHTENED_FRAMES`. Al comer el pellet, timer = 360 e invertir `dir` de los 4. Al inicio de `update`, si `frightenedFrames > 0`, restar 1. En `decideGhost`, si el timer `> 0`, random. En `draw`, si el timer `> 0`, `FRIGHTENED_COLOR`. Comprobar: azules ~6 s a 60 Hz, recarga con otro pellet, luego vuelven al color por índice. La colisión aún mata a Pac-Man.
5. En el loop de colisión: si timer `> 0`, +200 y respawn de ese fantasma (`GHOST_STARTS[i]`, `dir: 'up'`), sin `break`; si timer es 0, la muerte actual. Comprobar: comer un fantasma azul no quita vida; reaparece en fila 11; se puede volver a comer; sin frightened, un toque sigue costando vida.

## Acceptance criteria

- [ ] Al iniciar hay pellets en (1,3), (26,3), (1,23), (26,23) y no dots en esas celdas.
- [ ] Tile 4 es transitable. Pac-Man y fantasmas no lo tratan como muro.
- [ ] Comer un pellet: celda 0, +50, `dotsRemaining` baja 1, `frightenedFrames = 360`.
- [ ] Comer un pellet con el modo ya activo recarga a 360. No se apila por encima.
- [ ] A 60 Hz el modo dura ~6 s. En `update` se resta 1 por frame.
- [ ] Al activar frightened, cada fantasma invierte `dir`.
- [ ] Mientras el timer `> 0`, los 4 eligen random entre `choices` (sin 180° salvo callejón) y se dibujan `#2121ff`.
- [ ] Al llegar a 0, recuperan `kind` y `GHOST_COLORS[i]`.
- [ ] Colisión con timer `> 0`: +200, ese fantasma a `GHOST_STARTS[i]` con `dir: 'up'`, Pac-Man no pierde vida, el timer sigue, el fantasma se puede volver a comer.
- [ ] Colisión con timer 0: igual que SPEC 02 (vida / reset / `lost`).
- [ ] `dotsRemaining` inicial incluye los 4 pellets. No se gana si queda un pellet.
- [ ] `GHOST_SPEED`, `kind`, spawn fila 11 y tile 3 como muro no cambian.
- [ ] No hay eyes, parpadeo, combo, scatter, HUD de frightened ni sonido.

## Decisions

- **Sí:** 4 esquinas arcade, tile 4, char `'o'`. Sustituyen dots que ya estaban ahí.
- **Sí:** timer global `frightenedFrames`, no un flag por fantasma. Un solo reloj.
- **Sí:** 360 frames, recarga al comer otro pellet. El juego ya es frame-based (`PACMAN_SPEED` por frame), no se introduce `dt`.
- **Sí:** al comer un fantasma, respawn instantáneo en `GHOST_STARTS`. SPEC 02 dejó eyes fuera.
- **Sí:** sigue comible tras respawn. Sin invulnerabilidad; el camping del spawn se acepta.
- **Sí:** 180° una vez al entrar, después random. El `else` de `decideGhost` ya existe.
- **No:** 180° al salir del modo. El usuario solo pidió reverse al entrar.
- **Sí:** azul sólido `#2121ff`. Sin parpadeo.
- **Sí:** pellet 50, fantasma 200 fijo. Sin cadena arcade.
- **Sí:** tile 4 cuenta para ganar. Si no, se gana con pellets en el mapa.
- **Sí:** misma `GHOST_SPEED`. Speeds distintas son otra spec (`aligned()`).
- **No:** eyes, scatter, combo, flash, HUD, sonido. Cada uno va en su spec.
- **Sí:** `main.js` no se toca. El timer vive en `update`.

## Risks

| Risk                                                        | Mitigation                                                                                                                              |
| ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| rAF a 120 Hz acorta el modo a ~3 s                          | Igual que el resto del juego (speeds por frame). No meter `dt` aquí.                                                                    |
| Pac-Man acampa fila 11 y come en bucle                      | Aceptado. Sin invulnerabilidad en esta spec.                                                                                            |
| Fantasma aligned el mismo frame: `decideGhost` pisa el 180° | El reverse se asigna en `movePacman`; si `moveGhost` está aligned, el random puede sustituirlo. Se nota sobre todo si no están aligned. |
| `#2121ff` es también `WALL_COLOR`                           | Los fantasmas se dibujan en pasillos, no encima de paredes.                                                                             |

## What is **not** in this spec

- Eyes / volver a la pen.
- Parpadeo al acabar frightened.
- Combo 200/400/800/1600.
- Scatter/chase.
- Velocidad distinta en frightened.
- Invulnerabilidad post-respawn.
- HUD o sonido de frightened.

Cada uno de esos, si entra, va en su propia spec.
