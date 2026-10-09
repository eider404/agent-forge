# Guía de uso de las skills v2

Qué son las skills de `.agents/skills/` de este repositorio, cómo instalarlas, qué flujos de trabajo cubren y cómo ejecutar cada una.

---

## 1. Qué son

Son **12 skills de Claude Code**: instrucciones en markdown (`.agents/skills/<nombre>/SKILL.md`, más sus `templates/` y `references/`) que el agente carga cuando las invocas. Juntas forman un flujo de desarrollo asistido por IA en **dos bucles**:

- **Bucle externo: qué construir.** Se ejecuta una vez por feature: PRD → arquitectura → reglas del proyecto → tickets.
- **Bucle interno (PIV: Plan → Implement → Validate): cómo construirlo.** Se ejecuta una vez por ticket: preparar el contexto → plan → implementación → commit.

```
BUCLE EXTERNO (una vez por feature)
  plan-create-prd → plan-architecture → rules-create-global → plan-slice-tickets
                                         (greenfield, o cuando
                                          cambian las reglas)
                                                                    │
BUCLE INTERNO (una vez por ticket)                                  ▼
  piv-run-full-loop = prime-codebase → piv-plan-implementation → piv-implement → piv-commit
                                                                    │
ANTES DEL PR (opcional)                                             ▼
  piv-validate → piv-review-changes → (corregir → piv-validate → re-review) → rules-check-drift
```

Cada skill deja un **artefacto** (un archivo) que la siguiente lee. Por eso puedes cortar el flujo en cualquier punto, cerrar la sesión y retomarlo después: el estado vive en disco, no en la conversación.

### Las skills por grupo

| Grupo | Skill | En una frase |
|---|---|---|
| **Plan** | `plan-create-prd` | Te entrevista y escribe un PRD centrado en el problema (qué y por qué, nunca cómo) |
| | `plan-architecture` | Sesión de trabajo con un "CTO" para decidir stack, forma de datos, límites, riesgos y spikes |
| | `plan-slice-tickets` | Corta la feature en tickets del tamaño de un loop PIV, con criterios `AC-n` y grafo de dependencias |
| **Prime** | `prime-codebase` | Orienta al agente en el repo (estructura, docs, convenciones, git) antes de planear |
| **PIV** | `piv-plan-implementation` | Crea la rama del ticket y un plan listo para implementarse en una pasada |
| | `piv-implement` | Ejecuta el plan tarea por tarea validando cada una y escribe el reporte |
| | `piv-validate` | Corre toda la suite del proyecto (tests, tipos, lint) y da un veredicto PASS/FAIL |
| | `piv-review-changes` | Revisión técnica del cambio (y contra el plan, si se lo pasas); solo reporta |
| | `piv-commit` | Un commit atómico con mensaje convencional, sin secretos ni artefactos de build |
| | `piv-run-full-loop` | Construye un ticket de punta a punta con una sola pausa para preguntas |
| **Capa de IA** | `rules-create-global` | Escribe `CLAUDE.md`/`AGENTS.md` y `.claude/references/conventions.md` (el `/init` personalizable) |
| | `rules-check-drift` | Comprueba si las reglas y los comandos de `## validate` siguen siendo ciertos tras un cambio |

---

## 2. Instalación

1. Desde la raíz de este repositorio, copia las 12 carpetas de `.agents/skills/` a `<tu-proyecto>/.claude/skills/`:

   ```bash
   mkdir -p <tu-proyecto>/.claude/skills
   cp -r .agents/skills/* <tu-proyecto>/.claude/skills/
   ```

2. Abre Claude Code **en la raíz de tu proyecto**.

> **Instálalas en `<tu-proyecto>/.claude/skills/`, no solo en `~/.claude/skills/` ni en `.agents/skills/`.** En este repositorio viven en `.agents/skills/`, pero las skills se referencian entre sí con rutas relativas al proyecto destino (`.claude/skills/<nombre>/…`; por ejemplo, `piv-plan-implementation` lee `.claude/skills/piv-validate/references/discover-commands.md`). Si están en otra ubicación, esas referencias se rompen.

Requisitos: git (el bucle interno termina en un commit por rama). Opcionales: `gh` para GitHub Issues, el MCP de Atlassian para Jira/Confluence y `tree` (si no está instalado, `prime-codebase` usa `find`).

