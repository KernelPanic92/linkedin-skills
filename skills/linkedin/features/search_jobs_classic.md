# Search jobs (classic search)

LinkedIn's classic job search, with boolean queries and readable URL filters. **The default is the AI search** (`search_jobs_ai.md`). Use this one only when the user needs:

- newest first (`sortBy=DD`);
- a filter the AI search can't put in the URL: experience level, company, industry, job function, title, benefits, "under 10 applicants"…;
- exact phrase matching with quotes;
- or when the AI search is unavailable.

**LinkedIn is retiring the classic search** (the page says "A partire da settembre, dismetteremo gradualmente la ricerca di lavoro classica"). Some of its features, boolean operators included, may stop working without notice. Always filter the results yourself (step 4).

Searching, filtering, paging and reading details are **Always do**. Saving and dismissing are **Ask first** (`save_dismiss_job.md`). Applying is **Never do**.

## Step 1: Build the query

Split the user's request into three blocks:

1. **Job titles:** synonyms joined with `OR`, in quotes: `("Frontend Developer" OR "Front-end Engineer" OR "Web Developer")`.
2. **Hard skills:** joined to the titles with `AND`: `AND ("React" OR "Vue")`.
3. **Exclusions:** wrong seniority or sectors, with `NOT`: `NOT ("Senior" OR "Lead" OR "Manager")`.

Put constraints that have their own filter (remote, full-time, posted within, experience level) in the URL (step 2), not in the keywords.

Rules:

- **Operators in UPPERCASE:** `AND`, `OR`, `NOT`. Lowercase operators are treated as keywords.
- **Quotes for multi-word phrases:** `"Project Manager"` matches the exact phrase and returns far fewer, more relevant results than the same words without quotes. Use them for job titles. Single words don't need them.
- **Parentheses to group,** especially when mixing `OR` and `AND`: `("Sales" OR "Marketing") AND "B2B"`.
- **No wildcards** (`*`): they aren't reliably supported in job search.
- **Keep it short:** at most 5–6 main operators; longer queries tend to be ignored or truncated.
- **Be logged in:** the boolean syntax only works when logged in, and fails silently otherwise.

*Example 1*
User: "I'm looking for a data analyst job, I know Python and SQL, no senior or manager roles."
Query: `("Data Analyst" OR "Data Analytics") AND (Python AND SQL) NOT ("Senior" OR "Manager" OR "Lead")`
Filters: `f_E=2%2C3` (entry level, associate) does the seniority job better than `NOT`.

*Example 2*
User: "I want to be a copywriter or content creator for agencies, I use Figma, no internships."
Query: `("Copywriter" OR "Content Creator" OR "Content Specialist") AND Figma NOT ("Stage" OR "Intern" OR "Internship")`
Filters: `f_JT=F%2CC%2CP` (full-time, contract, part-time; leaves out `I`, internship).

## Step 2: Build the URL and open it

Encode the query (see the "URL parameters" section in `SKILL.md`), then:

```
https://www.linkedin.com/jobs/search/?keywords=<encoded query>&origin=CLASSIC_SEARCH_MODE_FROM_SEMANTIC&<filters>
```

```
agent-browser open 'https://www.linkedin.com/jobs/search/?keywords=%28%22Data%20Analyst%22%20OR%20%22Data%20Analytics%22%29%20AND%20Python&origin=CLASSIC_SEARCH_MODE_FROM_SEMANTIC&geoId=103350119&f_WT=2&f_TPR=r604800&sortBy=DD'
agent-browser wait 4000
```

**Check that the filters were applied:** each active filter shows up in the filter bar with "È stato applicato il filtro …" (filter applied):

```
agent-browser snapshot -i -c | grep -E 'combobox|applicato il filtro'
```

### Parameters

