# Cortex — CHANGELOG de parches (repo: corpus-cortex/)

> Historial de parches del repo semilla `corpus-cortex/`. Primer archivo de proceso del
> repo: hasta hoy solo había `LICENSE`, `README.md` y `docs/Cortex_ContratosEntrantes.md`
> (el módulo todavía no abrió su Block — ni código, ni `CLAUDE.md`).
>
> **Disciplina (heredada del ecosistema Corpus):**
> - Un parche nace `[PENDIENTE]` y pasa a `[APLICADO YYYY-MM-DD]` cuando se aplica y verifica.
>   Para prosa de doc, "verificado" = revisado contra su sede/árbol; no hay superficie de
>   runtime en este repo todavía.
> - **Nunca** se borra una entrada. **Nunca** se renumera un parche existente.
> - Cada sesión abre su **propia subsección**, con numeración independiente.
> - Este CHANGELOG es de **este repo** (`corpus-cortex/`). Cada repo hermano tiene el suyo.

---

## PARCHES DE sesión Reparación del gate de coherencia (acta 2026-07-22) — 2026-07-22

Tanda de reparación documental propuesta por el gate de coherencia en su corrida COMPLETO del
2026-07-22 (`../../corpus/docs/auditorias/2026-07-22_coherencia_docs.md`; el gate propone, el
autor dispone). Este archivo nace en esta misma tanda: por voto del autor, los dos hallazgos
sobre `Cortex_ContratosEntrantes.md` se registran acá en vez de quedar solo en el resumen.
Solo prosa; **ninguna norma cambió de contenido** (este repo no acuña `CTX-` todavía).

- PARCHE 1 — **Hallazgo 2.10 del acta (pase de valor):** `docs/Cortex_ContratosEntrantes.md`
  decía que las cinco firmas entrantes «vivían dispersas en **cuatro repos** distintos». Sus 6
  filas (§2) tienen sede en solo TRES repos: corpus-cargo (filas 1-3), corpus-caliber (filas
  4,6), corpus-stalker (fila 5). El 4 coincidía con la cantidad de DOCUMENTOS, no de repos.
  Corregido a «**tres repos** (corpus-cargo, corpus-caliber, corpus-stalker), repartidas en
  cuatro documentos». **[APLICADO 2026-07-22]**
- PARCHE 2 — **Hallazgo 2.11 del acta (pase de valor):** el mismo doc afirmaba que
  `corpus-cortex/` «contiene **solo** `LICENSE` y `README.md`». `find` devuelve más: `LICENSE`,
  `README.md`, el propio `Cortex_ContratosEntrantes.md` y —desde esta tanda— este `CHANGELOG.md`.
  Corregida la enumeración a esos cuatro, con «ni una línea de código, ni `CLAUDE.md`». La
  mención a este `CHANGELOG.md` es la reconciliación por haberlo creado en esta misma sesión (el
  acta, foto del 2026-07-22, listaba tres archivos porque este todavía no existía). **[APLICADO
  2026-07-22]**

Verificación: sin superficie de runtime (repo sin código). Cambios trazables al acta; el árbol
es árbitro de nivel 1 (§7.1). No commiteado ni pusheado (GIT-7).
