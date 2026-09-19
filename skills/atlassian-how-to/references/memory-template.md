# Index page — template

> An index page is a table of pages with, for each one, enough for a reader — human or Claude — to
> decide whether to open it and how to find it. Two variants of the same shape: the **root** index
> (`seamlex-portal-memory`, which also carries the settings) and a **section** index (the parent of
> `Minuta` pages). Sections in the order below; none left out.
> Pages are found by exact title and by their place under a parent — never by label.

---

## Root index — `seamlex-portal-memory`

**Title (exact):** `seamlex-portal-memory` · **Maintained by:** Seamlex. Never edited by the plugin.

### Propósito

<One paragraph. What this page indexes — the settings the Seamlex commands read and the pages that hold
the project's knowledge for `{{PROGRAM}}` at `{{COMPANY}}`.>

### Configuración

> One setting per row, key spelled exactly as the commands name it. A setting that does not apply is a
> missing row, never an empty value.

| Clave | Valor | Significado |
|---|---|---|
| `JIRA_PROJECT` | `ABC` | Jira project key of the engagement |
| `CONF_PARENT` | `Relevamientos` (page id `123456`) | Parent page of every `Minuta <KEY> — <task>` |
| `CONF_SCOPE_PAGE` | `Alcance firmado — <program>` (page id `123457`) | The signed scope page |
| `TYPE_TASK` | `Subtarea` | The project's sub-task type, used for pending items |
| `LABEL_REQUEST` | `cliente` | Jira label on every issue the plugin creates |
| `LABELS_EXTRA` | `fase-1` | Extra Jira labels on every issue the plugin creates (optional) |
| `SEAMLEX_CONTACT` | `<name>` | Who at Seamlex picks up pending items and unidentified features |
| `CONFIRM_WRITES` | `always` | Show and approve every write before it happens |
| `DETAIL` | `business` | Level of language for the customer: `business` / `technical` |
| `DRAFTS_DIR` | `seamlex` | Local workspace folder for discovery notes |
| `PROGRAM` | `<program name>` | The program's name as the customer says it |
| `COMPANY` | `<company name>` | The customer |

### Páginas

> One row per page. Title exactly as the page is titled. *Tipo* from the fixed vocabulary — `scope`,
> `discovery`, `index`, `minuta`, `glosario`, `no-identificado`. *Leer cuando* is one of `cada sesión` /
> `cuando la tarea toca <área>` / `a demanda`.

| Título (exacto) | Enlace | Tipo | Qué tomar de ella | Leer cuando |
|---|---|---|---|---|
| `Alcance firmado — <program>` | <link> | `scope` | Épicas e ítems incluidos, exclusiones escritas, supuestos, fases | cada sesión |
| `Discovery Brief — <program>` | <link> | `discovery` | §3 áreas y roles, §4 procesos, §6 actores, §7 dolores, §9 alcance y restricciones | cada sesión |
| `Relevamientos` | <link> | `index` | Índice de las minutas; las dos o tres últimas se leen completas | cada sesión |
| `Glosario — <company>` | <link> | `glosario` | El vocabulario del cliente; se usa en lugar del de la plataforma | cada sesión |
| `Features no identificados — <program>` | <link> | `no-identificado` | Lo ya estacionado fuera del alcance firmado, para no repetirlo | cada sesión |

### Cómo encontrar lo que no está listado

```
space = "<SPACE>" AND title ~ "<JIRA-KEY>"
space = "<SPACE>" AND ancestor = <Relevamientos id> AND title ~ "Minuta" ORDER BY lastmodified DESC
space = "<SPACE>" AND (title ~ "Minuta" OR title ~ "Acta" OR title ~ "Reunión") ORDER BY lastmodified DESC
```

### Mantenimiento

| | |
|---|---|
| **Mantiene** | <name, Seamlex> |
| **Última actualización** | <YYYY-MM-DD> — <what changed> |

---

## Section index — e.g. `Relevamientos`

**Title (exact):** the section's name as the root index lists it · **Maintained by:** whoever creates
pages under it — the plugin adds a row for every page it creates here.

### Propósito

<One paragraph: what lives under this page, one page per what, who writes them.>

### Páginas

| Título (exacto) | Enlace | Tipo | Qué tomar de ella | Leer cuando |
|---|---|---|---|---|
| `Minuta ABC-12 — Devoluciones de mercadería dañada` | <link> | `minuta` | Actores y proceso de devolución hoy y después; reglas de aceptación; 3 pendientes abiertos; estado `En progreso` | cuando la tarea toca devoluciones |
| `Minuta ABC-15 — Alta de clientes mayoristas` | <link> | `minuta` | Quién aprueba un alta y con qué información; sin pendientes; estado `Finalizado` | cuando la tarea toca clientes |

### Cómo encontrar lo que no está listado

```
space = "<SPACE>" AND title ~ "<JIRA-KEY>" AND title ~ "Minuta"
space = "<SPACE>" AND ancestor = <this page's id> ORDER BY lastmodified DESC
```

### Mantenimiento

| | |
|---|---|
| **Mantiene** | Seamlex / el plugin |
| **Última actualización** | <YYYY-MM-DD> — <row added or changed> |
