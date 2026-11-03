# Security policy

## Reporting a vulnerability

Email **hello@getglade.co.uk** with "Security" in the subject line. Please include steps to
reproduce and the endpoint or tool affected. Don't open a public issue for security reports.

We aim to acknowledge reports within 3 working days and to fix confirmed issues promptly.

## Scope

- The hosted MCP endpoint `https://getglade.co.uk/api/mcp`
- The REST API under `https://getglade.co.uk/api/v1/`
- The skill and plugin files in this repository

## What the server does with your data

Both MCP tools are read-only: they query Glade's own index of public outdoor locations and never
write data or call third-party services on your behalf. Requests are logged against your API key
(tool name, credits used, response time, status, and the postcode if one was sent) for rate
limiting and abuse prevention. See <https://getglade.co.uk/privacy>.

## Handling API keys

Never commit a `glade_…` key. The Claude Code plugin stores it in your OS credential store via
`userConfig.sensitive`. If a key leaks, contact us and we'll revoke and reissue it.
