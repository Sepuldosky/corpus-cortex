# Cortex — Estado de HOY

> **Foto del AHORA**, volátil. Es lo primero que se lee al retomar el módulo —
> **antes** que el doc de arquitectura. Se actualiza **en sitio** (no se agregan
> secciones ni historial). El historial vive en `git` + [`CHANGELOG.md`](CHANGELOG.md).
> Si crece de una pantalla, está mal redactado: recortar.

**Última actualización:** 2026-08-24 — **el repo se FUNDÓ**. Dejó de ser semilla: nació su `CLAUDE.md` (la sede de su familia de normas), más `cortex_estado.md`, `cortex_roadmap.txt` y `Cortex_Architecture.md`, siguiendo el template del flujo §2. Con eso se acuñaron las **cinco primeras normas** del módulo y la familia dejó de estar reservada-sin-entradas en `../../corpus/docs/ids.yaml`. **Sin una línea de código todavía**, y a propósito: lo que abrió es un bloque de **diseño**.

---

## Qué existe hoy

- **El repo está fundado.** `CLAUDE.md` + los cuatro docs del template. La familia de
  normas tiene sede y sus cinco primeras entradas están en el registro del ecosistema.
- **Los seis contratos ENTRANTES, inventariados desde el 2026-07-19** en
  [`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md): tres de Cargo, dos de
  Caliber, uno de corpus-stalker. Doc de **recepción, sin autoridad** — cada fila tiene su
  sede en otro repo. **Se lee entero antes de diseñar**; nada de ahí se re-deriva.
- **Las convenciones de commit**, con dos alcances en uso y **diez reservados**. El de
  escuadrones (`escuadra`) se sumó el 2026-08-24 al abrir el bloque, por voto del autor.
- **La identidad visual** (glifo y lockup, light y dark) y el README público.
- **El alcance del bloque abierto, votado por el autor el 2026-08-24:** un sistema de
  escuadrones para NPCs de **VJ Base y HL2**; DRG y ZBase **diferidas**. Es una **capa**
  sobre bases que ya existen, no una base de NPC nueva.

## Lo que NO existe, y conviene no leerlo de más

- **Cero código.** No hay `lua/`, ni entry point, ni registro contra el framework. Lo que
  hay es diseño y proceso.
- **El diseño de escuadrones NO está escrito.** Lo que existe es el **catálogo de órdenes**
  —levantado de la guía de SWAT de *Ready or Not*— en `../../corpus/docs/Corpus_Interaccion_Arquitectura.md`
  §8.bis, y la **forma de la pantalla** en el mock v2 del menú. ⚠ **Ese catálogo cubre las
  ÓRDENES, no la mecánica**: quién integra un escuadrón, qué estado guarda y cómo se encola
  una orden **no está decidido por nadie**.
- **CortexBase no está diseñada** y no es lo que se abrió. Sigue en el plan, **debajo** de
  la capa y después.

## Bloqueado por quién — y qué NO está bloqueado

- **El afecto (dolor, miedo) está gated por Caliber**, cuyo Block 3 no cerró: la superficie
  de eventos de daño/extremidad **no existe todavía**. El hook que hay hoy
  (`Caliber_LimbsUpdated`) es **un aviso de refresh, no un evento de daño** — su payload es
  `(npc, reason)`, sin zona y sin cuánto bajó el pool. Diseñar el afecto contra eso sería
  diseñar contra algo que no está.
- ⭐ **La táctica y el escuadrón NO están bloqueados por eso**, y es lo que hace que este
  bloque pueda abrir hoy. El roadmap del framework enuncia el gate sin partirlo — la
  partición vive en el CLAUDE.md de este repo.
- **La orden `Deploy` y la secuencia *throw flashbang and clear* están bloqueadas por
  Cargo**, no por Caliber: hoy **un NPC no puede ser dueño de un inventario** (`OwnerKey`
  llama a `SteamID64`, que es método de `Player` y no de `Entity` — con un NPC revienta, no
  cae en el `or`). Abrir esa puerta es un voto de Cargo.
- **Una decisión de diseño del framework sigue abierta y arrastra**: el mensaje de net del
  menú lleva `(id de acción, entidad, component)` y **no tiene campo para quién ejecuta**.
  Una orden de escuadra tiene **dos** objetivos. Está anotado en §8.bis del doc del menú
  como la decisión que más arrastra, y se resuelve por el lado del framework.

## Próximo paso

1. **Leer VJ Base** en `dev/other/VJ Base/` — qué expone para dar órdenes, qué estado de
   escuadrón trae de fábrica, y qué hace HL2 por su cuenta con `npc_*` y sus squads nativos.
   Es la lectura que decide si el escuadrón de Cortex **envuelve** o **reemplaza**.
2. **Contestar el contrato entrante #6** (scavenger-pickup: ¿es behavior de Cortex?). Su
   sede lo dejó explícitamente abierto *«a decidir en el diseño de Cortex»*, y este bloque
   es ese diseño.
3. **Recién ahí, diseñar el escuadrón**: quién lo integra, qué estado guarda, cómo se encola
   una orden — y decidir si merece doc particular (`Cortex_Escuadrones_Arquitectura.md`)
   según el criterio del flujo §2: *«¿un implementador necesita este doc solo, sin el general
   ni el chat, para ejecutar?»*.

---

*Rumbo / qué sigue → [`cortex_roadmap.txt`](cortex_roadmap.txt). Diseño → [`Cortex_Architecture.md`](Cortex_Architecture.md).
Contratos recibidos → [`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md).
Metodología → [`../../corpus/docs/corpus_flujo_trabajo.txt`](../../corpus/docs/corpus_flujo_trabajo.txt).*