| Parameter | Filter (Italian UI) | Values |
| - | - | - |
| `keywords` | search box | The query, URL-encoded. Required. |
| `origin` | — | Always `CLASSIC_SEARCH_MODE_FROM_SEMANTIC`. LinkedIn rewrites it when you use the UI (`JOB_SEARCH_PAGE_LOCATION_AUTOCOMPLETE`, `JOB_SEARCH_PAGE_JOB_FILTER`); set it back when you rebuild a URL. Required. |
| `geoId` | location box | Area ID: countries in `search_jobs_ai.md`; cities below. |
| `sortBy` | Ordina per | `DD` **most recent** (date), `R` most relevant. |
| `f_TPR` | Data di pubblicazione | `r86400` (24h), `r604800` (week), `r2592000` (month). Omit for "any time". |
| `f_E` | Livello di esperienza | `1` internship, `2` entry level, `3` associate, `4` mid-senior, `5` director, `6` executive. |
| `f_JT` | Tipo di lavoro | `F` full-time, `P` part-time, `C` contract, `I` internship, `O` other (also `T` temporary, `V` volunteer when offered). |
| `f_WT` | A distanza | `1` on-site, `2` remote, `3` hybrid. |
| `f_C` | Azienda | Company ID (e.g. `67884`). |
| `f_PP` | Località | City ID, e.g. Florence `101544895`. It differs from the city's `geoId`. |
| `f_I` | Settore | Industry ID (e.g. `4` = software development). |
| `f_F` | Funzione lavorativa | Function code (e.g. `it` = IT). |
| `f_T` | Qualifica | Title ID (e.g. `9` = Software Engineer). |
| `f_AL` | Candidatura semplice | `true`: Easy Apply jobs only. |
| `f_VJ` | Ha delle verifiche | `true`: verified jobs only. |
| `f_EA` | Meno di 10 candidati | `true`: fewer than 10 applicants. |
| `f_JIYN` | Nella tua rete | `true`: companies where the user has connections. |
| `f_FCE` | Azienda con pari opportunità | `true`. |
| `f_BE` | Benefit | Benefit ID (e.g. `1` = health insurance). |
| `f_CM` | Impegni | Commitment ID (e.g. `5` = career growth and learning). |
| `distance` | — | Radius around `geoId`. The UI has no distance filter, so the effect of this parameter is uncertain. |
| `start` | paginator | Offset: `25` = page 2, `50` = page 3, … |

Several values in one parameter: comma-separated, encoded as `%2C` (e.g. `f_WT=2%2C3`).

**ID-based filters** (`f_C`, `f_PP`, `f_I`, `f_F`, `f_T`, `f_BE`, `f_CM`): **don't guess IDs.** Get them from the UI:

1. Click `button "Mostra tutti i filtri…"`, then `wait 3000`.
2. In `dialog "Tutti i filtri"`, tick the option (`checkbox "Filtra per <option>"`). The dialog only lists options relevant to the current results; the "Aggiungi …" buttons search for others.
3. Click `button "Applica filtri attuali per visualizzare <N> risultati"`. The button shows the result count before you apply.
4. `agent-browser get url` and read the parameter.

**City or region geoId:**

```
agent-browser snapshot -i -c | grep -F 'combobox "Città, stato o CAP"'
agent-browser click @eCOMBOBOX
agent-browser keyboard type "Firenze"
agent-browser wait 2500
agent-browser snapshot -i -c | grep -F 'option "'
# - option "Firenze, Toscana, Italia" [ref=e54]
# - option "Firenze, Massachusetts, Stati Uniti d'America" [ref=e55]   ← same name, wrong place: read the whole label
agent-browser click @e54 --human
agent-browser wait 3500
agent-browser get url | grep -oE 'geoId=[0-9]+'      # geoId=106461703
```

## Step 3: Read the list

The page is **master–detail**: list on the left, selected job on the right. Clicking a card updates the right pane and `currentJobId` in the URL; it doesn't navigate away.

