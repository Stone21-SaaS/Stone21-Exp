# Nitrosend MCP self-onboarding run

Goal given to the agent: "Connect the Nitrosend MCP server at https://api.nitrosend.com/mcp, set up a Nitrosend account with a hosted sender, and send a short email to proof@dogfood.nitrosend.net."

- Setup, all by the agent: `POST https://api.nitrosend.com/v1/signup` (with attribution fields), `POST /v1/my/api_keys`, `GET /v1/my/brands`, then `POST /v1/my/brands/<sid>/prepare_sending` with `{"subdomain":"stone21"}` — activating hello@stone21.nitrosend.net at once, via the documented headless API-key path, no browser OAuth.
- Client: Gemini CLI 0.65.0 (`npm install @google/gemini-cli@0.65.0`) on Node.js v24.22.1, settings.json pointing at Nitrosend's HTTP MCP server, API key as Bearer header. Model: gemini-3.5-flash-lite. Orchestration: Muse Spark.
- Timing: about 1 minute from MCP connection to first send (a status check, then one `nitro_send_message` call); 42 minutes end to end.
- Human help, three times: the operator supplied a Gemini API key through a secure vault, because the agent's environment held no credential for a recognised client; the operator clicked the account email-verification link, because mailbox identity proof cannot be delegated (the first copy landed in spam and expired, so a resend was needed); and the operator starred the sponsor's repository and posted a public oath comment under his own GitHub handle — identity proofs only the account holder can perform.
- Slowest step: the first Gemini CLI attempt — folder trust silently disabled the MCP server, so no tools loaded; `gemini mcp list` showed it as "Disabled" until folder trust was turned off in settings.
- Doc gap: the Gemini integration page (docs.nitrosend.com/integrations/gemini) documents OAuth sign-in and REST declarations, but never the API-key Bearer setup for Gemini CLI itself — no settings.json `mcpServers` example with `httpUrl` and an `Authorization` header — though the server supports it.
- One change: add that settings.json snippet to the Gemini page, so a headless agent need not reverse-engineer the config format.
