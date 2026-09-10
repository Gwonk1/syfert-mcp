# Installing the Syfert MCP server

Syfert is a **hosted remote MCP server**. There is nothing to clone, build, or run locally.
You only need a token and one config entry.

## Step 1 — get a token (the human must do this)

Ask the user to open <https://syfert.com/mcp/>, enter their email, and open the one-time link
that arrives. It shows a private URL exactly once. The token is the path segment in that URL:
`https://mcp.syfert.com/<TOKEN>/mcp`. It is free.

Do not proceed until the user gives you the token. Never invent one.

## Step 2 — add the server

Add this to the MCP settings file, substituting the real token:

```json
{
  "mcpServers": {
    "syfert": {
      "type": "streamableHttp",
      "url": "https://mcp.syfert.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

If the client cannot set request headers, use the legacy path form instead and send no headers:

```json
{
  "mcpServers": {
    "syfert": {
      "type": "streamableHttp",
      "url": "https://mcp.syfert.com/YOUR_TOKEN/mcp"
    }
  }
}
```

## Step 3 — verify

List tools. A working install reports **13 tools**: `search_cases`, `get_case`,
`check_citation`, `check_brief`, `get_treatment`, `get_propositions`, `get_citing_cases`,
`find_related_cases`, `search_quotes`, `find_issues`, `find_court`, `search_statutes`,
`get_statute`. All are read-only.

Then call `check_citation` with `citation: "410 U.S. 113"` and
`expected_case_name: "Roe v. Wade"`. It should come back as a match.

## Troubleshooting

- **401 with `WWW-Authenticate: Bearer`** — the token is missing, mistyped, or revoked. Check
  the header is exactly `Authorization: Bearer <token>` with one space.
- **404** — the path is wrong. The endpoint is `/mcp`, not `/` and not `/sse`.
- **Rate limited** — the free tier allows 120 requests per minute per token.
- Anything else: <graham@syfert.com>.

Do not put the token in a file that gets committed to source control.
