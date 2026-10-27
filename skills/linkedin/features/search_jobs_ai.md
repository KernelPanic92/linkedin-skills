# Search jobs (AI search, default)

LinkedIn's natural-language job search. **Use it by default**: it takes the user's request as written, all 25 cards of a page are readable in one compact snapshot, and it's the search LinkedIn is keeping (the classic one is being retired).

Use `search_jobs_classic.md` instead only when the user needs:

- newest first (this search can't sort; `f_TPR=r86400`, last 24 hours, is the closest substitute);
- a filter this search can't put in the URL: experience level, company, industry, job function, title, benefits…;
- exact phrase matching with quotes;
- or when this search is unavailable.

Searching, filtering, paging and reading details are **Always do**. Saving and dismissing are **Ask first** (`save_dismiss_job.md`). Applying is **Never do**.

## Step 1: Turn the request into a query and filters

Split what the user asked into two parts.

**A. Hard constraints → filters.** These go in the URL, because words in the query only steer the results:

| User says | Filter | URL |
| - | - | - |
| last 24 hours / last week / last month | posted within | `f_TPR=r86400` / `f_TPR=r604800` / `f_TPR=r2592000` |
| only Easy Apply | Easy Apply | `f_AL=true` |
| only verified jobs | verified | `f_VJ=true` |
| remote | remote | `f_SAL` item `f_SA_id_225001:272001` |
| full-time | employment type | `f_SAL` item `f_SA_id_226001:272015` |
| part-time | employment type | `f_SAL` item `f_SA_id_226001:273001` |
| contract | employment type | `f_SAL` item `f_SA_id_226001:274001` |
| internship | employment type | `f_SAL` item `f_SA_id_226001:275001` |
| volunteer | employment type | `f_SAL` item `f_SA_id_226001:272002` |
| a country | location | `geoId=<id>` (table below) |

`f_SAL` holds one or more items joined by `$`. Write it already encoded: `:` → `%3A`, `$` → `%24`.

- remote only: `f_SAL=f_SA_id_225001%3A272001`
- remote + full-time: `f_SAL=f_SA_id_225001%3A272001%24f_SA_id_226001%3A272015`

The classic parameters `f_WT`, `f_JT` and `sortBy` **don't work here**: LinkedIn silently drops them. Don't use them.

**B. Everything else → the query.** One or two sentences in the user's language:

- role and synonyms,
- key skills,
- place and work mode,
- exclusions in words ("non senior", "senza Java").

Example: user says "cerco lavoro da full stack Angular/NestJS, non senior, da remoto in Italia, full-time, pubblicati questa settimana":

- query: `Full stack developer Angular e NestJS, non senior, in Italia da remoto`
- filters: `geoId=103350119`, `f_TPR=r604800`, `f_SAL=f_SA_id_225001%3A272001%24f_SA_id_226001%3A272015`

## Step 2: Build the URL and open it

Encode the query (see the "URL parameters" section in `SKILL.md`), then assemble:

```
https://www.linkedin.com/jobs/search-results/?keywords=<encoded query>&origin=SEMANTIC_SEARCH_LANDING_PAGE&geoId=<id>&<filters>
```

```
agent-browser open 'https://www.linkedin.com/jobs/search-results/?keywords=Full%20stack%20developer%20Angular%20e%20NestJS%2C%20non%20senior%2C%20in%20Italia%20da%20remoto&origin=SEMANTIC_SEARCH_LANDING_PAGE&geoId=103350119&f_TPR=r604800&f_SAL=f_SA_id_225001%3A272001%24f_SA_id_226001%3A272015'
agent-browser wait 4500
```

**Check that the filters were applied:**

```
agent-browser snapshot -i -c | grep -E 'textbox "Descrivi|checked=true'
# - textbox "Descrivi il lavoro che vorresti" [ref=…]: Full stack developer Angular e NestJS, …
#   - checkbox "Filtra per Lavoro" [checked=true]       ← always on
#   - checkbox "Ultima settimana" [checked=true]
#   - checkbox "Filtra per Da remoto" [checked=true]
#   - checkbox "A tempo pieno" [checked=true]
```

Every filter you put in the URL must appear with `checked=true`. If one is missing, check the URL encoding.

## Step 3: Read the list

```
agent-browser snapshot -i -c > /tmp/jobs.txt
grep -E '^\s*- button ".*Ignora offerta di lavoro' /tmp/jobs.txt | grep -v '^\s*- button "Ignora'
```

Each line is one card: a `button` whose name holds all the card text. That includes title, company, location with work mode ("Da remoto" / "Ibrido" / "In sede"), salary if shown, badges, and posting age ("Pubblicata 5 giorni fa").

**Getting the title right:** the order inside the card's name isn't fixed. Some cards start with the title, others with the company. The reliable title is in the card's child button `button "Ignora offerta di lavoro <Title>"`: read the title from there. **Reading it is fine; clicking it dismisses the job.**

Badges you may find in the name:

| Text | Meaning |
| - | - |
| "(offerta di lavoro verificata)" | verified job |
| "Valutazione attiva delle candidature" | actively recruiting |
| "Visualizzata" / "Candidatura inoltrata" / "Salvata" | already viewed / applied / saved by the user |
| "Candidatura semplice" | Easy Apply |
| "Promosso" | promoted (paid placement) |
| "Invia una delle prime candidature" | be one of the first applicants |
| "N collegamenti lavorano qui" | N connections work there |
| "Selezionato," at the start | the card currently open in the detail pane |

Other things on the page:

- **Result count:** `StaticText "<N> risultati"` (full snapshot only).
- **Loose matches:** after the paragraph "Abbiamo trovato altri risultati relativi alla tua ricerca…", the results are only loosely related. Treat them with extra suspicion.

## Step 4: Filter the results yourself

No query reliably excludes anything: the search is loose, and words like "da remoto" or "non senior" only steer it. **Before reporting, go through every card and drop the ones that break the user's constraints.** Check each card against each constraint:

| Constraint | Drop the card if… |
| - | - |
| seniority ("non senior") | the title contains Senior, Sr., Lead, Principal, Head, Manager, Director, CTO… |
| stack ("Angular", "senza Java") | the title names a different main stack, or the excluded one (for example "Java Developer" when Java is excluded) |
| work mode ("da remoto") | the location says "(In sede)" or "(Ibrido)" |
| place | the location is outside the requested area |
| already applied | the card says "Candidatura inoltrata" (mention it, don't list it as new) |

When the card doesn't say enough (for example, the stack isn't in the title), **keep it** and check it in the detail (step 5).

Then report:

- how many cards you read, how many you kept, how many you dropped and why (e.g. "12 dropped: 8 senior, 3 on-site, 1 Java");
- for each kept job: title, company, location/work mode, salary if shown, age, notable badges.

Don't dismiss the dropped jobs on LinkedIn unless the user asks (`save_dismiss_job.md`).

## Step 5: Read a job's detail

The page is **master–detail**: the list on the left, the selected job on the right. Clicking a card updates the right pane and the `currentJobId` in the URL; it doesn't navigate away, so there's no "back" step.

```
agent-browser click @eCARD --human        # the card's own button, NOT "Ignora offerta di lavoro …"
agent-browser wait 2500
agent-browser snapshot | awk '/heading "Informazioni sull’offerta di lavoro"/{f=1} f' | head -120
```

- Take a **full** `snapshot`. **Don't use `-c`**, since it drops bullet points, which hold the requirements.
- There's no `article` element on this page, so `get text article` fails here.
- **Check that the right job is open:** the card you clicked now starts with "Selezionato,", and the detail shows `link "<Title>"` with the same title.

The detail pane:

```
- link "Azienda, <Company>"
- paragraph > link "<Title>"
- paragraph "<Location> · <age> · <N> persone hanno cliccato sul pulsante “Candidati”"
- paragraph "Risposte gestite esternamente a LinkedIn"           ← optional: applications handled off LinkedIn
- link "Da remoto" / link "A tempo pieno"                       ← work mode / employment type
- link "Candidati sul sito web dell’azienda" (or an Easy Apply button)   ← APPLY: NEVER click
- button "Salva l’offerta di lavoro"                            ← save: Ask first
- paragraph "Il tuo profilo e il tuo CV soddisfano alcune qualifiche richieste" + link "Mostra dettagli corrispondenza"
- heading "Persone che puoi contattare"
  - link "<Name> … Autore dell’offerta di lavoro"               ← the job poster, when shown
  - link "Messaggio"                                            ← Ask first
- heading "Informazioni sull’offerta di lavoro"                 ← DESCRIPTION: paragraphs, strong headings, list items with ListMarker "• "
- heading "Informazioni sull’azienda"                           ← about the company
```

From the description, report what the user cares about: stack, seniority/years, work mode, location, salary (often written as "RAL"), contract.

## Step 6: More pages

- **No scrolling:** each page renders its 25 cards at once.
- **Next page by URL:** add `&start=25` (page 2), `&start=50` (page 3)…: `start = (page − 1) × 25`. This is the simplest way.
- **Or the paginator:** `button "Indietro"` (previous), up to 3 `button "Pagina N"`, `button "Avanti"` (next).
  - **Two "Avanti" buttons may exist.** The other one belongs to the company photo carousel in the detail pane. The paginator's "Avanti" comes right after the "Pagina N" buttons in the snapshot.
  - **On the last page the paginator has no "Avanti".**
- **Hard cap: 21 pages** (`start=500`), even when the count says "99+". From `start=525` the page shows `heading "Nessun risultato trovato"` (no results); that's the cap, not an error, and it doesn't mean you've seen every match.
- **Read 2–3 pages at most.** If the user needs more, the search is too broad: narrow it (filters, more specific query) instead of walking pages.

## Using the filters from the UI

To use a filter that isn't in the table above (experience level, company, topic chips…), set it from the bar under the search box, then read `agent-browser get url` to see its parameter. Don't guess IDs.

- Toggles (apply on click): "Filtra per Da remoto", "Filtra per Candidatura semplice", "Filtra per Meno di 10 candidati", "Filtra per Nella mia rete", "Filtra per Verificato".
- Dropdowns: `button "Filtra per Data di pubblicazione"`, `"Filtra per Livello di esperienza"`, `"Filtra per Tipo di impiego"`, `"Filtra per Azienda"`. Click the dropdown, tick the options (`generic "<option>"` or `checkbox "<option>"`), then click `link "Mostra risultati"`. `button "Ripristina"` resets the dropdown.
- Topic chips generated from the query ("Azure", "API", "SaaS"…) change with every search.

After you use a filter, LinkedIn changes `origin` to `JOB_SEARCH_PAGE_JOB_FILTER` and adds `referralSearchId`. When you rebuild a URL by hand, drop both and use `origin=SEMANTIC_SEARCH_LANDING_PAGE`.

## Starting from the search box

You can also start from the classic search page with `button "Prova la ricerca di lavoro con l’IA"`. It opens `/jobs/search-results/?origin=SEMANTIC_SEARCH_MODE_FROM_CLASSIC`, an empty page with example prompts. There, click `textbox "Descrivi il lavoro che vorresti"`, `keyboard type` the query and `press Enter`. Building the URL (step 2) is faster and lets you set the filters at once.

## Country geoIds

| Country | geoId |
| - | - |
| Australia | 101452733 |
| Brazil | 106057199 |
| Canada | 101174742 |
| China | 102890883 |
| France | 105015875 |
| Germany | 101282230 |
| India | 102713980 |
| Ireland | 104738515 |
| Italy | 103350119 |
| Japan | 101355337 |
| Mexico | 103323778 |
| Netherlands | 102890719 |
| New Zealand | 105490917 |
| Poland | 105072130 |
| Singapore | 102454443 |
| South Africa | 104035573 |
| South Korea | 105149562 |
| Spain | 105646813 |
| Sweden | 105117694 |
| Switzerland | 106693272 |
| United Arab Emirates | 104305776 |
| United Kingdom | 101165590 |
| United States | 103644278 |

For a city, get its `geoId` with the steps in `search_jobs_classic.md` ("City or region geoId"); for example, Florence is `106461703`. Alternatively, name the city in the query.

## Reference: annotated snapshot

A real `agent-browser snapshot` of this search, taken in an earlier session, after clicking a result, from an **Italian** account at 1600x1200. Personal data and company names are anonymized. Repeated cards, footer links and navbar entries are cut (`…`). The ref numbers are only examples; always use the ones from your own latest snapshot.

Comments marked `←` are annotations; they don't appear in the real output.

### Top bar and filters

```
- banner
  - search
    - textbox "Descrivi il lavoro che vorresti" [ref=e478]: ("Full Stack Developer" OR "Software Engineer") AND ("Angular" OR "NestJS") NOT ("Python") a tempo pieno da remoto pubblicate nell'ultimo mese
        ← the active query. LinkedIn appends the active filters in natural language
    - button "Annulla selezione" [ref=e479]          ← clears the query
  - navigation
    - link "Lavoro, 0 nuove notifiche" [ref=e125]     ← Jobs
    - link "Messaggistica, 0 nuove notifiche" [ref=e126]
    - link "Notifiche, 0 nuove notifiche" [ref=e127]
    - button "Tu" [expanded=false, ref=e128]          ← "Me" menu
- toolbar
  - navigation
    - radiogroup
      - checkbox "Filtra per Lavoro" [checked=true, ref=e522]
      - button "Filtra per Ultimo mese" [expanded=false, ref=e399]        ← dropdown filter (= f_TPR)
        - checkbox "Ultimo mese" [checked=true, ref=e495]
      - checkbox "Filtra per Da remoto" [checked=true, ref=e497]           ← toggle filter (= f_WT)
      - button "Filtra per A tempo pieno" [expanded=false, ref=e401]      ← dropdown filter (= f_JT)
      - button "Filtra per Azienda" [expanded=false, ref=e402]
      - checkbox "Filtra per Candidatura semplice" [checked=false, ref=e503]   ← Easy Apply only
      - button "Filtra per Livello di esperienza" [expanded=false, ref=e404]   ← seniority
      - checkbox "Filtra per Frontend" [checked=false, ref=e507]           ← suggested topic chips (Fintech, SaaS, API, AWS…): they vary per query
      - checkbox "Filtra per Nella mia rete" [checked=false, ref=e517]     ← "In my network"
      - checkbox "Filtra per Meno di 10 candidati " [checked=false, ref=e519]   ← "Under 10 applicants"
```

Filters are easier to set through URL parameters (see step 1). The `[checked=…]` state is useful to check which filters are really active.

### Left pane: result list

```
- main
  - sectionheader
    - paragraph > StaticText "99+ risultati"                    ← result count
    - button "Località Italia" [expanded=false, ref=e366]       ← location picker (read geoId from the URL after changing it)
  - generic
    ← CARD, full shape: every optional field present
    - button "Full Stack Developer Acme Consulting Italia (Da remoto) Altre opzioni Ignora offerta di lavoro Full Stack Developer 22K € /yr - 27K € yr Valutazione attiva delle candidature Visualizzata · Candidatura semplice · Promosso" [ref=e368]
      - paragraph > StaticText "Full Stack Developer"          ← title
      - paragraph > StaticText "Acme Consulting"                ← company
      - paragraph > StaticText "Italia (Da remoto)"             ← location (workplace type in brackets: Da remoto / Ibrido / In sede)
      - button "Altre opzioni" [ref=e413]                       ← card menu
      - button "Ignora offerta di lavoro Full Stack Developer" [ref=e414]   ← DISMISS: only with confirmation
      - paragraph > StaticText "22K € /yr - 27K € yr"           ← salary (optional)
      - paragraph > StaticText "Valutazione attiva delle candidature"   ← actively recruiting (optional)
      - paragraph > StaticText "Visualizzata"                   ← status: Visualizzata (viewed) | Applicato (applied) | Salvata (saved) | "Pubblicata N fa" (posted N ago)
      - paragraph > StaticText "Candidatura semplice"           ← Easy Apply (optional)
      - paragraph > StaticText "Promosso"                       ← sponsored (optional)

    ← CARD, verified job + connections working there
    - button "Senior Full Stack Developer – IoT (Full Remote) (offerta di lavoro verificata) Example Group Italia (Da remoto) Altre opzioni Ignora offerta di lavoro Senior Full Stack Developer – IoT (Full Remote) 40K € all'anno 2 collegamenti lavorano qui Visualizzata · Promosso" [ref=e379]
      - paragraph > StaticText "Senior Full Stack Developer – IoT (Full Remote) (offerta di lavoro verificata)"   ← "(verified job)" suffix
      - paragraph > StaticText "Example Group"
      - paragraph > StaticText "Italia (Da remoto)"
      - …
      - paragraph > StaticText "40K € all'anno"
      - paragraph > StaticText "2 collegamenti lavorano qui"    ← N connections work here
      - …

    ← CARD, minimal: no salary, no badges
    - button "Middle Full-Stack Developer (Next.js) Sample Labs Venezia (Da remoto) Ignora offerta di lavoro Middle Full-Stack Developer (Next.js) Pubblicata 1 mese fa" [ref=e371]
      - paragraph > StaticText "Middle Full-Stack Developer (Next.js)"
      - paragraph > StaticText "Sample Labs"
      - paragraph > StaticText "Venezia (Da remoto)"
      - button "Ignora offerta di lavoro Middle Full-Stack Developer (Next.js)" [ref=e418]
      - paragraph > StaticText "Pubblicata 1 mese fa"

    ← CARD, currently selected one: its name starts with "Selezionato,"
    - button "Selezionato, Fullstack Engineer su applicativi AI [Ral fino a 45k] (offerta di lavoro verificata) Example AI Italia (Da remoto) …" [ref=e383]

    - … (about 25 cards per page) …

    - paragraph > StaticText "Abbiamo trovato altri risultati relativi alla tua ricerca che potrebbero non essere corrispondenze esatte, ma risultare comunque adeguati."
        ← SEPARATOR: cards after this one are loose matches and may not satisfy the boolean query
    - button "Full Stack Developer Acme Consulting Biella (Ibrido) …" [ref=e386]
    - …

    - heading "Questi risultati sono utili?" [level=2, ref=e393]   ← feedback widget: ignore
    - link "Prova Premium per 0 €" [ref=e396]                       ← upsell: ignore
    - list                                                          ← PAGINATOR
      - listitem > button "Pagina 1" [ref=e481]
      - listitem > button "Pagina 2" [ref=e482]
      - listitem > button "Pagina 3" [ref=e483]
    - button "Avanti" [ref=e398]                                    ← NEXT. Present ⇒ another page exists
    - heading "Ricevi avvisi per questa ricerca" [level=3, ref=e467]
    - switch "Fai tap per attivare/disattivare l'impostazione" [checked=false, ref=e521]   ← job alert toggle: don't touch
```

### Right pane: detail of the selected job

```
  - generic
    ← sticky header (appears 2× in the tree)
    - paragraph
      - link "Fullstack Engineer su applicativi AI [Ral fino a 45k]" [ref=e613]
      - link "Offerta di lavoro verificata" [ref=e614]
    - paragraph > StaticText "Example AI • Italia (Da remoto)"
    - button "Salva l'offerta di lavoro" [ref=e605]                     ← save (only if asked)
    - link "Candidati sul sito web dell'azienda" [ref=e606]             ← APPLY (external): NEVER click
        (internal Easy Apply jobs show a button "Candidatura semplice" here: NEVER click)
    - …
    ← company + main header
    - link "Azienda, Example AI" [ref=e587]
    - paragraph > StaticText "Italia · 1 settimana fa · Più di 100 persone hanno cliccato sul pulsante "Candidati""
        ← location · posting age · applicant clicks
    - paragraph > StaticText "Risposte gestite esternamente a LinkedIn"   ← applications handled off LinkedIn
    - link "Da remoto" [ref=e589]                                          ← workplace type
    - link "A tempo pieno" [ref=e590]                                      ← employment type
    - paragraph > StaticText "Il tuo profilo e il tuo CV corrispondono bene alle qualifiche richieste"   ← match hint (optional)
    - link "Prova Premium per 0 €" [ref=e593]                              ← upsell: ignore
    - heading "Persone che puoi contattare" [level=2, ref=e596]          ← "People you can reach out to"
    - paragraph > StaticText "<First name> e altre persone nella tua rete"

    - heading "Informazioni sull'offerta di lavoro" [level=2, ref=e598]  ← ABOUT THE JOB
    - generic
      - paragraph > StaticText "Example AI sta evolvendo da agenzia di marketing a SaaS Factory AI-native…"
      - paragraph > StaticText "💸 " + strong "RAL:" + StaticText " 38K–45K"            ← salary inside the text
      - paragraph > StaticText "📚 " + strong "Esperienza:" + StaticText " 2+ anni …"
      - paragraph > StaticText "📌 " + strong "Modalità di lavoro:" + StaticText " Ibrido su Milano, con apertura al full remote"
      - paragraph > StaticText "💻 " + strong "Tecnologie:" + StaticText " TypeScript, Python, Next.js, …"
      - paragraph > StaticText "Ti occuperai di: "
      - paragraph > StaticText "👉 …"                                    ← responsibilities
      - paragraph > StaticText "Sei la persona giusta se: "
      - paragraph > StaticText "✅ …"                                    ← requirements
      - paragraph > StaticText "Plus graditi: "
      - paragraph > StaticText "⭐ …"                                    ← nice to have
      - paragraph > StaticText "Processo: 💬 Introduttivo → 🧪 Tecnico → 🤝 Finale → ✍️ Offerta"
        ← free-form text written by the recruiter: the structure varies per job

    - heading "Informazioni sull'azienda" [level=2, ref=e600]            ← ABOUT THE COMPANY
    - link "Example AI 188.317 follower" [ref=e601]
    - button "Segui Example AI" [ref=e602]                               ← follow: don't click
    - paragraph > StaticText "Servizi IT e consulenza IT"                ← industry
    - paragraph > StaticText "51-200 dipendenti"                         ← size
    - paragraph > StaticText "105 su LinkedIn"
    - generic > StaticText "…"                                           ← company blurb
    - link "Mostra altre informazioni sull'azienda" [ref=e603]
```

### Other elements in the tree

- A `complementary` with the messaging overlay (bottom right): `button "Componi messaggio"` (compose message). Ignore it during job search.
- An `Iframe` repeating the global navbar ("Navigazione globale"). Ignore it.
- `complementary "Assistente con IA…"` = LinkedIn AI assistant. Ignore it.
