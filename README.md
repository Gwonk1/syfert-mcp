# Syfert Legal Research — MCP server

**The verification layer for legal AI output.** Verify every citation, check whether a case is
still good law, audit a whole brief in one pass — and search statutes for every state.

- **Endpoint:** `https://mcp.syfert.com/mcp` (Streamable HTTP)
- **Auth:** `Authorization: Bearer <token>` — *optional*, see below
- **Free token:** one email click at <https://syfert.com/mcp/> — no card, no trial clock
- **Docs:** <https://syfert.com/mcp/> · **Tool detail:** <https://syfert.com/mcp/tools.php>
- **Maintainer:** Graham W. Syfert, Esq., P.A. — <graham@syfert.com>

## No token? Just connect.

Point any MCP client at `https://mcp.syfert.com/mcp` with no credential at all and it answers.
Every tool except the long-brief upload works anonymously; it is simply metered per IP
address: **20 calls per day** (and at most 20 a minute).
`initialize` and `tools/list` are never counted, so discovery always works.

A free token — one email click at <https://syfert.com/mcp/> — lifts those limits to 120 calls
per minute, restores the full result caps and the 512 KB `check_brief` document, unlocks
`brief_upload` for briefs of any length, and attaches your calls to your own account. Send it as `Authorization: Bearer YOUR_TOKEN`.

```bash
curl -s https://mcp.syfert.com/mcp -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## What is behind it

| | |
|---|---|
| Case law | **10.7 million U.S. opinions** — every state court and every federal court (Free Law Project / CourtListener bulk data, refreshed quarterly) |
| Citation map | ~175 million **citation events** — each place one opinion cites another, with the judge's sentence, parenthetical, quoted words and pin cite |
| Treatment | **Syfertize** citator flags: red = overruled or superseded, yellow = questioned or limited, green = followed |
| Statutes | All **50 states + D.C.**, plus the U.S. Code, the C.F.R., the federal rules and the sentencing guidelines — 55 corpora, each with its own index and scrape date |
| Quotes | 448,000 passages that courts quote from each other, each tied to its source opinion |

22 tools. 19 are **read-only**; the other three write only what you send them:
`suggest_treatment` (a flag correction your AI proposes, kept for human review) and
`brief_upload` / `brief_delete` (your own brief, held in server RAM while it is checked).
Nothing bills, nothing leaves the corpus.

## Tools

| Tool | What it does |
|---|---|
| `search_cases` | Full-text search across 10.7M opinions — boolean, phrases, proximity, court/date/state filters, authority-weighted ranking. |
| `get_case` | Full opinion text (majority, concurrence, dissent) with metadata, parallel citations, the Bluebook cite and the treatment flag. Long opinions page. |
| `check_citation` | The hallucination catcher: does the cite exist, what case really lives there, does the claimed case name match. Misses get did-you-mean suggestions. |
| `check_brief` | Whole-document audit — every citation, every claimed case name and every quoted passage in a finished draft, verified in one pass. Up to 200 citations inline; any longer brief goes through `brief_upload`. |
| `brief_upload` | Signed-in accounts: upload a brief of any length in pieces; `check_brief(brief_id)` then checks every citation in it. |
| `get_brief_results` | Read a long brief's results one page at a time (corrections, cites, unattached quotes, statutes). |
| `brief_delete` | Delete an uploaded brief and its results now (otherwise they go after the last page is read, or after an hour unused). |
| `find_case` | Fast direct lookup: a citation or a caption in, THE case out — or up to five candidates when it is not decisive. |
| `verify_quote` | One quotation checked word for word against the opinion it is attributed to, with the closest passage when it is off. |
| `find_authority` | Paste a proposition verbatim; get the cases courts cite for it. |
| `context_detector` | Does the case say what the brief says it says? The opinion's own words around the pinned page, beside the brief's sentence. |
| `get_treatment` | Is it still good law? Red/yellow/green with the citing cases behind each signal, who overruled it (including statutory supersession), and still-good replacement authority. |
| `get_propositions` | What courts actually cite the case *for*, proposition by proposition, each with its own flag — plus a quote-integrity verdict per famous passage. |
| `get_citing_cases` | The citator list: ranked citing cases with judge-written parentheticals, verbatim quotes, pin cites and each citing court's own signal. |
| `find_related_cases` | Hand it one good case, get the doctrinal line it belongs to — the frequently co-cited family around that anchor authority. |
| `search_quotes` | Phrase search across 448,000 quoted passages, ranked by adoption, each tied to the opinion it originates in. Catches misattributed quotes. |
| `find_issues` | Start from facts or an issue instead of a case name: the doctrine as courts in that state name it, the cases on **each** side, the statutes those opinions construe. |
| `find_court` | Turns "the Eleventh Circuit" or "S.D. Florida" into the court identifier the search filters expect. |
| `search_statutes` | Search statutes and codes for all 50 states + D.C., the U.S. Code, the C.F.R., the federal rules and the sentencing guidelines. |
| `get_statute` | Full section text with cross-references surfaced and Notes-of-Decisions annotations; federal sections carry a banner when they have been held unconstitutional. |
| `export_document` | Optional: a readable copy of a case or statute (Markdown or HTML, with a filename) to save locally; long documents come in parts. |
| `suggest_treatment` | Tell us a red/yellow/green flag is wrong; the suggestion is reviewed by a human and never changes a flag by itself. |

## Install

Every snippet below sends a token. You do not have to — drop the `Authorization` header (or
use the plain endpoint URL) and the same client connects anonymously at 20 calls a day.
To lift that: <https://syfert.com/mcp/> → enter your email → open the one-time link, then
replace `YOUR_TOKEN` everywhere below with what it gives you. **Treat it like a password.**

### Claude Code

```bash
claude mcp add --transport http syfert https://mcp.syfert.com/mcp \
  --header "Authorization: Bearer YOUR_TOKEN"
