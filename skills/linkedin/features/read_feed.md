# Read the feed

Reading is **Always do**. Every button on a post is a public or social action: **Ask first**.

## Open and read

```
agent-browser open "https://www.linkedin.com/feed/"
agent-browser wait 4000
agent-browser snapshot -c
```

`agent-browser get text main` also works, but its first part is the user's own sidebar (name, headline, profile views, post impressions, saved items, groups). Skip it, since the posts come after.

## A post

Each post has, in `snapshot -i -c`:

- `button "Apri il menu dei controlli per il post di <Author>"`: post menu;
- `button "Nascondi il post di <Author>"`: hide the post;
- `button "Segui <Author>"`: follow (only on posts from people or pages you don't follow);
- reactions: "Consiglia" (like), `button "Commenta"` (comment), `button "Diffondi il post"` (repost), `link "Invia"` (send to someone).

The author's name is in those button labels, which is the easiest way to tell posts apart.

How to report: for each post, the author, the first lines of the text, and whether it's promoted ("Promosso").

## More posts

The feed is the page itself, not an inner container: `agent-browser scroll down 1500`, `wait 2500`, snapshot again (see `scroll_lazy_lists.md`). Read a few screens, not the whole feed.
