# Cortex — CHANGELOG de parches (repo: corpus-cortex/)

> Historial de parches de `corpus-cortex/`. Nació el 2026-07-22 como primer archivo de
> proceso del repo, cuando éste era semilla: solo `LICENSE`, `README.md` y
> `docs/Cortex_ContratosEntrantes.md` (el módulo no había abierto su Block — ni código, ni
> `CLAUDE.md`). **El repo se fundó el 2026-08-24** y hoy tiene su set de docs completo;
> sigue sin código.
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

---

## PARCHES DE sesión Fundación del repo (bloque de escuadrones) — 2026-08-24

El autor abre el **bloque de escuadrones** y con eso el módulo recibe contenido real por
primera vez, así que corresponde el punto (c) del flujo §2: el repo hermano recibe su propio
`CLAUDE.md` + `docs/{estado, roadmap, CHANGELOG, convenciones_commits, Architecture}`, mismo
template que `corpus/`, apuntando al `flujo_trabajo.txt` canónico en vez de duplicarlo.

**Lo que desbloquea, y por qué era un bloqueante duro y no una formalidad:** hasta hoy la
familia de normas de este repo estaba **reservada sin entradas**, porque su sede declarada
—este `CLAUDE.md`— no existía. El CHECK 2a del checker de IDs se pone rojo si alguien acuña la
primera norma de una familia contra un archivo fantasma. **Sin fundar el repo, Cortex no podía
tener ni una norma**, y su doc de convenciones llevaba desde el 2026-07-22 una regla real
esperando en prosa a que hubiera dónde acuñarla.

Alcance: **sólo prosa**. Ni una línea de Lua — lo que abrió es un bloque de DISEÑO.

- PARCHE 1 — **`CLAUDE.md`**: la sede de la familia del módulo. Acuña sus **cinco primeras
  normas** —la regla cardinal de que Cortex es una CAPA sobre bases de NPC que ya existen y no
  una base nueva; la partición en dos mitades con gates independientes; que el escuadrón vive
  acá y el menú de Corpus es sólo su front-end; que Cortex no parchea la propiedad de
  inventario de Cargo; y los alcances de commit— más el mapa de docs, el workspace multi-repo,
  las diez normas del ecosistema que este repo **cita y no redefine**, y las dos guardias que
  corren antes de cerrar prosa (el checker de IDs y el de voseo). **[APLICADO 2026-08-24]**
- PARCHE 2 — **`docs/cortex_estado.md`**: foto del AHORA. Qué existe, qué no existe (y no
  leerlo de más), **quién bloquea qué** —el afecto lo bloquea Caliber, `Deploy` lo bloquea
  Cargo, y el campo «quién ejecuta» del net lo bloquea el framework— y los tres próximos pasos.
  **[APLICADO 2026-08-24]**
- PARCHE 3 — **`docs/cortex_roadmap.txt`**: el rumbo. Abre con la partición en dos mitades,
  porque es lo que ordena todo lo demás; después el bloque inmediato de escuadrones en cinco
  puntos, las tres cosas que el bloque **no puede resolver solo**, y los bloques futuros
  (facciones, afecto, loot table, CortexBase, las bases diferidas). **[APLICADO 2026-08-24]**
- PARCHE 4 — **`docs/Cortex_Architecture.md`**: doc de fundación. Marca cada afirmación como
  **DECIDIDO**, **HEREDADO** o **ABIERTO**, y su §6 —lo abierto— es la sección más larga a
  propósito: presentar como diseño lo que todavía es una pregunta es la forma en que este tipo
  de doc miente. **[APLICADO 2026-08-24]**
- PARCHE 5 — **`docs/cortex_convenciones_commits.txt`**: (a) el bloque ESTADO decía «el repo es
  semilla», que dejó de ser cierto en esta misma tanda; (b) la regla de la §3 **toma su ID**,
  con la espera anotada para que se entienda por qué una regla vieja tiene un ID nuevo; (c)
  nace el alcance **`escuadra`**, por voto del autor al abrir el bloque —el escuadrón guarda
  estado propio que no es ni táctica ni plumbing, y meterlo en `tactica` haría indistinguible
  un commit de formación de uno de supresión—; (d) el alcance `caliber` decía que era «el gate
  del módulo», y es el gate de **una mitad**; (e) el pie listaba `CLAUDE.md` como
  `NO EXISTE todavía`. **[APLICADO 2026-08-24]**
- PARCHE 6 — **`docs/Cortex_ContratosEntrantes.md`**: ratificación, sin tocar una sola fila de
  la tabla. El encabezado y §5 afirmaban que el repo no tenía `CLAUDE.md` y que la familia
  estaba reservada sin entradas. **La nota de método sobre el `HUERFANO_DOC` se conserva** —la
  lección no caducó, y de hecho **le volvió a pasar a otra redacción el 2026-08-24**, en
  `Corpus_Interaccion_Arquitectura.md` §5.4. Y §3.6 gana una línea: el contrato 6 se decide «en
  el diseño de Cortex», y ese diseño **abrió hoy**. **[APLICADO 2026-08-24]**
- PARCHE 7 — **`README.md`**: decía «Estado: sin empezar», y era falso en dos mitades — el
  bloque abrió, y el gate de Caliber cubre el afecto y no el módulo entero. Reescrito en
  lenguaje de visitante, sin IDs internos (misma disciplina que la nota de STK-8: un ID interno
  ensucia la cara pública del repo en GitHub). **[APLICADO 2026-08-24]**

