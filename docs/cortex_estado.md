# Cortex — Estado de HOY

> **Foto del AHORA**, volátil. Es lo primero que se lee al retomar el módulo —
> **antes** que el doc de arquitectura. Se actualiza **en sitio** (no se agregan
> secciones ni historial). El historial vive en `git` + [`CHANGELOG.md`](CHANGELOG.md).
> Si crece de una pantalla, está mal redactado: recortar.

**Última actualización:** 2026-08-24 (dos tandas). La segunda **cerró el punto [2] del roadmap**: se censaron las bases de NPC contra el árbol real y la respuesta bajó a `Cortex_Architecture.md` §7, con dos normas nuevas — el escuadrón corre **en paralelo** al squad del engine (voto del autor) y la cola de órdenes **sondea**. La primera: **el repo se FUNDÓ**. Dejó de ser semilla: nació su `CLAUDE.md` (la sede de su familia de normas), más `cortex_estado.md`, `cortex_roadmap.txt` y `Cortex_Architecture.md`, siguiendo el template del flujo §2. Con eso la familia dejó de estar reservada-sin-entradas en `../../corpus/docs/ids.yaml`. **Entre las dos tandas van siete normas.** **Sin una línea de código todavía**, y a propósito: lo que abrió es un bloque de **diseño**.

---

## Qué existe hoy

- **El repo está fundado.** `CLAUDE.md` + los cuatro docs del template. La familia de
  normas tiene sede y **siete entradas** en el registro del ecosistema.
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
  —de la guía de SWAT de *Ready or Not*— en `../../corpus/docs/Corpus_Interaccion_Arquitectura.md`
  §8.bis, la **forma de la pantalla** en el mock v2, y desde el 2026-08-24 **qué expone cada
  base** (§7 de la arquitectura). ⚠ **Ese catálogo cubre las ÓRDENES, no la mecánica**: quién
  integra un escuadrón, qué estado guarda y cómo se encola una orden **sigue sin decidirse**.
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

## Lo que el censo de bases contestó — 2026-08-24

- **ENVUELVE, y se midió qué:** la **maquinaria de schedules**, no el squad del engine. VJ Base
  **no tiene escuadrones ni órdenes** (`Leader`: 0 hits en 28.431 líneas; los 40 hits de
  `command` son consolas y botones de menú), pero **sí** `ENT:Follow` con su hook `OnFollow`
  vacío y **ocho métodos `SCHEDULE_*` con punto de extensión**.
- ⭐ **VJ y HL2 puro comparten el canal de destino** (`LastPosition`), así que *Move To*
  **no son dos implementaciones**.
- ⚠ **Cuatro trampas medidas que no dan error**, en §7.4 de la arquitectura. La peor: el destino
  **no es argumento** de los métodos de movimiento de VJ, y pasarle un vector no falla — manda al
  NPC al último punto guardado.

## Próximo paso

1. **Contestar el contrato entrante #6** (scavenger-pickup: ¿es behavior de Cortex?). Su
   sede lo dejó explícitamente abierto *«a decidir en el diseño de Cortex»*, y este bloque
   es ese diseño.
2. **Diseñar el escuadrón**: quién lo integra, qué estado guarda, cómo se encola una orden — y
   decidir si merece doc particular (`Cortex_Escuadrones_Arquitectura.md`) según el criterio del
   flujo §2: *«¿un implementador necesita este doc solo, sin el general ni el chat, para
   ejecutar?»*. Tres de las cuatro preguntas de §6.1 siguen abiertas; la primera ya no.

---

*Rumbo / qué sigue → [`cortex_roadmap.txt`](cortex_roadmap.txt). Diseño → [`Cortex_Architecture.md`](Cortex_Architecture.md).
Contratos recibidos → [`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md).
Metodología → [`../../corpus/docs/corpus_flujo_trabajo.txt`](../../corpus/docs/corpus_flujo_trabajo.txt).*
