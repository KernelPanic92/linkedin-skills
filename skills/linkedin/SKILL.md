---
name: linkedin
description: Read and search LinkedIn without leaving the chat. Reads the inbox and notifications, the home feed, and profiles. Searches people by role, company, school or location and filters by connection degree. Searches job offers in natural language or with classic filters such as date, experience level, company, remote work and city, and opens the full job description. Saves, dismisses and tracks jobs. Manages received and sent invitations. Sends messages and connection requests only after the user confirms the exact text and recipient. Use when the user asks to check LinkedIn messages or notifications, find a person or a recruiter, look at a profile, search or filter job offers, save or dismiss a job, or write to a contact on LinkedIn. Not for publishing posts, editing the profile, or scraping data outside the user's own account.
author: kernelpanic92
version: 2.1.0
license: MIT
---

# LinkedIn

Use [agent-browser](https://github.com/vercel-labs/agent-browser) to interact with LinkedIn.

This file holds the rules that apply to every task. Each action is described in its own file under `features/`. **Before doing an action, read its feature file** (see the "Features" table); read only the ones you need.

## Before starting

verify that the agent-browser CLI is installed:

! agent-browser --version

If it is not installed, proceed to install it.

verify that the agent-browser skills are installed:
! npx skills list

If it is not installed, proceed to install it:

```
npx skills add vercel-labs/agent-browser
```

otherwhise use the CLI skills

```
agent-browser skills get core --full
```

###  agent-browser Global Installation (recommended)

Installs the native Rust binary:

```bash
npm install -g agent-browser
agent-browser install  # Download Chrome from Chrome for Testing (first time only)
```

### agent-browser Project Installation (local dependency)

For projects that want to pin the version in `package.json`:

```bash
npm install agent-browser
agent-browser install
```

Then use via `package.json` scripts or by invoking `agent-browser` directly.

### agent-browser Homebrew (macOS)

```bash
brew install agent-browser
agent-browser install  # Download Chrome from Chrome for Testing (first time only)
```

### agent-browser Cargo (Rust)

```bash
cargo install agent-browser
agent-browser install  # Download Chrome from Chrome for Testing (first time only)
```

### agent-browser From Source

Requires Node.js 24+, pnpm 11+, and Rust.

```bash
git clone https://github.com/vercel-labs/agent-browser
cd agent-browser
pnpm install
pnpm build
pnpm build:native   # Requires Rust (https://rustup.rs)
pnpm link --global  # Makes agent-browser available globally
agent-browser install
```

### agent-browser Linux Dependencies

On Linux, install system dependencies:

```bash
agent-browser install --with-deps
```

This exits nonzero if the package manager cannot install every required browser library.

### agent-browser Updating

Upgrade to the latest version:

```bash
agent-browser upgrade
```

Detects your installation method (npm, Homebrew, or Cargo) and runs the appropriate update command automatically.

### agent-browser Requirements

- **Chrome** - Run `agent-browser install` to download Chrome from [Chrome for Testing](https://developer.chrome.com/blog/chrome-for-testing/) (Google's official automation channel). Existing Chrome, Brave, Playwright, and Puppeteer installations are detected automatically. No Playwright or Node.js required for the daemon.
- **Node.js 24+ and pnpm 11+** - Only needed when building from source.
- **Rust** - Only needed when building from source (see From Source above).


## Safety Rules (read first)

Every action falls into one of three tiers. If an action isn't listed, treat it as **Ask first**.

### ✅ Always do (no need to ask)

- **Read without side effects:** feed, profiles, people/message/job search results, job details, saved jobs, and the inbox and notification **lists** (previews included). Opening an *unread* item is not side-effect free; see Ask first.
- **Navigate:** open URLs, search, apply search filters, paginate, scroll lazy-loaded lists, take snapshots and screenshots.
- **Keep the session:** use `--session`/`--profile` so that the login is saved and reused, and check login status (see the "Check login status" section).
- **Pace every action:** wait a random 2–5 seconds between commands that touch the page (`agent-browser wait 2000` … `wait 5000`, vary the value), and stay under ~30 page actions per hour.
- **Accept cookie and privacy consent banners:** if a cookie or privacy policy banner or dialog covers the page, click its accept button (role `button`, e.g. "Accetta"/"Accept"), then take a new snapshot. This applies only to consent banners. Other dialogs (Premium offers, "Add a note", sign-up prompts) follow the other rules. With a persistent `--profile` the choice is remembered, so it should appear only once.
- **Hand over to the user** as soon as a login, 2FA, CAPTCHA or security checkpoint appears (see the "Troubleshooting" section).

### ⚠️ Ask first

Ask, wait for an explicit "yes", then act. Confirmation covers that single action with that exact content: if the text, recipient or target changes, ask again.

- **Send a message:** show the recipient and the exact text (`features/send_message.md`). Quick-reply buttons ("OK", "👍") send immediately, so they count as sending.
- **Send, accept or ignore a connection request** or any other invitation (event, group, page): show who it is and, for requests you send, the exact note.
- **Mark a message or notification as read:** opening an unread conversation or clicking an unread notification marks it as read, and the sender may see a read receipt. Report unread items from the list preview; open them only once the user agrees. Careful: **opening `/messaging/` opens the most recent conversation by itself**. Read the inbox from the messaging overlay instead (`features/read_messages.md`).
- **Change the state of a job:** save or unsave it, dismiss it ("Ignora offerta di lavoro"), or turn a job alert on or off.
- **Public or social actions:** follow or unfollow, react, comment, repost, post. Show the exact content for anything written.
- **Anything else that changes the account:** profile, settings, privacy or visibility.
- **Keep trying after you're stuck:** a step is stuck when **3 attempts** fail. That means the same command or an equivalent alternative (another ref, selector or URL), each followed by a fresh snapshot. Failure signs: element not found, a click with no visible effect, a snapshot identical to the previous one, an unexpected URL, a list that doesn't grow.
  When that happens, stop and ask the user for help instead of trying yet another variant:
  1. **Goal:** what you were trying to do.
  2. **Attempts:** each command you tried and what you saw.
  3. **Evidence:** a screenshot (`agent-browser screenshot`).
  4. **Proposal:** what you'd try next, or whether to reopen the session with `--headed` so the user can look.

  Wait for their answer. Don't count reformulations of the same idea as new approaches.

### ❌ Never do

- **Apply to a job**, "Easy Apply" / "Candidatura semplice" included, even when asked: the user always clicks "apply" themselves.
- **Put the password on a command line** or read it into the context (see the "Login" section).
- **Solve or bypass a CAPTCHA, 2FA or security checkpoint, in any way.** This covers guessing the answer, clicking tiles or images at random, analysing the image, sending it to external services, reloading until it goes away, and switching browser or session. A CAPTCHA is the user's job, always.
- **Invent creative workarounds when something doesn't work.** Interact with LinkedIn only through the page, the way a person would, using the commands in this skill. In particular, never:
  - run `eval` to click, hide or change elements, or to read data the page doesn't show;
  - call LinkedIn's internal APIs (`voyager`, GraphQL) or `curl` with the session cookies;
  - edit cookies or storage by hand;
  - change the user agent or add stealth/anti-detection flags;
  - fetch LinkedIn content from other sites, caches or search engines;
  - use another account.

  If the documented way fails, follow the stuck rule in Ask first: stop and ask.
- **Harvest or act in bulk:** crawling many profiles, exporting contact or result lists, collecting data for third parties, sending messages or invitations in batches. Reading on the user's behalf at human pace is the intended use; bulk collection and automated actions at volume violate LinkedIn's User Agreement and get the account restricted.
- **Retry fast after a block:** after a rate limit, forced logout or checkpoint, stop and don't resume on your own.
- **Do anything irreversible:** delete conversations or notifications, remove connections, withdraw invitations. Tell the user to do it themselves.
- **Accept Premium trials, upsells or anything involving a payment.**

## Features

Read the file in the last column before doing the action. Paths are relative to this skill's folder.

| Action | What it covers | Read it when | File |
| - | - | - | - |
| Read messages | Inbox list without opening conversations, unread detection, reading a thread, searching messages | The user asks about their messages, inbox, unread conversations, or what someone wrote | `features/read_messages.md` |
| Send a message | Composer, typing, checking and sending; existing chat or from a profile | The user asks to write, reply or send something to someone | `features/send_message.md` |
| Read notifications | Notification list, how to tell which are new, quick actions to avoid | The user asks about notifications or "what's new" | `features/read_notifications.md` |
| Read the feed | Posts and their action buttons | The user asks what's on their feed or about recent posts | `features/read_feed.md` |
| Search people | People search, degree filters, pagination | The user asks to find a person or people with a given role, company or place | `features/search_people.md` |
| View a profile | Reading a profile, header actions, what not to click | The user asks about a specific person or profile | `features/view_profile.md` |
| Send a connection request | Connect button, note dialog, pending state | The user asks to connect with someone | `features/send_connection_request.md` |
| Manage invitations | Received invitations (accept/ignore) and sent invitations | The user asks about pending invitations or connection requests | `features/manage_invitations.md` |
| Search jobs (AI, default) | Natural-language job search, URL filters, list, pagination, job detail | Any job search, unless the classic search is needed | `features/search_jobs_ai.md` |
| Search jobs (classic) | Boolean query, sort by date, all `f_*` filters, city `geoId` lookup | The user needs newest-first, a filter the AI search lacks (experience level, company, industry…), exact phrases, or the AI search is unavailable | `features/search_jobs_classic.md` |
| Save or dismiss a job | Save/unsave, dismiss/undo, saved jobs tracker | The user asks to save a job, drop jobs they don't want, or see saved jobs | `features/save_dismiss_job.md` |
| Scroll lazy lists | Loading more items in conversations, notifications, classic job list, feed | A feature file tells you to load more items | `features/scroll_lazy_lists.md` |

## Setup

This skill was written against a **1600x1200** viewport. Set it at the start of every session:

```
agent-browser set viewport 1600 1200
```

**When you're done**, close the browser so that the session doesn't stay open: `agent-browser close`. With `--profile` the login is kept for the next run.

### Global parameters

Commands in this skill omit global parameters; add them as needed.

| Name | Description |
| - | - |
| `--session <label>` | Named, isolated session. Recommended, so that LinkedIn work doesn't mix with other browsing. |
| `--profile <dir>` | Persistent browser profile: the login survives between runs. Recommended (see the "Login" section). |
| `--headed` | Visible browser window. Required whenever the user has to act (login, 2FA, CAPTCHA). Also needed when LinkedIn shows a Cloudflare page in headless mode. |

## Working with the page

### The basic loop

Every action follows the same loop. Don't skip steps, even when you think you know the page.

1. **Open or act:** `agent-browser open '<url>'` or `agent-browser click @eN --human`.
2. **Wait:** `agent-browser wait 3000` to `4000` after an `open`, `2500` after a click that changes the page. When you expect a specific element (a dialog, a toast), wait for its text: `agent-browser wait --text "<text>"`.
3. **Look:** take a fresh snapshot (see below), and only then pick the ref for the next step.
4. **Check:** confirm that what you expected happened. Each feature file tells you what to check. Never assume a click worked.

**Read the output of every command.** `✓ Done` means the command ran; a line starting with `✗` means it failed (element not found, timeout…), even if the next commands seem to work.

### Finding elements and refs

- **Refs are written `@eN`** (`click @e42`) and are valid only for the latest snapshot. After any navigation, click that changes the page, or scroll that loads items, take a new snapshot before using refs. Never reuse a ref from an older snapshot.
- **Don't read a whole snapshot to find one element.** Filter it with `grep -F` on the exact label and take the ref from the matching line:
  ```
  agent-browser snapshot -i -c | grep -F 'button "Invia"'
  # - button "Invia" [ref=e63]          → use @e63
  ```
  To get just the ref in a script:
  ```
  ref=$(agent-browser snapshot -i -c | grep -F 'button "Invia"' | head -1 | grep -oE 'ref=e[0-9]+' | cut -d= -f2)
  agent-browser click @$ref --human
  ```
  If `grep` finds several lines, look at them all and pick the right one by its full label. Feature files list the look-alikes to avoid.
- **Prefer refs to `find`.** On LinkedIn, `agent-browser find role button click --name "…"` often fails with `✗ Element not found` even when the element is there. When it does, use the snapshot + `grep` + ref method above.
- **LinkedIn's UI follows the account language.** The labels in this skill come from an Italian account ("Avanti" = Next, "Pagina 2" = Page 2, "Candidatura semplice" = Easy Apply, "Ignora offerta di lavoro" = Dismiss job). With another language, identify elements by role and position in the tree, not by their literal label.

### Snapshots and reading text

| Command | Use it for | Watch out |
| - | - | - |
| `snapshot -i -c` | Finding elements and refs; lists whose cards hold all their text in the button/link name | Shows only interactive elements: plain text is missing |
| `snapshot -c` | A quick look at structure and text | **Drops the text of list items and bold text**, which often holds the important parts |
| `snapshot` | Reading content (thread, job description) | Long: filter it with `grep`/`awk` or scope it with `-s '<css selector>'` |
| `get text <selector>` | Clean text of a region, e.g. `get text main`, `get text article` | Fails if the selector doesn't exist on that page |
| `snapshot -i -c -u` | Same as `-i -c`, plus the URL of each link | |
| `screenshot <path>` | Seeing what the page really shows, when the snapshot doesn't make sense | Look at the image before deciding |

To read just one part of a long snapshot, cut it from a known heading:

```
agent-browser snapshot | awk '/heading "Informazioni sull’offerta di lavoro"/{f=1} f' | head -80
```

- **Clicks:** prefer `click @eN --human` (curved, eased mouse movement). Don't pass `--seed` in normal use: a fixed seed makes every movement identical. Use it only to reproduce a run while debugging. `--human` needs a recent agent-browser (`agent-browser upgrade`); if the flag is rejected, drop it.
- **Typing:** to write text into a field, click the field first, then `agent-browser keyboard type "<text>"`. Accented letters and emoji are fine.
- **When the snapshot doesn't match what you expect** (an element is missing, a dialog doesn't show up), wait 2 seconds and snapshot again; dialogs often appear with a delay. If it still doesn't match, take a screenshot and look at it.

### URL parameters

Values in a URL (search keywords, message search terms) must be URL-encoded. Don't encode by hand; use:

```
python3 -c 'import urllib.parse,sys;print(urllib.parse.quote(sys.argv[1], safe=""))' 'full stack developer "Angular" (remote)'
# → full%20stack%20developer%20%22Angular%22%20%28remote%29
```

Then wrap the whole URL in **single quotes**, since `&` would otherwise be interpreted by the shell:

```
agent-browser open 'https://www.linkedin.com/search/results/people/?keywords=full%20stack%20developer'
```

Add optional parameters only when you use them; never leave placeholders (`QUERY`, `GEOID`…) in a URL.

## Check login status

```
agent-browser open "https://www.linkedin.com/feed/"
agent-browser wait 3500
agent-browser get url
```

| Final URL contains | Meaning |
| - | - |
| `/feed` | Logged in. |
| `/login`, `/uas/login`, `/authwall`, `/signup` | Not logged in. Go to the "Login" section. |
| `/checkpoint/` | Security checkpoint (2FA, CAPTCHA, identity check). Hand over to the user. |

## Login

Never type credentials through `fill` with a literal password: it ends up in shell history and in the model context. In order of preference:

**1. Persistent profile, manual login (recommended).** The user logs in once in a visible window; later runs reuse the profile.

```
agent-browser --headed --profile ~/.agent-browser/linkedin open "https://www.linkedin.com/login"
# ask the user to log in (and complete 2FA), then:
agent-browser wait --url "**/feed/**" --timeout 180000
```

Pass the same `--profile` on every following run. If a browser is already running without it, `agent-browser close` first: global options only apply when the browser starts.

**2. agent-browser auth vault.** The user stores the credentials once, from their own terminal:

```
agent-browser auth save linkedin --url "https://www.linkedin.com/login" --username "EMAIL" --password-stdin
```

Then the agent logs in without seeing the password:

```
agent-browser auth login linkedin
agent-browser get url   # check for /checkpoint/ → hand over to the user
```

## Troubleshooting

| Symptom | Action |
| - | - |
| Logged out (`/login`, `/authwall`) | Go to the "Login" section. With `--profile`, the session has probably expired: ask the user to log in again in `--headed` mode. |
| Title "Attention Required! \| Cloudflare" | Happens in headless mode, even before login; the same URL loads fine with `--headed`. Close the browser (`agent-browser close`) and reopen the session with `--headed --profile …`. Don't try to get past the Cloudflare page yourself. |
| `/checkpoint/` or CAPTCHA | Stop. `agent-browser screenshot /tmp/linkedin-checkpoint.png` and show it to the user. Reopen the same session with `--headed` so they can solve it, then `wait --url "**/feed/**" --timeout 180000` and resume. |
| Rate limited ("too many requests", empty pages, forced logout) | Stop all LinkedIn activity for 24 hours, then resume at a lower pace. |
| A ref isn't found or clicks the wrong thing | Stale ref: take a new snapshot and look the ref up again. |
| `find … --name` says `✗ Element not found` but the element is in the snapshot | Use the snapshot + `grep` + ref method (see the "Finding elements and refs" section). |
| A dialog or toast you expect isn't in the snapshot | It's slow to appear: `wait --text "<its text>"`, then snapshot again. If still missing, take a screenshot. |
| Search returns off-target jobs | Expected: searches are loose. Set the hard constraints as filters and filter the cards yourself (see the job search feature files). Also check login and the URL encoding. |
| A lazy list loads nothing | See `features/scroll_lazy_lists.md`; take a screenshot to see whether a loader or an error is visible. |
| No display available (server or container), or the browser won't open a window | Don't fall back to headless: LinkedIn blocks it. Ask the user to start Xvfb and run with `DISPLAY=:99 --headed`. For login or checkpoints, they connect via the dashboard (`agent-browser dashboard start`, reached through an SSH tunnel) or via VNC (localhost + SSH tunnel). |
