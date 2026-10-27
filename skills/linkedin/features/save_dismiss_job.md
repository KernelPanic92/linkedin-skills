# Save or dismiss a job

Saving and dismissing change the account: **Ask first**. Reading the saved jobs is **Always do**.

The steps come from the AI search (`search_jobs_ai.md`). The classic search uses the same "Ignora" button, and its save button is labelled "Salva <Title> presso <Company>".

## Dismiss ("Ignora")

Dismissing removes a job the user doesn't want, so that it stops coming back in later searches. It's also a signal to LinkedIn's recommendations. Use it when the user says a job is off target.

1. **Ask once for the whole list**, not job by job: "Dismiss these 5 jobs? 1. … 2. …". Name each job by title + company.
2. For each job, take a fresh snapshot and find its dismiss button:
   ```
   agent-browser snapshot -i -c | grep -F 'button "Ignora offerta di lavoro <Title>"'
   ```
   - **If more than one line matches** (same title, different companies), find the card that contains both the title and the right company, and use the "Ignora…" button just below it in the snapshot.
   - **Don't confuse it with the card itself:** the card's name also contains "Ignora offerta di lavoro …", but it starts with the title or company. The button to click is the line that **starts** with `button "Ignora offerta di lavoro`.
3. `agent-browser click @eN --human`, then `wait 2000`.
4. **Check:** the card stays where it is, and its name now contains "Offerta di lavoro <Title> ignorata, annulla Non consiglieremo più questa offerta di lavoro." ("we won't recommend this job again"):
   ```
   agent-browser snapshot -i -c | grep -F 'ignorata'
   ```
5. Pause 2–5 seconds, then go on with the next job.

**Undo:** the same button is now `button "Offerta di lavoro <Title> ignorata, annulla"`. Clicking it restores the card. It works only until the page reloads.

After a reload, a dismissed job is **gone from the list** (one card fewer). If the URL's `currentJobId` still points to it, the detail pane keeps showing it: that's normal.

The card's `button "Altre opzioni"` only contains "Perché vedo questa offerta di lavoro?" (why am I seeing this job?), so it isn't needed here.

## Save ("Salva")

Saving keeps a job for later. It's the right action when the user wants to apply later, since applying is theirs to do.

1. Ask: "Save <Title> at <Company>?"
2. Click the job's card (not "Ignora…"), `wait 2500`, and check that the detail pane shows that job (`link "<Title>"`).
3. Find and click the save button:
   ```
   agent-browser snapshot -i -c | grep -F 'Salva l’offerta di lavoro'
   agent-browser click @eN --human
   agent-browser wait 2000
   ```
   In the classic search, the button is `button "Salva <Title> presso <Company>"`.
4. **Check:**
   ```
   agent-browser snapshot -i -c | grep -E 'Hai salvato|Annulla salvataggio'
   # generic "Hai salvato questa offerta di lavoro. Vedi le offerte di lavoro salvate"   ← toast
   # button "Annulla salvataggio dell’offerta di lavoro"                                  ← the button changed
   ```
   The card also gets the badge "Salvata".

**Unsave:** click `button "Annulla salvataggio dell’offerta di lavoro"`. The toast says "Hai annullato il salvataggio di questa offerta di lavoro.", and the button goes back to "Salva l’offerta di lavoro".

## Saved jobs

```
agent-browser open "https://www.linkedin.com/jobs-tracker/"
agent-browser wait 4000
agent-browser snapshot -i -c
```

The page is a tracker with one filter per stage, each with a counter:

- "Salvate · N" (saved), the default;
- "In corso · N" (in progress);
- "Candidatura inoltrata · N" (applied);
- "Colloquio · N" (interview);
- "Archiviate" (archived);
- a "Data di pubblicazione" dropdown.

Click a stage's checkbox to switch to it. An empty stage shows `heading "Non ci sono offerte di lavoro qui"` (no jobs here).
