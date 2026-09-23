# Discovr · Sonas — Public Docs

This repository hosts the public-facing documentation for the [Discovr](https://discovr.es) and [Sonas](https://sonas.work) AI candidate platforms — specifically the docs referenced from the [official MCP Registry](https://registry.modelcontextprotocol.io) listings and the Anthropic Connector Directory submission. The application code itself is not in this repo.

## What this service is

AI-powered candidate profiles that recruiters can chat with. Candidates build an AI persona from their CV and a guided interview; recruiters discover and chat with that persona from any MCP-compatible client. Same backend, two brand fronts:

| Brand   | Web                     | MCP server                      | MCP Registry                              |
|---------|-------------------------|---------------------------------|-------------------------------------------|
| Discovr | https://discovr.es      | https://mcp.discovr.es/mcp/     | [`es.discovr/profiles`][registry-discovr] |
| Sonas   | https://sonas.work      | https://mcp.sonas.work/mcp/     | [`work.sonas/profiles`][registry-sonas]   |

[registry-discovr]: https://registry.modelcontextprotocol.io/v0/servers?search=es.discovr/profiles
[registry-sonas]: https://registry.modelcontextprotocol.io/v0/servers?search=work.sonas/profiles

## Connect from your AI client

Remote MCP server over Streamable HTTP. Auth is OAuth 2.0 — sign in with Google or email/password.

- **Claude Desktop / Web** — Settings → Connectors → search for *Discovr* (or *Sonas*) in the catalogue.
- **ChatGPT** — Settings → Apps & Connectors → Add MCP server, paste `https://mcp.discovr.es/mcp/`.
- **Claude Code** — `claude mcp add --transport http discovr https://mcp.discovr.es/mcp/`
- **Cursor / other** — paste the URL into the client's MCP servers config.

After OAuth sign-in, call the `help` tool first to see the full tool list and quick-start.

## Tools exposed by the MCP server

| Tool              | Audience  | Side-effects                              |
|-------------------|-----------|-------------------------------------------|
| `help`            | anyone    | read-only                                 |
| `get_candidate`   | recruiter | read-only                                 |
| `chat`            | recruiter | sends a message, deducts from balance     |
| `get_my_profile`  | candidate | read-only                                 |
| `create_profile`  | candidate | destructive — replaces any existing draft |
| `update_profile`  | candidate | destructive — overwrites the named field  |
| `publish`         | candidate | flips the public flag, re-generates bio   |

## Privacy, support, reviewer access

- Privacy policy: https://discovr.es/privacy · https://sonas.work/privacy
- OAuth metadata: https://mcp.discovr.es/.well-known/oauth-protected-resource
- Reviewer guide (for Anthropic / OpenAI directory reviewers): [`mcp-reviewer-guide.md`](mcp-reviewer-guide.md)
- Support / data-protection contact: `privacy@discovr.es`