---

## 3. Cómo se ejecuta una skill

Hay dos formas:

- **Explícita, con barra:** `/<nombre-de-skill> <argumentos>`. Por ejemplo:
  ```
  /piv-run-full-loop .agents/features/2026-10-08-csv-export T-2
  ```
- **Implícita, en lenguaje natural:** el agente elige la skill por su `description`. Por ejemplo: "planifica el T-2", "revisa mis cambios", "haz commit".

Lo que conviene saber al ejecutarlas:

- **Sin argumentos, la mayoría infiere la entrada:** ofrecen la carpeta de feature más reciente que encaja (la última con `prd.md` y sin `architecture.md`, la última arquitectada, etc.) y la nombran para que puedas corregirla.
- **Los workflows interactivos se detienen en un `GATE`:** publican una tanda de preguntas, cada una con un *default* recomendado, y esperan tu respuesta. Puedes contestar "acepta los defaults". Lo que no se sabe no se inventa: queda marcado `TBD — needs validation` o `Assumed — …`.
- **Al terminar, cada skill sugiere el siguiente paso** con el comando exacto.

---

## 4. Conceptos clave

### La carpeta de la feature

Todo lo que genera una feature vive junto y entra en el PR con el código:

```
.agents/features/<YYYY-MM-DD>-<slug>/
  prd.md                          ← plan-create-prd
  architecture.md                 ← plan-architecture   (los spikes rellenan su Outcome:)
  tickets.md                      ← plan-slice-tickets
  plans/t-2-<nombre>.md           ← piv-plan-implementation
  reports/t-2-<nombre>-report.md  ← piv-implement
  reviews/t-2-<nombre>-review.md  ← piv-review-changes
```

- `prd.md`, `architecture.md` y `tickets.md` son **canónicos**: hay uno por feature y solo se sobrescriben si lo confirmas.
- Los planes, reportes y reviews **nunca se sobrescriben**: una segunda versión lleva el sufijo `-2`, `-3`… y la de número más alto es la vigente.

Lo que pertenece al proyecto y no a una feature va fuera de esa carpeta: `CLAUDE.md`/`AGENTS.md` en la raíz y `.claude/references/conventions.md`.

### `conventions.md`: cómo se valida y cómo se commitea

`.claude/references/conventions.md` lo crea `rules-create-global` y lo puedes editar tú. Tiene dos secciones que otras skills leen por su nombre literal:

- **`## validate`**: un bullet por check, con su comando y su directorio. Lo usan `piv-validate`, `piv-plan-implementation` (para los `VALIDATION COMMANDS` del plan) y `rules-check-drift`. Si falta, las skills descubren los comandos en el CI y en los manifiestos (`package.json`, `pyproject.toml`, `Makefile`…).
- **`## commit`**: solo existe si tu proyecto se aparta de los defaults de conventional commits. Lo usa `piv-commit`.

### Un ticket = una rama = un PR

- `piv-plan-implementation` crea la rama `feature/<id>-<nombre>` (por ejemplo, `feature/t-2-source-adapter`) **antes** de escribir el plan.
- Si estás en la rama de otro ticket, el planner y el full-loop se detienen y te preguntan.
- **Si un ticket depende de otro, mergea primero el PR del anterior:** desde la rama base, el reporte del ticket previo solo es visible cuando ya está mergeado.

### Estado de un ticket

`piv-run-full-loop` lo deduce de la carpeta de la feature:

| Estado | Señal |
|---|---|
| **built** | Hay un reporte `reports/t-<n>-…-report.md` con `Status: COMPLETE` |
| **planned** | Hay un plan, o una rama `feature/t-<n>-…` sin mergear, pero no un reporte `COMPLETE` |
| **ready** | No está ni planificado ni construido, y todas sus dependencias (`Depends on`) están *built* |

### Idioma

Escribes en tu idioma y los documentos salen en tu idioma. La **estructura** que leen otras skills queda en inglés: encabezados y campos de `tickets.md`, del plan y del reporte, los IDs `T-n`/`AC-n`, valores como `COMPLETE`/`PARTIAL` y `Met`/`Partial`/`Not met`, y las marcas `TBD — needs validation`, `Assumed — …` y `Outcome:`. No traduzcas esa estructura a mano: el loop la busca por su nombre exacto.

