# linkedin-skills

Agent Skills for LinkedIn, packaged so they can be installed by Claude Code, the
`skills` CLI (`npx skills`), and any other agent that follows the
[Agent Skills](https://agentskills.io/specification) open format.

The skill drives LinkedIn through the
[agent-browser](https://github.com/vercel-labs/agent-browser) CLI: it checks the
session, reads messages and notifications, reads the feed, searches people and
job offers, opens profiles and job details, saves or dismisses jobs, manages
invitations and — only after explicit confirmation — sends messages and
connection requests.

## Repository layout

```
linkedin-skills/
├── .claude-plugin/
│   └── marketplace.json        # Claude Code plugin marketplace catalog
├── skills/
│   └── linkedin/
│       ├── SKILL.md            # Metadata + core rules (progressive disclosure)
│       └── features/           # One file per action, read on demand
└── README.md
```

Each action lives in its own file under `skills/linkedin/features/`. The agent
reads `SKILL.md` first and only opens the feature file it needs, which keeps the
context window small.

## Requirements

- [agent-browser](https://github.com/vercel-labs/agent-browser) CLI installed and
  authenticated against LinkedIn.
- Node.js 18+ if you install through `npx skills`.
- Claude Code (optional) if you install through the plugin marketplace.

## Installation

### Claude Code (plugin marketplace)

```bash
claude plugin marketplace add KernelPanic92/linkedin-skills
claude plugin install linkedin@linkedin-skills
```

Inside a session you can use the equivalent slash commands:

```
/plugin marketplace add KernelPanic92/linkedin-skills
/plugin install linkedin@linkedin-skills
```

### skills CLI (npx skills)

```bash
npx skills add KernelPanic92/linkedin-skills --skill linkedin
```

Add `-g` to install globally instead of at project level, and `-a claude-code`
(or another supported agent) to target a specific agent.

### Manual

Copy `skills/linkedin/` into your agent's skills directory. For Hermes Agent:

```bash
cp -r skills/linkedin ~/.hermes/skills/social-media/linkedin
```

## Usage examples

Once installed, ask the agent in natural language:

- "Check my LinkedIn notifications and tell me if anything needs a reply."
- "Find recruiters at fintech companies in Milan and show me their profiles."
- "Search LinkedIn jobs for remote Python roles posted in the last week."
- "Read my unread messages and summarize them."

Sending a message or a connection request is gated: the skill requires explicit
user confirmation before performing either action.

## Structure and conventions

The skill follows the Agent Skills specification:

- `SKILL.md` frontmatter carries `name`, `description`, `author` and `version`;
  `name` matches the parent directory (`linkedin`).
- The main file stays under 500 lines; detailed procedures live in
  `features/`, one level deep, and are referenced with relative paths.
- Everything the agent needs to route to the skill is in the description, so it
  triggers on LinkedIn tasks without loading the whole skill up front.

## Notes

LinkedIn's terms of service restrict automated access. Use this skill on your own
account, at your own risk, and keep the confirmation gate in place for any
outbound action.
