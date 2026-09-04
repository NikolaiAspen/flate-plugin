# Flate-plugin

Kunden peker og kommenterer i appen din. Du får kommentarene som verktøy i Claude Code/Codex, og agenten bygger.

## Claude Code
    /plugin marketplace add NikolaiAspen/flate-plugin
    /plugin install flate@flate
    /mcp            → velg «flate» → Authenticate (logg inn i nettleseren)

Deretter: `/flate:installer` (én gang per app), så ett av:
- **`/flate:del`** – deler appen du kjører **lokalt** med kunden uten å deploye: tunnelerer `localhost` til en offentlig URL og lager runden i én kommando. Bruk denne under utvikling.
- **`/flate:ny-runde Runde 1 https://preview.din-app.no`** – når du allerede har en offentlig URL (deploy/preview).

Del lenken kunden får, og `/flate:runde` når de har kommentert. `/flate:status` viser åpne runder.

**`/flate:issues`** gjør åpne kommentarer om til issues i din egen tracker (Linear/GitHub/Azure) via dine MCP-koblinger — Flate holder ingen integrasjoner selv.

## Codex
Se `codex/README.md`.
