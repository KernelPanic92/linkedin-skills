# Search people

Searching and reading results is **Always do**. Each result has a connect button: **Ask first**.

## Search

Encode the query (see the "URL parameters" section in `SKILL.md`), then:

```
agent-browser open 'https://www.linkedin.com/search/results/people/?keywords=<encoded query>'
agent-browser wait 4500
agent-browser snapshot -i -c
```

Role, company and city can go in the keywords: `full stack developer firenze`.

## Results

10 results per page. Each result is a link whose name holds the whole card:

```
- link "<Name> [Verificato] • <degree> <headline> <location> Invita <Name> a collegarsi Attuale: <current role>"
  - link "<Name> [Verificato]"                     ← opens the profile
  - link "Invita <Name> a collegarsi"             ← SENDS A CONNECTION REQUEST: Ask first
  - link "<Mutual connection>" …, link "altri N collegamenti in comune"
```

- **Degree:** `• 1°` = already connected, `• 2°` = friend of a friend, `• 3°` = further away.
- To list the results: `agent-browser snapshot -i -c | grep -E '^\s*- link "[^"]+ • (1°|2°|3°)'`
- To open a profile, take its URL from `snapshot -i -c -u` (the `link "<Name> …"` line) and `open` it, or click that link. Never click "Invita … a collegarsi" by mistake: it sits right next to it.
- Some results are Premium ads (`link "Attiva Premium per 0 €"`): skip them.

How to report: name, degree, headline, location, current role, mutual connections.

## Filters

At the top:

- checkboxes "1°", "2°", "3° e oltre": connection degree;
- `button "Filtra per Località"`, `button "Filtra per Aziende attuali"`: dropdowns;
- `button "Tutti i filtri"`: all filters.

The degree filter is in the URL, so you can set it directly:

| Degree | URL |
| - | - |
| 1st | `&network=%5B%22F%22%5D` |
| 2nd | `&network=%5B%22S%22%5D` |
| 3rd and beyond | `&network=%5B%22O%22%5D` |

Several degrees: `%5B%22F%22%2C%22S%22%5D` (= `["F","S"]`). LinkedIn adds `origin=FACETED_SEARCH`; that's fine.

For other filters, set them in the UI and read `agent-browser get url` to see the parameter.

## Pages

`button "Pagina N"` and `button "Avanti"` at the bottom, or `&page=2`, `&page=3`, … in the URL. Read only the pages you need (see the pacing rule in `SKILL.md`).