```

### claude.ai (custom connector)

claude.ai custom connectors cannot set request headers, so they use the legacy path form.
Prefilled link — paste your own token into the URL first:

```
https://claude.ai/customize/connectors?modal=add-custom-connector&connectorName=Syfert&connectorUrl=https%3A%2F%2Fmcp.syfert.com%2FYOUR_TOKEN%2Fmcp
```

Or by hand: Settings → Connectors → Add custom connector → `https://mcp.syfert.com/YOUR_TOKEN/mcp`.

### ChatGPT (developer mode)

Settings → Apps & Connectors → Advanced → Developer mode, then Create.
URL `https://mcp.syfert.com/YOUR_TOKEN/mcp`, authentication **No authentication**
(the token is already in the URL). ChatGPT cannot set headers either.

### Codex CLI

```bash
export SYFERT_MCP_TOKEN=YOUR_TOKEN
codex mcp add syfert --url https://mcp.syfert.com/mcp \
  --bearer-token-env-var SYFERT_MCP_TOKEN
```

### Cursor

One-click — paste your token in first (see "Building your own deeplink" below):

```
cursor://anysphere.cursor-deeplink/mcp/install?name=syfert&config=eyJ1cmwiOiJodHRwczovL21jcC5zeWZlcnQuY29tL21jcCIsImhlYWRlcnMiOnsiQXV0aG9yaXphdGlvbiI6IkJlYXJlciBZT1VSX1RPS0VOIn19
```

Or edit `~/.cursor/mcp.json` (see [`mcp.json`](mcp.json) in this repo).

<details>
<summary>Building your own deeplink</summary>

The `config` parameter is the base64 of this object with your token in it:

```json
{"url":"https://mcp.syfert.com/mcp","headers":{"Authorization":"Bearer YOUR_TOKEN"}}
```

```bash
python3 - <<'PY'
import json, base64, urllib.parse
cfg = {"url": "https://mcp.syfert.com/mcp",
       "headers": {"Authorization": "Bearer YOUR_TOKEN"}}
b = base64.b64encode(json.dumps(cfg, separators=(',', ':')).encode()).decode()
print("cursor://anysphere.cursor-deeplink/mcp/install?name=syfert&config=" +
      urllib.parse.quote(b, safe=''))
PY
```
</details>

### VS Code / GitHub Copilot

This link prompts for the token and stores it as a secret input:

```
vscode:mcp/install?%7B%22name%22%3A%22syfert%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.syfert.com%2Fmcp%22%2C%22headers%22%3A%7B%22Authorization%22%3A%22Bearer%20%24%7Binput%3Asyfert_token%7D%22%7D%2C%22inputs%22%3A%5B%7B%22type%22%3A%22promptString%22%2C%22id%22%3A%22syfert_token%22%2C%22description%22%3A%22Syfert%20MCP%20token%22%2C%22password%22%3Atrue%7D%5D%7D
```

