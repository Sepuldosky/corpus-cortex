# Cortex — Arquitectura del ESCUADRÓN

> **Doc PARTICULAR** (flujo §2). Se desprendió del general el 2026-08-25 por el criterio de esa
> sección — *«¿un implementador necesita este doc solo, sin el general ni el chat, para ejecutar?»* — y
> la respuesta es sí: acá está el modelo de estado completo, el arbitraje, la cola y la formación.
> [`Cortex_Architecture.md`](Cortex_Architecture.md) §5 queda como **resumen corto + link**, y el
> contenido **no se duplica** entre los dos.
>
> **Sigue sin haber una línea de código en este repo.** Este doc diseña por delante del código a
> propósito (mock-first, flujo §3), y lo dice para que nadie lo lea como descripción de algo construido.
>
> **Cómo leerlo.** Cada afirmación es de uno de cuatro tipos, marcado en el texto:
> - **DECIDIDO** — voto del autor, con fecha. Vinculante.
> - **HEREDADO** — viene de una sede de otro repo, y acá se **cita**, jamás se re-deriva.
> - **MEDIDO** — salió del árbol real, con su call-site. No se re-mide.
> - **ABIERTO** — nadie lo decidió, y está en §9.

---

## Índice

1. [El contrato entrante #6, contestado](#1-el-contrato-entrante-6-contestado)
2. [El arbitraje del cuerpo](#2-el-arbitraje-del-cuerpo)
3. [Quién integra un escuadrón](#3-quién-integra-un-escuadrón)
4. [Los grupos: amarillo es la UNIÓN](#4-los-grupos-amarillo-es-la-unión)
5. [El estado, entero — y qué se persiste](#5-el-estado-entero--y-qué-se-persiste)
6. [La formación es del ESCUADRÓN](#6-la-formación-es-del-escuadrón)
7. [La cola y el tick que sondea](#7-la-cola-y-el-tick-que-sondea)
8. [Qué pasa cuando alguien muere](#8-qué-pasa-cuando-alguien-muere)
9. [Lo que sigue ABIERTO, y lo que se mide en juego](#9-lo-que-sigue-abierto-y-lo-que-se-mide-en-juego)

---

## 1. El contrato entrante #6, contestado

**DECIDIDO por el autor el 2026-08-25.** La fila 6 de
[`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md) preguntaba si el *scavenger-pickup* es
behavior de Cortex. **La respuesta es sí — y el archivo NO se mueve.** Es una adjudicación de
**autoridad**, no de **hosting**, y las dos cosas se venían leyendo como una sola.

### 1.1 Por qué la pregunta escondía un supuesto

La fila se leía como *«¿es behavior? ⇒ entonces vive en Cortex»*. **MEDIDO sobre
`corpus-caliber/lua/corpus_caliber/server/corpus_caliber_scavenger.lua` (783 líneas): el archivo no es
una cosa, son tres**, y sólo una de las tres es comportamiento:

| Pieza | Qué es | De quién es el concepto |
|---|---|---|
| **(A) Valuación de armas** | `GetWeaponWeight`, los overrides, su persistencia (`Corpus.Data("caliber", "scav_weights")`), 3 concommands, 2 mensajes de net y la pestaña del browser | **arma** ⇒ Caliber |
| **(B) Contabilidad del mundo** | `MarkWeaponAsDroppedBy`, `IsScavengeable`, las ventanas de vida y de propiedad, y los tres hooks de detección (`OnNPCKilled`, `PlayerDroppedWeapon`, `OnEntityCreated`) | **arma tirada** ⇒ Caliber |
| **(C) El comportamiento** | el `Think` global, `ProcessScavengerNPC`, `FindBestWeapon`, `MoveNPCToWeapon`, `TryPickupAnimation`/`TryCrouchFallback`, `EquipWeapon` | **NPC** ⇒ Cortex |

**Y la dirección del acoplamiento es la contraria a la que sugiere la frase «acoplado al drop de
`Limbs`»:** el scavenger **no llama a limbs ni una vez** —cero referencias de código; las seis
menciones de `limb` en el archivo son comentarios—, es **limbs quien lo llama**, en 5 call-sites
(`corpus_caliber_limbs.lua:57,58,70,75,76`), **todos ya guardados con `if CALIBER.X then`**. Su única
dependencia hacia Caliber que él no define es `CALIBER.IsUserBlacklisted`, de core. Es decir: el
acoplamiento ya tiene **forma de contrato entre pares**, no de enredo interno.

### 1.2 Lo que costaba mover (C), medido

- **17 convars** `caliber_scavenger_*`, todas `FCVAR_REPLICATED + FCVAR_ARCHIVE`. **14 son de
  comportamiento** (`search_radius`, `pickup_distance`, `think_interval`, `interrupt_combat`,
  `force_all_npcs`, `movement_mode`, `post_drop_cooldown`, las tres de `retrieve_*`, las dos de
  `crouch_*`, `enabled` y `debug`) y sólo 3 de contabilidad. ⚠ **Archivadas** significa que renombrarlas
  **resetea en silencio la configuración de todo servidor ya instalado** — y ⚠ `FCVAR_REPLICATED` es
  invisible para el harness, así que eso se verificaría por comportamiento y no leyendo un archivo.
- **Y una regresión que el re-home no puede evitar:** hoy **Caliber solo** hace que los NPC recojan
  armas. Después del re-home, Caliber sin Cortex **pierde la feature entera**. Se pagaría una regresión
  real en una configuración real, a cambio de prolijidad de propiedad.

### 1.3 ⚠⚠ Lo urgente no era el re-home: hoy ya hay una colisión, y no da error

**MEDIDO, y es lo que cambió el marco.** El scavenger conduce el cuerpo del NPC por **el mismo canal
exacto** que [`Cortex_Architecture.md`](Cortex_Architecture.md) §7.3 midió como el canal de Cortex.
Tres colisiones, ninguna ruidosa:

1. **`MoveNPCToWeapon` hace `npc:SetLastPosition(weapon:GetPos())`** y arranca
   `SCHEDULE_GOTO_POSITION` (VJ) o `SCHED_FORCED_GO_RUN` (nativo). ⇒ **le reescribe el destino a una
   orden de Cortex en vuelo.** Es el hallazgo de §7.3 dado vuelta: *el canal compartido también se
   comparte con quien no querías*.
2. **La animación de pickup en la rama VJ usa `VJ_ACT_PLAYACTIVITY` con `lockAnim=true`**, que —según el
   comentario del propio archivo, que documenta por qué se eligió— **hace `StopMoving` + `ClearSchedule`**.
   ⇒ **mata el schedule de la orden**, no sólo lo desvía.
3. ⭐ **Y la peor, porque premia su modo de falla: CTX-7 dice que la cola SONDEA `IsCurrentSchedule`.**
   Si el scavenger cambió el schedule por abajo, el sondeo de Cortex lee **un schedule que Cortex no
   puso** y puede concluir *«orden terminada»* cuando el NPC se fue a buscar una escopeta. **Un falso
   verde que sólo se descubre en el juego, y que mientras tanto acredita.**

**El gate que existe hoy no alcanza, y por un motivo honesto:** `ProcessScavengerNPC` sólo mira combate
(`IsValid(npc:GetEnemy())` más la convar `interrupt_combat`). **No hay gate de «este NPC está bajo
órdenes»** porque, cuando se escribió, no existía nadie que diera órdenes. Fuera de combate, un NPC
armado se va igual a buscar un arma mejor dentro de `search_radius` (**800 u** por defecto, ≈ 20 m con
la constante de la casa de 1 u = 2,5 cm).

### 1.4 El veredicto

> **El scavenger-pickup ES comportamiento de NPC, y por eso queda sujeto a la autoridad de Cortex sobre
> el cuerpo del NPC. Pero NO se re-homea: se queda entero en Caliber y pasa a ser el PRIMER CLIENTE del
> arbitraje de §2.** (**CTX-8**)

Lo que eso compra, y por qué es mejor que las otras dos salidas:

- **Cero migración.** No se renombra una convar, no se mueve un `.json`, no se toca la pestaña del
  browser ni un `net string`.
- **Cero regresión.** Sin Cortex montado, `Corpus.GetModule("cortex")` da `nil` y el scavenger corre
  **exactamente como hoy**. ⭐ **El arbitraje sólo hace falta cuando Cortex existe** — que es
  justamente cuando está disponible. La soft-dep cae en el lugar correcto sola.
- **Y no parte el archivo por la mitad**, que es lo que el handoff advertía y con razón: dejar (A) de un
  lado y (C) del otro habría sido peor que no moverlo.

⚠ **Lo que este veredicto NO dice:** no dice que (C) esté bien escrito donde está, ni que nunca se
mueva. Dice que **hoy** el costo del movimiento supera al beneficio, y que **el problema real era otro**.
Si algún día Cortex tiene su propia primitiva de animación y (C) queda duplicándola, la conversación se
reabre — pero se reabre **midiendo la duplicación**, no citando la prolijidad.

---

## 2. El arbitraje del cuerpo

**DECIDIDO — CTX-9.** Cortex es la autoridad sobre **quién conduce a un NPC en un momento dado**, y
todo comportamiento autónomo del ecosistema pide el volante antes de mover un cuerpo y lo suelta al
terminar.

### 2.1 La forma

```
Cortex.Body.Claim(npc, holder, priority)  -> bool   -- ¿me lo das?
Cortex.Body.Release(npc, holder)                    -- lo suelto
Cortex.Body.Holder(npc)                   -> string|nil
```

- **`holder` es un string namespaced** que dice quién es: `"cortex.order"`, `"caliber.scavenger"`, …
  Un string y no una tabla, para que se pueda loguear e imprimir sin resolver nada.
- **`priority` es un número, y la regla que lo ordena no es negociable: la orden de un jugador SIEMPRE
  gana.** El jugador es la autoridad del sistema; un comportamiento autónomo cede. Un autónomo **no
  puede quitarle** el cuerpo a una orden.
- **`Release` es obligatorio**, y un claim tiene vencimiento: si el holder muere, se descarga o
  simplemente se olvida, el claim caduca solo. Un volante que se puede quedar trabado para siempre es
  peor que no tener volante.

### 2.2 ⚠ Las dos cosas que un claim NO es, y hay que escribirlas

1. **NO es un lock del engine.** Nada impide que VJ, otro addon, o el propio FSM del NPC lo muevan
   igual. El claim **coordina a los módulos de Corpus entre sí y no puede prometer más que eso**.
   Leerlo como *«está claimeado, entonces nadie lo toca»* es exactamente el falso verde que este doc
   está tratando de evitar en otro lado.
2. **NO reemplaza al sondeo de CTX-7.** Tener el volante no dice que la orden terminó. Son dos
   preguntas distintas — *«¿me toca a mí?»* y *«¿ya llegó?»* — y confundirlas hace que una orden
   quede colgada esperando un evento que no existe.

### 2.3 Cómo lo consume el scavenger

Un solo punto de entrada, dentro de `ProcessScavengerNPC`, junto al gate de combate que ya está:

```
si Cortex está montado y Cortex.Body.Holder(npc) no soy yo y no es nil
    -> no scavengeo este tick
```

y `Claim` antes de `MoveNPCToWeapon`, `Release` después del equip (éxito o fracaso — el
`ApplyPostDropCooldown` que ya corre en las dos ramas es el sitio natural).

⚠ **Cuidado con el `timer.Simple` de `EquipWeapon`:** el equip está diferido a `animDuration * 0.7`
y valida a los 0,1 s más. **El `Release` va en el último de esos timers, no en la llamada
sincrónica** — soltarlo antes deja al NPC clavado en su animación de pickup con el volante en manos
de una orden que ya empezó a moverlo.

---

## 3. Quién integra un escuadrón

**DECIDIDO.** El escuadrón tiene **un solo roster**, y en él conviven NPCs y jugadores. Lo que los
distingue **no es la pertenencia**: son **dos ejes independientes**, y tratarlos como uno solo es el
error caro.

| Eje | NPC | Jugador |
|---|---|---|
| **Ingreso** (consentimiento) | se le **asigna**, unilateral | se lo **invita**, y hace falta que acepte |
| **Ejecución** (qué le pasa al cuerpo) | la orden **le mueve el cuerpo** | la orden **le llega como aviso**; su cuerpo no se toca jamás |

**HEREDADO** de `../../corpus/docs/Corpus_Interaccion_Arquitectura.md` §8.ter y del mock v2 (nota 27):
*«Al NPC se lo asigna; al jugador se lo invita»*, la invitación pendiente se dibuja punteada en ámbar
con `INV`, **el invitado todavía no es miembro** (no tiene tecla, no cuenta en el grupo), y acepta desde
`self → Group`, porque unirse a un grupo es algo que uno hace **sobre sí mismo**. *Ningún sistema mete a
un jugador en un grupo sin que él diga sí.*

### 3.1 ⚠ El comandante NO es una fila del roster

**El jugador que manda es el DUEÑO de la escuadra, no un miembro de ella.** El menú lo llama *Team
Leader* (§5.3). Confundirlo con un miembro rompe dos cosas a la vez: se contaría a sí mismo en `ALL`, y
se le podría dar una orden a sí mismo.

### 3.2 ⚠⚠ Y hay DOS «líderes», que no son lo mismo

Esta es la trampa que la pregunta *«¿qué pasa al morir el líder?»* traía escondida:

- **El COMANDANTE** — el jugador dueño. No es una fila. Su muerte es §8.1.
- **El rol `LEADER`** — un **rol de un miembro**, del mismo tipo que `MEDIC` o `POINT`. El mock lo
  muestra así: `BRAVO-1 LEADER 12 m`, con tecla y distancia, o sea **un NPC**. Su muerte es §8.2.

### 3.3 La identidad de un miembro: dos tipos de clave, y no es un capricho

**Un miembro NPC es una `Entity`. Un miembro jugador se referencia por `SteamID64`.** No se pueden
unificar, y ya hay precedente medido de qué pasa cuando se intenta: **CTX-4** — `Inventory.OwnerKey` de
Cargo hace `ply:SteamID64()`, que es método de `Player` y **no** de `Entity`, así que con un NPC
**revienta en vez de caer en el `or`**. El roster lleva `kind` explícito (`"npc"` | `"player"`) y **el
despachador de órdenes tiene dos sumideros**, no uno con un `if` adentro.

⚠ **Y una honestidad de cara al jugador que sale de acá:** la columna `ORDER` del roster, para una fila
de jugador, muestra **la orden que RECIBIÓ**, no una que esté ejecutando. Una etiqueta que sugiera lo
segundo estaría mintiendo — misma regla que el bloque del menú aplicó a los gestos de GCAL.

---

## 4. Los grupos: amarillo es la UNIÓN

**DECIDIDO por el autor el 2026-08-25**, resolviendo una contradicción entre dos fuentes (**CTX-12**).

- La columna WHO del mock v2 rotula la primera fila **`ALL · YELLOW GROUP`** y le pone al lado **el
  total** — 5, contra `RED GROUP` 2 y `BLUE GROUP` 2. **Su propia aritmética dice que el amarillo es la
  unión**, no un subconjunto.
- La prosa de §8.ter del doc del menú dice *«los cuatro grupos: amarillo, rojo, azul y verde»*.

> **Gana la unión. Los ELEMENTOS son TRES —rojo, azul y verde— y el amarillo es «todos».**

Consecuencias directas sobre el modelo de datos, que es la razón por la que había que decidirlo:

- El campo `group` de un miembro vale **`red` | `blue` | `green` | `nil`**, y `nil` significa *en la
  escuadra, en ningún elemento*. **Nunca vale `yellow`**: el amarillo no se guarda, se **calcula**.
- `ALL` (tecla `0`) selecciona el roster entero, incluidos los de `group = nil`.
- **El techo de tres elementos no es de la UI: es del theme.** §8.ter lo midió — son los colores
  señalizables que hay, y ⚠ `olive` **no declara `green`**, por eso los grupos llevan **letra R·B·G·Y
  además del color**.

⚠ **Deuda declarada, y es de la otra sede:** la frase *«los cuatro grupos»* de §8.ter queda
desactualizada. **No se la toca desde acá** — `corpus/` está siendo trabajado por otra sesión en
paralelo, y la sede del **estado de escuadra** es este repo (**CTX-3**), así que la corrección de esa
prosa es un parche del lado del framework, anotado en el CHANGELOG.

---

## 5. El estado, entero — y qué se persiste

**DECIDIDO — CTX-10.** Primero el inventario completo, porque hasta hoy no existía en ningún lado.

### 5.1 Por escuadra (una por jugador comandante)

| Campo | Qué es |
|---|---|
| `owner` | `SteamID64` del comandante |
| `members[]` | `{ kind, ref, callsign, role, group, key }` — ver §3.3 |
| `invites[]` | `SteamID64` + cuándo se mandó. **No son miembros** |
| `formation` | la formación vigente — §6 |
| `queue[]` | filas `{ orderId, group, target, args }`, **cada una con su PROPIO objetivo** |
| `delayArmed` | si el modificador `Delay` está armado |
| `frozen` | la cola está congelada (comandante muerto — §8.1) |

### 5.2 Por miembro, y en la entidad

| Campo | Por qué vive en la entidad y no en la tabla |
|---|---|
| `currentOrder` + su estado de sondeo | lo lee el tick de §7, que ya tiene al NPC en la mano |
| `originalSquad` | **HEREDADO de CTX-6**: el squad del engine se guarda **desde el día uno**. Tiene que sobrevivir a que el NPC SALGA de la escuadra, y sin el NPC no significa nada |
| `returnPos` | el punto de retorno de *throw flashbang and clear* (§8.bis del menú). Sólo mientras dura esa secuencia |

**Derivados, y no se guardan nunca:** salud, distancia, clase, y **la pertenencia al amarillo**.
Guardar un derivado es fabricarse una segunda fuente de verdad que se desincroniza sin avisar.

### 5.3 ⭐ Casi nada de esto se puede persistir, y decirlo es la mitad de la respuesta

La pregunta abierta era *«qué estado guarda, y dónde»*, con la nota de que persistir va por
`Corpus.Data` namespaced (**COR-3**). **La respuesta es que el roster NO se persiste, y no por
elección: por lo que es.**

- **Los miembros NPC son ENTIDADES.** No sobreviven a un cambio de mapa, ni a un `cleanup`. Rehidratar
  un roster guardado significaría resolver referencias a entidades que ya no existen. ⚠ Y ni siquiera
  se puede guardar el `EntIndex` como sustituto: **los índices se reciclan** — el propio scavenger lo
  dice en su código, en el comentario de `RecordOwnWeaponDrop`: *«entity reference, never EntIndex»*.
- **La cola guarda destinos en coordenadas de ESTE mapa.** Restaurarlos en otro es mandar NPCs a un
  punto que ya no significa nada.
- **Los callsigns y los roles cuelgan de miembros que no van a existir.**

> **⇒ El roster, la cola y los roles son estado VOLÁTIL de runtime. No se persisten.**

**Lo único que califica para `Corpus.Data`**, porque sobrevive a un mapa por naturaleza:

- **Los nombres de los grupos**, si el autor mantiene el *renombrar* de §8.ter.
- **La membresía de un JUGADOR** en la escuadra de otro, que se referencia por `SteamID64`.

Y va con **`opts = { scope = "save" }`** —estado de partida, muere con ella— **no `config`**. La
primitiva ya toma el parámetro y hoy los dos scopes resuelven a la misma carpeta a propósito
(`corpus_data.lua`, cabecera): declararlo hoy no mueve un archivo y evita la migración del día que el
layout de perfiles llegue.

---

## 6. La formación es del ESCUADRÓN

**DECIDIDO por el autor el 2026-08-25 — CTX-11.** *Fall In* trae cuatro (fila simple, fila doble,
diamante, cuña), y la pregunta era si son argumento de la orden o estado de la escuadra. **Son estado
de la escuadra**: persisten, y la escuadra se mueve manteniéndolas.

⚠ **Y hay que decir de dónde NO salió esta respuesta.** El mock v2 **no la decide**: no dibuja ni una
formación —sus dos apariciones de `Wedge` son la orden de **puerta**, que traba una hoja— y `Fall In`
es **una hoja sin hijos** en la lista de `ORDERS`. Leer su silencio como *«entonces no es estado»*
habría sido un argumento por ausencia sobre un dibujo que nunca cubrió el tema. **Es un voto, no una
lectura.**

### 6.1 Lo que el voto arrastra, dicho en voz alta

La opción barata era la contraria, y la diferencia es un subsistema entero. Con la formación como
estado hacen falta tres cosas que la otra no necesitaba:

1. **Asignación de slot por miembro** — quién va en qué punta de la cuña, y que no se reasigne cada
   tick (o los NPC se cruzan entre sí eternamente).
2. **Un ancla y un rumbo.** Una formación no existe sin un punto de referencia y una dirección de
   encare: en *Fall In* el ancla es el comandante y el rumbo es su mirada; en *Move To* es el destino
   y el rumbo es la dirección de avance.
3. **Re-emisión periódica del destino de cada miembro**, porque el ancla se mueve.

### 6.2 ⚠ Y una regla de suspensión, o la escuadra entra en cuña a un tiroteo

**El estado persiste; su APLICACIÓN es condicional.** La formación se aplica sólo mientras la escuadra
está en un modo **cohesionado**, y estas cosas la **suspenden sin borrarla**:

| Qué la suspende | Por qué |
|---|---|
| **Combate** | la IA de la base toma el control: cubrirse y disparar gana sobre mantener el slot |
| **`Hold`** | *«para todo; sólo se defiende»* — cada uno aguanta donde está |
| **`Cover`** | fija la atención en un ángulo; la posición no la manda la formación |
| **`Stack Up`** | la puerta dicta las posiciones, y son explícitas |

Al terminar la suspensión, la formación **vuelve sola**, porque nunca se borró.

### 6.3 ⭐ MEDIDO: `ENT:Follow` de VJ NO sirve para mantener una formación

Antes de escribir el mecanismo se midió si VJ ya lo tenía, porque §7.2 del general anotó `ENT:Follow`
como punto de extensión. **Se leyó entero (`dev/other/VJ Base/lua/vj_base/ai/core.lua:1759` y el bloque
de mantenimiento en `npc_vj_human_base/init.lua:2638-2695`) y la respuesta es no**, por una razón
específica:

- **La tolerancia de `Follow` es un ANILLO, no un punto.** `FollowData.MinDist = FollowMinDistance +
  self:OBBMaxs().y + ent:OBBMaxs().y`, con **`FollowMinDistance = 100` u** por defecto (≈ 2,5 m), y el
  schedule apunta a `TASK_MOVE_TO_TARGET_RANGE, MinDist * 0.8`. El NPC **para cuando entra al anillo**,
  en cualquier punto de él. ⇒ **±2,5 m de error por miembro, más ancho que la separación entre slots.**
- **`Follow` sigue una ENTIDAD** (`SetTarget` + `SCHEDULE_GOTO_TARGET`), no una posición. Un slot no es
  una entidad, salvo que se cree un ancla invisible por miembro — y aun con el ancla queda el anillo.
- ⚠ **Y sólo existe en VJ.** HL2 nativo no tiene nada equivalente. Delegar en `Follow` sería escribir
  *dos* implementaciones de la formación, cuando el canal de §7.3 permite **una**.

⚠ **Tres trampas de `Follow` que quedan escritas igual, porque muerden si alguien lo usa para otra
cosa** (por ejemplo, un *Follow* individual sobre un NPC apuntado, que **sí** está en el catálogo):

1. **Le imprime en el chat al jugador.** `ent:PrintMessage(HUD_PRINTTALK, …)` en cinco sitios —
   *«X is now following you»*, *«… is no longer following you»*, *«… has been killed»*, *«… isn't
   friendly»*, *«… is following another entity»* — gateado por `self.CanChatMessage`. Con una escuadra
   de seis, cada reagrupe le spamea seis líneas.
2. **Un NPC de VJ sigue a UNA entidad a la vez** (código de fallo `2`), así que colisiona con cualquier
   otro sistema que use `Follow` — incluido el propio menú de VJ. **Eso lo cubre el arbitraje de §2.**
3. **`Follow` pisa `SetTarget` y resetea `GuardData`**, que son campos que VJ usa para otras cosas.

### 6.4 El mecanismo que sí

**Sobre el canal compartido de §7.3** —`SetLastPosition` + `SCHEDULE_GOTO_POSITION` (VJ) o
`SCHED_FORCED_GO`/`SCHED_FORCED_GO_RUN` (nativo)—, con el destino de cada miembro calculado como
`ancla + rotar(offset_del_slot, rumbo)`, **re-emitido por el tick de §7**. Una sola implementación para
las dos bases, que es exactamente lo que el hallazgo de §7.3 compró.

⭐ **La cadencia no hay que inventarla: VJ re-emite su propio follow cada 0,5 s** (`NextUpdateT =
curTime + 0.5`, medido en las dos bases de VJ) **y le alcanza**. Ése es el punto de partida honesto, y
sale de la base, no de una estimación.

⚠ **Pero es un punto de partida, no un resultado.** Re-emitir un schedule forzado es pisar lo que el
NPC esté haciendo, y a qué cadencia se puede hacer **sin romper el combate ni las animaciones**
es **medición en juego**, no lectura. Va a §9.

⚠ **Dos umbrales más que VJ ya calibró y conviene no re-inventar:** re-emite sólo si la distancia supera
`MinDist`, y **si supera `MinDist * 4` corta las actividades** (`StopAct = true`) para alcanzar. Un
miembro que quedó muy atrás **rompe formación para alcanzar** — eso ya es una decisión tomada por una
base que funciona.

---

## 7. La cola y el tick que sondea

**HEREDADO** del menú (§8.bis): `Delay` **no es una orden, es un MODIFICADOR**. Con Delay armado, cada
orden **se va a la cola** en vez de ejecutarse, y `Execute` las manda todas juntas — es la única forma
de que dos grupos entren al mismo tiempo. ⚠ **Cada fila guarda su PROPIO objetivo**: el destino se
resuelve **cuando diste la orden**, no cuando disparas.

**HEREDADO de CTX-7:** no hay evento de *«orden terminada»*. Para un NPC del engine corriendo un
`SCHED_` nativo el único tell es `IsCurrentSchedule(SCHED_X)`, y ⚠ `TaskComplete()` **no sirve** —es
algo que un NPC se llama a sí mismo dentro de un schedule *custom*, no un callback que el engine le
entregue a un tercero. **La cola necesita un tick que sondee.**

> ⭐ **Y ese tick es el mismo que mantiene la formación (§6.4).** Un solo tick con dos trabajos —
> *«¿ya llegó?»* y *«¿sigue en su slot?»*— y no dos loops que se pisen. La restricción de CTX-7 y el
> costo del voto de §6 **se pagan una sola vez**.

⚠ **Y acá vuelve la colisión de §1.3, ahora del lado correcto:** el sondeo lee un schedule que **puede
no ser el que Cortex puso**. El arbitraje de §2 es lo que hace que la lectura signifique algo — y aun
así el sondeo **tiene que verificar que el schedule que lee es el suyo**, no confiar en que nadie lo
tocó. Un tercero fuera de Corpus no pide volante.

---

## 8. Qué pasa cuando alguien muere

### 8.1 El COMANDANTE — DECIDIDO por el autor el 2026-08-25

> **La escuadra sobrevive, la cola se CONGELA, y los miembros pasan a `Hold`. Al respawnear, el
> comandante la recupera intacta.**

El motivo: la escuadra es suya y va a respawnear. **Con el comandante muerto no se ejecutan órdenes
nuevas** —no puede ver ni corregir nada— y **aguantar posición es lo único que no lo compromete**.
Disolverla habría sido castigar una muerte con trabajo manual en la pantalla de administración, que es
justo lo que el juego de referencia no hace.

Implementación: es el flag `frozen` de §5.1. **La cola no se vacía**: se congela y se descongela.

### 8.2 Un miembro con ROL — DECIDIDO

**Se cae el ROL, no la escuadra.** El rol queda **vacante** y no se reasigna solo: reasignarlo es
*designar*, y eso es una acción del jugador (§8.bis del menú lo pone como acción sobre un NPC apuntado).

⭐ **Y el vocabulario para mostrarlo ya existe y no hay que inventarlo:** el mock deja `Heal` **en
gris** con la explicación *«existe, pero el grupo seleccionado no tiene médico»*. **Perder un rol
grisea lo que dependía de él** — el sistema existe y la condición dio `false`, que es exactamente lo que
el gris significa en §6 del menú, y **no** el punteado, que significa *no hay sistema*.

### 8.3 Un miembro cualquiera

Sale del roster. Si tenía una fila en la cola dirigida a su grupo, la fila **sigue viva** mientras
quede al menos un miembro en ese grupo; si el grupo quedó vacío, la fila se descarta **y se dice**.

---

## 9. Lo que sigue ABIERTO, y lo que se mide en juego

### 9.1 Sigue abierto, y no lo destraba este bloque

- **El campo «quién ejecuta» del mensaje de net.** Lleva `(id, entidad, component)` y **no tiene dónde
  poner al ejecutor**; una orden de escuadra tiene **dos** objetivos. Es decisión del **framework**, se
  trabaja en otra sesión, y **bloquea el cable, no el diseño**.
- **`Deploy` y *throw flashbang and clear*** — bloqueadas por **CTX-4** (Cargo), no por Caliber.
- **Qué es una «puerta»** para el sistema. Sin esa definición no existen las cinco órdenes de puerta.
- **Munición finita para NPCs**, que es lo que justificaría el `Ammo box` y una orden de reabastecer.
  Es una decisión de **simulación**, no de inventario, y el menú la dejó explícitamente como pregunta
  para este módulo.
- **Dónde se elige la formación en la pantalla.** Con el voto de §6 la formación es **estado**, y la
  columna WHO es **la** sede del estado por su propia definición — pero **el mock no la dibuja**. Es un
  hueco del mock, no de este diseño, y se resuelve del lado del menú.

### 9.2 Los cinco huecos que se miden EN JUEGO — candidatos a la planilla `AP`

Los tres primeros vienen del censo de bases; los dos últimos los abrió este bloque.

1. **Qué coordina realmente el C++ con un squad.** Es código del engine: se mide, no se lee.
2. **Si `SetSquad` en runtime reordena esa coordinación o sólo cambia una etiqueta.** ZBase lo usa como
   si funcionara; eso es evidencia de **uso**, no de **efecto**.
3. **Qué capabilities trae cada `npc_*` de HL2.** ⚠ `CapabilitiesAdd`: si una orden exige una que el
   NPC no tiene, **no falla ruidoso: no hace nada.** Una orden de puerta depende de esto.
4. **A qué cadencia se puede re-emitir el destino de formación sin romper combate ni animaciones**
   (§6.4). El 0,5 s de VJ es el punto de partida medido, **no el resultado**.
5. **Si el arbitraje de §2 realmente elimina la colisión de §1.3.** ⚠ Y ojo con cómo se escribe ese
   check: *«el NPC llegó al destino»* **no** lo prueba —puede llegar igual, tarde, después de un
   desvío— y un check que no puede fallar no mide nada. Lo que hay que hacer fallar es el caso **con el
   arbitraje desconectado**, y esa suite de sabotaje tiene que dar el total **en rojo**.

---

*Resumen y link desde el general → [`Cortex_Architecture.md`](Cortex_Architecture.md) §5.
Estado de HOY → [`cortex_estado.md`](cortex_estado.md). Rumbo → [`cortex_roadmap.txt`](cortex_roadmap.txt).
Normas del módulo → [`../CLAUDE.md`](../CLAUDE.md). Contratos recibidos → [`Cortex_ContratosEntrantes.md`](Cortex_ContratosEntrantes.md).
El menú que hace de front-end → [`../../corpus/docs/Corpus_Interaccion_Arquitectura.md`](../../corpus/docs/Corpus_Interaccion_Arquitectura.md).*