---

## 5. Flujos de trabajo

### Flujo A: proyecto nuevo (greenfield), de la idea al primer PR

Es el flujo completo y el caso para el que se diseñaron las skills.

```
1. /plan-create-prd <idea> · [docs de research]
      → .agents/features/2026-10-06-agrupar-hilos/prd.md
2. /plan-architecture .agents/features/2026-10-06-agrupar-hilos
      → architecture.md (stack, datos, límites, spikes)
3. /rules-create-global .agents/features/2026-10-06-agrupar-hilos
      → CLAUDE.md + .claude/references/conventions.md + stubs
4. /plan-slice-tickets .agents/features/2026-10-06-agrupar-hilos
      → tickets.md (+ tracker si confirmas)
5. /piv-run-full-loop .agents/features/2026-10-06-agrupar-hilos
      → elige el siguiente ticket listo, lo planifica (pausa para preguntas),
        lo implementa, escribe el reporte y lo commitea en su rama
6. (opcional) /piv-validate · /piv-review-changes <plan> · /rules-check-drift main...HEAD
7. push → PR → merge → vuelve a la rama base → repite el paso 5 con el siguiente ticket
```

Notas:

- En los pasos 1 y 2 hay varios GATE: es una conversación, no un documento que se genera de un tiro.
- El paso 3 solo convierte en reglas las decisiones asentadas. Las marcadas `TBD` y los spikes sin `Outcome:` no se vuelven reglas. Vuelve a ejecutarlo cuando un spike registre su resultado.
- En el paso 5 basta con pasar la carpeta: el loop elige el ticket por `Suggested execution order` y `Depends on`, y te dice cuál eligió y por qué. Para forzar uno concreto: `/piv-run-full-loop <carpeta> T-3`.

### Flujo B: feature nueva en un proyecto existente (brownfield)

```
1. /rules-create-global
      (solo la primera vez que incorporas el repo al flujo)
      → deriva CLAUDE.md y conventions.md del código existente
2. /plan-create-prd <idea>
3. /plan-architecture <carpeta>
      → detecta que es brownfield y explora el código antes de proponer
4. /plan-slice-tickets <carpeta>
5. /piv-run-full-loop <carpeta>   (una vez por ticket, como en el flujo A)
```

Si el repo ya tiene un `CLAUDE.md` y un `conventions.md` con `## validate`, sáltate el paso 1. `rules-create-global` nunca reescribe un `conventions.md` existente.

### Flujo C: alcance pequeño, sin tickets

Si la feature cabe en un solo loop, no hace falta cortarla en tickets:

```
1. /plan-create-prd <idea>
2. /plan-architecture <carpeta>
3. /piv-run-full-loop <carpeta>
      (o /piv-plan-implementation <carpeta> → /piv-implement <plan> → /piv-commit)
```

Con una carpeta que tiene `architecture.md` pero no `tickets.md`, el planner toma el camino de alcance pequeño: propone criterios derivados del MVP y te avisa si el MVP no cabe en un loop. En ese caso, vuelve a `plan-slice-tickets`.

> El loop **no** planifica directamente desde una carpeta que solo tiene `prd.md`: se detiene y te sugiere `plan-architecture`. Si planificara desde el PRD, decidiría el stack y el modelo de datos dentro de un plan de un ticket.

### Flujo D: cambio puntual (bugfix, ajuste, tarea suelta)

Para algo que no merece PRD, usa texto libre o una clave de tracker:

```
/piv-run-full-loop Añadir paginación al endpoint GET /orders (25 por página)
/piv-run-full-loop ACC-42
/piv-run-full-loop https://github.com/org/repo/issues/17
```

El plan y el reporte se guardan en una carpeta nueva, `.agents/features/<hoy>-<slug>/`. Si la descripción coincide claramente con una feature existente, el planner te ofrece usar esa carpeta.

### Flujo E: paso a paso, sin el full-loop

Úsalo cuando quieras revisar o editar el plan antes de construir, o dejar el código sin commitear para revisarlo:

