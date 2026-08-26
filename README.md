<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/cortex_lockup_dark.svg">
    <img src="assets/cortex_lockup_light.svg" width="200" alt="Cortex">
  </picture>
</p>

# Cortex

**NPC AI** module of the [Corpus](https://github.com/Sepuldosky/corpus) ecosystem for
**Garry's Mod**: squads, combat tactics, and affect — pain and fear that change behavior.
Independent addon that **hard-depends** on Corpus (the ecosystem's only hard dependency) and
detects the other modules at runtime, never assumes them.

> **Status: in code, first vertical slice.** The open block is **squads**: giving tactical
> orders to VJ Base and HL2 NPCs, team-leader style. It's a **layer over NPC bases that
> already exist**, not a new NPC base.
>
> The slice covers body arbitration, a minimal roster, one order end-to-end (*Move To*) with
> its polling tick, the player notice, and the scavenger gate in Caliber — five Lua files,
> written and verified offline, but **not yet confirmed in-game**. The public contract so far
> is `CORTEX.Body.*` only.
>
> **Affect** (pain and fear) still waits separately: it needs zone damage events that
> [Caliber](https://github.com/Sepuldosky/corpus-caliber) doesn't expose yet. Two halves on
> different timelines, and only one is still waiting.

The module's design lives in [`docs/`](docs/); the ecosystem's direction lives in the
[Corpus roadmap](https://github.com/Sepuldosky/corpus/blob/main/docs/corpus_roadmap.txt).

## Planned dependencies

- **Corpus** (hard — the ecosystem's framework).
- **Caliber** (soft — zone damage events for pain/fear). Without it, Cortex degrades to
  tactical behavior with generic reaction to damage, without per-limb detail.

Reference design for the ecosystem and the dependency graph →
[`CORPUS_Architecture.md`](https://github.com/Sepuldosky/corpus/blob/main/docs/CORPUS_Architecture.md)
(§1-§2, §9).
