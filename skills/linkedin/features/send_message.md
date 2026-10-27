# Send a message

**Ask first.** Before sending, the user must have approved **the recipient and the exact text**. If either changes, ask again.

## 1. Prepare the text and ask

Write the message, then show it to the user exactly as it will be sent:

> To: <Name> (<headline>)
> Text: "Ciao! …"
> Send it?

Wait for an explicit "yes". "Ok, but change X" is not a yes: apply the change and ask again.

## 2. Open the conversation

Pick one way:

- **Existing conversation:** open the messaging overlay (see `read_messages.md`, "Read the inbox"), then click the person's `heading "<Name>"`. Or use `/messaging/?searchTerm=<name>` and click the result.
- **From the profile** (also for someone you've never written to): open `https://www.linkedin.com/in/<username>/`, then click the header's `link "Invia messaggio"`. It opens a chat window in the bottom-right overlay, on the same page. Make sure it's the header link, inside `region "CTA principale"`; the right column has "Invia un messaggio a <Other person>" links for other people.

Then `wait 2500`.

## 3. Check you're in the right conversation

```
agent-browser snapshot -i -c | grep -E 'Chiudi la conversazione con|heading "'
```

The conversation must be with the approved recipient: the heading or the "Chiudi la conversazione con <Name> e <User>" button shows the name. If it's someone else, stop.

## 4. Find the composer

```
agent-browser snapshot -i -c | grep -E 'textbox "Scrivi un messaggio|button "Invia"|opzioni di invio'
# - textbox "Scrivi un messaggio…" [ref=e66]:
# - button "Invia" [disabled, ref=e63]          ← disabled while the textbox is empty
# - button "Apri le opzioni di invio" [ref=e67]
```

If there are several composers (two chat windows open), close the extra one with its `button "Chiudi la conversazione con …"` and start again from step 3.

## 5. Type

```
agent-browser click @eTEXTBOX --human
agent-browser wait 1000
agent-browser keyboard type "Approved text, exactly"
```

- For a line break: `agent-browser press Shift+Enter` between two `keyboard type` commands.
- **Never press `Enter` in the composer.** The send options have two modes, "Premi Invio per inviare" (Enter sends) and "Clicca su Invia" (Enter adds a new line), and you don't know which one the user has.
- Don't use `fill` here: type with `keyboard type`, like a person would.

## 6. Check the text before sending

```
agent-browser snapshot -i -c | grep -E 'textbox "Scrivi un messaggio|button "Invia"'
# - textbox "Scrivi un messaggio…" [ref=e66]: Approved text, exactly
# - button "Invia" [ref=e63]                    ← enabled now
```

The text after `:` must be **identical** to the approved text. If it isn't (missing letters, text typed twice, leftovers from a draft), clear it and type again:

```
agent-browser click @eTEXTBOX
agent-browser press Control+a     # on macOS: Meta+a
agent-browser press Backspace
```

then repeat steps 5 and 6.

## 7. Send

```
agent-browser click @eINVIA --human
agent-browser wait 3000
```

## 8. Check it was sent

```
agent-browser snapshot -i -c | grep -E 'textbox "Scrivi un messaggio|button "Invia"'
# - textbox "Scrivi un messaggio…" [ref=e66]:      ← empty again
# - button "Invia" [disabled, ref=e63]             ← disabled again
agent-browser snapshot | grep -F '<first words of the text>'
```

The full snapshot must contain "<User name> ha inviato i seguenti messaggi (HH:MM)" followed by the text. Tell the user it was sent. If the text isn't there, don't send it again: take a screenshot and tell the user.

## Closing the chat window

`button "Chiudi la conversazione con <Name> e <User>"` closes the overlay chat. Do it also when you opened a chat and the user decided not to send.