```
1. /prime-codebase                                   (opcional; el planner reutiliza su salida)
2. /piv-plan-implementation <carpeta> T-2            → crea la rama y escribe plans/t-2-<nombre>.md
      ✋ revisa o edita el plan (sección AMENDMENTS si cambias algo)
3. /piv-implement .agents/features/<…>/plans/t-2-<nombre>.md
                                                     → código + reports/t-2-<nombre>-report.md
4. /piv-validate <plan>                              (opcional)
5. /piv-review-changes <plan>                        (opcional)
6. /piv-commit
```

Diferencias con el full-loop:

- `piv-implement` ejecutado por separado hace su propio GATE si el plan trae supuestos abiertos.
- En el full-loop, esas preguntas ya se hicieron en el GATE del planner y no se repiten.

### Flujo F: retomar un ticket interrumpido o `PARTIAL`

- **Se cortó la sesión después del plan:** vuelve a ejecutar `/piv-run-full-loop <carpeta>`. Detecta el ticket *planned*, te dice cuál es y en qué rama está, y te ofrece retomarlo con `piv-implement <plan>` + `piv-commit`, en lugar de escribir un plan `-2`.
- **El reporte quedó `PARTIAL`:** el loop se detiene **sin** hacer commit y explica qué falta y por qué. Termina el trabajo (a mano o pidiéndoselo al agente) y ejecuta `/piv-commit`. También puedes decidir commitear el trabajo tal como está.
- **No hay ningún ticket listo:** el loop lista cada ticket con su estado (*built* / *planned* / esperando a `T-x`). Lo habitual es que falte mergear el PR de una dependencia.

### Flujo G: control de calidad antes del PR

El full-loop no incluye `piv-validate` ni `piv-review-changes`: `piv-implement` ya ejecuta los checks del plan, y un reporte `COMPLETE` exige que pasen. Añade este ciclo cuando el cambio necesite la suite completa o una segunda mirada:

```
/piv-validate                         → suite completa del proyecto: Overall PASS/FAIL
/piv-review-changes <plan>            → reviews/<plan>-review.md + tabla AC-n → Yes/Partial/No
   ¿hallazgos? → corrige (a mano o pidiéndoselo al agente con el archivo de review)
              → /piv-validate → /piv-review-changes <plan>   (repite hasta que esté limpio)
/piv-commit                           (si las correcciones dejaron cambios sin commitear)
/rules-check-drift main...HEAD        → ¿las reglas y ## validate siguen siendo ciertos?
```

`piv-validate` y `piv-review-changes` **solo reportan, nunca corrigen**. `rules-check-drift` solo sugiere cambios. Si quieres aplicarlos, pídelo y se commitean con `piv-commit`.

### Flujo H: spikes (decisiones arriesgadas)

1. `plan-architecture` marca como spike cada decisión irreversible o arriesgada, con `Question · Spike · Decision rule · Outcome:`. El `Outcome:` queda vacío.
2. `plan-slice-tickets` convierte cada spike sin `Outcome:` en un ticket de tipo `spike`. Los tickets que dependen de esa decisión quedan bloqueados (`Blocked by open questions / spikes`).
3. Construye el spike como cualquier otro ticket (`/piv-run-full-loop <carpeta> T-1`). La última tarea de su plan escribe el resultado en el `Outcome:` de `architecture.md`, y `piv-implement` lo verifica.
4. Vuelve a ejecutar `/rules-create-global <carpeta>` si la decisión debe convertirse en regla del proyecto.

### Flujo I: con tracker (Jira, GitHub, Linear)

- `plan-create-prd` publica el PRD en el tracker (Confluence/Jira) si se lo pides. Si el PRD vive allí, `plan-architecture` publica también la arquitectura como página hija, enlazada en ambos sentidos.
- `plan-slice-tickets` escribe siempre `tickets.md` y, **solo tras tu confirmación**, crea los tickets en el tracker.
- Las skills PIV aceptan la clave o la URL del ticket: `/piv-run-full-loop ACC-30`. Una clave de Jira también se pasa a `prime-codebase` para anclar la exploración.
- La carpeta local manda sobre el tracker cuando existen ambas.

---

## 6. Referencia por skill

### `plan-create-prd`

