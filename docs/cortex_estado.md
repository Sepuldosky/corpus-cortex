# Cortex — Estado de HOY

> **Foto del AHORA**, volátil. Es lo primero que se lee al retomar el módulo —
> **antes** que el doc de arquitectura. Se actualiza **en sitio** (no se agregan
> secciones ni historial). El historial vive en `git` + [`CHANGELOG.md`](CHANGELOG.md).
> Si crece de una pantalla, está mal redactado: recortar.

**Última actualización:** 2026-08-25. **Cerró el bloque de DISEÑO del escuadrón: puntos [3] y [4] del roadmap.** Se contestó el **contrato entrante #6** —la última fila que este repo tenía pendiente de decidir— y las **tres preguntas abiertas** que el censo no había cerrado, así que **las cuatro del escuadrón están contestadas**. El diseño se desprendió a **doc particular** ([`Cortex_Escuadrones_Arquitectura.md`](Cortex_Escuadrones_Arquitectura.md)) por el criterio del flujo §2, y §5 del general quedó como resumen + link. **Cinco normas nuevas (CTX-8..CTX-12), y van DOCE.** ⚠ En el mismo parche se corrigió que **CTX-6 y CTX-7 vivían dentro del bloque `CRG` del registro**. **Sigue sin una línea de código**, y sigue siendo a propósito: lo que cerró hoy es lo que hacía falta para escribirlo bien.

---

## Qué existe hoy

- **El repo está fundado**, con `CLAUDE.md` + los docs del template. La familia de normas
  tiene sede y **doce entradas** en el registro del ecosistema.
- ⭐ **El DISEÑO DEL ESCUADRÓN está escrito**, en doc particular:
  [`Cortex_Escuadrones_Arquitectura.md`](Cortex_Escuadrones_Arquitectura.md). Nueve secciones:
  el arbitraje del cuerpo, quién integra el roster, el modelo de estado completo, los grupos,
  la formación, la cola y su tick, y qué pasa cuando alguien muere. **Es el doc que hay que
  leer para implementar**; §5 del general es su resumen.
- **Los seis contratos ENTRANTES** en [`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md),
  y **el último que faltaba decidir —el #6— está CONTESTADO** desde el 2026-08-25. Doc de
  **recepción, sin autoridad**: cada fila tiene su sede en otro repo y nada se re-deriva ahí.
- **Las convenciones de commit**, con dos alcances en uso y **diez reservados** (`escuadra`
  entre ellos, sumado al abrir el bloque).
- **La identidad visual** (glifo y lockup, light y dark) y el README público.
- **El alcance del bloque, votado el 2026-08-24:** escuadrones para NPCs de **VJ Base y HL2**;
  DRG y ZBase **diferidas**. Es una **capa** sobre bases que ya existen, no una base nueva.

## Lo que NO existe, y conviene no leerlo de más

- **Cero código.** No hay `lua/`, ni entry point, ni registro contra el framework. **Nada de lo
  que dice el doc del escuadrón corre todavía** — el arbitraje de CTX-9 es una firma escrita,
  no una función que exista.
- **Ninguna de las doce normas tiene evidencia de tipo `planilla`.** Cinco citan código de
  **otros** repos; el resto es `INTENCION`. Convierten en el punto [5], no antes.
- **CortexBase no está diseñada** y no es lo que se abrió. Sigue en el plan, **debajo** de la
  capa y después.

## Bloqueado por quién — y qué NO está bloqueado

- **El afecto (dolor, miedo) está gated por Caliber**, cuyo Block 3 no cerró: la superficie
  de eventos de daño/extremidad **no existe todavía**. El hook que hay hoy
  (`Caliber_LimbsUpdated`) es **un aviso de refresh, no un evento de daño** — su payload es
  `(npc, reason)`, sin zona y sin cuánto bajó el pool.
- ⭐ **La táctica y el escuadrón NO están bloqueados por eso**, y es lo que hizo que este
  bloque pudiera abrir y cerrar.
- **La orden `Deploy` y la secuencia *throw flashbang and clear* están bloqueadas por
  Cargo**, no por Caliber: hoy **un NPC no puede ser dueño de un inventario** (`OwnerKey`
  llama a `SteamID64`, que es método de `Player` y no de `Entity` — con un NPC revienta, no
  cae en el `or`). Abrir esa puerta es un voto de Cargo.
- **El campo «quién ejecuta» del mensaje de net sigue abierto y arrastra**: lleva
  `(id, entidad, component)` y una orden de escuadra tiene **dos** objetivos. Es del
  **framework**, se trabaja en otra sesión, y **bloquea el cable, no el diseño**.
- **Qué es una «puerta»** para el sistema. Sin esa definición no existen las cinco órdenes de
  puerta — y una de ellas depende además de capabilities que nadie censó.

## Lo que este bloque contestó — 2026-08-25

- **Contrato #6: el scavenger ES behavior — pero NO se re-homea** (**CTX-8**). Autoridad, no
  hosting. ⭐ Y la medición dio vuelta dos cosas: el archivo **no es una cosa, son tres**, y el
  acoplamiento **va al revés** de lo que decían dos docs — el scavenger **no llama a limbs ni
  una vez**.
- ⚠ **Lo urgente era otra cosa, y ya está roto hoy** (**CTX-9**): ese comportamiento conduce el
  cuerpo del NPC **por el mismo canal** que Cortex, así que le pisa el destino a una orden y le
  **borra el schedule** — sin dar un error, y con un falso verde en el sondeo de CTX-7.
- **El roster no se persiste** (**CTX-10**), y no por elección: sus miembros son **entidades**.
- **La formación es del ESCUADRÓN** (**CTX-11**, voto contra la opción barata), con regla de
  suspensión. ⚠ El mock **no la decidía**: no dibuja ni una formación.
- **El amarillo es la UNIÓN**, no un cuarto elemento (**CTX-12**) — resolviendo una
  contradicción entre el mock y la prosa de §8.ter del doc del menú.

## Lo que el censo de bases contestó — 2026-08-24

**ENVUELVE la maquinaria de schedules, no el squad del engine.** ⭐ VJ y HL2 puro **comparten el
canal de destino** (`LastPosition`) — y es el mismo canal por el que colisiona el scavenger. ⚠ Cuatro
trampas medidas que no dan error. Todo el detalle en **§7 de [`Cortex_Architecture.md`](Cortex_Architecture.md)**,
que es autosuficiente: acá no se repite.

## Próximo paso

**Punto [5] del roadmap: bajar a código, por vertical slice** — registro contra Corpus → estado
de escuadrón → **una sola orden de punta a punta** → net → el cable al menú. *Una orden completa
vale más que siete a medias.* Criterio de entrada **cumplido**: el diseño está escrito y es
autosuficiente.

⚠ **Dos cosas antes de escribir la primera línea.** (1) El cable al menú sigue **bloqueado** por
el campo «quién ejecuta», que es del framework — el resto del slice **no** lo está. (2) Los
**cinco huecos** que se miden en juego (los tres del censo más la cadencia de formación y si el
arbitraje elimina la colisión) son los candidatos a la **primera planilla del módulo**, letra `AP`.

---

*Rumbo / qué sigue → [`cortex_roadmap.txt`](cortex_roadmap.txt). Diseño → [`Cortex_Architecture.md`](Cortex_Architecture.md).
Contratos recibidos → [`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md).
Metodología → [`../../corpus/docs/corpus_flujo_trabajo.txt`](../../corpus/docs/corpus_flujo_trabajo.txt).*
