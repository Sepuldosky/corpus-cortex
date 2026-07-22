# Cortex — Contratos ENTRANTES

> **Qué es este documento:** el registro único de las firmas que **otros repos ya congelaron contra Cortex**, antes de que Cortex tenga una línea de código. Es un doc de **RECEPCIÓN**, no de diseño.
>
> **Qué NO es:** la arquitectura de Cortex. Acá no se diseña IA, ni CortexBase, ni táctica, ni afecto. Nada de lo que se escriba en este archivo decide cómo funciona el módulo — solo inventaría lo que ya se le prometió desde afuera, para que el Block de diseño empiece sabiendo qué tiene que honrar.
>
> **Por qué existe** (deuda D-13, hueco H5 del COMPLETO del 2026-07-19): `corpus-cortex/` contiene `LICENSE`, `README.md`, este mismo documento y su `CHANGELOG.md` —ni una línea de código, ni `CLAUDE.md`—, pero **es el destinatario de al menos cinco contratos congelados por otros repos**. Hasta hoy esas cinco firmas vivían dispersas en tres repos (corpus-cargo, corpus-caliber, corpus-stalker), repartidas en cuatro documentos, y **nadie podía auditar si eran mutuamente consistentes, porque el repo que las recibe no tenía dónde contradecirlas**. El hallazgo 2.25 del COMPLETO fue la punta de ese iceberg.
>
> **Autoridad: NINGUNA.** Cada fila de abajo tiene su **sede en otro repo** y este doc la **cita**. Si algo acá contradice a su sede, este doc está desactualizado (cita **FLU-22**: la sede gana siempre). No se acuña ni un solo `CTX-` desde acá — la familia `CTX` está reservada pero **sin entradas**, y acuñar la primera exige antes crear el `CLAUDE.md` del repo.

---

## Índice

