# Cortex — Arquitectura del módulo

> **Estado: DOC DE FUNDACIÓN.** Escrito el 2026-08-24, el día que el repo dejó de ser semilla. Contiene lo que **ya está decidido** —por voto del autor, o por una sede de otro repo— y **nombra explícitamente lo que no lo está**. No describe un sistema construido: **no hay una línea de código en este repo**.
>
> **Cómo leerlo, para no leer de más:** cada afirmación de acá es de uno de tres tipos, y el tipo está marcado en el texto.
> - **DECIDIDO** — hay un voto del autor o una sede de otro repo que lo fija. Vinculante.
> - **HEREDADO** — viene de un contrato entrante. Su sede está en otro repo y acá se **cita**, jamás se re-deriva (→ [`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md)).
> - **ABIERTO** — nadie lo decidió. **Está en §6, y §6 es la parte más importante de este documento.**
>
> Un doc que describe un slice sin código no está mintiendo; está haciendo su trabajo (flujo §7.1). Lo que sí sería mentir es presentar como diseño lo que todavía es una pregunta — por eso §6 existe y es larga.

---

## Índice

1. [Qué es Cortex, y qué NO es](#1-qué-es-cortex-y-qué-no-es)
2. [La partición en dos mitades](#2-la-partición-en-dos-mitades)
3. [Dependencias: qué se asume y qué se detecta](#3-dependencias-qué-se-asume-y-qué-se-detecta)
4. [Lo que Cortex POSEE, y lo que sólo consume](#4-lo-que-cortex-posee-y-lo-que-sólo-consume)
5. [El escuadrón — lo poco que ya está fijado](#5-el-escuadrón--lo-poco-que-ya-está-fijado)
6. [Lo que está ABIERTO](#6-lo-que-está-abierto)
7. [Verificación](#7-verificación)

---

## 1. Qué es Cortex, y qué NO es

**DECIDIDO (voto del autor, 2026-08-24).** Cortex es el módulo de IA de NPC del ecosistema: **escuadrones**, **táctica de combate** y **afecto** (dolor y miedo que cambian el comportamiento).

**El bloque que se abrió es el de escuadrones**, y su alcance se estrechó a propósito:

> *«Un sistema mejorado de escuadrones para NPCs de tipo VJ y HL2; las otras bases quedan diferidas.»*

⭐ **Ese estrechamiento tiene una consecuencia que conviene decir en voz alta, porque es la que se pierde primero: el sistema de escuadrones es una CAPA sobre NPCs que YA EXISTEN, no una base de NPC nueva** (**CTX-1**). Es distinto de la decisión vieja —registrada fuera de este repo— de que **CortexBase se construye propia**, tomando lo mejor de VJ/DRG/ZBase. **Las dos conviven**: la capa se escribe primero y CortexBase después, **debajo**. Lo que no puede pasar es que el bloque de la capa se convierta calladamente en el bloque de la base.

**Lo que este módulo NO es:**

- **No es una base de SNPC** (todavía; ver el punto [9] del roadmap).
- **No es el menú interactivo.** El menú es la séptima primitiva del **framework**, vive en `corpus/`, y su diseño está cerrado y votado en `../../corpus/docs/Corpus_Interaccion_Arquitectura.md`. Cortex es **el sistema que la rama `command` de ese menú ejecuta** — no su dueño (**CTX-3**).
- **No es dueño del inventario, ni de la armadura, ni de la sangre, ni del hambre.** Esas son de Cargo, Caliber, Coagulant y Craving, y la frontera la fija COR-10 desde el framework.

## 2. La partición en dos mitades

**DECIDIDO (CTX-2).** Cortex son dos mitades con **gates independientes**, y ésta es la afirmación que más ordena el trabajo del módulo:

| | **Mitad FRÍA** — escuadrón y táctica | **Mitad CALIENTE** — afecto |
|---|---|---|
| Qué es | formaciones, cola de órdenes, coordinación, puertas | dolor y miedo, y cómo cambian el comportamiento |
| ¿De qué depende? | **de nada que falte** | de eventos de daño/extremidad **con zona y magnitud** |
| ¿Se puede escribir hoy? | **sí** | **no** |

⚠ **Por qué la mitad caliente no se puede diseñar todavía, y no es una cuestión de ganas.** Lo único que Caliber emite hoy es `hook.Run("Caliber_LimbsUpdated", npc, reason)`, heredado verbatim de ADS. Su payload es `(npc, reason)` con `reason ∈ spawn|damage|heal`: **sin zona, sin cuánto bajó el pool, sin `dmginfo`**. Es un **aviso de refresh, no un evento de daño**. Con eso, el afecto no puede distinguir un rasguño en el brazo de una pierna destrozada — que es exactamente lo que necesita. El pendiente de Caliber **no es agregar el emit, ya existe: es enriquecer su payload** y recién ahí elevarlo a contrato. Sede: `../../corpus-caliber/docs/Caliber_Architecture.md` §9.a, y **CAL-22**, que existe justamente para que nadie lea ese contrato como servido.

⚠ **Y el gate del ecosistema está enunciado SIN partir.** `../../corpus/docs/corpus_roadmap.txt` dice que Cortex *«arranca cuando el Block 3 de Caliber»* exponga esa superficie — Cortex entero. **La partición es de acá**, y es la razón por la que el bloque de escuadrones puede abrir con el Block 3 de Caliber todavía en su tramo 0.

## 3. Dependencias: qué se asume y qué se detecta

**HEREDADO del framework.** Una sola hard-dep (**COR-11**): **Corpus**. Todo lo demás se detecta en runtime vía `Corpus.GetModule`/`Corpus.HasModule` y **nunca se asume** (**COR-5**), porque el orden de mount entre addons de Gmod no está garantizado.

| Peer | Tipo | Para qué, y qué pasa si falta |
|---|---|---|
| **Corpus** | **hard** | Registro, persistencia, net, log, UI shell, ready barrier — y el menú interactivo que hará de front-end. Sin él el módulo **no arranca** y lo dice ruidoso |
| **Caliber** | soft | Eventos de daño/extremidad para el afecto. Sin él: táctica y escuadrón intactos, **sin afecto localizado** |
| **Cargo** | soft | Consume `GetFactionInfo` para el header del inventario (contrato #1) y es dueño del cadáver looteable (#2). ⚠ Y **bloquea** `Deploy` por otra vía — ver §6.2 |
| **corpus-stalker** | soft (al revés) | **Él registra contra Cortex**, no al revés: defs de NPC y facciones de la Zona (#5). Sin Cortex se queda sin NPCs propios y el resto de su contenido sigue |
| **VJ Base**, **HL2** | COMPAT-RUNTIME | Las bases sobre las que corre la capa. Se **detectan y se consumen por API**; no se copia su código |

**Sobre las bases de terceros:** son **COMPAT-RUNTIME**, no RECICLAR (`../../dev/mods_workshop_mapa.md`). Es lo que **CTX-1** significa en la práctica — si el módulo terminara copiando el código de VJ adentro, dejó de ser una capa.

## 4. Lo que Cortex POSEE, y lo que sólo consume

Esta tabla es la razón por la que [`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md) existe: **seis firmas se congelaron contra este repo antes de que tuviera una línea de código**, y ninguna se re-deriva acá.

| Cosa | Dueño | Nota |
|---|---|---|
| Facción, rango y relaciones | **Cortex** | **HEREDADO** (#1). Y con una forma ya fijada por la sede de Cargo: **framework de PROVIDERS**, no tabla hardcodeada. Cargo **sólo renderiza** ⇒ la firma devuelve algo **presentable**, no un identificador crudo |
| La **muerte** como evento semántico | **Cortex** | **HEREDADO** (#3) |
| El **contenido** del drop (loot table) | **Cortex/Caliber** | **HEREDADO** (#3), por exclusión. La tabla se cierra al diseñar el bloque dueño |
| El **estado del escuadrón** | **Cortex** | **DECIDIDO** — **CTX-3** |
| El cadáver como **contenedor**, su UI y su **GC** | **Cargo** | **HEREDADO** (#2). No es de Cortex, y eso mantiene el loot agnóstico a su diseño |
| El **scavenger-pickup** | **sin decidir** | **ABIERTO** (#6) — §6.1 |

⚠ **La cadena de facción se lee entera o no se entiende:** corpus-stalker **registra** sus facciones → Cortex las **resuelve** → Cargo las **pinta**. De ahí sale un orden de implementación que no es opcional: **el framework de providers antes que las defs**.

## 5. El escuadrón — lo poco que ya está fijado

**Casi todo el escuadrón está abierto.** Lo que sigue es lo único con respaldo, y viene de fuera de este repo.

### 5.1 El catálogo de órdenes — **DECIDIDO**, y con una advertencia

Levantado de la guía de SWAT de *Ready or Not* y del pedido del autor, vive en `../../corpus/docs/Corpus_Interaccion_Arquitectura.md` §8.bis:

- **Órdenes base:** *Move To · Fall In* (con cuatro formaciones: fila simple, fila doble, diamante, cuña) *· Cover · Hold · Deploy · Search and Secure · Delay/Synced*.
- **Órdenes de puerta:** *Stack Up · Scan* (tres métodos) *· Breach* (cinco tipos) *· Wedge · Open/Close*.

⚠⚠ **Esa fuente cubre las ÓRDENES y NO la mecánica.** No dice quién integra un escuadrón, qué estado guarda, ni cómo se encola una orden. **Todo eso es §6.**

### 5.2 Tres decisiones de forma que ya se tomaron del lado del menú — **HEREDADO**

Están en §8.bis del doc del menú, y **arrastran diseño para acá**:

1. **`Delay` es un MODIFICADOR, no una orden.** Con Delay armado, cada orden **se va a la cola** en vez de ejecutarse, y `Execute` las manda todas juntas — es la única forma de que dos grupos entren al mismo tiempo. ⚠ **Cada fila de la cola guarda su PROPIO objetivo**: el destino se resuelve **cuando diste la orden**, no cuando disparas. **Eso es estado de la escuadra, o sea de este módulo.**
2. **La asimetría NPC/jugador.** *Al NPC se lo asigna, al jugador se lo invita*, y **un jugador no ejecuta órdenes: las recibe**.
3. **Los grupos son CUATRO** (rojo, azul, verde, amarillo), y llevan **letra además de color** — el techo es de la paleta del theme, no de la UI.

### 5.3 `Deploy` no es una acción, es un OBJETO — **HEREDADO**, y bloqueado

Se despliega *algo*, así que sus hijos son objetos y no verbos: **la lista la arma el módulo que registra cada objeto, no el árbol**. ⚠ Consecuencia directa: **`Deploy` no existe sin Cargo** — sin inventario no hay nada que sacar. Y hay una segunda pared, más dura, en §6.2.

## 6. Lo que está ABIERTO

**Esta sección es el trabajo del bloque.** Nada de acá está decidido, y ninguna de estas preguntas se contesta sin leer antes el árbol real.

### 6.1 Las cuatro preguntas del escuadrón

1. **¿Envuelve o reemplaza?** VJ Base y los `npc_*` de HL2 ya coordinan algo por su cuenta. **Lo que el engine y la base ya hacen no se reimplementa: se envuelve.** Contestar esto exige leer `dev/other/VJ Base/` y la superficie real de squads de HL2 — **del árbol y de la wiki, no de memoria**. Es el punto [2] del roadmap y **decide la forma del módulo entero**.
2. **¿Quién integra un escuadrón?** ¿Sólo NPC? ¿Jugadores también? La asimetría de §5.2 dice que un jugador *recibe* y no *ejecuta*, pero no dice si es miembro.
3. **¿Qué estado guarda, y qué le pasa al morir el líder?** La cola de §5.2 y la formación de §5.1 son estado que **sobrevive al menú**. Persistir va por `Corpus.Data` namespaced (**COR-3**), si es que hay algo que persistir entre mapas.
4. **¿La formación es de la ORDEN o del ESCUADRÓN?** *Fall In* trae cuatro. Si la formación persiste, es otra pieza de estado; si es un argumento de la orden, no lo es.

### 6.2 Dos paredes que este módulo no puede tirar solo

**(a) El campo «quién ejecuta» del menú.** El mensaje de net del menú lleva `(id de acción, entidad, component)` y **no tiene dónde poner al ejecutor**. Una orden de escuadra tiene **dos objetivos** —*quién* actúa y *dónde*— y **ninguna otra acción del menú tiene eso**. Es decisión del **framework** y se resuelve por el lado de `corpus/`; acá sólo se registra que **bloquea el cable** entre el menú y este módulo.

**(b) Un NPC no puede ser dueño de un inventario de Cargo — CTX-4, y está MEDIDO.** `Inventory.OwnerKey` (`../../corpus-cargo/lua/corpus_cargo/server/corpus_cargo_inventory.lua:101-104`) hace `ply:SteamID64() or ("bot" .. ply:EntIndex())`, y **`SteamID64` es un método de `Player`, no de `Entity`** — con un NPC **no cae en el `or`: revienta**. ⚠ **La puerta no está entornada: no existe**, y abrirla es una decisión de Cargo (*¿el dueño de un inventario deja de ser un jugador?*), no un parche de una línea desde acá.

Eso bloquea `Deploy` y la secuencia que el autor pidió, **«throw flashbang and clear»**, que además arrastra dos cosas más: **el efecto de flash sobre NPCs es pobre** (medido en juego: quedan ciegos unos pocos segundos — sede a leer: `dev/other/Arc9 EFT explosives/`), y **volver a la posición original** implica que la orden guarda un **punto de retorno**, o sea otra vez estado por escuadra.

### 6.3 El contrato entrante que este bloque tiene que contestar

**#6 — ¿el scavenger-pickup es behavior de Cortex?** El comportamiento de recolección de armas por NPC vive hoy en Caliber (`corpus_caliber_scavenger.lua`), acoplado al drop de `Limbs`. Su sede (§9.c) reconoce que **elegir target, la animación y el timing huelen a comportamiento NPC** —territorio de este módulo— pero deja la decisión abierta a propósito: se re-homea **sólo si** se confirma como behavior al diseñar Cortex. **No hay nada que honrar: hay una pregunta que contestar**, y si se re-homea, se re-homea con parche en los **dos** repos.

### 6.4 Qué es una «puerta» para el sistema

`prop_door_rotating` y `func_door` son entidades del engine. **Las cinco órdenes de puerta no existen sin esta definición**, y el menú ya le puso una precondición fuerte: la puerta **sólo trae sus cinco acciones si comunica dos lados navegables** —decorativa, tapiada o sin sala detrás no las trae, y **abierta o cerrada da igual**. Referencia disponible: el mod `immersive door openable` de `dev/other/`, que ya mapeó sus **siete keyvalues de sonido** en dos familias.

## 7. Verificación

No hay test runner (es un addon GMod). Mientras el repo sea sólo docs, **«verificado» significa revisado contra su sede y contra el árbol real** — nunca contra el chat. La jerarquía de §7.1 del flujo pone al código por encima de todo doc, y **este módulo diseña por delante del código a propósito** (mock-first, flujo §3).

Cuando nazca código, el criterio de aceptación **no** puede ser paridad con nada: a diferencia de Caliber —que migró desde ADS y podía compararse contra el original— **Cortex no tiene predecesor**. Su verificación va a ser **planilla en juego con IDs de check estables** (flujo §1, Paso 4), citables desde el registro como evidencia tipo `planilla`.

⚠ **Y hay una lección de la casa que aplica antes de escribir el primer check:** el ecosistema tiene un catálogo de controles que dieron el veredicto equivocado, y la primera regla de ese catálogo es que **un check que no puede fallar no mide nada**. Cada tanda de checks va con su **suite de sabotaje**, que tiene que dar el total en rojo.

---

*Estado de HOY → [`cortex_estado.md`](cortex_estado.md). Rumbo → [`cortex_roadmap.txt`](cortex_roadmap.txt).
Contratos recibidos → [`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md). Normas del módulo → [`../CLAUDE.md`](../CLAUDE.md).
Framework y grafo de dependencias → [`../../corpus/docs/CORPUS_Architecture.md`](../../corpus/docs/CORPUS_Architecture.md).
El menú que hará de front-end → [`../../corpus/docs/Corpus_Interaccion_Arquitectura.md`](../../corpus/docs/Corpus_Interaccion_Arquitectura.md).*