**Qué hace:** te entrevista por fases (fundación → profundización → hipótesis → MVP y *door check*) para escribir un PRD de producto: problema, evidencia, tesis, hipótesis falsable, usuario y JTBD, restricciones, MVP, métricas, non-goals y preguntas abiertas. No toma decisiones de ingeniería.

**Ejecución:**

```
/plan-create-prd                                         # empieza preguntando
/plan-create-prd Agrupar hilos cuando el volumen supera ~100/día
/plan-create-prd Agrupar hilos · docs/research/encuestas.md   # lee primero el research como evidencia
```

**Produce:** `.agents/features/<fecha>-<slug>/prd.md` (y una página en el tracker, si lo pides).

**Siguiente paso:** `plan-architecture`.

### `plan-architecture`

**Qué hace:** sesión iterativa con un asesor CTO/staff-engineer. Propone 2–3 enfoques con sus trade-offs, recomienda uno y decide stack, librerías, forma de los datos, límites de seguridad y contratos, y qué decisiones necesitan un spike. Detecta si el proyecto es greenfield o brownfield. Exige un PRD.

**Ejecución:**

```
/plan-architecture                                       # ofrece el PRD más reciente sin arquitectura
/plan-architecture .agents/features/2026-10-06-agrupar-hilos
/plan-architecture .agents/features/2026-10-06-agrupar-hilos/prd.md · docs/adr/
```

**Produce:** `architecture.md` en la carpeta del PRD (si ya existe, pide confirmación antes de sobrescribirlo).

**Siguiente paso:** `rules-create-global` (greenfield) y luego `plan-slice-tickets`; o `piv-plan-implementation` si el alcance es pequeño.

### `plan-slice-tickets`

**Qué hace:** descompone la feature en tickets que un loop resuelve de una vez (~500–1500 líneas de cambio, un concern testeable por ticket). Cada ticket lleva `Type` (`build`/`spike`), `Scope`, `Acceptance criteria` (`AC-n`), `Architecture decisions`, `Per-ticket context`, `Files touched` y `Depends on`. También genera el grafo de dependencias y el orden de ejecución sugerido.

**Ejecución:**

```
/plan-slice-tickets                                      # ofrece la feature arquitectada más reciente
/plan-slice-tickets .agents/features/2026-10-06-agrupar-hilos
```

**Produce:** `tickets.md`. Al volver a cortar, los tickets que ya tienen plan o reporte conservan su número. Tras tu confirmación, también crea los tickets en el tracker.

**Siguiente paso:** `piv-run-full-loop <carpeta>`.

### `prime-codebase`

**Qué hace:** orienta al agente. Lee la estructura (`tree`/`find`), `CLAUDE.md`/`AGENTS.md`, `conventions.md` y los README; identifica puntos de entrada, configuración y modelos clave; y revisa el estado de git (también funciona fuera de un repo). Devuelve un resumen escaneable.

**Ejecución:**

```
/prime-codebase
/prime-codebase ACC-2,ACC-3            # carga antes esos issues de Jira
/prime-codebase ACC-2 4521098          # issue de Jira + página de Confluence
```

**Produce:** solo el resumen en la conversación. El planner lo reutiliza.

### `piv-plan-implementation`

**Qué hace:**

1. Lee el ticket, las decisiones de arquitectura que le afectan, el alcance del PRD y el plan y reporte de sus dependencias.
2. Analiza el código y hace una tanda corta de preguntas (GATE).
3. Investiga la documentación externa.
4. **Crea la rama** y escribe el plan con la forma de `templates/plan.md`: decisiones heredadas, patrones a seguir, tareas atómicas (`IMPLEMENT/PATTERN/IMPORTS/GOTCHA/VALIDATE/SATISFIES`), `VALIDATION COMMANDS` y `ACCEPTANCE CRITERIA`.

**Ejecución:**

```
/piv-plan-implementation .agents/features/2026-10-08-csv-export T-2
/piv-plan-implementation T-2                             # usa la carpeta más reciente con tickets.md
/piv-plan-implementation ACC-30                          # clave o URL de tracker
/piv-plan-implementation .agents/features/2026-10-08-csv-export   # alcance pequeño, sin tickets
/piv-plan-implementation Añadir login con JWT            # texto libre
```

**Produce:** la rama `feature/<id>-<nombre>` y `plans/<id>-<nombre>.md`.

