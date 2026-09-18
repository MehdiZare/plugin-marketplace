# Bina Options plugin for Grok Build

Connect Grok Build to [Bina Capital](https://bina.capital) options analytics:
implied-volatility context, options-implied expected moves, option-strategy
ideas for a directional view, and (where enabled) short-premium
structures ranked by the models' expected return on capital.

Everything is **read-only, model-based education over delayed listed-options
data**. It is not investment advice, and there is no order placement.

## Installation

In Grok Build, open `/plugin`, search for **Bina Options**, and install.

On first connection, Grok opens Bina sign-in in the browser. Sign in with a
Bina account and approve access. Do not paste an API key or token into chat.

## Components

- MCP server `bina-options` (streamable HTTP, remote, no local code).
- Skill `bina-options`: which tool answers which question, how to read the
  results, and the rules for presenting them.

## Network endpoints and credentials

- `https://mcp.bina.capital/mcp` — the hosted MCP server.
- `https://clerk.bina.capital` — OAuth 2.1 sign-in (authorization code + PKCE),
  advertised by the server's protected-resource metadata.

Credentials: a Bina account. The plugin requests the scopes `options:read`
and `offline_access`, and stores no API key. The server's tools are
read-only.

## License

Proprietary. Use of the hosted service is governed by Bina Capital's terms.
