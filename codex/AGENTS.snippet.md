## Flate-gjennomgang

Dette repoet er koblet til Flate via MCP-serveren `flate`. Verktøyene er tilgjengelige som `mcp__flate__<navn>`, f.eks. `mcp__flate__list_rounds`, `mcp__flate__get_round`, `mcp__flate__update_comment`, `mcp__flate__reply`.

Når noen ber deg «ta runden» eller hente kommentarer fra Flate:

1. Kall `mcp__flate__list_rounds`. Får du feil om `project_id`: kall `mcp__flate__list_projects` og velg prosjektet som matcher dette repoet (mappenavn eller `githubRepo`); spør brukeren hvis det er uklart. Er et rundenavn/-id gitt, bruk den; ellers velg den åpne runden med flest åpne kommentarer, og si hvilken du valgte.
2. Kall `mcp__flate__get_round` med `round_id`. Les hver kommentar med status `open` eller `planned`: `anchor.route` er siden i appen, `anchor.testId` (mest stabil) / `anchor.component` / `anchor.text` / `anchor.cssPath` er elementet, og `surface.device` sier hvilken flate kunden så (iphone/ipad/android/desktop). Finn koden som rendrer elementet (søk etter `data-testid`/`testID`, tekst, rutefil).
3. Implementer én kommentar om gangen, med en commit per kommentar. Følg repoets konvensjoner. Kjør repoets typecheck/tester.
4. Etter hver kommentar: `mcp__flate__update_comment` med `status: "implemented"` og `note` på norsk (1–2 setninger, skrevet til kunden – hva ble endret, ikke hvordan). Må du gjøre noe stort som ikke er avgrenset: `status: "planned"` + `note` om hva som trengs.
5. Er kommentaren tvetydig: kall `mcp__flate__reply` med et konkret spørsmål (gjerne to alternativer) og sett `status: "planned"`. Ikke gjett.
6. Sett ALDRI `verified` – det gjør kunden. Kjør ALDRI handlinger med eksterne effekter (sende SMS/e-post, betaling, deploy) som del av en kommentar.
7. Avslutt med en kort oppsummering: hva som ble gjort per kommentar, hva som må deployes for at kunden skal se det, og eventuelle spørsmål som venter på svar.
