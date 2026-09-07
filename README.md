# Flate-plugin

Kunden peker og kommenterer i den ekte appen din. Du får kommentarene som verktøy i Claude Code/Codex, og agenten bygger.

## Claude Code
    /plugin marketplace add NikolaiAspen/flate-plugin
    /plugin install flate@flate
    /mcp            → velg «flate» → Authenticate (logg inn i nettleseren)

Alt kan sies i prosa — «koble appen til Flate», «lag en runde fra denne branchen», «del appen jeg kjører lokalt», «ta runden», «lukk runden» — eller kjøres som slash-kommandoer:

- **`/flate:installer`** – én gang per app: monterer `@flateapp/sdk` og åpner CSP (`connect-src`) for portalen. Deploy etterpå.
- **`/flate:ny-runde Runde 1 https://preview.din-app.no`** – runde mot en deployet/preview-URL, eller fra en branch når prosjektet er koblet til GitHub i portalen (Flate finner deploy-previewen selv).
- **`/flate:del`** – deler appen du kjører **lokalt** uten å deploye: tunnelerer `localhost` til en offentlig URL og lager runden i én kommando.
- **`/flate:runde`** – henter kommentarene, fikser én om gangen, melder status tilbake til kunden.
- **`/flate:status`** – åpne runder og hva som venter. **`/flate:issues`** – åpne kommentarer som issues i din egen tracker (Linear/GitHub/Azure) via dine MCP-koblinger.
- **`/flate:oversikt`** – slik henger alt sammen (token-flyt, statuser, verktøy).

Kunden trenger ingen konto: åpner lenken, skriver navnet sitt, trykker **k** eller Kommentér, og peker. Innlogging i appen (også Vipps/BankID) virker som vanlig, fordi Flate kjører som overlay i appen og ikke i en iframe.

Oppdater pluginen med `/plugin update flate@flate`.

## Codex
Se `codex/README.md`.
