# Cortex

Módulo de **IA de NPC** del ecosistema [Corpus](https://github.com/Sepuldosky/corpus) para
**Garry's Mod**: táctica de combate y afecto — dolor y miedo que cambian el comportamiento.
Addon independiente que **hard-depende** de Corpus (la única dependencia dura del ecosistema)
y detecta a los demás módulos en runtime, nunca los asume.

> **Estado: sin empezar.** Este repo aún no tiene código. Cortex espera su bloque de diseño,
> gated por la superficie de eventos de daño/extremidad que
> [Caliber](https://github.com/Sepuldosky/corpus-caliber) expondrá con su pipeline de jugador
> (Block 3). El rumbo del ecosistema vive en el
> [roadmap de Corpus](https://github.com/Sepuldosky/corpus/blob/main/docs/corpus_roadmap.txt).

## Dependencias previstas

- **Corpus** (dura — framework del ecosistema).
- **Caliber** (soft — eventos de daño por zona para dolor/miedo). Sin él, Cortex degrada a
  comportamiento táctico con reacción genérica al daño, sin detalle por extremidad.

Diseño de referencia del ecosistema y grafo de dependencias →
[`CORPUS_Architecture.md`](https://github.com/Sepuldosky/corpus/blob/main/docs/CORPUS_Architecture.md)
(§1-§2, §9).
