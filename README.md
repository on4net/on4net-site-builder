# on4net-site-builder

A Claude skill for building, redesigning and editing a website on an **ON4NET** installation through the ON4NET MCP server
(designs, theme layouts, sections, elements, pages, menus, media, blog, forms, Google tools).

## Contents

| Path | Purpose |
|---|---|
| `SKILL.md` | The main guide: session start, model, build order, rules, quality bar, tool map |
| `reference/` | Topic guides read on demand (`structure`, `shortcodes`, `builtins`, `theme-and-style`, `content-and-data`, `blueprint`, `google`, `recipes`) |
| `design-director/` | The bundled design skill `on4net-design-director` (brand discovery, visual direction, animation) |

## Connecting the MCP server

The skill only tells an agent *what to do*; the agent needs the ON4NET MCP server to *do* it. Two things are needed first:

1. **An agent key**: in the panel, Settings > AI agents > new key (`o2n_agent_…`). It names one site, shown once, revocable.
2. **The address**: `https://<site>/mcp` when the installation runs the hosted `mcp` service, or `ON4NET_API_URL=https://<site>` for the local (stdio) server.

| Client | Transport | How the key is given |
|---|---|---|
| Claude Code / Desktop / Cursor | local (stdio) `npx -y @on4net/mcp`, or hosted URL | environment variables, or an `Authorization: Bearer` header |
| Claude in the cloud (claude.ai) | hosted URL | `x-api-key` header, or the OAuth sign-in page |
| ChatGPT / OpenAI | hosted URL | OAuth sign-in page (needs `ON4NET_OAUTH_SECRET` on the server), or a header via the API |
| OpenClaw and other agents | stdio or hosted URL | whichever the agent supports (see below) |

### Which header carries the key

The hosted server accepts the key in **either** header, for every client:

| Header | Value |
|---|---|
| `x-api-key` | `o2n_agent_…` (the bare key, no prefix) |
| `Authorization` | `Bearer o2n_agent_…` |

**Prefer `x-api-key`** when the client lets you set a custom header, and use it wherever a client reserves `Authorization` for OAuth
(claude.ai custom connectors do). Both give exactly the same access; the key is checked the same way and is revoked the same way in the panel.
Clients that can only do OAuth (ChatGPT connectors) use the sign-in page instead.

```
claude mcp add --transport http on4net https://<site>/mcp --header "x-api-key: o2n_agent_…"
```

### Claude in the cloud (claude.ai)

Settings > Connectors > **Add custom connector**:

- URL: `https://<site>/mcp`
- Authentication: claude.ai reserves the `Authorization` header for OAuth, so send the key as **`x-api-key: o2n_agent_…`** (the server
  accepts the bare key there). If the installation has `ON4NET_OAUTH_SECRET` set, you can instead leave the header empty and sign in on
  the page the connector opens: type the site's address and the key once.

Then give the cloud agent the skill (Settings > Capabilities > Skills > upload the `on4net-site-builder` folder as a zip). Claude Code in
the cloud / on the web takes the connector the same way, or `claude mcp add --transport http on4net https://<site>/mcp --header "x-api-key: o2n_agent_…"`.

### OpenAI (ChatGPT and the API)

- **ChatGPT** (Developer mode > Connectors > **Create**): MCP server URL `https://<site>/mcp`, authentication **OAuth**. ChatGPT cannot paste a
  header, so the server must be started with `ON4NET_OAUTH_SECRET` (32+ characters; keep it secret). ChatGPT opens the server's sign-in
  page; type the site's address and the agent key, and it receives a sealed token (the key stays revocable in the panel). If your
  installation uses another client address, list its host in `ON4NET_OAUTH_REDIRECT_HOSTS`; `chatgpt.com`, `openai.com`, `claude.ai` are allowed by default.
- **OpenAI API** (Responses API, remote MCP tool): the key goes in as the tool's authorization/headers, no OAuth page involved:

  ```json
  { "type": "mcp", "server_label": "on4net", "server_url": "https://<site>/mcp",
    "authorization": "o2n_agent_…", "require_approval": "always" }
  ```

  Keep `require_approval` on for anything that publishes or deletes; the tools carry read-only/destructive annotations to help.
- **Skills**: ChatGPT has no `SKILL.md` loader. The same text is available to any client as the MCP tool `get_guide` (and resource
  `on4net://guide`), so tell the agent to call `get_guide` first; copy `SKILL.md` into the custom instructions / system prompt if you want it always on.

