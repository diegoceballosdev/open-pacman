# SPEC 02 — Salida de los fantasmas fuera de la pen

> **Estado:** Implemented
> **Depende de:** SPEC 01
> **Fecha:** 2026-09-03
> **Objetivo:** Los 4 fantasmas nacen fuera de la pen (fila 11, x 12–15) y el tile 3 es muro para todos.

## Scope

**In:**

- Cambiar `GHOST_STARTS` en `src/js/maze.js`: misma fila de `kind`, `y` de 14 a 11.
- En `isWall` (`src/js/game.js`), el tile 3 (puerta) es muro para cualquier actor, no solo Pac-Man.
- `resetPositions` sigue usando `GHOST_STARTS`: tras perder una vida, las mismas celdas que al iniciar.

**Out of scope (for future specs):**

- Salir andando por la puerta (AI de “house mode”).
- Teletransporte al pulsar Start.
- Puerta unidireccional.
- Salida escalonada por tiempo.
- Volver a la pen al ser comidos (eyes).
- Cambiar personalidades, velocidades o colores.
- Power pellets y modo frightened.

## Data model

`GHOST_STARTS` en `src/js/maze.js` queda así:

```js
const GHOST_STARTS = [
  { x: 12, y: 11, kind: "hunter" },
  { x: 13, y: 11, kind: "ambusher" },
  { x: 14, y: 11, kind: "flanker" },
  { x: 15, y: 11, kind: "shy" },
];
```

No hay estructuras nuevas. Cada fantasma en `createGame` sigue siendo `{ x, y, dir: 'up', speed: GHOST_SPEED, kind }`.

`isWall` trata el tile 3 como muro sin filtrar por actor:

```js
if (v === 3) return true;
```

## Implementation plan

1. En `src/js/maze.js`, poner `y: 11` en las 4 entradas de `GHOST_STARTS`. Comprobar a ojo: cuatro fantasmas encima de la pen, no se apilan, se mueven por el laberinto.
2. En `src/js/game.js`, en `isWall`, el tile 3 es muro para todos. Comprobar: ningún fantasma entra en la pen; Pac-Man sigue sin pasar la puerta.

## Acceptance criteria

- [ ] Al iniciar, las posiciones son hunter (12,11), ambusher (13,11), flanker (14,11), shy (15,11).
- [ ] Tras perder una vida, los 4 vuelven a esas mismas celdas.
- [ ] Los 4 se mueven por el laberinto; no quedan atrapados en la pen.
- [ ] Un fantasma no puede pasar el tile 3.
- [ ] Pac-Man no puede pasar el tile 3.
- [ ] Los `kind`, `GHOST_SPEED`, colores por índice y la regla de no 180° salvo callejón no cambian.
- [ ] No hay timers de salida, eyes, frightened ni nombres en el HUD.

## Decisions

- **Sí:** cambiar `GHOST_STARTS` a la fila 11. Cero lógica nueva de pathfinding; no se atascan.
- **No:** nacer en la pen y teletransportar al Start. Extra estado para el mismo resultado visual en partida.
- **No:** AI que apunte a la puerta. Más frágil con el greedy y la regla de no 180°.
- **Sí:** x 12–15 en y=11. Encima de la pen, cuatro celdas distintas, mismo orden que SPEC 01.
- **Sí:** `resetPositions` usa las mismas celdas. Un solo spawn.
- **Sí:** tile 3 es muro para fantasmas. Si reentran, se atascan otra vez; la pen queda decorativa.
- **No:** puerta solo de salida. Más lógica de la necesaria.
- **Sí:** esto sustituye el criterio de SPEC 01 “los fantasmas sí pasan la puerta” y “salen de la pen en cuanto pueden”.

## Risks

| Risk                                                    | Mitigation                                                                |
| ------------------------------------------------------- | ------------------------------------------------------------------------- |
| `dir: 'up'` inicial choca con muro en (13,10) y (14,10) | `decideGhost` ya elige otra dirección legal; no hace falta cambiar `dir`. |
| Un fantasma greedy podría querer entrar en la pen       | Tile 3 es muro; no hay camino hacia dentro.                               |

## What is **not** in this spec

- Salida andando por la puerta.
- Teletransporte al Start.
- Puerta unidireccional.
- Salida escalonada.
- Eyes / volver a la pen.
- Cambios de personalidad, velocidad o color.

Cada uno de esos, si entra, va en su propia spec.
