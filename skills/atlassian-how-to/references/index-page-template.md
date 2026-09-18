# Index page — template

> An index page is a table of pages with, for each one, enough for a reader — human or Claude — to
> decide whether to open it and how to find it. Two variants of the same shape: the **root** index
> (`claude-client-config`, which also carries the settings) and a **section** index (the parent of
> `Minuta` pages, *Procesos*, *Referencia*, *Delivery*). Sections in the order below; none left out.
> Labels on the page: `index`, plus `config` on the root.

---

## Root index — `claude-client-config`

**Title (exact):** `claude-client-config` · **Labels:** `index`, `config` · **Maintained by:** Seamlex.
Never edited by the plugin.

### Propósito

<One paragraph. What this page indexes — the settings the Seamlex commands read and the pages that hold
the project's knowledge for `{{PROGRAM}}` at `{{COMPANY}}` — and what it deliberately does not cover
(e.g. "the delivery team's technical documents are indexed on the *Delivery* page").>

### Configuración

> One setting per row, key spelled exactly as the commands name it. A setting that does not apply is a
> missing row, never an empty value.

| Clave | Valor | Significado |
|---|---|---|
| `JIRA_PROJECT` | `ABC` | Jira project key of the engagement |
| `CONF_PARENT` | `Relevamientos` (page id `123456`) | Parent page of every `Minuta <task>` |
| `CONF_SCOPE_PAGE` | `Alcance firmado — <program>` (page id `123457`) | The signed scope page |
| `TYPE_RELEVAMIENTO` | `Relevamiento` | Issue type (or, if stated, the label) that marks a relevamiento |
| `TYPE_TASK` | `Subtarea` | The project's sub-task type, used for pending items |
| `LABEL_REQUEST` | `cliente` | Label on everything the plugin creates |
| `LABELS_EXTRA` | `fase-1` | Extra labels on everything the plugin creates (optional) |
| `SEAMLEX_CONTACT` | `<name>` | Who at Seamlex picks up pending items and unidentified features |
| `CONFIRM_WRITES` | `always` | Show and approve every write before it happens |
| `DETAIL` | `business` | Level of language for the customer: `business` / `technical` |
| `DRAFTS_DIR` | `seamlex` | Local workspace folder for discovery notes |
| `PROGRAM` | `<program name>` | The program's name as the customer says it |
| `COMPANY` | `<company name>` | The customer |

### Páginas

> One row per page. Title exactly as the page is titled. *Tipo* from the fixed vocabulary — `scope`,
> `discovery`, `index`, `minuta`, `proceso`, `glosario`, `referencia`, `no-identificado`,
> `analisis-funcional`, `hld`. *Leer cuando* is one of `cada sesión` / `cuando la tarea toca <área>` /
> `a demanda`.

| Título (exacto) | Enlace | Tipo | Etiquetas | Qué tomar de ella | Leer cuando |
|---|---|---|---|---|---|
| `Alcance firmado — <program>` | <link> | `scope` | `scope` | Épicas e ítems incluidos, exclusiones escritas, supuestos, fases | cada sesión |
| `Discovery Brief — <program>` | <link> | `discovery` | `discovery` | §3 áreas y roles, §4 procesos, §6 actores, §7 dolores, §9 alcance y restricciones | cada sesión |
| `Relevamientos` | <link> | `index` | `index` | Índice de las minutas; las dos o tres últimas se leen completas | cada sesión |
| `Procesos` | <link> | `index` | `index` | Índice de procesos de negocio, uno por página | cuando la tarea toca un proceso |
| `Glosario — <company>` | <link> | `glosario` | `glosario` | El vocabulario del cliente; se usa en lugar del de la plataforma | cada sesión |
| `Referencia` | <link> | `index` | `index` | Organigrama, muestras de documentos, políticas | a demanda |
| `Features no identificados — <program>` | <link> | `no-identificado` | `no-identificado` | Lo ya estacionado fuera del alcance firmado, para no repetirlo | cada sesión |
| `Delivery` | <link> | `index` | `index` | Análisis funcionales y diseños del equipo de delivery | a demanda |

### Cómo encontrar lo que no está listado

```
space = "<SPACE>" AND label = "minuta" ORDER BY lastmodified DESC
space = "<SPACE>" AND label = "<jira-key>"
space = "<SPACE>" AND (title ~ "Minuta" OR title ~ "Acta" OR title ~ "Reunión") ORDER BY lastmodified DESC
```

### Mantenimiento

| | |
|---|---|
| **Mantiene** | <name, Seamlex> |
| **Última actualización** | <YYYY-MM-DD> — <what changed> |

---

## Section index — e.g. `Relevamientos`

**Title (exact):** the section's name as the root index lists it · **Labels:** `index` · **Maintained
by:** whoever creates pages under it — the plugin adds a row for every page it creates here.

### Propósito

<One paragraph: what lives under this page, one page per what, who writes them.>

### Páginas

| Título (exacto) | Enlace | Tipo | Etiquetas | Qué tomar de ella | Leer cuando |
|---|---|---|---|---|---|
| `Minuta Devoluciones de mercadería dañada` | <link> | `minuta` | `minuta`, `abc-12` | Actores y proceso de devolución hoy y después; reglas de aceptación; 3 pendientes abiertos; estado `En progreso` | cuando la tarea toca devoluciones |
| `Minuta Alta de clientes mayoristas` | <link> | `minuta` | `minuta`, `abc-15` | Quién aprueba un alta y con qué información; sin pendientes; estado `Finalizado` | cuando la tarea toca clientes |

### Cómo encontrar lo que no está listado

```
space = "<SPACE>" AND ancestor = <this page's id> ORDER BY lastmodified DESC
space = "<SPACE>" AND label = "minuta" AND label = "<jira-key>"
```

### Mantenimiento

| | |
|---|---|
| **Mantiene** | Seamlex / el plugin |
| **Última actualización** | <YYYY-MM-DD> — <row added or changed> |