**Verificación:** sin superficie de runtime (el repo sigue sin código), así que «verificado» =
revisado contra su sede y contra el árbol real. Las citas a otros repos se adjudicaron
**abriendo el archivo**, no copiándolas entre docs (flujo §7.3.b): la línea de `OwnerKey` de
Cargo se leyó en el archivo, las carpetas de `dev/other/` se listaron, y el propio checker de
IDs corrió en verde antes y después. El barrido de ratificación cruzó las siete raíces por el
**valor** —«repo semilla», «sin `CLAUDE.md`», «familia sin entradas»— y no por la lista de
destinos que traía el plan; lo que apareció en `corpus/` se corrigió en el mismo parche y está
registrado en el CHANGELOG de ese repo. No commiteado ni pusheado (GIT-7).

---

## PARCHES DE sesión Censo de las bases de NPC — punto [2] CERRADO — 2026-08-24

Segunda tanda del mismo día, sobre el repo recién fundado. **Punto [2] del roadmap:** leer las
bases de NPC y contestar si el escuadrón de Cortex **envuelve** lo que ya existe o lo
**reemplaza**. Sólo prosa: el repo sigue sin código.

**La respuesta: ENVUELVE — pero no el squad del engine, sino la MAQUINARIA DE SCHEDULES.** La
pregunta estaba mal planteada, y eso salió de medir: *«squad»* nombra dos cosas distintas.

- PARCHE 1 — **`docs/Cortex_Architecture.md` §7 (nueva)**, con la §8 renumerada. Autosuficiente a
  propósito: el detalle con los call-sites quedó en `dev/CORTEX_CENSO_BASES_NPC.md`, y **esa
  carpeta está fuera de git** — si el censo se pierde, la sección sigue alcanzando para ejecutar.
  La pregunta 1 de §6.1 pasa de ABIERTA a cerrada. **[APLICADO 2026-08-24]**
- PARCHE 2 — **`CLAUDE.md` acuña CTX-6**: el escuadrón corre **en PARALELO** al squad del engine;
  no se pisa `SetSquad` y el original **se guarda desde el día uno**. Votado por el autor sobre el
  censo. El motivo es que **son el mismo campo** y el del engine es un **BANDO**: de las 60 defs de
  NPC de HL2, **46 nacen en siete nombres globales de facción** (`overwatch` 18, `resistance` 12,
  `zombies` 7…), con desborde a `overwatch0`/`overwatch1` al pasar el tope de 16. Meter un combine
  en «el grupo rojo del jugador» **le sacaría la coordinación con los otros combines**, que el
  engine hace en C++. Costo aceptado y dicho en voz alta: los miembros de un grupo **no se
  coordinan entre sí** por el motor. **[APLICADO 2026-08-24]**
- PARCHE 3 — **`CLAUDE.md` acuña CTX-7**: la cola de órdenes **SONDEA**. Medido: 40 usos de
  `IsCurrentSchedule` contra **cero** de cualquier callback de fin de schedule. ⚠ Y la trampa que
  la norma existe para evitar: **`TaskComplete()` no sirve**, aunque tenga 185 usos — es algo que
  un NPC **se llama a sí mismo** dentro de un schedule *custom*, confirmado abriendo sus
  call-sites. Quien lo lea por el nombre diseña la cola por eventos y la cola no avanza nunca.
  **[APLICADO 2026-08-24]**
- PARCHE 4 — **`docs/cortex_estado.md`** y **`docs/cortex_roadmap.txt`**: el punto [2] pasa a
  HECHO con sus cinco resultados, el [3] (contrato #6) es lo próximo y su criterio de entrada ya
  está cumplido. Del roadmap se **eliminó** la receta del censo ya ejecutado: conservaba la línea
  *«y hoy no está»*, que la propia tanda volvió falsa. Queda el MÉTODO, que es lo reutilizable.
  **[APLICADO 2026-08-24]**

### ⚠ Lo que se dio vuelta a mitad del censo, y se registra porque el error era plausible

Mirando **sólo** el sandbox de GMod y VJ Base —las dos fuentes obvias— el squad parece algo que
**se fija con un keyvalue al spawnear y no se toca más**, porque es lo único que esas dos hacen.
Con esa foto, la conclusión escrita habría sido *«no se puede reasignar en runtime de forma
útil»*, **y es falsa**: `NPC:SetSquad()`/`GetSquad()` existen y **dos addons sin relación entre
sí** los usan en vivo. Se descartó además que ZBase los definiera él mismo —cero definiciones en
sus 155 archivos—, que era la hipótesis que invalidaba todo.

*Un censo acotado a donde crees que está el sospechoso no lo encuentra donde no está.*

### Y un hallazgo que ahorra trabajo, en vez de crearlo

**VJ y HL2 puro comparten el canal de destino.** `SCHEDULE_GOTO_POSITION` de VJ arma
`TASK_GET_PATH_TO_LASTPOSITION`, y el patrón del engine es `SetLastPosition` + `SCHED_FORCED_GO`:
**las dos bases leen `LastPosition`**. *Move To* **no son dos implementaciones** — es un
`SetLastPosition` común y dos formas de arrancar el schedule.

**Verificación:** sin superficie de runtime. Cada afirmación se adjudicó **abriendo el archivo**
(§7.3.b) y con **denominador declarado**; el instrumento fue `rg --no-ignore` acotado por carpeta,
no el Grep del harness. Tres huecos quedan **declarados y no tapados**: qué coordina el C++ con un
squad, si `SetSquad` en runtime reordena esa coordinación o sólo cambia una etiqueta, y qué
capabilities trae cada `npc_*` de HL2 —de lo que depende una orden de puerta—. Los tres se miden
**en juego**. No commiteado ni pusheado (GIT-7).
