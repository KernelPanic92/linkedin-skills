# Read notifications

Reading the list is **Always do**. Clicking a notification or its quick action is **Ask first**.

## 1. Count the new ones first

The snapshot doesn't show which notifications are unread. The only signal is the counter in the top bar, so **read it before opening the notifications page**, because it may reset afterwards:

```
agent-browser snapshot -i -c | grep -E 'Notifiche'
# - link "Notifiche, 2 nuove notifiche" [ref=e19]     ← N = 2
#   or "2 nuove notifiche Notifiche", or "Notifiche, 1 nuova notifica"
```

No number in the label means 0 new notifications.

## 2. Open the list

```
agent-browser open "https://www.linkedin.com/notifications/"
agent-browser wait 4000
agent-browser snapshot -c > /tmp/notifications.txt
```

Notifications are sorted newest first, so **the first N are the new ones**.

Each notification looks like this:

```
- generic "<full text of the notification>"
  - link [ref=…]                                 ← author's picture
  - paragraph "<Name> ha confermato la tua competenza …"   ← the text
  - link "Ringrazia" | "È il tuo turno" …         ← quick action (optional): Ask first
- paragraph "2m" | "1h" | "3g"                    ← age
- button "Altre opzioni"                          ← menu: see below
- link "<full text>"                              ← opens the notification's content
```

To list them quickly:

```
grep -E '^\s*- generic "' /tmp/notifications.txt | head -20
```

How to report: the N new ones first, with their age, then older ones if the user asked.

Filters at the top: "Tutto" (all), "Offerte di lavoro" (jobs), "I miei post" (my posts), "Menzioni" (mentions).

## What not to click

- **The notification link** opens its content (a post, a profile…). That's fine when the user asks to open it.
- **Quick actions** ("Ringrazia" = thank, "È il tuo turno" = your turn, endorse back, reply…) act on the user's behalf: Ask first.
- **`button "Altre opzioni"`** opens a menu with:
  - "Cambia le preferenze per le notifiche": Ask first;
  - "Elimina notifica": deletes it, irreversible, so don't;
  - "Mostra meno contenuti come questo": Ask first.

## More notifications

The list is lazy (about 18 at first, then about 10 more per round). See `scroll_lazy_lists.md`, counting `button "Altre opzioni"`.
