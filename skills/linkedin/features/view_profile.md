# View a profile

Reading is **Always do**. The header buttons send requests or messages, and one of them withdraws an invitation: see "What not to click".

## Open and read

```
agent-browser open 'https://www.linkedin.com/in/<username>/'
agent-browser wait 4000
agent-browser get text main
```

**Use `get text main`, not `snapshot -c`.** `-c` drops list items, and experience, education and skills are lists. `get text main` returns clean text (a few thousand characters) in this order:

- name, connection degree ("· 1°"), headline, location;
- connections and mutual connections ("Andrea, Simone e altri 44 collegamenti in comune");
- "Punti chiave" (highlights), "Informazioni" (About);
- activity, "Esperienza" (experience), "Formazione" (education), skills;
- at the end, suggestions of other profiles ("Altri profili per te"), which are **not** part of this person's profile.

Two limits:

- The About text is truncated ("… altro").
- Long sections show only their first items. Their "Mostra tutto…" link opens the full list on a separate page. Get its URL with `agent-browser snapshot -i -c -u | grep -F 'Mostra tutt'` and `open` it.

If you only have the person's name, find the profile first (see `search_people.md`).

## Header actions

In `snapshot -i -c`, inside `region "CTA principale"`:

| What you see | Meaning |
| - | - |
| `link "Invita <Name> a collegarsi"` + `link "Invia messaggio"` | Not connected yet |
| only `link "Invia messaggio"` | Already connected (1st degree) |
| `link "In sospeso, clicca per ritirare l’invito inviato a <Name>"` | Request already sent, pending |
| `button "Altro"` | Menu with more actions; they change from profile to profile |

## What not to click

- `link "Invita <Name> a collegarsi"`: sends a request. See `send_connection_request.md`.
- `link "Invia messaggio"`: opens a chat. See `send_message.md`.
- `link "In sospeso, clicca per ritirare l’invito…"`: **withdraws the invitation**. Irreversible, so never click it.
- **The right column** ("Altri profili per te", "Persone che potresti conoscere", "Esplora i profili Premium") has "Collegati", "Segui" and "Invia un messaggio a <Other person>" buttons for **other people**. Before clicking anything, check that its label contains this profile's name and that it's inside `region "CTA principale"`.
