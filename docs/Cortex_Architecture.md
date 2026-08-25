# Cortex — Arquitectura del módulo

> **Estado: DOC DE FUNDACIÓN.** Escrito el 2026-08-24, el día que el repo dejó de ser semilla. Contiene lo que **ya está decidido** —por voto del autor, o por una sede de otro repo— y **nombra explícitamente lo que no lo está**. No describe un sistema construido: **no hay una línea de código en este repo**.
>
> **Cómo leerlo, para no leer de más:** cada afirmación de acá es de uno de tres tipos, y el tipo está marcado en el texto.
> - **DECIDIDO** — hay un voto del autor o una sede de otro repo que lo fija. Vinculante.
> - **HEREDADO** — viene de un contrato entrante. Su sede está en otro repo y acá se **cita**, jamás se re-deriva (→ [`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md)).
> - **ABIERTO** — nadie lo decidió. Está en §6. ⚠ **Al 2026-08-25 §6 ya no es lo más importante del doc: las cuatro preguntas del escuadrón cerraron y el contrato entrante #6 se contestó.** Lo que queda abierto ahí son las dos paredes que este módulo no puede tirar solo, y la definición de «puerta».
>
> Un doc que describe un slice sin código no está mintiendo; está haciendo su trabajo (flujo §7.1). Lo que sí sería mentir es presentar como diseño lo que todavía es una pregunta — por eso §6 existe y es larga.

---

## Índice

1. [Qué es Cortex, y qué NO es](#1-qué-es-cortex-y-qué-no-es)
2. [La partición en dos mitades](#2-la-partición-en-dos-mitades)
3. [Dependencias: qué se asume y qué se detecta](#3-dependencias-qué-se-asume-y-qué-se-detecta)
4. [Lo que Cortex POSEE, y lo que sólo consume](#4-lo-que-cortex-posee-y-lo-que-sólo-consume)
5. [El escuadrón — resumen, y el link al doc particular](#5-el-escuadrón--resumen-y-el-link-al-doc-particular)
6. [Lo que está ABIERTO](#6-lo-que-está-abierto)
7. [Las bases de NPC: qué expone cada una — MEDIDO](#7-las-bases-de-npc-qué-expone-cada-una--medido)
8. [Verificación](#8-verificación)

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
| El **scavenger-pickup** | **Caliber** (el código) · **Cortex** (la autoridad) | **CONTESTADO el 2026-08-25** (#6) — **CTX-8**. Es behavior, y por eso el cuerpo del NPC responde a Cortex; **pero el archivo no se mueve**. Ver §6.3 |
| **Quién conduce el cuerpo** de un NPC | **Cortex** | **DECIDIDO** — **CTX-9**. Todo behavior autónomo del ecosistema pide y suelta el volante |

⚠ **La cadena de facción se lee entera o no se entiende:** corpus-stalker **registra** sus facciones → Cortex las **resuelve** → Cargo las **pinta**. De ahí sale un orden de implementación que no es opcional: **el framework de providers antes que las defs**.

## 5. El escuadrón — resumen, y el link al doc particular

> ⭐ **El diseño del escuadrón se desprendió a doc PARTICULAR el 2026-08-25:**
> [`Cortex_Escuadrones_Arquitectura.md`](Cortex_Escuadrones_Arquitectura.md). Ahí viven el modelo de
> estado completo, el arbitraje del cuerpo, la cola y la formación — y **es el doc que hay que leer
> para implementar**, por el criterio del flujo §2 (*«¿un implementador necesita este doc solo, sin el
> general ni el chat, para ejecutar?»*). **Esta sección es su resumen y no duplica su contenido.**

**Lo que cerró el 2026-08-25**, en una línea cada uno:

| | Qué se decidió |
|---|---|
| **CTX-8** | El **scavenger-pickup** ES behavior de NPC ⇒ queda bajo la autoridad de Cortex — **pero NO se re-homea**. Autoridad, no hosting |
| **CTX-9** | La **conducción del cuerpo se arbitra**: un dueño a la vez, la orden del jugador gana. ⚠ Hace falta porque **hoy ya se rompe sin dar un error** |
| **CTX-10** | El **roster es VOLÁTIL**: no se persiste, y no por elección — sus miembros son entidades |
| **CTX-11** | La **formación es del ESCUADRÓN**, con regla de suspensión |
| **CTX-12** | El **amarillo es la UNIÓN**: los elementos son **tres** |

**Y quién integra un escuadrón:** un solo roster con NPCs y jugadores, distinguidos por **dos ejes
independientes** —ingreso (al NPC se lo asigna, al jugador se lo invita) y ejecución (al NPC la orden
le mueve el cuerpo, al jugador le llega como aviso)—; ⚠ **el comandante NO es una fila**, y ⚠⚠ hay
**dos «líderes»** que no son lo mismo: el comandante (jugador) y el rol `LEADER` (un miembro).

Lo que sigue es el respaldo que ya existía antes de ese bloque, y viene de fuera de este repo.

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

**Esta sección era el trabajo del bloque, y el bloque la cerró.** Las cuatro preguntas de §6.1 están
contestadas — una el 2026-08-24 por medición, tres el 2026-08-25 por voto— y el contrato entrante de §6.3
también. **Lo que sigue realmente abierto es §6.2 y §6.4**, y ninguna de las dos se destraba desde este repo.

> ⭐ **Y una lección de las tres que cerraron, porque se repitió en las tres:** ninguna se contestó como
> estaba formulada. La del scavenger escondía un supuesto (*ser behavior* ⇒ *vivir en Cortex*), la del estado
> venía con el *dónde* dado por sentado (persistir), y la del líder tenía **dos sujetos distintos** adentro.
> *Una pregunta abierta que se contesta tal cual vino suele ser una pregunta que nadie miró.*

### 6.1 Las cuatro preguntas del escuadrón — **LAS CUATRO CERRADAS**

1. ~~**¿Envuelve o reemplaza?**~~ **CERRADA el 2026-08-24 — ver §7.** Envuelve, y se midió qué. La respuesta cambió la pregunta: lo que se envuelve **no es el squad del engine** sino la maquinaria de schedules.
2. ~~**¿Quién integra un escuadrón?**~~ **CERRADA el 2026-08-25.** Un solo roster, con NPCs **y** jugadores; la asimetría no es de pertenencia sino de **dos ejes** (ingreso y ejecución). Detalle: doc particular §3.
3. ~~**¿Qué estado guarda, y qué le pasa al morir el líder?**~~ **CERRADA el 2026-08-25** (**CTX-10**). ⭐ Y la respuesta al *dónde* dio vuelta la premisa: **el roster no se persiste**, porque sus miembros son entidades y la cola guarda coordenadas de este mapa. Lo del líder traía además **dos sujetos distintos** adentro. Detalle: doc particular §5 y §8.
4. ~~**¿La formación es de la ORDEN o del ESCUADRÓN?**~~ **CERRADA el 2026-08-25 por voto del autor** (**CTX-11**): **del escuadrón**, con regla de suspensión. ⚠ Y **el mock no la decidía**: no dibuja ni una formación. Detalle: doc particular §6.

### 6.2 Dos paredes que este módulo no puede tirar solo

**(a) El campo «quién ejecuta» del menú.** El mensaje de net del menú lleva `(id de acción, entidad, component)` y **no tiene dónde poner al ejecutor**. Una orden de escuadra tiene **dos objetivos** —*quién* actúa y *dónde*— y **ninguna otra acción del menú tiene eso**. Es decisión del **framework** y se resuelve por el lado de `corpus/`; acá sólo se registra que **bloquea el cable** entre el menú y este módulo.

**(b) Un NPC no puede ser dueño de un inventario de Cargo — CTX-4, y está MEDIDO.** `Inventory.OwnerKey` (`../../corpus-cargo/lua/corpus_cargo/server/corpus_cargo_inventory.lua:101-104`) hace `ply:SteamID64() or ("bot" .. ply:EntIndex())`, y **`SteamID64` es un método de `Player`, no de `Entity`** — con un NPC **no cae en el `or`: revienta**. ⚠ **La puerta no está entornada: no existe**, y abrirla es una decisión de Cargo (*¿el dueño de un inventario deja de ser un jugador?*), no un parche de una línea desde acá.

Eso bloquea `Deploy` y la secuencia que el autor pidió, **«throw flashbang and clear»**, que además arrastra dos cosas más: **el efecto de flash sobre NPCs es pobre** (medido en juego: quedan ciegos unos pocos segundos — sede a leer: `dev/other/Arc9 EFT explosives/`), y **volver a la posición original** implica que la orden guarda un **punto de retorno**, o sea otra vez estado por escuadra.

### 6.3 ~~El contrato entrante que este bloque tiene que contestar~~ — **CONTESTADO el 2026-08-25**

**#6 — ¿el scavenger-pickup es behavior de Cortex? Sí, y por eso queda bajo la autoridad de Cortex sobre el cuerpo del NPC — pero NO se re-homea** (**CTX-8**). Es una adjudicación de **autoridad**, no de **hosting**.

⭐ **Lo que la medición dio vuelta, y por eso conviene tenerlo acá aunque el detalle esté en el doc particular §1:** el archivo **no es una cosa, son tres**, y sólo una es behavior; el acoplamiento **va al revés** de lo que decía la frase heredada —el scavenger no llama a limbs ni una vez, es limbs quien lo llama—; y **lo urgente no era el re-home**: hoy ese comportamiento ya conduce el cuerpo del NPC **por el mismo canal de §7.3**, así que le pisa el destino a una orden y le borra el schedule **sin dar un error**. De ahí sale **CTX-9**.

### 6.4 Qué es una «puerta» para el sistema

`prop_door_rotating` y `func_door` son entidades del engine. **Las cinco órdenes de puerta no existen sin esta definición**, y el menú ya le puso una precondición fuerte: la puerta **sólo trae sus cinco acciones si comunica dos lados navegables** —decorativa, tapiada o sin sala detrás no las trae, y **abierta o cerrada da igual**. Referencia disponible: el mod `immersive door openable` de `dev/other/`, que ya mapeó sus **siete keyvalues de sonido** en dos familias.

## 7. Las bases de NPC: qué expone cada una — MEDIDO

> **DECIDIDO por medición, el 2026-08-24.** Censo sobre el árbol real: VJ Base (**96 archivos, 28.431 líneas**), la fuente de Garry's Mod en disco (**60 defs de NPC** en `base_npcs.lua`), y —sólo como fuente de información sobre la API del engine, **no** como base a soportar— ZBase, `combat intelligence ai fixed` y `terminator nextbot`.
>
> El detalle con los call-sites uno por uno quedó en `dev/CORTEX_CENSO_BASES_NPC.md`. **Esa carpeta está fuera de git**, así que lo de acá es autosuficiente a propósito: si el censo se pierde, esta sección alcanza para ejecutar.

### 7.1 La respuesta: ENVUELVE — pero no lo que uno esperaría

**Cortex envuelve la MAQUINARIA DE SCHEDULES, no el squad del engine.** El motivo es que el «squad» de HL2 y el escuadrón de Cortex **no son la misma cosa**, aunque el nombre invite a creerlo:

| | **Squad del engine** | **Escuadrón de Cortex** |
|---|---|---|
| Qué agrupa | un **BANDO** | **a quién obedece el jugador** |
| Quién lo asigna | el spawn, por def del NPC | el jugador, en runtime |
| Tamaño | tope **16** | cuatro grupos de color |
| Para qué sirve | que el C++ coordine entre sí a los de la misma facción | dar órdenes con cola y formación |

**La prueba de que es un bando:** de las 60 defs de NPC de HL2, **46 nacen con squad**, en **siete nombres globales** — `overwatch` (18 NPCs), `resistance` (12), `zombies` (7), `antlions` (4), `poison` (2), `novaprospekt` (2), `npc_stalker_squad` (1). Todos los combines del mapa caen en el mismo hasta el tope de 16, y el sandbox desborda a `overwatch0`, `overwatch1`… ⇒ **a qué squad pertenece un NPC de HL2 es un accidente del orden de spawn.**

⚠ **Y son el MISMO CAMPO:** un NPC tiene un solo squad. Por eso hizo falta el voto que fija **CTX-6**.

### 7.2 VJ Base: no tiene escuadrones ni órdenes, pero sí la maquinaria

Sobre sus 28.431 líneas: **`Leader` da CERO hits**. `squad` da 10, y **ocho son un solo bloque del spawner copiado literal del sandbox de GMod** (su propio comentario cita la procedencia); los otros dos son enums del engine. De los 40 hits de `command`, **ninguno es una orden**: son `concommand`, `RunConsoleCommand` y el campo `Command` de botones de menú.

**Lo que sí tiene, y Cortex CONSUME en vez de duplicar:**

- **`ENT:Follow(ent, doToggle)`** — API pública documentada; devuelve `(bool, código)` con cuatro códigos de fallo: `1` estacionario, `2` **ya sigue a otra entidad**, `3` hostil o neutral. ⭐ El código 2 es una restricción de diseño **que ya existe**: un NPC de VJ sigue a UNA entidad a la vez.
- **`ENT:OnFollow(status, ent)`** — hook definido **vacío** en las dos bases. Es el punto de extensión sin fork.
- **Ocho métodos `SCHEDULE_*`**, todos con un parámetro `customFunc(schedule)` que corre sobre el schedule antes de arrancarlo ⇒ **se le agregan tasks sin tocar VJ**. `GOTO_POSITION` es *Move To*, `IDLE_STAND` es *Hold*, `COVER_*` y `FACE` cubren *Cover*.

### 7.3 HL2 puro: el mismo canal, distinto arranque

El patrón para mandar a un NPC del engine a un punto, **verificado en cuatro sitios independientes**:

```lua
npc:SetLastPosition(pos)
npc:SetSchedule(SCHED_FORCED_GO_RUN)   -- SCHED_FORCED_GO para caminar
```

> ⭐⭐ **Acá está el hallazgo que ahorra la mitad del trabajo: es el MISMO canal que usa VJ.** `SCHEDULE_GOTO_POSITION` arma `TASK_GET_PATH_TO_LASTPOSITION`, o sea que **las dos bases leen el destino de `LastPosition`**. *Move To* **no son dos implementaciones**: es un `SetLastPosition` común y dos formas de arrancar el schedule. Igual con entidades — VJ lee `GetTarget()` y el engine tiene `SCHED_TARGET_CHASE`/`SCHED_TARGET_FACE`.

El engine cubre solo *Move To*, *Cover*, *Hold*, *Search and Secure* y mirar. ⚠ **No tiene nada** para formaciones, *Stack Up*, *Breach* ni *Wedge*: todo eso lo escribe Cortex.

### 7.4 ⚠ Cuatro trampas medidas, y ninguna da error

1. **El destino NO es argumento de los métodos de movimiento de VJ.** El primer parámetro de `SCHEDULE_GOTO_POSITION(moveTask, customFunc)` es el **task de movimiento** (`TASK_RUN_PATH`/`TASK_WALK_PATH`), no la posición. Quien lea el nombre y le pase un vector **no recibe un error: recibe un NPC que camina al último punto que tuviera guardado.**
2. **`SCHEDULE_COVER_ENEMY` tiene el mismo cuerpo que `SCHEDULE_COVER_ORIGIN`** — los dos arman `TASK_FIND_COVER_FROM_ORIGIN`. `TASK_FIND_COVER_FROM_ENEMY` existe y VJ lo usa, pero en otro archivo. **El nombre del método miente sobre lo que hace.**
3. **No hay API para LISTAR los miembros de un squad.** `ai.GetSquadMemberCount` devuelve un **conteo** de un nombre que hay que conocer de antemano ⇒ **Cortex lleva su propia lista, obligatoriamente.**
4. **`CapabilitiesAdd`**: un NPC del engine sólo hace lo que sus capabilities permiten. Si una orden exige una que no tiene, **no falla ruidoso: no hace nada.** No se censó cuáles trae cada `npc_*`, y una orden de puerta depende de eso.

### 7.5 Lo que este censo NO puede contestar

- **Qué coordina realmente el C++ con un squad.** Es código del engine: se mide **en juego**, no se lee.
- **Si `SetSquad` en runtime reordena esa coordinación o sólo cambia una etiqueta.** ZBase lo usa como si funcionara; eso es evidencia de **uso**, no de **efecto**.
- **Las bases diferidas no se auditaron.** DRG y ZBase se consultaron por una pregunta puntual sobre la API del engine; este censo **no las evalúa** como bases a soportar.

## 8. Verificación

No hay test runner (es un addon GMod). Mientras el repo sea sólo docs, **«verificado» significa revisado contra su sede y contra el árbol real** — nunca contra el chat. La jerarquía de §7.1 del flujo pone al código por encima de todo doc, y **este módulo diseña por delante del código a propósito** (mock-first, flujo §3).

Cuando nazca código, el criterio de aceptación **no** puede ser paridad con nada: a diferencia de Caliber —que migró desde ADS y podía compararse contra el original— **Cortex no tiene predecesor**. Su verificación va a ser **planilla en juego con IDs de check estables** (flujo §1, Paso 4), citables desde el registro como evidencia tipo `planilla`.

⚠ **Y hay una lección de la casa que aplica antes de escribir el primer check:** el ecosistema tiene un catálogo de controles que dieron el veredicto equivocado, y la primera regla de ese catálogo es que **un check que no puede fallar no mide nada**. Cada tanda de checks va con su **suite de sabotaje**, que tiene que dar el total en rojo.

---

*Estado de HOY → [`cortex_estado.md`](cortex_estado.md). Rumbo → [`cortex_roadmap.txt`](cortex_roadmap.txt).
Contratos recibidos → [`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md). Normas del módulo → [`../CLAUDE.md`](../CLAUDE.md).
Framework y grafo de dependencias → [`../../corpus/docs/CORPUS_Architecture.md`](../../corpus/docs/CORPUS_Architecture.md).
El menú que hará de front-end → [`../../corpus/docs/Corpus_Interaccion_Arquitectura.md`](../../corpus/docs/Corpus_Interaccion_Arquitectura.md).*
