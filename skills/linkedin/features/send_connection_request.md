# Send a connection request

**Ask first.** The user must approve the person and, if there is one, the exact note.

## 1. Open the profile and find the button

```
agent-browser open 'https://www.linkedin.com/in/<username>/'
agent-browser wait 4000
agent-browser snapshot -i -c | awk '/region "CTA principale"/{f=1} f' | head -20
```

Look for `link "Invita <Name> a collegarsi"` (shown as "Collegati") **with this person's name**, inside `region "CTA principale"`. The right column has identical-looking buttons for other people: ignore them.

| You find | Do this |
| - | - |
| `link "Invita <Name> a collegarsi"` | Go on |
| nothing, but there's `button "Altro"` | Click "Altro", snapshot, and look for the connect item in the menu |
| only `link "Invia messaggio"` | Already connected: tell the user |
| `link "In sospeso, clicca per ritirare…"` | Request already pending: tell the user, and **don't click it** |
| no connect anywhere, only "Segui" | The profile only allows following: tell the user |

## 2. Ask

> Send a connection request to <Name> (<headline>)? With a note or without?

If they want a note, write it and show the exact text. LinkedIn notes are short (about 200 characters for free accounts).

## 3. Click and wait for the dialog

```
agent-browser click @eCONNECT --human
agent-browser wait --text "Vuoi aggiungere una nota"
agent-browser snapshot -i -c | grep -E 'nota'
# - button "Aggiungi una nota" [ref=e68]
# - button "Invia senza nota" [ref=e69]
```

The dialog is `dialog "Vuoi aggiungere una nota al tuo invito?"` ("add a note to your invitation?"). It appears with a delay: **if the snapshot doesn't show it, wait and snapshot again**. Don't click "Collegati" a second time.

## 4a. Without a note

Click `button "Invia senza nota"`.

## 4b. With a note

1. Click `button "Aggiungi una nota"`, `wait 2000`, snapshot.
2. Click the dialog's textbox and `keyboard type` the approved note.
3. Snapshot and check that the textbox holds exactly the approved note.
4. Click the dialog's send button.

## 5. Check

```
agent-browser wait 3000
agent-browser snapshot -c | grep -E 'Invito inviato|In sospeso'
# StaticText "Invito inviato a <Name>."                                      ← toast
# link "In sospeso, clicca per ritirare l’invito inviato a <Name>"         ← new header link
```

**Never click the "In sospeso…" link**, since that withdraws the invitation. The request is also listed in the sent invitations (see `manage_invitations.md`).
