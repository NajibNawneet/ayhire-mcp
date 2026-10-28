# AyHire MCP server

Hire engineers by what they shipped. Describe a role and AyHire finds developers from their public GitHub work, judges each one against the role's criteria with links to the evidence, says who is open to work, and keeps your roles and shortlist in sync with the [AyHire](https://ayhire.com) web app.

- **Server URL:** `https://ayhire.com/mcp` (Streamable HTTP)
- **Sign-in:** OAuth 2.1 with dynamic client registration. Your AI app opens AyHire, you sign in and click **Connect**. No key to copy.
- **Official MCP Registry:** [`com.ayhire/ayhire`](https://registry.modelcontextprotocol.io/v0/servers?search=com.ayhire/ayhire)
- **Plans:** 14-day free trial, no card. See [pricing](https://ayhire.com/pricing).
- **Docs for agents:** [ayhire.com/agents.md](https://ayhire.com/agents.md)

AyHire is a hosted service. This repository holds what you need to connect to it: the server manifest, a Claude Code plugin, a Gemini CLI extension, the `ayhire` command line and the agent skills. The server itself is not open source.

## Connect

### Claude (web, desktop and mobile)

**Customize** › **Connectors** › **+ Add** › **Add custom connector**, name it **AyHire**, paste `https://ayhire.com/mcp`, choose **Sign in now** and **Register automatically**, then sign in when AyHire opens.

### ChatGPT

[ChatGPT Plugins](https://chatgpt.com/plugins) › **+** › **Add custom MCP server**, paste `https://ayhire.com/mcp`, choose **OAuth**, then sign in when AyHire opens.

### Claude Code

```
claude mcp add --transport http --scope user ayhire https://ayhire.com/mcp
claude mcp login ayhire
```

Or install the plugin from this repository, which adds the AyHire skills too:

```
/plugin marketplace add NajibNawneet/ayhire-mcp
/plugin install ayhire@ayhire
```

### Codex

```
codex mcp add ayhire --url https://ayhire.com/mcp
codex mcp login ayhire
```

### Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "ayhire": { "url": "https://ayhire.com/mcp" }
  }
}
```

### VS Code (GitHub Copilot)

`~/.copilot/mcp-config.json`:

```json
{
  "mcpServers": {
    "ayhire": { "type": "http", "url": "https://ayhire.com/mcp" }
  }
}
```

### Gemini CLI

```
gemini mcp add --transport http --scope user ayhire https://ayhire.com/mcp
```

Or as an extension: `gemini extensions install https://github.com/NajibNawneet/ayhire-mcp`.

### Any other MCP client

Point it at `https://ayhire.com/mcp`. Clients without OAuth can send an API key instead: create one under **Integrations** › **API keys** in AyHire and send it as `Authorization: Bearer ayh_...`.

Check it worked: ask your agent *"Use AyHire to check who I'm signed in as."* It answers with your email.

## What it can do

| Area | Tools |
|---|---|
| Find people | `find_people_for_role`, `find_people_behind_repos`, `search_people`, `search_repos`, `search_commits`, `search_candidates` |
| Judge them | `analyze_github_user`, `analyze_github_users`, `score_candidate`, `score_role_person` |
| Read the evidence | `get_person_evidence`, `contribution_profile`, `list_pull_requests`, `list_commits`, `read_readme` |
| Reach them | `enrich_person` (email, LinkedIn, X), `check_availability` (already hired?) |
| Roles and shortlist | `create_role`, `suggest_role_criteria`, `set_role_criteria`, `list_roles`, `get_role`, `add_people_to_role`, `decide`, `get_shortlist` |
| Watch people | `watch_person`, `unwatch_person`, `list_watches` |
| Workspaces | `list_workspaces`, `switch_workspace`, `create_workspace`, `list_workspace_members`, `invite_to_workspace` |
| Account | `whoami`, `disconnect` |

Every tool carries MCP annotations (read-only, destructive, idempotent, open-world), so your app can ask before the ones that cost money or change something.

Example prompts:

- *"Who should I talk to first for our senior Rust platform role?"*
- *"Who actually builds the most popular open-source vector databases?"*
- *"Is @someone a good fit for a staff frontend role? Show me the evidence."*

## What's in this repository

| Path | What it is |
|---|---|
| `server.json` | The server's entry in the [official MCP Registry](https://registry.modelcontextprotocol.io) |
| `.claude-plugin/` | Claude Code plugin and marketplace manifests |
| `.mcp.json` | The plugin's MCP connection; its key comes from `bin/ayhire-mcp-headers` |
| `bin/ayhire` | `ayhire login`, `status`, `logout`. Needs only bash and curl. Saves a key to `~/.config/ayhire/credentials.json` |
| `skills/` | Agent skills: how to use the tools, and a playbook for sourcing a role |
| `gemini-extension.json` | Gemini CLI extension manifest |
| `.cursor-plugin/`, `mcp.json` | Cursor plugin manifest and its MCP connection |

## Privacy and support

AyHire reads public GitHub work only, never private repositories. [Privacy policy](https://ayhire.com/privacy) · [Terms](https://ayhire.com/terms) · Support: [team@ayhire.com](mailto:team@ayhire.com)

## License

The files in this repository are MIT licensed. The AyHire service is covered by its [terms](https://ayhire.com/terms).
