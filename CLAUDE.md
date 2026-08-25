# CLAUDE.md

Guía para trabajar en **Cortex** — el módulo de IA de NPC del ecosistema Corpus (addon GLua para Garry's Mod). Léela antes de tocar código o docs de este repo.

## Qué es

Cortex es el módulo de **IA de NPC** del ecosistema Corpus: **escuadrones** (quién obedece, qué órdenes hay, cómo se ejecutan), **táctica de combate** y **afecto** — dolor y miedo que cambian el comportamiento. Es un addon Gmod independiente con su propio git, que **hard-depende** de Corpus (la única dependencia dura del ecosistema — cita COR-11) y de nadie más. Detecta a los otros módulos en runtime vía `Corpus.GetModule`/`Corpus.HasModule`, nunca los asume (cita COR-5).

Este repo se **fundó el 2026-08-24**, al abrir el autor el bloque de escuadrones. Antes era semilla: `LICENSE`, `README.md`, `docs/Cortex_ContratosEntrantes.md`, `docs/CHANGELOG.md`, `docs/cortex_convenciones_commits.txt` y los SVG de identidad — sin `CLAUDE.md`, o sea **sin sede para su familia de normas**. El diseño del módulo → [`docs/Cortex_Architecture.md`](docs/Cortex_Architecture.md). El framework y el grafo de dependencias → [`../corpus/docs/CORPUS_Architecture.md`](../corpus/docs/CORPUS_Architecture.md).

**Regla cardinal (CTX-1): Cortex es una CAPA de comportamiento sobre bases de NPC que ya existen, no una base de NPC.** El bloque abierto cubre **VJ Base y los NPC nativos de HL2**; **DRG y ZBase quedan diferidas**. Esto **no deroga** la decisión vieja de construir CortexBase propia tomando lo mejor de VJ/DRG/ZBase: las dos conviven, la capa se escribe primero y CortexBase después, **debajo**. Confundirlas es el error caro — escribir una base de NPC nueva cuando lo pedido era una capa sobre las que hay.

**Regla cardinal (cita COR-10 y COR-1; su sede es el framework):** nada de lógica de dominio sube a Corpus. Formaciones, cola de órdenes, percepción, facción, dolor y miedo viven **acá**, y otros módulos lo consumen vía el registro de Corpus.

## Docs del proyecto — jerarquía de lectura

Antes de tocar código o diseño, lee en este orden (los tres primeros son **docs vivos**):

1. **Estado de HOY** → [`docs/cortex_estado.md`](docs/cortex_estado.md). Foto del AHORA, ≤1 pantalla. **Léelo ANTES** que la arquitectura — dice qué existe hoy y qué está bloqueado por quién.
2. **Rumbo** → [`docs/cortex_roadmap.txt`](docs/cortex_roadmap.txt). Qué sigue y en qué orden.
3. **Historial de parches** → [`docs/CHANGELOG.md`](docs/CHANGELOG.md). `[PENDIENTE]`/`[APLICADO YYYY-MM-DD]`, nunca se borra ni renumera.
4. **Metodología de trabajo** → [`../corpus/docs/corpus_flujo_trabajo.txt`](../corpus/docs/corpus_flujo_trabajo.txt). **Doc canónico compartido** por todo el ecosistema — no se duplica acá. Su **§7 arbitra los hechos**; léelo antes de escribir cualquier norma.
5. **Contratos ENTRANTES** → [`docs/Cortex_ContratosEntrantes.md`](docs/Cortex_ContratosEntrantes.md). **Léelo entero antes de diseñar.** Inventaría las SEIS firmas que Cargo, Caliber y corpus-stalker congelaron contra este repo **antes de que existiera**. Es doc de **recepción, sin autoridad**: cada fila tiene su sede en otro repo y ninguna se re-deriva acá.
6. **Arquitectura del módulo** → [`docs/Cortex_Architecture.md`](docs/Cortex_Architecture.md). Doc general; los subsistemas grandes se desprenden a doc particular (flujo §2) y acá queda resumen + link.
7. **Convenciones de commit** → [`docs/cortex_convenciones_commits.txt`](docs/cortex_convenciones_commits.txt). Alcances específicos de **este** repo — **el doc manda** (cita CTX-5).

## Idioma

**Cita CRG-48 y GIT-4 —** código y comentarios en **inglés**; strings de cara al jugador en **inglés**; docs, mensajes de commit y `Corpus.Log` en **español**. Los `<tipo>` de commit van en inglés (ver convenciones).

## El workspace multi-repo

Este repo (`corpus-cortex/`) es una de las **siete** raíces git del workspace `corpus.code-workspace`. La raíz `corpus/` es el framework del que todos hard-dependen; otras cuatro (`corpus-caliber/`, `corpus-coagulant/`, `corpus-craving/`, `corpus-cargo/`) son módulos hermanos. La séptima, `corpus-stalker/`, no es un módulo sino el **addon de contenido** de S.T.A.L.K.E.R.: consumidor puro que espera registrar contra Cortex sus defs de NPC y sus facciones (contrato entrante #5).

Hay además una carpeta `dev/` en la raíz del workspace que **no es un repo** (fuera de todos los git, nunca se publica). Ahí viven las bases de NPC de terceros que este módulo tiene que leer antes de diseñar: `dev/other/VJ Base/`, `dev/other/drgbase/`, `dev/other/zbase/` y el detector de IA `nead` de la misma carpeta. **Al integrar con mods ajenos**, consulta [`../dev/mods_workshop_mapa.md`](../dev/mods_workshop_mapa.md): distingue **RECICLAR** (copiar código/assets ⇒ importa la licencia) de **COMPAT-RUNTIME** (detectar y consumir por API ⇒ la licencia no importa). Cortex es COMPAT-RUNTIME sobre VJ y HL2 por construcción — es lo que CTX-1 significa en la práctica.

## Mapa de archivos

**No hay código todavía.** El repo lleva docs, identidad visual y licencia. Cuando nazca el primer archivo Lua, el patrón template es el de Caliber (**manifest de carga explícito**: un único entry en `lua/autorun/corpus_cortex_init.lua` que registra el módulo, declara el contrato público y hace `include()` en orden determinista, con los sub-archivos **fuera** de `lua/autorun/` para que no se auto-ejecuten y dupliquen la carga). Ver §3-§5 de `Caliber_Architecture.md`.

| Archivo | Rol |
|---|---|
| [`docs/Cortex_ContratosEntrantes.md`](docs/Cortex_ContratosEntrantes.md) | Las seis firmas que otros repos congelaron contra este. Recepción, sin autoridad |
| [`docs/Cortex_Architecture.md`](docs/Cortex_Architecture.md) | Diseño del módulo: qué es, qué posee, qué está abierto |
| [`docs/cortex_estado.md`](docs/cortex_estado.md) · [`docs/cortex_roadmap.txt`](docs/cortex_roadmap.txt) | Foto del AHORA · rumbo |
| [`docs/cortex_convenciones_commits.txt`](docs/cortex_convenciones_commits.txt) | Tipos y **alcances** de commit de este repo |
| `assets/cortex_{logo,lockup}_{dark,light}.svg` | Identidad visual. Fuente de verdad: `../corpus/docs/Corpus_Identidad.md` |

## Contratos que no debes romper

Esta es la **sede** de la familia `CTX-nn` (§7.4 del flujo): la definición canónica de cada contrato vive acá y el resto del ecosistema la **cita**, nunca la redefine. El registro [`../corpus/docs/ids.yaml`](../corpus/docs/ids.yaml) los indexa.

1. **CTX-1 — Cortex es una CAPA sobre bases de NPC que ya existen, no una base de NPC.** Alcance del bloque abierto: **VJ Base y HL2**; DRG y ZBase **diferidas**. No deroga a CortexBase — la capa va primero y la base propia después, debajo. Sede: §Qué es, más arriba.
2. **CTX-2 — El módulo tiene DOS mitades con gates INDEPENDIENTES, y confundirlas bloquea trabajo que no está bloqueado.** El **afecto** (dolor, miedo) está gated por la superficie de eventos de daño/extremidad que Caliber expondrá con su Block 3 — hoy **no existe** (contrato entrante #4, y **CAL-22** existe justamente para que nadie la lea como servida). La **táctica** y el **escuadrón** **no dependen de esa superficie** y se pueden escribir ya. El roadmap del framework enuncia el gate sin partirlo; **acá se parte**.
3. **CTX-3 — El escuadrón vive acá: su estado, su cola de órdenes y su formación son de Cortex, y el menú interactivo de Corpus es sólo su FRONT-END.** El registro de acciones del framework (`Corpus.Interact`) dibuja el árbol y transporta el commit; **no guarda una sola línea de estado de escuadra**, porque COR-1 y COR-10 se lo prohíben. Adjudicado en `../corpus/docs/Corpus_Interaccion_Arquitectura.md` §5.4, que enuncia la frontera sin acuñarla; **su sede normativa es ésta**.
4. **CTX-4 — Cortex no parchea la propiedad de inventario de Cargo.** Está **medido** que un NPC no puede ser dueño de un inventario hoy: `Inventory.OwnerKey` (`../corpus-cargo/lua/corpus_cargo/server/corpus_cargo_inventory.lua:101-104`) hace `ply:SteamID64() or ("bot" .. ply:EntIndex())`, y **`SteamID64` es un método de `Player`, no de `Entity`** — con un NPC **no cae en el `or`: revienta**. La puerta no está entornada, **no existe**, y abrirla es una decisión de Cargo (*¿el dueño de un inventario deja de ser un jugador?*), no un parche de una línea desde acá. **Toda orden que exija que un NPC PORTE ítems queda diferida hasta ese voto** — entre ellas `Deploy` y la secuencia *throw flashbang and clear*.
5. **CTX-5 — Los alcances de commit de este repo son los de §3 de [`docs/cortex_convenciones_commits.txt`](docs/cortex_convenciones_commits.txt), y ese doc manda.** Dos EN USO (`assets`, `docs`) y diez RESERVADOS que entran en vigor con su código (`base`, `core`, `escuadra`, `tactica`, `afecto`, `facciones`, `caliber`, `config`, `dev`, `init`). Aplica GIT-6: la §3 es por-repo y jamás se hereda del framework. El CLAUDE.md los resume; el doc los define.

Las normas del ecosistema que este repo **cita y no redefine**: **COR-2** (namespace único — todo cuelga de `Corpus`, cero globals sueltos), **COR-3**/**COR-18** (persistencia namespaced vía `Corpus.Data`, nada de `file.*` para estado propio), **COR-4** (net namespaced), **COR-5** (detección, nunca asunción), **COR-6** (prefijo `corpus_cortex_*` en todo archivo Lua), **COR-9** (cada archivo autosuficiente), **COR-11** (Corpus es la única hard-dep), **COR-15** (UI vía `Corpus.UI.RegisterTab`), **COR-16** (log vía `Corpus.Log`) y **COR-17** (los assets de terceros no se versionan).

## Verificación

No hay test runner automatizado (es un addon GMod) — el patrón es el de ADS/Kontrol: cargar el mapa, confirmar en consola/juego, no asumir. Ver `../corpus/docs/corpus_flujo_trabajo.txt` §1 (Paso 4). **Mientras el repo sea sólo docs, "verificado" significa revisado contra su sede y contra el árbol real**, no contra el chat: la jerarquía de autoridad de §7.1 pone al código y al árbol por encima de cualquier doc, y este módulo **diseña por delante del código a propósito** (mock-first, flujo §3).

Dos guardias que corren **antes** de cerrar cualquier cambio de prosa:

- `..\corpus\.claude\check-ids\corpus_check_ids.ps1` — el checker de IDs (§7.7). ⚠ **No distingue «cito un ID» de «acuño un ID»**: hasta el 2026-08-24 este repo no tenía sede, y escribir la etiqueta de una norma `CTX` **en prosa** —aunque fuera como ejemplo— lo ponía en rojo con `HUERFANO_DOC`. Cazó a dos redacciones distintas. **Desde que existe este archivo eso dejó de aplicar**, pero la lección queda: una norma que se nombra tiene que estar en el registro **en el mismo parche**.
- `python ..\dev\chequear_voseo.py <archivo>...` — al autor se le escribe en **español de Chile, de tú**. Falsos positivos conocidos: «esté», «estás», «caché».

Al cerrar un cambio con superficie de runtime: refresca [`docs/cortex_estado.md`](docs/cortex_estado.md) en sitio y actualiza [`docs/CHANGELOG.md`](docs/CHANGELOG.md) (`[PENDIENTE]` → `[APLICADO YYYY-MM-DD]`, sin borrar ni renumerar).

## Git / commits

Sigue [`docs/cortex_convenciones_commits.txt`](docs/cortex_convenciones_commits.txt): `<tipo>(<alcance>): <descripción>` — tipo en inglés, descripción en español, minúscula inicial, sin punto final, imperativo. **Tipos** (§2): `feat`, `fix`, `refactor`, `docs`, `chore`, `test`. **Alcances**: los de §3, resumidos en CTX-5. Ojo: `chore` **no** es un alcance sino un tipo.

**Este repo está publicado en GitHub** (`github.com/Sepuldosky/corpus-cortex`, público, MIT, remote `origin` cableado y al día con `origin/main`). **Push y commit sólo cuando se pidan explícitamente** (**GIT-7**).

**GIT-5 — No agregues el trailer `Co-Authored-By: Claude` (ni ninguna atribución de co-autoría a Claude/Anthropic) en los mensajes de commit.** Esto sobreescribe el comportamiento por defecto del harness.
