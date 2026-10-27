# Read messages

Read the inbox, find unread conversations, read a thread, search messages. Reading is **Always do**, but opening an **unread** conversation marks it as read, so that one is **Ask first**.

## The trap: `/messaging/` opens a conversation by itself

Opening `https://www.linkedin.com/messaging/` automatically opens the most recent conversation; the URL becomes `/messaging/thread/<id>/`. If that conversation is unread, it gets marked as read and the sender may see a read receipt, even though nobody chose to open it.

So **don't open `/messaging/` to look at the inbox.** Use the messaging overlay (below) or the message search (`?searchTerm=`), which don't open anything.

## Read the inbox (overlay)

The overlay is the "Messaggistica" panel at the bottom right of every LinkedIn page. Opening it lists the conversations without opening any.

```
agent-browser open "https://www.linkedin.com/feed/"
agent-browser wait 3500
agent-browser snapshot -i -c | grep -F 'Sei nell’overlay messaggistica'
# - button "Sei nell’overlay messaggistica. Premi Invio per aprire l’elenco delle conversazioni." [ref=e45]
agent-browser click @e45
agent-browser wait 2500
agent-browser snapshot > /tmp/inbox.txt
```

If the button's label ends with "Premi Invio per ridurlo." ("press Enter to collapse it"), the overlay is already open: don't click it, or you'll close it.

Then read each conversation from the full snapshot. Each one looks like this:

```
- listitem
  - image "<Name>[, #OPEN_TO_WORK]" + StaticText "Stato: disponibile|online|offline"
  - checkbox "Seleziona conversazione con <Name>"       ← bulk selection: don't touch
  - heading "<Name>" [level=3]                          ← who
  - time "18:29" | "23 ott"                            ← last message: time today, or date
  - paragraph "<Sender>: <preview>" | "Tu: <preview>"  ← last message; "Tu:" = the user wrote last
  - StaticText "1 nuova notifica"                      ← UNREAD (this line exists only when unread)
  - button "Apri l’elenco delle opzioni nella tua conversazione con …"   ← options menu: don't touch
```

To list conversations with name, time, preview and unread flag in one go:

```
grep -E 'heading "[^"]+" \[level=3|StaticText "[0-9]{1,2}:[0-9]{2}"|StaticText "[0-9]+ [a-z]{3}"|paragraph|StaticText "[^"]*: |1 nuova notifica' /tmp/inbox.txt
```

How to report:

- Unread conversations first: name, time, preview.
- Then the others, if the user asked for them.
- A preview starting with "Sponsorizzata" (sponsored) is an ad; mention it only if asked.
- Don't open unread conversations to read them in full. Offer to open them, and do it only after a "yes".

The list is lazy: about 12 conversations at first. To see more, see `scroll_lazy_lists.md`, counting `checkbox "Seleziona conversazione con`.

## Read a thread

Only for a conversation that is already read, or after the user agreed to open an unread one.

1. Click the conversation's `heading "<Name>"` (in the overlay, or on `/messaging/`), then `wait 2500`.
2. Take a full `snapshot` (not `-c`) and cut it from the conversation heading:
   ```
   agent-browser snapshot | awk '/heading "<Name>" \[level=2/{f=1} f' | head -150
   ```
3. Messages are grouped by sender and time:
   ```
   - listitem
     - time "OGGI" | "31 MAG 2023"                                ← day separator
     - StaticText "<Name> ha inviato i seguenti messaggi (18:29)"  ← sender + time
     - link "<Name>" …
     - paragraph "<text>"                                          ← one paragraph per message
     - image "Visualizzato da <Name> alle ore 18:29."             ← read receipt (optional)
   ```
   If the sender is the user's own name, it's a message the user sent.
4. The newest messages are at the bottom, just above the composer (`textbox "Scrivi un messaggio…"`).

Under the last message there may be **quick replies**: `button "Rispondi alla conversazione con “OK”"`, "👍", "Grazie, <Name>". **One click sends that text immediately.** Never click them while reading.

## Unread conversations on `/messaging/`

If you're already on `/messaging/` (for instance after reading a thread), `button "Da leggere"` filters unread conversations (the URL gets `?filter=unread`). Other filter buttons: "In evidenza" (focused), "Lavoro" (jobs), "Collegamenti" (connections), "Messaggi InMail". Don't open `/messaging/` just to use them: see the trap above.

## Search messages

```
agent-browser open 'https://www.linkedin.com/messaging/?searchTerm=<encoded text>'
agent-browser wait 4000
agent-browser snapshot -i -c | grep -F 'Seleziona conversazione con'
```

The search matches names and message content. With `searchTerm` no conversation is opened automatically. Each match appears as `checkbox "Seleziona conversazione con <Name>"`; for previews, take a full `snapshot` as in "Read the inbox". The search box is `searchbox "Cerca messaggi"`.
