# Manage invitations

Reading invitations is **Always do**. Accepting or ignoring one is **Ask first**. Withdrawing a sent one is **Never do**.

## Check for pending invitations

```
agent-browser open "https://www.linkedin.com/mynetwork/grow/"
agent-browser wait 4000
agent-browser snapshot -i -c | awk '/region "CTA principale"/{f=1} f' | head -12
```

Inside `region "CTA principale"` there's a heading with the status, e.g. `heading "Nessun invito in sospeso"` (no pending invitations), and `link "Gestisci tutti gli inviti"` (manage all invitations).

The rest of `/mynetwork/grow/` is **suggestions, not invitations** ("Persone che potresti conoscere…", "Popolare su LinkedIn", "Suggerimenti per te"). Their buttons are:

- `button "Invita <Name> a collegarsi"`: sends a request;
- `button "Segui <Name>"`: follows;
- `button "Rimuovi <Name> dai suggerimenti"`: removes from suggestions.

All three are Ask first. Don't touch them while managing invitations.

## Received invitations

```
agent-browser open "https://www.linkedin.com/mynetwork/invitation-manager/received/"
agent-browser wait 4000
agent-browser snapshot -c | awk '/- main/{f=1} f' | head -80
```

Shape of the page:

```
- main
  - region "CTA principale"
    - paragraph "Gestisci gli inviti"                    ← Manage invitations
    - button "Impostazioni degli inviti"                 ← invitation settings: don't touch
    - navigation
      - link "Ricevuti"                                  ← Received  → /invitation-manager/received/
      - link "Inviati"                                   ← Sent      → /invitation-manager/sent/
    - navigation
      - link "Tutto (N)"                                 ← filter with counter; N = pending invitations
    - paragraph "Nessun nuovo invito"                    ← shown when N = 0
    - heading "Suggerimenti per te"                      ← BELOW THE INVITATIONS: suggestions, not invitations
      - link "<Name> <headline> … Invita <Name> a collegarsi"
        - button "Invita <Name> a collegarsi" ("Collegati")   ← sends a NEW request: not part of this flow
```

Everything between the counters and `heading "Suggerimenti per te"` is an invitation. Report each one: who it is (name, headline), the type (person, event, page, newsletter, group) and the attached note, if any.

## Accept or ignore

1. Show the user the list, and for each invitation ask whether to accept, ignore or leave it.
2. For each invitation they decided on, take a fresh `snapshot -i -c` and find **that person's** buttons: their labels contain the person's name. They're expected to be "Accetta" (accept) and "Ignora" (ignore). If they look different, stop and ask (stuck rule).
3. Never click a button whose label names someone else, and never use an "accept all" control.
4. `click @eN --human`, `wait 2500`, snapshot again. Check that the invitation has disappeared or shows the new state, and that the "Tutto (N)" counter went down by one.
5. Pause 2–5 seconds before the next invitation.

## Sent invitations

```
agent-browser open "https://www.linkedin.com/mynetwork/invitation-manager/sent/"
agent-browser wait 4000
agent-browser snapshot -c | awk '/- main/{f=1} f' | head -60
```

The invitations are grouped by type, e.g. `link "Persone (N)"` (people). Each one shows:

- the name and the headline;
- "Inviato oggi" (sent today) or a date;
- `link "Ritira l’invito inviato a <Name>"` ("Ritira" = withdraw). **Withdrawing is irreversible: never click it.** If the user wants to withdraw an invitation, tell them to do it themselves.

With no sent invitations, the page shows "Nessun invito inviato".