1. [Cómo se lee esta tabla](#1-cómo-se-lee-esta-tabla)
2. [Los contratos entrantes](#2-los-contratos-entrantes)
3. [Detalle por contrato](#3-detalle-por-contrato)
4. [Consistencia mutua: qué ya se cruzó](#4-consistencia-mutua-qué-ya-se-cruzó)
5. [Lo que NO está congelado](#5-lo-que-no-está-congelado)

---

## 1. Cómo se lee esta tabla

Un contrato entrante puede estar en tres situaciones, y la diferencia importa mucho al diseñar:

- **VIVO** — hay un call-site real en el árbol de otro repo, ejecutándose hoy con degradación honesta. Cortex tiene que honrar la firma **tal cual** o romper a un consumidor que ya existe.
- **ANTICIPATORIO** — el call-site existe pero el peer que lo llama sabe que Cortex no está y degrada. Igual de vinculante en la firma; menos urgente.
- **DECLARADO** — todavía no hay call-site: es una frontera adjudicada en prosa, con dueño decidido. Vinculante como **decisión de propiedad**, no como firma.

## 2. Los contratos entrantes

| # | Contrato | Quién lo congeló | Situación | Sede (manda) |
|---|---|---|---|---|
| 1 | `GetFactionInfo(ply)` | Cargo | **ANTICIPATORIO** | `corpus-cargo/docs/Cargo_Architecture.md` §6 |
| 2 | Cadáver looteable = contenedor de Cargo, y su **GC es de Cargo** | Cargo (voto del autor) | **DECLARADO** | `corpus-cargo/docs/Cargo_Trade_Arquitectura.md` §9 |
| 3 | Loot table (*qué* dropea un NPC al morir) | Cargo (por exclusión) | **DECLARADO** — dueño: Cortex/Caliber | `corpus-cargo/docs/Cargo_Trade_Arquitectura.md` §9 |
| 4 | `CALIBER.Limbs.*` + eventos de daño/limb | Caliber | **NO CONGELADO** (ver §5) | `corpus-caliber/docs/Caliber_Architecture.md` §8, §9.a |
| 5 | Defs de NPC y facciones de la Zona | corpus-stalker | **DECLARADO** | `corpus-stalker/docs/STALKER_Arquitectura.md` §3-§4 |
| 6 | Scavenger-pickup: ¿es behavior de Cortex? | Caliber | **DECLARADO — a decidir en el diseño de Cortex** | `corpus-caliber/docs/Caliber_Architecture.md` §9.c |

## 3. Detalle por contrato

### 3.1 · `GetFactionInfo(ply)` — el header de perfil de Cargo

**Situación: ANTICIPATORIO.** El call-site vive en `corpus_cargo_inventory.lua`, con lazy-check + `pcall`. Sin Cortex montado, el header del inventario simplemente omite facción y rango.

Lo que la sede de Cargo (§6) **ya decidió y Cortex hereda**:

- **Cortex es dueño del dato** — facción, rango y relaciones son concepto de IA/comportamiento, no de inventario.
- **Cargo solo renderiza** lo que el provider activo reporte. Cargo no almacena ni interpreta facción; no hay caché ni copia del lado del inventario.
- Se espera un **framework de providers dentro de Cortex**, para que otros mods de facciones se registren — no una tabla hardcodeada.
- **El cruce es simétrico:** Cargo emite qué traje está equipado en el slot Body, y Cortex resuelve la **facción aparente** (disguise) a partir de esa señal. Ninguno de los dos importa al otro: ambos pasan por el registro de Corpus (cita **COR-5**).

> **Ojo al diseñar:** "Cargo solo renderiza" implica que la firma devuelve algo **presentable** (nombre de facción, rango, probablemente color), no un identificador crudo que Cargo tendría que traducir — traducirlo sería interpretar, que es justo lo que la sede le prohíbe.

### 3.2 · El cadáver looteable, y quién lo limpia

**Situación: DECLARADO.** Es el **voto B** del COMPLETO del 2026-07-19, resuelto por el autor.

- El cadáver (NPC muerto, o jugador) es una **entidad con inventario**: su loadout más lo que llevaba. Abrirlo es el estado **"loot"** de la UI de Cargo (sin precios, con *Take all*), no "trade".
- **El cadáver looteable es un contenedor de Cargo** — cita **CRG-21**: contenedor, trader y cadáver son el **mismo primitivo** (`Containers.Attach`), nunca un inventario paralelo.
- **El GC —*cuándo* se limpia un cadáver ya looteado— es de CARGO.** La razón del voto: mantiene el loot **agnóstico al diseño de Cortex**, y el dueño del primitivo de inventario-en-entidad es quien ya reacciona vía `CallOnRemove`.
- *Cómo se ve y se transfiere* ese inventario también es de Cargo.

**Qué le queda a Cortex:** la **muerte** como evento semántico y el **contenido** (§3.3). No el contenedor, no la UI, no el GC. Cortex queda como **capa de IA sobre la base de NPC**.

### 3.3 · Loot table — *qué* dropea un NPC al morir

**Situación: DECLARADO.** Adjudicada a **Cortex/Caliber** por exclusión, en la misma sede que 3.2. Cargo provee el inventario-en-entidad y la UI de loot; el **contenido y la muerte** son del dueño del dominio.

La tabla en sí **se cierra al diseñar el bloque dueño** — es la regla del flujo de que el dueño de una frontera se decide en diseño (cita **CRG-47**). Este doc solo registra que la pelota está de este lado.

### 3.4 · `CALIBER.Limbs.*` y los eventos de daño/limb

**Situación: NO CONGELADO — y es el más peligroso de malinterpretar.**

Lo que **existe hoy** en Caliber:

- `CALIBER.HealLimbs` es superficie pública. `CALIBER.Limbs.*` está **vacío como contrato** en el Block 2.
- Hay un `hook.Run("Caliber_LimbsUpdated", npc, reason)` en `corpus_caliber_limbs.lua`, heredado verbatim de ADS, con `reason ∈ spawn|damage|heal`. Corre en el path de daño **y en todos los paths de heal**. Hoy **no tiene ningún consumidor** en el ecosistema.

Por qué **no** sirve tal cual (sede: Caliber §9.a): ese hook es un **aviso de refresh, no un evento de daño**. Su payload es `(npc, reason)` — **sin zona, sin cuánto bajó el pool, sin `dmginfo`**. Cortex necesita dolor y miedo *localizados*; con este payload no puede distinguir un rasguño en el brazo de una pierna destrozada.

El pendiente de Caliber **no es agregar el emit** —ya existe— sino **enriquecer su payload** y recién ahí elevarlo a contrato.

> **Bloqueante doble.** (a) Ese enriquecimiento llega con el **Block 3 de Caliber**, que no cerró. (b) **CAL-22** advierte que la Limbs API es agnóstica NPC/jugador *por diseño* pero **NPC-only en la práctica**: el contrato de `CORPUS_Architecture.md` §4 **no está cumplido hoy**. Esa norma existe exactamente para que Cortex no lea ese contrato como servido. Diseñar Cortex asumiendo eventos de limb ricos sería diseñar contra algo que no existe.

### 3.5 · Defs de NPC y facciones de la Zona

**Situación: DECLARADO.** `corpus-stalker` —el addon de contenido— espera registrar contra Cortex sus defs de NPC (stalkers, mutantes) y sus datos de facción (nombre, relaciones, colores).

Es la relación estándar del ecosistema, en la dirección habitual: **el módulo pone el framework, el addon de contenido pone el contenido**. corpus-stalker es **consumidor puro** (cita **STK-1**) y hard-depende solo de Corpus (cita **COR-11**): si Cortex no está, se queda sin NPCs propios y el resto de su contenido sigue funcionando.

Encaja con 3.1: los datos de facción que stalker registre son los que Cortex después le reporta a Cargo vía `GetFactionInfo`. **Es la misma cadena**, y es la razón por la que conviene que el framework de providers de 3.1 exista antes de cablear las defs.

### 3.6 · Scavenger-pickup — una frontera abierta a propósito

**Situación: DECLARADO, a decidir en el diseño de Cortex.**

El comportamiento de recolección de armas por NPC vive hoy en Caliber (`corpus_caliber_scavenger.lua`), acoplado al drop de `Limbs`. Su sede (§9.c) reconoce que **elegir target, la animación y el timing huelen a comportamiento NPC — territorio de Cortex**, pero deja la decisión explícitamente abierta: se re-homea **solo si** se confirma como behavior al diseñar Cortex.

No hay nada que honrar acá. Lo que hay es una **pregunta que el Block de Cortex tiene que responder**, y este doc existe para que no se pase por alto.

## 4. Consistencia mutua: qué ya se cruzó

El valor de juntar las seis filas en un solo lugar es poder preguntarse si se contradicen. Al 2026-07-19:

- **3.2 y 3.3 son complementarios, no solapados.** Cargo se queda el contenedor, la UI y el GC; Cortex se queda la muerte y el contenido. La línea está donde tiene que estar: el primitivo de inventario es de quien ya lo tiene, la semántica es de quien tiene el dominio.
- **3.1 y 3.5 son la misma cadena** vista desde los dos extremos (stalker registra → Cortex resuelve → Cargo pinta). Consistentes, y con un orden de implementación implícito: el framework de providers antes que las defs.
- **3.4 es el único con riesgo real de diseño-en-falso**, y está señalizado por dos normas de Caliber (§9.a y **CAL-22**).
- **3.6 no está decidido**, y eso es correcto: decidirlo ahora sería adivinar.

**No se encontró contradicción entre las seis.** Es la primera vez que se pueden mirar juntas.

## 5. Lo que NO está congelado

Para que nadie lea de más en este documento:

- **Nada de la IA de Cortex.** Táctica de combate, peek, dolor, miedo, CortexBase — todo eso es el Block de diseño de Cortex y **no existe todavía**. Este doc no lo prejuzga.
- **La familia `CTX` está reservada, sin entradas.** Está en `familias` del registro con `pendiente: true`. Acuñar su **primera entrada** exige antes crear `corpus-cortex/CLAUDE.md`, que es la sede declarada de la familia — el checker se pone rojo si se acuña sin eso. **Este documento no acuña ninguna.**

  *(Nota de método: la primera redacción de esta viñeta escribía la etiqueta de esa primera entrada de forma literal, y el checker la cazó al instante como `HUERFANO_DOC` — un ID citado y no definido. Correcto: no distingue "cito una norma" de "menciono un identificador", y no debería. Se reescribió en prosa.)*
- **Los eventos de daño/limb (3.4) NO son un contrato**, por más que dos docs los mencionen. Son un pendiente de Caliber.
- **La decisión de base de NPC** (que CortexBase se construye propia, tomando lo mejor de VJ/DRG/ZBase, con compat después) es una decisión del autor registrada fuera de este repo; no es un contrato entrante y no se documenta acá.

---

*Mantenimiento: este doc se actualiza cuando **otro repo congela algo nuevo contra Cortex**, en el mismo parche que lo congela (misma disciplina que FLU-30). Cuando el Block de Cortex abra y el repo reciba su `CLAUDE.md` + set de docs, este archivo se conserva como el inventario de entrada — no se disuelve en la arquitectura.*
