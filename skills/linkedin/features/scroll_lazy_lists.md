# Scroll lazy lists

Several LinkedIn lists load more items only when the last one comes into view: conversations, notifications, the classic job list and the feed. Scrolling is **Always do**, but each round is a page action: load only what you need.

## Which method

| List | Method | What to count |
| - | - | - |
| Conversations (overlay or `/messaging/`) | `scrollintoview` the last item | `checkbox "Seleziona conversazione con` |
| Notifications | `scrollintoview` the last item | `button "Altre opzioni"` |
| Classic job search | `scrollintoview` the last card | `get count "[data-job-id]"` vs `get count "li[data-occludable-job-id]"` (see `search_jobs_classic.md`) |
| Feed | `scroll down 1500` (the feed is the page itself) | posts (`button "Nascondi il post di`) |

The conversation list is an **inner scroll container**, so `scroll down` on the page does nothing there. `scrollintoview` works on any container, which is why it's the default.

The AI job search doesn't need scrolling: all 25 cards of a page are already there.

## The loop

One round:

```
before=$(agent-browser snapshot -i -c | grep -c 'checkbox "Seleziona conversazione con')
last=$(agent-browser snapshot -i -c | grep -F 'checkbox "Seleziona conversazione con' | tail -1 | grep -oE 'ref=e[0-9]+' | cut -d= -f2)
agent-browser scrollintoview @$last
agent-browser wait 3000
after=$(agent-browser snapshot -i -c | grep -c 'checkbox "Seleziona conversazione con')
echo "$before -> $after"
```

Replace the counted pattern for other lists (see the table). Each round loads about 5–10 items.

**Stop when:**

- you have enough items for what the user asked; **or**
- `after` equals `before` (the list ended, or it's loading slowly: try one more round with `wait 5000`, then stop); **or**
- you've done 5 rounds. Tell the user how many items you've read and ask whether to go on.

Two notes:

- Don't use `wait --load networkidle` on LinkedIn: realtime connections keep the network busy, so it often times out.
- If nothing loads, take a screenshot to see whether a loader or an error is visible.