**Siguiente paso:** `piv-implement <plan>`.

### `piv-implement`

**Qué hace:**

1. Verifica que está en la rama del plan. Si el plan se escribió en la rama base, crea la rama y se lleva los cambios de la carpeta de la feature.
2. Hace un GATE si el plan trae supuestos abiertos.
3. Implementa tarea por tarea y ejecuta el `VALIDATE` de cada una antes de pasar a la siguiente.
4. Corre todos los `VALIDATION COMMANDS` y marca los criterios cumplidos.

Respeta el `Out of Scope` del plan y las decisiones heredadas.

**Ejecución:**

```
/piv-implement .agents/features/2026-10-08-csv-export/plans/t-2-source-adapter.md
```

**Produce:** el código y `reports/<plan>-report.md`, con `Status` (`COMPLETE`/`PARTIAL`), la tabla de `AC-n`, las desviaciones, `Assumptions carried` y `Built (for dependent tickets)`. En un spike, verifica que el `Outcome:` quedó escrito en `architecture.md`.

**Siguiente paso:** `piv-commit`.

### `piv-validate`

**Qué hace:** descubre los comandos (`## validate` → CI → manifiestos), te muestra la tabla de comandos resueltos y los ejecuta todos: tests, type-check y lint/format de cada parte del stack. Sigue aunque uno falle y devuelve una fila por check y un veredicto `Overall: PASS/FAIL`. Un check sin comando aparece como `⏭️ skipped`, con su motivo. No escribe archivos ni corrige nada.

**Ejecución:**

```
/piv-validate                                            # suite completa del proyecto
/piv-validate .agents/features/<…>/plans/t-2-<nombre>.md # + checks del plan y tabla AC-n → resultado
```

**Siguiente paso:** `piv-review-changes`, o `piv-commit`.

### `piv-review-changes`

**Qué hace:** revisa el cambio leyendo cada archivo completo: lógica, seguridad, rendimiento y adherencia a `CLAUDE.md` y `conventions.md`. Revisa los cambios sin commitear o, con el árbol limpio, los commits `<base>...HEAD`. Con un plan, revisa además los `AC-n`, el `Out of Scope` y las `Inherited decisions`. Las desviaciones documentadas en el reporte no cuentan como hallazgo; un `AC-n` declarado `Met` que el código no cumple, sí. Solo reporta.

**Ejecución:**

```
/piv-review-changes
/piv-review-changes .agents/features/<…>/plans/t-2-<nombre>.md
```

**Produce:** `reviews/<plan>-review.md`, con hallazgos `severity/file/line/issue/detail/suggestion` y, si pasas un plan, la tabla `AC-n → Yes/Partial/No`. Sin plan, lo guarda en la carpeta de la feature afectada o en una nueva con la fecha de hoy.

**Siguiente paso:** corregir → `piv-validate` → repetir la review; o `piv-commit`.

### `piv-commit`

**Qué hace:** crea un solo commit atómico con todos los cambios pendientes y un mensaje `<tag>: <descripción>`. Si existe `conventions.md › ## commit`, sus reglas mandan. Deja fuera del commit los secretos (`.env`, claves, credenciales, también dentro del contenido de un archivo) y los archivos de build o de más de ~5 MB, y te avisa de lo que dejó fuera. **No hace push.**

**Ejecución:**

```
/piv-commit
```

**Produce:** el commit, y un resumen con el hash, qué cambió, `Left Out of the Commit` y `AI Layer Changes` si se tocó `.claude/`.

### `piv-run-full-loop`

**Qué hace:** encadena `prime-codebase` → `piv-plan-implementation` → `piv-implement` → `piv-commit` para **un** ticket. Solo se detiene una vez, en el GATE del planner, y después llega hasta el commit, salvo que algo falle.

**Ejecución:**

```
/piv-run-full-loop .agents/features/2026-10-08-csv-export        # siguiente ticket listo
/piv-run-full-loop .agents/features/2026-10-08-csv-export/prd.md # ídem
/piv-run-full-loop .agents/features/2026-10-08-csv-export T-3    # un ticket concreto
/piv-run-full-loop ACC-30                                        # ticket del tracker
/piv-run-full-loop Añadir paginación a GET /orders               # texto libre
```