Or from the command line: `code --add-mcp '{"name":"syfert","type":"http","url":"https://mcp.syfert.com/mcp","headers":{"Authorization":"Bearer YOUR_TOKEN"}}'`

### Gemini CLI

As an extension (this repo — it carries [`gemini-extension.json`](gemini-extension.json)):

```bash
gemini extensions install https://github.com/Gwonk1/syfert-mcp
```

You will be asked for the **Syfert MCP Token** setting on install; it is stored in your
system keychain, not in the manifest. Or add the server directly:

```bash
gemini mcp add syfert https://mcp.syfert.com/mcp --transport http \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Goose

One-click (Goose will prompt for the `Authorization` header value):

```
goose://extension?type=streamable_http&url=https%3A%2F%2Fmcp.syfert.com%2Fmcp&id=syfert&name=Syfert%20Legal%20Research&description=Verify%20citations%2C%20check%20whether%20a%20case%20is%20still%20good%20law%2C%20audit%20a%20whole%20brief%2C%20and%20search%20statutes%20for%20all%2050%20states&header=Authorization%3DBearer%20%3CYOUR_SYFERT_TOKEN%3E
```

Or: Extensions → Add custom extension → type **Streamable HTTP**, URL
`https://mcp.syfert.com/mcp`, request header `Authorization: Bearer YOUR_TOKEN`.

### Windsurf

`~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "syfert": {
      "serverUrl": "https://mcp.syfert.com/mcp",
      "headers": { "Authorization": "Bearer YOUR_TOKEN" }
    }
  }
}
```

### Any client that only speaks stdio (`mcp-remote` fallback)

```json
{
  "mcpServers": {
    "syfert": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.syfert.com/mcp",
               "--header", "Authorization: Bearer YOUR_TOKEN"]
    }
  }
}
```

### Two URL forms

| Form | Use it when |
|---|---|
| `https://mcp.syfert.com/mcp` + `Authorization: Bearer <token>` | **Preferred.** Your client can set request headers (everything above except claude.ai and ChatGPT). |
| `https://mcp.syfert.com/<token>/mcp` | Legacy fallback for clients that cannot set headers — claude.ai custom connectors, ChatGPT developer mode. |

Both reach the same server. An unauthenticated request returns `401` with a
`WWW-Authenticate: Bearer` challenge and RFC 9728 protected-resource metadata.

## Privacy — what is and is not logged

**Your searches are not logged.** Not the citation you checked, not the case name, not your
query, not a brief you pasted in. What is kept is operational only: which tool ran, whether it
succeeded, and how long it took — enough to keep the lights on and catch abuse. That data is
seen by Graham W. Syfert, Esq., P.A. and nobody else. It is not sold, not shared with third
parties, and not fed to any advertising system.

One exception, for long briefs: a brief you upload with `brief_upload` is held, with its check
results, in server memory only (never on disk, in backups or in logs), readable only with your
credential, and deleted as soon as the last page of results is read, or after an hour unused.
And if your AI calls `suggest_treatment`, the case, the flag it proposes, its reason and the
authority it names are kept for human review — nothing about you goes with it.

Your token is a secret; anyone holding it can search against your access. If it leaks, email
<graham@syfert.com> and a fresh one will be issued and the old one retired.

Be sensible anyway: do not paste client secrets into a search box, here or anywhere.

- Privacy policy: <https://syfert.com/privacy>
- Terms: <https://syfert.com/terms>

## Support

Email <graham@syfert.com>, or open an issue on this repo. Bugs get fixed.

## What is in this repo, and what is not

This repository holds **manifests and install instructions only** — the JSON files that let
MCP clients and catalogs find the hosted service. It is published under the
[MIT licence](LICENSE) so catalogs can vendor and redistribute the manifests freely.

**The Syfert service itself is proprietary and closed-source.** The corpus builders, the
Syfertize treatment engine, the citation-event pipeline and the server code are not published
here or anywhere. Nothing in this repository grants any right to the service, its data, or its
output beyond the [terms](https://syfert.com/terms).

## Not legal advice

Syfert is a research tool. It checks the citations; you are still the lawyer, and the filings
are still yours. Using it creates no attorney-client relationship.