### OpenClaw

OpenClaw takes MCP servers from its own config, so the entry is the standard one. Use whichever form your version supports:

```json
{ "mcpServers": { "on4net": { "command": "npx", "args": ["-y", "@on4net/mcp"],
    "env": { "ON4NET_API_URL": "https://<site>", "ON4NET_AGENT_KEY": "o2n_agent_…" } } } }
```

or, for a client that connects to a URL: `https://<site>/mcp` with `Authorization: Bearer o2n_agent_…`. Put `SKILL.md` (and the
`reference/` folder) where OpenClaw reads skills or instructions; if it has no skill folder, rely on `get_guide`. The exact file name
and key names differ between OpenClaw versions: check its MCP documentation and keep the values above.

### After connecting: check it

Ask the agent to run `get_site`. It should answer with the site's name and what the key may do. A `401` means the key is wrong or revoked, a
`403` means the key's permissions do not cover the act (tell the owner, do not retry), and "could not reach" means the address is wrong.
Never paste the key into a chat: put it only in the connector/config field.

## Companion skills

The site builder does not do design or SEO research itself; it hands those to companion skills and uses them whenever the task
matches. `SKILL.md` ("Companion skills") tells the agent to check for them at the start of every session.

| Area | Skill | Source | Provides |
|---|---|---|---|
| Design | `on4net-design-director` | bundled, `design-director/` | brand-aware design process, industry presets, motion |
| Design | `ui-ux-pro-max` | https://github.com/nextlevelbuilder/ui-ux-pro-max-skill | styles, palettes, font pairings, UX/a11y rules |
| SEO | `claude-seo` | https://github.com/AgriciDaniel/claude-seo | `/seo audit`, `page`, `technical`, `schema`, `sitemap`, `hreflang`, `local`, `content` … |
| GEO | `geo-seo-claude` | https://github.com/zubair-trabzada/geo-seo-claude | `/geo audit`, `quick`, `citability`, `crawlers`, `llmstxt`, `brands`, `schema` … |

### Install

Run these yourself; the agent will only suggest them and never runs them without your yes.

**1. on4net-design-director** - already in this repo. Make it discoverable by copying (or linking) `design-director/` next to
this skill, e.g. into `~/.claude/skills/on4net-design-director/`.

**2. ui-ux-pro-max** (inside Claude Code):

```
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```

**3. claude-seo** (inside Claude Code 1.0.33+):

```
/plugin marketplace add AgriciDaniel/claude-seo
/plugin install claude-seo@agricidaniel-claude-seo
/seo setup
/seo doctor
```

Manual alternative on Windows: `git clone --depth 1 https://github.com/AgriciDaniel/claude-seo.git`, then
`powershell -ExecutionPolicy Bypass -File claude-seo\install.ps1`.

**4. geo-seo-claude** (Windows: use **Git Bash**, not PowerShell; needs Python 3.10+ and Git):

```
git clone https://github.com/zubair-trabzada/geo-seo-claude.git
cd geo-seo-claude
./install-win.sh
```

It installs into `~/.claude/skills/geo*` and `~/.claude/agents/geo-*.md`. Review the script before running it.

Restart Claude Code afterwards so the new skills are listed.

### Check what is installed

In Claude Code, `/plugin` lists installed plugins and `/skills` lists available skills. You should see `on4net-site-builder`,
`on4net-design-director`, `ui-ux-pro-max`, `claude-seo` (`/seo`) and `geo` (`/geo`).

### How the agent uses them

1. Session start: `get_site`, inventory, and a check of which companions are available.
2. Design work (new site, redesign, theme layout, new section): `on4net-design-director` + `ui-ux-pro-max` first, then build with
   ON4NET sections, elements and theme layouts.
3. After pages are published: `claude-seo` for SEO, `geo-seo-claude` for AI-search visibility; fixes are applied through ON4NET
   tools (page SEO fields, sections, `set_google_tags`, menus, media).
4. Missing companion: the agent says so, shows the command above, and continues with the main guide until you decide.

### Safety

Companion skills are third-party code. Read their install scripts before running them. Their suggestions never override the rules
in `SKILL.md` section 4: no third-party scripts, fonts or embeds without the owner's approval, nothing published without a yes.