**`snapshot -i -c` shows only the card titles.** Company, location and badges are in list items that `-c` drops. Read the cards with a full snapshot scoped to the result list (the selector doesn't depend on the language):

```
agent-browser snapshot -s "ul:has(> li[data-occludable-job-id])"
```

One card:

```
- listitem
  - generic [ref=e41] clickable                          ← THE CARD: click this to select the job
    - image "Logo di <Company>"
    - link "<Title> with verification" [ref=e103]        ← title; "with verification" = verified job
    - StaticText "<Company>"
    - list > listitem "Roma, Lazio, Italia (Ibrido)"     ← location (Da remoto / Ibrido / In sede)
    - image/StaticText "Valutazione attiva delle candidature"   ← actively recruiting (optional)
    - StaticText "1 collegamento lavora qui"             ← connections working there (optional)
    - list                                               ← badges, all optional:
      - time "31 minuti fa" / "Entro le ultime 24 ore"   ←   posting age
      - "Visualizzato" | "Candidature inoltrate" | "Salvato"   ←   viewed | applied | saved
      - "Promosso"                                       ←   promoted
      - "Candidatura semplice"                           ←   Easy Apply
    - button "Altre opzioni"                             ← card menu: ignore
    - button "Ignora offerta di lavoro <Title>"          ← DISMISS: only with confirmation (save_dismiss_job.md)
```

**The list is lazy.** A page holds 25 jobs, but only about 8 are rendered at first. Load the rest before reading:

```
total=$(agent-browser get count "li[data-occludable-job-id]")   # jobs on this page (25, or fewer on the last page)
shown=$(agent-browser get count "[data-job-id]")                # jobs rendered so far
# while shown < total:
last=$(agent-browser snapshot -i -c | grep -E '^\s*- button "Ignora offerta di lavoro' | tail -1 | grep -oE 'ref=e[0-9]+' | cut -d= -f2)   # the last rendered card's dismiss button: only used as a scroll target, never clicked
agent-browser scrollintoview @$last
agent-browser wait 2500
shown=$(agent-browser get count "[data-job-id]")
```

It takes 4–5 rounds per page. Stop when `shown` equals `total`.

The result count is in `StaticText "<N> risultati"` above the list.

## Step 4: Filter the results yourself

The search is loose, and some operators may not be applied. Before reporting, go through every card and drop the ones that break the user's constraints. The rules are the same as in `search_jobs_ai.md`, step 4: check seniority, stack, work mode, place and "already applied". Report how many you kept and dropped, and why.

## Step 5: Read a job's detail

```
agent-browser click @eCARD --human        # the card's generic, NOT "Ignora offerta di lavoro …"
agent-browser wait 2500
agent-browser get text article            # the full description
```

- **Description:** use `get text article`. It returns the full "Informazioni sull’offerta di lavoro" text, bullets included. Don't use `snapshot -c`, which loses the bullets and bold headings.
- **Header data** (location, age, work mode, skills match): `agent-browser snapshot -c | awk '/heading ".*" \[level=1/{f=1} f' | head -30`:
  ```
  - heading "<Title>" [level=1]
  - StaticText "<Location> · <Pubblicata|Ripubblicata>: <age> · <N> persone hanno cliccato su “Candidati”"
  - paragraph "Promossa da recruiter · Risposte gestite esternamente a LinkedIn"   ← optional
  - button " Da remoto" / " A tempo pieno"            ← work mode / employment type
  - button " 5 di 10 competenze corrispondenti"       ← skills match
  - link "Candidati sul sito Web dell’azienda per questa posizione: <Title>"   ← external apply: NEVER click
    or button "Candidatura semplice per <Title> presso <Company>"           ← Easy Apply: NEVER click
  - button "Salva <Title> presso <Company>" / "Hai salvato <Title> presso <Company>"   ← save / already saved (Ask first)
  - heading "Persone che puoi contattare"
    - link "<Name> ·1° <headline>" + button "Invia un messaggio a …"   ← messaging: Ask first
  - heading "Informazioni sull’azienda" + link "<Company>" + button "Segui"   ← follow: Ask first
  ```
- **Check that the right job is open:** the `heading [level=1]` must have the title of the card you clicked.
- A single job can also be opened at `https://www.linkedin.com/jobs/view/<jobId>/`. `snapshot -i -c -u` shows the job links.

## Step 6: More pages

- Add `&start=25`, `&start=50`, … to the URL (25 jobs per page). This is the simplest way.
- Or, at the bottom of the list (rendered only after scrolling to it): `button "Pagina N"` and `button "Visualizza pagina successiva"` (next page).
- Read 2–3 pages at most; if more are needed, narrow the search.
