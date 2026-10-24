## Zuplo API

This is a sample Zuplo project that shows how to configure the
[MCP Gateway](https://zuplo.com/docs/mcp-server/introduction) with OAuth and
role-based access to MCP tools. **All values are placeholders** (`example.com`
URLs, dummy client IDs, made-up tool names). Replace them with your own before
running it.

## What's configured

### Route: `/mcp/example`

Proxies MCP traffic (`GET`, `POST`) to the upstream MCP server at
`https://mcp.example.com/mcp` using `McpProxyHandler`. Inbound policies run in
this order:

1. `oidc-managed-oauth`: authenticates the user
2. `mcp-token-exchange-example`: gets an upstream token for that user
3. `mcp-capability-filter-example`: limits which tools the user can see and call

### Policies (`config/policies.json`)

**`oidc-managed-oauth`** (`mcp-oauth-inbound`): the gateway acts as the OAuth
server for MCP clients and sends users to your OIDC identity provider to log
in.

| Option | Placeholder |
| --- | --- |
| `oidc.issuer` | `https://idp.example.com` |
| `oidc.jwksUrl` | `https://idp.example.com/.well-known/jwks.json` |
| `oidc.audience` | `dummy-mcp-gateway-audience` (optional ID-token audience override; omit it if ID tokens use the client ID as the audience) |
| `browserLogin.url` | `https://idp.example.com/oauth2/authorize` |
| `browserLogin.tokenUrl` | `https://idp.example.com/oauth2/token` |
| `browserLogin.clientId` / `clientSecret` | `$env(OIDC_CLIENT_ID)` / `$env(OIDC_CLIENT_SECRET)` |
| `browserLogin.scope` | `openid profile email` |

MCP clients can register with the gateway using Client ID Metadata Documents
(CIMD, on by default via `gateway.cimdEnabled`) or Dynamic Client Registration
(`/oauth/register`).

**`mcp-token-exchange-example`** (`mcp-token-exchange-inbound`): per-user
OAuth to the upstream MCP server (`authMode: user-oauth`). With
`clientRegistration.mode: auto`, the gateway registers itself with the upstream
using CIMD and falls back to DCR. Use `mode: manual` with a client ID and
secret if the upstream requires a pre-registered client.

**`mcp-capability-filter-example`** (`mcp-capability-filter-inbound`): role-based
tool access (`accessControl.mode: rolesAndGroups`), reading roles from the
`roles` claim and groups from the `groups` claim.

| Tool | Allowed for |
| --- | --- |
| `list_items` | Everyone (public) |
| `search_items` | `viewer`, `editor`, `admin` roles |
| `update_item` | `editor`, `admin` roles |
| `delete_item` | `admin` role or `dummy-ops-team` group |

Only the tools listed are exposed. Other upstream tools are hidden, so list
your real tool names here.

### Other files

- `zuplo.jsonc`: `compatibilityDate` is `2026-03-01`, the minimum for the MCP
  Gateway.
- `modules/zuplo.runtime.ts`: registers `McpGatewayPlugin`.
- `.env.example`: copy to `.env` and set `OIDC_CLIENT_ID` and
  `OIDC_CLIENT_SECRET`.

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Open [http://localhost:9000](http://localhost:9000) with your browser to see the
result.

You can start editing the API by modifying `config/routes.oas.json`. The dev
server will automatically reload the API with your changes.

## Debugging

In VS Code, open **Run and Debug**, select **Launch & Attach Zuplo**, and click
the green play button.

For other editors and more details, see the
[debugging guide](https://zuplo.com/docs/articles/local-development-debugging).

## Learn More

To learn more about Zuplo, you can visit the
[Zuplo documentation](https://zuplo.com/docs).

To connect with the community join [Discord](https://discord.zuplo.com).