**Se detiene si:**

- no estás en un repositorio git;
- estás en la rama de otro ticket;
- hay cambios sin commitear en la rama base fuera de `.agents/features/`;
- la carpeta solo tiene `prd.md` (te sugiere `plan-architecture`);
- hay un ticket *planned* (te ofrece retomarlo);
- ningún ticket está listo;
- el reporte queda `PARTIAL` (no hace commit).

**Después:** push → PR → merge → vuelve a la rama base y ejecuta el loop otra vez.

### `rules-create-global`

**Qué hace:** escribe un `CLAUDE.md` (o `AGENTS.md`) raíz y ligero: mapa del código, dónde va el código nuevo, convenciones concretas, comandos y los principios de trabajo que tú eliges (los pregunta en un GATE). También crea `.claude/references/conventions.md` (con un `## validate` verificable y, si hace falta, `## commit`) y stubs de contexto. En greenfield deriva las reglas de `architecture.md`; en brownfield, del código (ya preparado con `prime-codebase`, de un documento de análisis o de una lectura mínima).

**Ejecución:**

```
/rules-create-global                                     # feature arquitectada más reciente, o el código
/rules-create-global .agents/features/2026-10-06-agrupar-hilos/
/rules-create-global docs/prd.md docs/architecture.md    # rutas sueltas
/rules-create-global docs/codebase-analysis.md           # brownfield con documento de análisis
```

**Siguiente paso:** `plan-slice-tickets` (greenfield) o el primer loop PIV (brownfield). Más adelante, `rules-check-drift`.

### `rules-check-drift`

**Qué hace:** compara `CLAUDE.md`/`AGENTS.md` y el `## validate` de `conventions.md` con el diff. Solo reporta cuatro tipos de problema: una regla que ya es falsa, una entrada del mapa desactualizada, un comando de `## validate` que ya no existe y un invariante nuevo que merezca una línea. Si nada cambió de verdad, no sugiere nada. Es asesora: no edita.

**Ejecución:**

```
/rules-check-drift                     # detecta la base como piv-review-changes
/rules-check-drift main...HEAD
```

**Siguiente paso:** si pides aplicar los cambios, se commitean con `piv-commit`.

---

## 7. Chuleta: "quiero…"

| Quiero… | Ejecuta |
|---|---|
| Empezar un producto o feature desde una idea | `/plan-create-prd <idea>` |
| Decidir cómo construir un PRD | `/plan-architecture <carpeta>` |
| Generar `CLAUDE.md` y los comandos de validación | `/rules-create-global [<carpeta>]` |
| Cortar la feature en tickets | `/plan-slice-tickets <carpeta>` |
| Construir el siguiente ticket de punta a punta | `/piv-run-full-loop <carpeta>` |
| Construir un ticket concreto | `/piv-run-full-loop <carpeta> T-n` |
| Arreglar algo puntual sin PRD | `/piv-run-full-loop <descripción>` |
| Solo planificar y revisar el plan antes de construir | `/piv-plan-implementation <carpeta> T-n` |
| Construir un plan ya escrito | `/piv-implement <ruta-del-plan>` |
| Saber si todo está en verde | `/piv-validate [<plan>]` |
| Una revisión de código antes del PR | `/piv-review-changes [<plan>]` |
| Commitear lo pendiente sin colar secretos | `/piv-commit` |
| Comprobar que `CLAUDE.md` sigue siendo cierto | `/rules-check-drift main...HEAD` |
| Que el agente conozca el repo | `/prime-codebase [claves-jira]` |

---

## 8. Herramientas complementarias (originales, opcionales)

No forman parte del flujo, ninguna skill v2 depende de ellas y **no están incluidas en este repositorio** (`.agents/skills/` solo contiene las 12 skills del flujo). Si las quieres, instálalas por separado en `<tu-proyecto>/.claude/skills/`.

- **`agent-browser`**: automatización de navegador (Chrome por CDP) para navegar, rellenar formularios, hacer clic, capturar pantalla o extraer datos. Antes de usarla, carga su guía real con `agent-browser skills get core`.
- **`ast-grep`**: búsqueda estructural de código por AST, por ejemplo "funciones async sin try/catch", con `ast-grep run` o `ast-grep scan`.
